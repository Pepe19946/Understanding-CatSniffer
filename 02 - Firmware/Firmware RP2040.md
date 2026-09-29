# Firmware RP2040 — USB, puente y control local

## En una frase

Esta imagen convierte al [[RP2040 - El procesador de interfaz|RP2040]] en la puerta USB del sistema, puente hacia CC1352P7 e intérprete de las funciones Shell/LoRa locales.

## Cómo leer esta nota

Primero recuerda el mapa [[USB CDC - Cómo CatSniffer aparece ante la PC]]. Después usa esta nota para comprobar cómo Zephyr, callbacks, buffers y comandos implementan ese modelo. “Firmware Zephyr” significa que la aplicación usa el sistema operativo y drivers Zephyr; no significa que Zephyr también corra en CC1352P7.

## 1. Papel

El firmware de `RP2040/catsniffer/` corre en U3 RP2040. Es la frontera USB del equipo, un puente serie hacia el CC1352P7 y el controlador directo del SX1262. También maneja reset/boot del CC, `CTF1..3`, LEDs y una consola propia.

**Conclusión:** el RP2040 coordina transportes; no sustituye al firmware del CC1352. Para `Cat-Bridge` mueve bytes sin interpretarlos. Para `Cat-LoRa` sí interpreta o agrupa entrada y llama la API de radio que gobierna el SX1262.

## 2. Arquitectura de build

- `CMakeLists.txt`: aplicación Zephyr; incorpora `src/*.c`, `src/USB/usbd_init.c` y `src/shell_commands.c`; genera metadatos de versión/commit.
- `prj.conf`: UART interrupt-driven, USB device stack next + CDC ACM, SPI, GPIO, LoRa native backend, flash/NVS/settings. Consola/log/printk están deshabilitados.
- `west.yml`: fork `https://github.com/wero1414/zephyr`, revisión `fsk-native-driver`. Es una rama, no un commit inmutable: **limitación de reproducibilidad**.
- `boards/rpi_pico.overlay`: board lógico `rpi_pico`, UART0, SPI0/SX1262, GPIO, tres CDC y partición NVS.
- `VERSION`: `0.2.0.0-dual-mode`.
- **Documentado:** README indica Zephyr 4.1, `west build ... -b rpi_pico` y UF2/HEX/ELF. No se construyó ni se validó el toolchain.

La carga documentada usa BOOTSEL del RP2040, enumeración ROM `RPI-RP2` por USB y copia del UF2. El comando local `reboot` llama `reset_usb_boot(0, 0)`. Esta ruta de carga no es una de las tres interfaces CDC del firmware ya iniciado.

## 3. Entrada y secuencia de inicialización

La entrada de aplicación es `src/main.c:main()`.

Secuencia observada:

1. Obtiene dispositivos UART0, CDC0/1/2 y configura GPIO de reset/boot, LEDs y `CTF1..3`.
2. Lee el pin `BOOT`: bajo selecciona modo `BOOT`/500000 baud; alto selecciona `PASSTHROUGH`/921600 baud. La espera del pin tiene tope de 500 ms.
3. Inicializa configuraciones por defecto LoRa y FSK a 915 MHz.
4. Llama `enable_usb_device_next()` → `usbd_init_device()` y habilita/deshabilita el dispositivo con el estado VBUS.
5. Inicializa seis ring buffers estáticos de 16 KiB.
6. Configura UART, instala callbacks de los tres CDC y UART, habilita interrupciones RX y resetea el CC1352.
7. En boot fuerza BOOT; en modo normal intenta seleccionar banda `GIG`.
8. Llama `initialize_lora()`, que obtiene `DT_ALIAS(lora0)`, comprueba readiness, aplica `lora_config()` y arma recepción asíncrona.
9. Crea `lora_thread_func()` como hilo cooperativo, prioridad 5, stack 4096 bytes.
10. El hilo `main` permanece en la animación/servicio de LED; el tráfico ocurre en callbacks, interrupciones y el hilo de radio.

**Orden entre procesadores no demostrado:** el reset del CC se configura inicialmente alto y sólo más tarde se pulsa. El código no demuestra que el CC espere inmóvil hasta que el RP esté completamente listo.

## 4. USB

`src/USB/usbd_init.c` define el contexto de la nueva pila USBD de Zephyr, descriptores (VID `0x1209`, PID `0xBABB`, producto `Catsniffer`) y registra todas las clases configuradas a full speed. DFU se excluye expresamente.

El overlay crea:

| CDC | Label | Callback en `main.c` | Destino |
| --- | --- | --- | --- |
| 0 | `Cat-Bridge` | `cdc0_interrupt_handler()` | UART/CC1352 transparente |
| 1 | `Cat-LoRa` | `cdc1_interrupt_handler()` | hilo/APIs SX1262 |
| 2 | `Cat-Shell` | `cdc2_interrupt_handler()` | `process_command()` local |

