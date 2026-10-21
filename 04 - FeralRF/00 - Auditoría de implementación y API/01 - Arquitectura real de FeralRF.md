# Arquitectura real de FeralRF

## Vista de conjunto

FeralRF no es un único firmware ni una conexión USB directa al CC1352P7. La ruta normal es:

```text
Aplicación / integración
        │ API Python síncrona
        ▼
feralrf.Radio ── COBS + CRC16, USB CDC Cat-Bridge ──► RP2040
        ▲                                             │ UART0 921600 bit/s
        │                                             ▼
decodificación de respuestas                 CC1352P7 / TI-RTOS
        │                                      UART task ⇄ RF task
        └──────── eventos y respuestas ◄────── RF Driver / RF core

Cat-Shell (otro CDC) ──► RP2040: reset/boot y GPIO de selección de banda
```

La API Python controla el estado lógico y las solicitudes de radio del CC1352P7. El RP2040 transporta bytes y controla señales físicas externas. El CC1352P7 ejecuta el protocolo binario, mantiene el estado de RX/TX y usa TI RF Driver. Confundir estas responsabilidades produce dos errores frecuentes: atribuir `band1/2/3` al firmware CC1352P7 y tratar el ACK del CC como confirmación del RF switch.

## Mapa de componentes

| Componente | Procesador | Fuente principal | Interfaz | Recursos/estado |
|---|---|---|---|---|
| `Radio` | PC/Python | `python/feralrf/radio.py` | API pública síncrona; pyserial | puerto, secuencia, cola asíncrona, estado de conexión |
| framing | PC/Python | `python/feralrf/protocol.py` | COBS, delimitador `0x00`, CRC16-CCITT | trama máxima implícita por `uint8 length` |
| builders y presets | PC/Python | `commands.py`, `presets.py` | payloads binarios | conversión/masking de parámetros |
| integraciones | PC/Python | `integrations/killerbee.py`, `emulation/*`, `_jamming.py`, `_spectrum.py` | adaptadores/módulos de nivel superior | no son pilas completas |
| Cat-Bridge | RP2040 | `CatSniffer-Firmware/app/src/main.c:L202-L265` | USB CDC0 ⇄ UART0 | dos ring buffers, contadores locales |
| Cat-Shell | RP2040 | `main.c:L193-L225,L345-L425` | USB CDC2, texto | reset/boot, baudrate, GPIO RF switch |
| overlay USB/UART/GPIO | RP2040 | `app/boards/catsniffer_v3.overlay:L42-L60,L123-L143` | GPIO2/3, GPIO8/9/10, UART0 | boot/reset y CTF1/2/3 |
| entrada TI-RTOS | CC1352P7 | `firmware/cc1352/src/main_rtos.c:L112-L190` | dos tareas estáticas | stacks 4096, prioridad 3 |
| `HostIFTask` | CC1352P7 | `host_if_task.c` | UART y protocolo | parser, un comando pendiente, cola de salida |
| `DataTask` | CC1352P7 | `data_task.c` | eventos entre tareas | despacho de comando, RX/TX diferido |
| `CommandProcessor` | CC1352P7 | `command_processor.c` | IDs y payloads | validación estructural, ACK/ERROR/resultado |
| `ControlTask` | CC1352P7 | `control_task.c` | máquina de estado | PHY/canal/potencia, RX, TX raw/burst/continuous/jam |
| `RadioIF` | CC1352P7 | `radio_if.c` | TI RF Driver | handles, configuraciones RF, colas RX, métricas |
| criptografía | CC1352P7 | `crypto_engine.c` | drivers TI | TRNG, AES, SHA, ECDSA, ECDH |
| cola de salida | CC1352P7 | `output_if.c`, `packet_queue.c` | frames a UART task | profundidad 32 y contador de drops interno |

## PC y biblioteca Python

`Radio.connect()` descubre/abre Cat-Bridge, inicializa el transporte y opcionalmente llama a `reset_device()` (`radio.py:L136-L257`). `Radio.init()` envía `RADIO_INIT`, espera ACK y consulta `GET_INFO` (`radio.py:L399-L453`). Los métodos restantes construyen payloads con `commands.py`, asignan una secuencia y delegan a `_send_command()` / `_read_response()` (`radio.py:L263-L397`).

Una trama lógica es `command:uint8, sequence:uint8, length:uint16-le, payload, crc16:uint16-le`, después COBS y `0x00` (`protocol.py:L18-L90`; espejo C en `firmware/cc1352/src/protocol.c`). El enlace no tiene negociación de versión, ventana ni retransmisión general. Los reintentos están codificados sólo en algunos métodos.

`_read_response()` mezcla dos responsabilidades: correlacionar la respuesta síncrona y atender eventos. Guarda ciertos mensajes con secuencia cero, pero descarta una respuesta inesperada con otra secuencia. Los paquetes RX producidos por firmware usan una secuencia propia 0…254; por tanto, los RX que coinciden con una espera síncrona suelen descartarse. Las excepciones de parseo dentro del bucle también se ignoran (`radio.py:L278-L397`).

## RP2040: frontera de integración

El firmware stock publica tres CDC: `Cat-Bridge`, `Cat-LoRa` y `Cat-Shell` (`catsniffer_v3.overlay:L123-L135`). CDC0 mueve bytes entre USB y UART0 a 921600 bit/s. No conoce COBS, comandos FeralRF ni estados RF. El camino CC→USB registra overrun/drop; el camino USB→CC inserta bytes al ring buffer sin comprobar el retorno de `safe_ring_buf_put()`, por lo que una saturación puede perder bytes sin señalización al host (`main.c:L202-L265`). Es un riesgo de transporte, no una causa demostrada de los timeouts históricos.