En RX, cada callback lee `uart_fifo_read()` del CDC. En TX, drena el ring correspondiente con `uart_fifo_fill()`. CDC2 acumula una línea en `catsniffer.command_data[256]`, hace eco y llama `process_command()` al recibir fin de línea.

## 5. Comunicación RP2040 ↔ CC1352P7

### Configuración

- `chosen uart-cc1352 = &uart0`.
- RP GPIO0=UART TX y GPIO1=UART RX; en U8 corresponden a DIO12/DIO13 según PCB/Fase 1.
- Velocidad normal `921600`; boot `500000`.
- El código sólo cambia `baudrate` sobre la configuración obtenida por `uart_config_get()`: el framing no se declara explícitamente aquí y no debe afirmarse más allá de los defaults del dispositivo.

### PC → CC

`Cat-Bridge RX → cdc0_interrupt_handler() → rb_usb_to_cc1352 → habilita UART TX → cc1352_uart_interrupt_handler() → uart_fifo_fill(UART0)`.

No hay parser en esta ruta. La interpretación comienza, si existe, en la imagen del CC.

### CC → PC

`UART0 RX → cc1352_uart_interrupt_handler() → rb_cc1352_to_usb → habilita CDC0 TX → cdc0_interrupt_handler() → host`.

El ISR lee lotes de hasta 64 bytes y detecta `UART_ERROR_OVERRUN`; cuenta descartes cuando se llena el ring CC→USB. La dirección USB→CC inserta byte por byte sin comprobar el retorno, por lo que no contabiliza sus descartes.

### Boot/reset

- `reset_cc1352()`: lleva reset bajo 100 ms y luego alto 100 ms.
- `boot_mode_cc1352()`: configura BOOT como salida baja, espera y resetea.
- `change_mode(BOOT)`: ejecuta la secuencia y usa 500000 baud.
- `change_mode(PASSTHROUGH)`: BOOT como entrada pull-up, resetea y usa 921600 baud.
- `src/shell_commands.c`: comandos `boot` y `exit` llaman esos cambios.

**Conclusión:** el RP facilita entrar al ROM bootloader y expone el UART al host, pero no implementa por sí solo el programador serie; el README remite a `cc2538-bsl`. El flujo del host se deja para Fase 3.

## 6. Control del SX1262

### Configuración activa

El nodo `sx1262@0` de `boards/rpi_pico.overlay` usa:

- SPI0: CIPO/MISO GPIO16, CS GPIO17 activo bajo, SCK GPIO18, COPI/MOSI GPIO19; máximo 1 MHz.
- RESET GPIO24 activo bajo.
- BUSY GPIO4.
- DIO1 GPIO5.

El comentario final del overlay que dice BUSY=GPIO5/DIO1=GPIO6 contradice la configuración ejecutable. El nodo activo coincide con el PCB (BUSY=4, DIO1=5): **conclusión: comentario obsoleto**.

### Driver y operaciones

La aplicación usa la API LoRa de Zephyr: `lora_config()`, `lora_send()` y `lora_recv_async()`. El driver Semtech no está dentro de esta aplicación; `CMakeLists.txt` apunta al árbol externo `${ZEPHYR_BASE}/drivers/lora/native/sx126x`, suministrado por la revisión de Zephyr del manifest.

- `initialize_lora()` obtiene el dispositivo, configura parámetros y arma RX asíncrona.
- `lora_start_rx_async()` instala `lora_rx_cb()`; ante `-EBUSY` cancela y reintenta.
- `lora_start_fsk_rx_async()` hace lo equivalente con `fsk_rx_cb()`.
- Los callbacks formatean la recepción y la ponen en `rb_sx1262_to_usb` para CDC1.
- `lora_thread_func()` espera `lora_data_sem` o un timeout de 100 ms. En modo stream envía bytes directamente; en modo comandos procesa líneas LoRa/FSK.
- Para transmitir se cancela RX, se aplica configuración TX, se llama `lora_send()` y luego se rearma RX.

DIO1/IRQ y los comandos SPI de bajo nivel son responsabilidad del driver externo; no son visibles en esta línea base. La aplicación no referencia DIO2/DIO3.

## 7. Selección RF

`change_band()` controla U2 mediante `CTF1..3`:

| enum/comando | CTF1, CTF2, CTF3 | Significado del código |
| --- | --- | --- |
| `GIG` / `band1` | `0,1,0` | ruta 2.4 GHz |
| `SUBGIG_1` / `band2` | `0,0,1` | ruta Sub-GHz 1 |
| `SUBGIG_2` / `band3` | `1,0,0` | ruta Sub-GHz 2/SX según relación de Fase 1 |

Los GPIO20/21 asociados en PCB a `ANT_SW`/`DIO22` y U6 no aparecen en el overlay ni en el código. Tampoco DIO2/DIO3 del SX1262. **No está demostrado que este firmware controle U6 directamente.**