Cat-Shell actúa fuera del protocolo binario:

- `boot` fuerza boot/reset y cambia UART a 500000 bit/s.
- `exit` restaura passthrough/reset y 921600 bit/s.
- `band1`, `band2`, `band3` escriben GPIO CTF para `GIG`, `SUBGIG1`, `SUBGIG2` (`main.c:L193-L225,L379-L425`).

`Radio.reset_device()` deduce por defecto Cat-Shell como número de COM de Cat-Bridge + 2, cierra el bridge, escribe `boot`, espera, escribe `exit` y reconecta (`radio.py:L184-L246`). No lee ni valida respuestas de shell. El commit histórico `5e0f9d5` mejoró el descubrimiento en macOS, pero la aritmética sigue siendo una suposición en Windows/Linux. No hay API pública para seleccionar banda o verificar GPIO/front-end.

## CC1352P7 y TI-RTOS

`main_rtos.c` crea dos tareas estáticas de la misma prioridad (3): UART y RF, cada una con stack de 4096 bytes (`main_rtos.c:L112-L190`). La tarea RF inicializa criptografía, abre al arranque un handle RF persistente en modo propietario 433 MHz y entra en `DataTask_run()`. Esto contradice la descripción documental de que todos los `RF_open()` son exclusivamente lazy: modos adicionales sí se abren bajo demanda, pero existe un handle 433 persistente desde el arranque (`main_rtos.c:L141-L178`).

`DataTask_init()` llama una sola vez a `RadioIF_init()` (`data_task.c:L67-L79`). `CMD_RADIO_INIT` llama a `ControlTask_onRadioInit()` y devuelve ACK, pero no vuelve a ejecutar `RadioIF_init()` (`command_processor.c`, caso `CMD_RADIO_INIT`; `control_task.c:L194-L219`). Por tanto, `Radio.init()` reinicia estado de control y detiene RX, pero no equivale a reinicialización completa del backend RF.

### Flujo entre tareas

1. `HostIFTask_poll()` primero extrae hasta cuatro frames de la cola de salida y los escribe por UART.
2. Lee hasta 128 bytes de UART, alimenta el parser y deposita como máximo un comando pendiente para la tarea RF.
3. `DataTask_poll()` procesa ese comando, luego stop RX, TX diferido/start RX y finalmente publica hasta ocho paquetes RX.
4. `OutputIF_send*()` serializa respuestas/eventos y los encola para la tarea UART.

La cola `PacketQueue` tiene profundidad 32. Productor y consumidor pertenecen a tareas diferentes, pero sus índices/contador no usan lock, sección crítica ni atomics (`packet_queue.c`). Esto es un riesgo de concurrencia confirmado en la estructura, aunque no está demostrado que sea la causa de una ejecución fallida. Si la cola está llena, `OutputIF` pierde la respuesta; su contador no aparece en `GET_STATS`.

## Estado y recursos RF

`ControlTask` posee la intención de alto nivel: PHY seleccionado, canal, potencia, RX habilitado y operaciones pendientes raw/burst/continuous/jam. `RadioIF` posee handles, configuraciones TI, colas RX y métricas. Esta división explica por qué varias órdenes reciben ACK antes de operar RF.

- `SET_PHY`, `SET_CHANNEL` y `SET_POWER` actualizan estado; no prueban hardware.
- `SET_PROP_CONFIG` modifica estructuras de comando TI en memoria; es `void` y el handler responde ACK sin conocer aceptación del RF core.
- `RX_START` agenda un evento y responde ACK; el open/start ocurre después. El fallo posterior se emite como `RSP_ERROR` asíncrono.
- TX raw/frame/burst/continuous se acepta en un slot de control; la transmisión ocurre después.
- CW/PRBS sí intenta abrir/configurar/postear el comando antes del ACK, pero el ACK sólo demuestra que `RF_postCmd()` devolvió un handle no negativo.
- Criptografía es síncrona dentro del handler y responde con un resultado específico o ERROR.

El backend RX usa tres entradas de data queue TI y una cola software de ocho `RadioIF_RxPacket`. Después existe la cola de salida de 32 frames, el UART del CC, los buffers RP2040, USB y la lectura Python. `GET_STATS` sólo representa parte de esa cadena.

## Dependencias y target

`firmware/cc1352/CMakeLists.txt` selecciona por defecto `CC1352P7`, `USE_TIRTOS=ON`, compila los módulos anteriores y enlaza bibliotecas precompiladas de RF driver, drivers, driverlib y SYS/BIOS del SDK 8.30.01.01. Los modos RF usan estructuras generadas bajo `syscfg/` y módulos SmartRF locales. La potencia solicitada se resuelve contra tablas locales: Sub-GHz contiene -20…20 dBm y 2.4 GHz contiene -20…5 y 11…20 dBm (`radio_if.c:L209-L317`). La resolución busca un valor exacto y, si no existe, elige la entrada más cercana; la API no comunica el valor efectivo ni el tipo de PA.

## Hardware y límites de afirmación

El CC1352P7 controla RF core y comandos de radio. El RP2040 controla reset/boot y las líneas externas de selección de banda. El código demuestra las escrituras de GPIO, pero sin esquema/medición no demuestra la ruta física exacta de una revisión ni la antena activa. Del mismo modo, `RF_postCmd()` o `RF_runCmd()` exitoso no certifica emisiones OTA.

Existe además una discrepancia documental de pines: material de FeralRF menciona reset en GPIO15, mientras el overlay ejecutable de `CatSniffer-Firmware@c0cd5a4` usa GPIO3 y boot GPIO2. Se conserva como incertidumbre de versión/nomenclatura, no como defecto probado.