## 8. Modelo de ejecución y buffers

- `main`: inicialización y bucle de LED.
- Hilo cooperativo LoRa/FSK prioridad 5: procesamiento CDC1 y radio.
- Callbacks/IRQ UART: tres CDC y UART CC.
- Callbacks asíncronos del driver: recepción LoRa/FSK.
- Semáforo `lora_data_sem`: despierta el hilo por entrada de CDC1.
- Seis ring buffers estáticos de 16384 bytes: CC↔USB, SX↔USB y config↔USB. `rb_usb_to_config` se inicializa pero no se vuelve a usar.
- `safe_ring_buf_put/get()` protege las operaciones con `irq_lock()`.

## 9. Errores y recuperación normal

- Fallos de dispositivos/USB críticos hacen que `main()` retorne.
- Fallo de inicialización LoRa se reporta en `Cat-Shell`; el flujo sigue y crea el hilo.
- UART revisa overrun y cuenta algunos bytes descartados.
- Los errores de configuración/transmisión radio se convierten en mensajes para CDC1/CDC2 en varias rutas.
- Recepción asíncrona recupera `-EBUSY` cancelando/reintentando.
- Boot/reset del CC permite recuperación controlada desde la shell.

## 10. Flujos representativos

### CC

`USB CDC0 → callback CDC0 → ring USB→CC → ISR UART TX → CC`; regreso simétrico por ISR UART RX, ring CC→USB y callback CDC0 TX. El RP no interpreta el payload.

### SX1262

`USB CDC1 → callback CDC1 → ring USB→SX + semáforo → hilo LoRa → parser o stream → API LoRa → driver → SPI/GPIO`; regreso mediante callback RX, ring SX→USB y CDC1 TX.

CDC2 ofrece un segundo origen local de control mediante `radio ...`, pero sus detalles de comandos no son objeto de esta fase.

## 11. Potenciales problemas, no evaluados

1. USB se habilita antes de inicializar los ring buffers; podría existir una ventana de callbacks tempranos. Requiere evaluación, no se afirma impacto.
2. CDC1/CDC2 no pasan por una comprobación explícita `device_is_ready()`; que sus punteros existan no equivale a readiness.
3. `process_command()` se llama desde el callback CDC2 y algunos handlers duermen, acceden NVS o reconfiguran radio; debe verificarse el contexto real permitido por Zephyr.
4. `catsniffer.band` parte en cero, igual a `GIG`; la llamada inicial `change_band(GIG)` puede salir antes de escribir `CTF1..3`, dejando el estado de selección dependiente de inicialización eléctrica.
5. En ciertas rutas stream se ignora el retorno de `lora_send()` o de inserciones en rings.

No se proponen correcciones en Fase 2.

## 12. Preguntas abiertas

- ¿Qué revisión exacta del fork Zephyr corresponde a `fsk-native-driver` en un build oficial? El manifest no fija SHA.
- ¿Cuál es la semántica host completa de los tres puertos y cómo los descubre Catnip? Fase 3.
- ¿Cómo gestiona internamente el driver externo DIO1/BUSY y el switch TX/RX? Requiere disponer de la dependencia exacta o pruebas posteriores.
- ¿Qué versión RP está instalada en la unidad? Requiere inspección/prueba.

## 13. Evidencia principal

- `RP2040/catsniffer/CMakeLists.txt`, `prj.conf`, `west.yml`, `VERSION`.
- `RP2040/catsniffer/boards/rpi_pico.overlay`.
- `RP2040/catsniffer/src/main.c`: `main`, callbacks CDC/UART, `change_mode`, `change_band`, inicialización y hilo LoRa.
- `RP2040/catsniffer/src/USB/usbd_init.c`: pila/descriptores USB.
- `RP2040/catsniffer/src/shell_commands.c`: dispatch local, boot/exit/band/reboot.
- Baseline: `c0cd5a45e019dbd14ed11d039aacb13f300e5731`.

## Por qué esto importa después

Este firmware define la infraestructura que reutilizan tanto Catnip como FeralRF. También separa dos conductas: Cat-Bridge transporta, mientras Cat-Shell y Cat-LoRa requieren interpretación local.

## Comprueba tu comprensión

1. ¿Por qué un byte de Cat-Bridge puede llegar al CC sin convertirse en un comando RP2040?
2. ¿Qué código del RP2040 controla directamente SX1262?
3. ¿Por qué actualizar RP2040 puede cambiar la enumeración USB aunque el firmware CC no cambie?
4. ¿Qué señales permiten al RP2040 iniciar programación del CC?

**Anterior:** [[Mapa de firmware oficial]]  
**Siguiente:** [[Firmware CC1352P7]]  
**Relacionado:** [[Protocolos host-dispositivo]]
