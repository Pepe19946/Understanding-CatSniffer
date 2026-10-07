# FeralRF - Wiki técnica integral

> Encabezado original conservado: # FeralRF — Wiki técnica integral

## Estado de definición y validación — reconciliación del 7 de octubre de 2026

Esta nota es la definición integral canónica. El cuerpo anterior fue elaborado el 6-10-2026 con un corte de evidencia al 5-10 y permanece íntegro; sus afirmaciones “TX actual no demostrado” o “EV-12/13/14 pendientes” se leen como históricas. No constituyen el estado actual. [[FeralRF - Matriz de pruebas]] es la autoridad documental de coberturaactual; [[Registro de validación FeralRF]] explica montaje/cronología y [[Auditoría técnica de validación FeralRF - EV ejecutadas]] interpreta brechas.

EV-12 fechado 6-10 aporta marcadores RAW/FRAME IEEE recibidos por segunda placa, repetición CONT0 de 99 hits y conteos insuficientes con intervalos positivos ensayados. EV-13 aporta control CW/PRBS y timeoutsRX_STOP a baja carga, con instrumentación física diferida. EV-14 aporta bytes IEEE, transición mínima BLE→IEEE y tres mocks host; no provoca error RF real ni reproduce firma. Estos resultados amplían la evidencia, pero no validan íntegramente multi-PHY/presets/crypto/stacks ni demuestran cese/potencia/frecuencia.

La definición se contrasta con [[Arquitectura FeralRF]], [[Matriz de capacidades]], [[Protocolo y API Python]] y [[Fuentes FeralRF]]. Pendientes de contrato identificados: SEQ de RX frente a frase genérica SEQ0; `get_info` público frente a información obtenida por `init()`; GPIO/revisión de bridge; rangos de potencia y clamping default; RF_open/close por ruta, no universal. Las tres versiones (proyecto/FW/Python) son componentes distintos, no incompatibilidad confirmada. El firmware cargado no está ligado a un hash/commit en estas notas. No se realizó una nueva inspección de código o hardware durante esta reconciliación.

GAP/GATT/conexión/active scan BLE retirados no se cuentan como FAIL. Spectrum/reactive/pattern/MIOTY/RSA/AIS/802.15.4g y HighPA requieren distinguir soporte incompleto/pending de capacidad lista pero sin prueba. WMBus/Wi-SUN/Sidewalk/Zigbee/Thread/Matter nombran PHY/raw o integración externa; ningún EV actual acredita pila superior. Guía canónica: [[FeralRF - Guía de validación experimental]].

## Registro documental previo — conservar su fecha y alcance


> [!info] Línea base y alcance
> **Fecha de análisis:** 6 de octubre de 2026 (America/Mexico_City)  
> **Repositorio:** `C:/Users/Support/Documents/ec-projects/catsniffer-feralrf/FeralRF`  
> **Rama:** `main`, siguiendo `origin/main` según las referencias locales (no se hizo `fetch`)  
> **HEAD:** `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`  
> **Fecha/asunto de HEAD:** 2026-07-22 12:34:03 -06:00 — `docs: fix remaining 'RF_open at boot' folklore in architecture layer rules`  
> **Árbol de trabajo al iniciar:** el superproyecto no tenía cambios propios salvo el submódulo TI marcado modificado: `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01`. Dentro del submódulo, `HEAD` estaba separado (*detached*) en `5b31d0a4903351e544546e23ef3330eaa4291ceb`, tag/descripción `lpf2-8.30.01.01-dirty`, con `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md` modificado. No se alteró ese estado.  
> **Alcance:** reconstrucción técnica del sistema actual mediante código, configuración, documentación e historia Git; contraste secundario con el Vault y con la evidencia experimental existente. No es una guía de validación, no reproduce una campaña de pruebas y no afirma resultados RF nuevos. No se compiló, flasheó ni operó hardware.

## Cómo leer la evidencia

Este documento usa etiquetas deliberadas:

- **Documentación dice:** afirmación de README, docs o comentarios.
- **Código implementa:** conducta alcanzable en el `HEAD` actual.
- **Git muestra:** cambio o intención registrada; no prueba funcionamiento.
- **Evidencia experimental muestra:** resultado preservado por la campaña del Vault o por la matriz histórica, con su alcance.
- **Inferencia:** interpretación técnica razonada, no declaración explícita del proyecto.
- **Conclusión:** síntesis después de comparar las anteriores.

La jerarquía es: código/configuración actual y Git del repositorio; después, notas del Vault; finalmente, contexto externo oficial. Un nombre de preset no equivale a un stack, un test unitario no equivale a hardware y un `ACK` no equivale a emisión RF observada.

---

## 1. Qué es FeralRF

FeralRF es una combinación de:

1. firmware para el **CC1352P7** de CatSniffer;
2. un protocolo binario host↔firmware;
3. el paquete Python `feralrf`, que expone control, RX/TX, presets y aceleradores criptográficos;
4. ejemplos, pruebas host y utilidades de integración, especialmente KillerBee.

Su objetivo práctico es convertir la radio integrada multibanda del CC1352P7 en una plataforma programable desde Python para recibir paquetes crudos, transmitir bytes, cambiar PHY/configuración propietaria y ejecutar modos de prueba. Opera principalmente en la capa física y en una porción pequeña de enlace: conserva metadatos y clasifica PDU BLE, pero no implementa hoy stacks completos de BLE, Zigbee, Thread, Matter, Wi-SUN, Wireless M-Bus o Sidewalk.

**Documentación dice:** el README lo llama firmware universal y API host multi-PHY, con RX/TX crudo y crypto.

**Código implementa:** una API síncrona, comandos COBS+CRC, backends BLE/IEEE 802.15.4/propietario, colas RX, TX por paquetes, CW/PRBS, jamming experimental y crypto. No hay decodificadores Zigbee/Thread/Wi-SUN ni una pila BLE pública actual.

**Conclusión:** es, a la vez, firmware de radio, framework de transceptor, plataforma de experimentación PHY y base de análisis de paquetes. Puede alimentar un analizador o flujo de investigación, pero no es por sí solo un analizador de espectro calibrado, una pila de red completa ni equipo de certificación.

Modelo mental verificado:

```text
usuario / script
  → feralrf.Radio (PC)
  → frame COBS + CRC16 por Cat-Bridge USB CDC
  → RP2040 stock, passthrough USB↔UART
  → UART0 del CC1352P7
  → HostIFTask / CommandProcessor
  → ControlTask / DataTask
  → RadioIF
  → TI RF Driver
  → RF Core del CC1352P7
  → front-end / antena
```

Fuentes principales: `README.md`; `docs/ARCHITECTURE.md`; `python/feralrf/radio.py`; `firmware/cc1352/src/{command_processor,control_task,data_task,radio_if}.c`.

---

## 2. Contexto de ejecución de hardware

### 2.1 RP2040: interfaz y control de placa

El RP2040 es un MCU dual-core Arm Cortex-M0+ con USB y UART; esos rasgos son contexto externo del fabricante ([especificaciones RP2040](https://www.raspberrypi.com/products/rp2040/specifications/), [datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf)). En CatSniffer ejecuta firmware Zephyr de Electronic Cats, no FeralRF.

Responsabilidades relevantes:

- termina el USB de la PC y presenta tres CDC: Cat-Bridge, Cat-LoRa y Cat-Shell;
- copia bytes Cat-Bridge↔UART hacia/desde el CC1352P7;
- interpreta comandos de texto Cat-Shell y controla BOOT/RESET del CC;
- controla el SX1262 por SPI/GPIO en la ruta Cat-LoRa;
- participa en selección/configuración de placa según el firmware oficial.

En runtime FeralRF, Cat-Bridge es transparente semánticamente: el RP2040 mueve bytes, pero quien entiende COBS, CRC y comandos es el CC1352P7. Para flasheo, Catnip usa Cat-Shell (`boot`/`exit`) y Cat-Bridge hacia el bootloader serie del CC. `Radio.reset_device()` también intenta usar Shell.

**Código implementa:** FeralRF no contiene firmware RP2040 actual; Git eliminó la implementación no mantenida en `ebdfcf3` (2026-07-21). La dependencia es el firmware stock o compatible de CatSniffer.

### 2.2 CC1352P7: procesador y radio de FeralRF

El CC1352P7 integra un Arm Cortex-M4F a 48 MHz, memoria, periféricos y un subsistema RF programable Sub-1 GHz/2.4 GHz. TI lista 2/4-(G)FSK, MSK, OOK, BLE e IEEE 802.15.4 entre sus capacidades de silicio ([datasheet CC1352P7](https://www.ti.com/lit/ds/symlink/cc1352p7.pdf)). Que el chip anuncie un protocolo no significa que este firmware incluya su stack.

En él ejecutan:

- TI-RTOS7/SYSBIOS y dos tareas cooperativas de prioridad 3;
- `HostIF`, el parser del protocolo y `CommandProcessor`;
- `ControlTask`, `DataTask`, `PhyManager`, `LLManager` y `RadioIF`;
- TI Drivers y el firmware interno del RF Core;
- el motor crypto TI mediante `crypto_engine.c`.

El Cortex-M4F ejecuta la aplicación y envía *radio operations* al RF Core mediante TI RF Driver. Los callbacks y colas transfieren resultados al M4F; después el firmware los empaqueta para el host.

### 2.3 SX1262 / ruta LoRa

El SX1262 es un transceptor periférico comandado por el RP2040. **FeralRF no lo usa:** no hay driver SX1262, acceso Cat-LoRa ni comando FeralRF dirigido a él. Sidewalk LR, basado en modulación tipo LoRa, queda explícitamente fuera de los presets del CC1352P7. La presencia física del SX1262 no convierte la ruta FeralRF en LoRa.

### 2.4 Front-end y antena

El firmware contiene selección dinámica de antena/ruta mediante configuración SysConfig y referencias a DIO28/29/30; el hardware CatSniffer, sin embargo, también conecta señales de selección del switch externo al RP2040. El Vault identifica una discrepancia de propiedad/control de `CTF1..3` entre configuración tipo LaunchPad y la placa CatSniffer.

**Conclusión:** puede afirmarse qué GPIO pretende usar el código, pero no que cada ruta RF externa quede correctamente seleccionada sólo por ejecutar `set_phy()`. Los resultados OTA históricos sugieren que existieron configuraciones funcionales de banco; no documentan de forma suficiente el estado del selector para eliminar la duda de diseño.

---

## 3. Arquitectura extremo a extremo

### 3.1 Fronteras, procesador y representación

| Frontera | Ejecuta en | Interfaz/representación | Responsabilidad |
|---|---|---|---|
| usuario→API | PC | llamadas Python | intención: init, config, RX, TX, crypto |
| API→codec | PC | `Command` + payload `bytes` | validar argumentos y construir payload LE |
| codec→serial | PC | frame COBS terminado en `00` | delimitar, CRC y secuencia |
| PC→RP2040 | PC/RP | USB CDC Cat-Bridge | transporte serial virtual |
| RP→CC | RP/CC | UART 921600, 8N1, sin flow control | passthrough de bytes |
| UART→comando | CC M4F | buffer COBS estático | validar frame y despachar por ID |
| comando→estado | CC M4F, RF task | eventos/estado estático | excluir RX/TX, programar operación |
| estado→radio | CC M4F/RF Core | TI `RF_Op`, SmartRF structs | setup, frecuencia, RX/TX |
| radio→host | RF Core→CC→RP→PC | data queue→`RSP_RX_PACKET`→COBS | paquete y metadatos asíncronos |

### 3.2 Modelo de tareas actual

`src/main_rtos.c` crea primero `RfTask_taskFxn` y luego `UartTask_taskFxn`, ambas con pila estática de 4096 bytes. UART task lee como máximo 128 bytes por vuelta y sólo conserva un comando pendiente. RF task ejecuta el comando pendiente, bombea estado TX/RX y drena hasta ocho paquetes RF por vuelta. La cola host tiene 32 frames y se vacían hasta cuatro por vuelta.

```text
UartTask                         RfTask
---------                        ------
HostIFTask_poll                  HostIFTask_processPendingCommand
  lee UART                         CommandProcessor
  acumula hasta 0x00               ControlTask / RadioIF
  copia 1 frame pendiente           DataTask / colas RF
  vacía cola de salida  ←────────   OutputIF_sendResponse
```

Si llega un segundo comando mientras uno sigue pendiente, `HostIFTask` envía `RSP_ERROR/ERR_INVALID_STATE` asíncrono con secuencia 0. Una cola host llena descarta frames y marca `TASK_EVENT_PACKET_QUEUE_DROPPED`; no existe retransmisión.

### 3.3 Camino de control

Ejemplo `Radio.set_phy(PHY.IEEE_802_15_4, 25)`:

```text
Radio.set_phy
  → CommandBuilder.set_phy: <BHI>
  → build_frame(CMD_SET_PHY, seq, payload)
  → USB CDC / RP passthrough / UART
  → HostIFTask_processPendingCommand
  → CommandProcessor: valida 1 o 7 bytes
  → ControlTask_onSetPhy
  → PhyManager_select + LLManager_select + RadioIF_setPhy
  → RSP_ACK con el mismo seq
```

El `ACK` confirma que la selección lógica fue aceptada. `RadioIF_setPhy` ajusta estado y comandos, pero el RF Core puede no haber transmitido ni recibido todavía.

`RADIO_INIT` reinicia estado lógico, detiene RX, borra métricas y devuelve `ACK`; `GET_INFO` devuelve versión firmware `1.0.0`, capabilities `0x07` y serial constante ASCII `FERALRF1`; no es el serial físico de la placa.

### 3.4 Camino de datos RX

```text
RF Core → TI data queue → RadioIF_rfCallback
 → RadioIF_poll / process{Ble,Ieee154,Prop}Packets
 → cola interna RadioIF_RxPacket
 → LLManager_processRxPacket
 → DataTask_emitRxPacket
 → OutputIF / cola de 32
 → UART / RP2040 / CDC
 → Radio.read_packets()
 → Packet o RxStreamError
```

La respuesta RX incluye timestamp de 64 bits, canal, RSSI con signo, LQI, bandera CRC, longitud/datos y tres bytes de metadata LL. Python preserva esos campos en `Packet`.

---

## 4. Mapa del repositorio

| Ruta | Rol | Ejecuta en | Tecnología | Relaciones importantes |
|---|---|---|---|---|
| `firmware/cc1352/` | aplicación embebida | CC1352P7 | C, TI-RTOS7, TI Drivers | protocolo, tareas, RF, crypto |
| `firmware/cc1352/src/` | implementación | CC Cortex-M4F | C | módulos explicados en §5 |
| `firmware/cc1352/include/` | contratos internos y wire IDs | build/CC | headers C | espejo con enums Python |
| `firmware/cc1352/syscfg/` | configuración generada/derivada | build/RF Core | SysConfig C/H | drivers, BLE, IEEE y 433 |
| `firmware/cc1352/linker/` | memoria/entrada P/P7 | linker | GNU ld | target elegido por CMake |
| `firmware/sdk/...` | SDK fabricante | build | submódulo TI 8.30.01.01 | RF driver, RTOS, headers/libs |
| `python/feralrf/` | API pública y helpers | PC | Python ≥3.9 | serializa el mismo protocolo |
| `python/feralrf/emulation/` | personalidades/payloads | PC | Python | usa API TX; no crea stacks completos |
| `python/feralrf/integrations/` | adaptador KillerBee | PC | Python | IEEE 802.15.4 RX/inject/jam |
| `python/examples/` | smokes y gates | PC + hardware | Python/Bash | pruebas operativas, no evidencia por existir |
| `python/tests/` | tests host | PC | pytest, mocks | sin RF/hardware salvo futuros markers |
| `docs/` | contrato y estado declarado | lector | Markdown | puede quedar detrás del código |
| `hardware/` | fuentes KiCad y referencias | diseño | KiCad/PDF/STEP | copia con procedencia/revisión ambigua |
| `docker/` | entorno reproducible parcial | host/build | Docker | toolchain, no garantiza libs TI completas |
| `.github/` | automatización | CI | YAML | Python test/lint; firmware best-effort |

`hardware/` no debe tomarse automáticamente como la revisión física conectada: el Vault halló etiquetas de revisiones distintas entre PCB, esquema y silkscreen. `firmware/sdk` es código de fabricante, no parte funcional escrita por FeralRF; sus miles de scripts no forman parte del inventario de scripts propios.

---

## 5. Arquitectura del firmware CC1352P7

| Módulo | Quién lo llama / qué llama | Estado y hardware que posee |
|---|---|---|
| `main_rtos.c` | reset/runtime→tasks | dominios, GPIO, semáforos, tareas; abre handle 433 |
| `host_if.c` | `HostIFTask`→UART driverlib | UART0 DIO12/13, 921600 8N1 |
| `host_if_task.c` | UART task; difiere a RF task | buffer COBS, overflow, un frame pendiente |
| `protocol.c/.h` | host parser/output | COBS, CRC16, frame y IDs canónicos |
| `command_processor.c` | RF task→Control/Radio/Crypto | validación de payload, respuesta síncrona |
| `control_task.c` | comandos/DataTask→RadioIF | PHY/canal/potencia, exclusión y scheduler TX/jam |
| `data_task.c` | RF task | eventos RX/TX y emisión host de paquetes |
| `task_event.c` | tareas | bitmask compartido; no es cola |
| `output_if.c` | cualquier productor→PacketQueue | frame de respuesta y drop no bloqueante |
| `packet_queue.c` | output/UART | 32 frames host estáticos |
| `phy_manager.c` | Control/Radio | ocho IDs PHY y perfil LL BLE/default |
| `ll_manager.c` | DataTask | recorta/clasifica PDU BLE; pasa crudo lo demás |
| `radio_if.c` | Control/Data | toda la sesión TI RF, queues, RX/TX/test/jam |
| `crypto_engine.c` | CommandProcessor | TRNG/AES/SHA/ECDH/ECDSA TI |
| `smartrf_*.c`, `syscfg/*` | RadioIF/TI Driver | setup, FS, RX/TX, patches, overrides |
| `tx_queue.c` | sin caller actual significativo | legado de BLE central; 8 entradas |
| `main.c` | ruta `USE_TIRTOS=OFF` | variante NoRTOS, no baseline predeterminado |
| `startup/main_rtos.c` | no enlazado por CMake actual | artefacto alterno/histórico |

### 5.1 Arranque y contradicción `RF_open`

`main_rtos.c::RfTask_taskFxn` inicializa crypto y abre al arrancar un handle dedicado 433 MHz con `Prop0_mode433`, postea `Prop0_cmdFs433` y lo registra en `RadioIF`. Para otros modos, `RadioIF_switchRfMode` abre/configura bajo demanda.

**Documentación dice:** `docs/ARCHITECTURE.md` y el asunto de HEAD corrigen folklore hacia “`RF_open` lazy, no en boot”.

**Código implementa:** hay una excepción material: 433 sí abre en boot (`main_rtos.c`); no-433 es lazy. Comentarios de `data_task.c`, `radio_if.h` y `radio_if.c` lo confirman.

**Conclusión:** describir “ningún `RF_open` en boot” es falso para el binario predeterminado.

### 5.2 HostIF, framing y errores

`HostIFTask_poll` delimita por `0x00`. Un frame demasiado largo produce `ERR_FRAME_TOO_LONG` con seq 0. `CommandProcessor_processEncodedFrame` decodifica COBS, verifica CRC/longitud y después llama `handle_command`. `OutputIF_sendResponse` protege el límite de 255 bytes y encola el frame codificado.

### 5.3 ControlTask y scheduler

Todo estado es estático; no hay `malloc`. RX y TX por paquetes se excluyen. `TX_RAW`, `TX_FRAME`, `TX_BURST` y `TX_CONTINUOUS` copian hasta 125 bytes. `TX_BURST` conserva contador, intervalo y próxima fecha; `TX_CONTINUOUS` conserva el mismo estado sin contador. El tiempo de scheduling se integra desde SysTick y no constituye un temporizador de precisión metrológica.

Sólo el fallo tardío de `TX_RAW` genera actualmente `RSP_ERROR` asíncrono. En burst/continuous, un fallo de `RadioIF_transmitRaw` limpia el estado en silencio. Esto importa al interpretar scripts que imprimen `scheduled=N` o `PASS` justo después del ACK.

### 5.4 RadioIF y TI RF Driver

`RadioIF` posee handles, objetos, comandos SmartRF, data queue, ring lógico RX, métricas y configuración propietaria. TI explica que un PHY combina setup command, patches y overrides ([guía PHY oficial](https://software-dl.ti.com/simplelink/esd/simplelink_cc13xx_cc26xx_sdk/latest/exports/docs/proprietary-rf/proprietary-rf-users-guide/rf-core/phy-configuration.html)); eso coincide con los `smartrf_*`, patches enlazados y `RF_open` del proyecto.

- BLE: canales advertising, access address `0x8E89BED6`, CRC init `0x555555`, hopping opcional 37–39.
- IEEE 802.15.4: configuración 2.4 GHz y cola con RSSI/correlation+CRC/timestamp.
- propietario: frecuencia, modulación, rate, deviation, BW, sync word y `formatConf`; incluye variantes 433, 868/915 y experimento 2.4 GHz.
- OOK: carga patches genook y tiene un lock de sesión/frecuencia documentado; se recomienda reset.

`RF_postCmd` es asíncrono; `RF_runCmd` bloquea hasta terminar una operación según TI ([RF Driver overview](https://software-dl.ti.com/lprf/simplelink_academy/modules/prop_01_basic/prop_01_basic.html)). FeralRF usa ambos según la ruta.

**Discrepancia adicional:** `docs/ARCHITECTURE.md` afirma que `CMD_FS` va siempre por `RF_postCmd`. Las rutas normales y test lo hacen, pero `RadioIF_startJamSession` llama `RF_runCmd(...fs_cmd...)` en `radio_if.c:2895`. El jamming debe considerarse experimental y la regla documental no es universal en HEAD.

### 5.5 LLManager, estadísticas y RX

Para BLE, `LLManager` exige al menos dos bytes, usa longitud de header, recorta trailing bytes y clasifica ADV/SCAN/CONNECT/DATA. Para IEEE/propietario no interpreta protocolo. `GET_STATS` devuelve 36 bytes: `rx_ok`, `rx_crc_err`, `rx_drop`, `rx_overflow` y cinco contadores BLE LL. Los drops de `PacketQueue` host no se incorporan explícitamente a esos cuatro contadores.

### 5.6 Crypto y código residual

Crypto expone TRNG, AES-128 ECB/CTR/CBC/CCM/GCM, SHA-256, ECDH P-256/Curve25519 y ECDSA P-256. RSA aparece como capacidad del silicio/roadmap, no como comando actual.

`radio_if.h` conserva prototipos BLE central/follower y `radio_if.c` conserva primitivas sin comandos públicos actuales; el stack BLE/GATT y sus IDs se retiraron. Son residuos internos/históricos, no API soportada.

---

## 6. Arquitectura Python

### 6.1 Capas

| Archivo | Papel |
|---|---|
| `protocol.py` | COBS, CRC16-CCITT, header y parseo |
| `enums.py` | PHY, command/response y clasificación estable/experimental |
| `commands.py` | payloads; no incluye ID ni framing |
| `radio.py` | conexión, secuencia, respuestas, API pública y crypto |
| `_responses.py` | parsers auxiliares; `SpectrumDataResponse` queda sin ruta E2E |
| `exceptions.py` | jerarquía de error host |
| `presets.py` | 31 configuraciones propietarias |
| `_spectrum.py` | contenedor/plot host; feature pendiente |
| `_jamming.py` | modelos de políticas no conectados a comandos reactivos/pattern |
| `emulation/*` | payloads/personalidades PHY |
| `integrations/killerbee.py` | adapter IEEE 802.15.4 para KillerBee |

### 6.2 Conexión, reset y secuencia

`Radio.list_devices()` filtra varios VID genéricos; para VID CatSniffer `0x1209` exige “Bridge” en descripción/producto. Sin puerto explícito usa el primer candidato, sin sondeo FeralRF. `connect()` abre pyserial a 921600 y limpia buffers después de un segundo.

La secuencia va 0..254 y salta `0xFF`. `_read_response()` ignora ecos de comando, correlaciona respuestas síncronas, almacena eventos seq 0 y acepta errores async seq 0/0xFF por compatibilidad. Frames inválidos se descartan silenciosamente dentro del bucle.

`reset_device()` calcula Shell como el sufijo numérico de Bridge + 2, envía `boot`/`exit`, reabre Bridge e invoca `init()`.

`disconnect()` cierra Serial y limpia el estado local. No existe un `get_info()` público independiente en el `HEAD`: `init()` envía `CMD_RADIO_INIT` y después `CMD_GET_INFO`. La jerarquía de fallos separa `ConnectionError`, `ProtocolError`, `CommandError`, `TimeoutError`, `RadioError` y `CryptoError`; además, `RxStreamError` representa un `RSP_ERROR` asíncrono observado durante streaming.

**Evidencia experimental muestra:** KI-15 se reprodujo en Windows: Bridge COM88, Shell real COM87, pero la API calculó COM90. Reset manual explícito funcionó 1/1. Por ello el mecanismo físico existe, pero la API pública de reset es problemática en esa enumeración.

### 6.3 Métodos RF públicos y semántica real

| Método | Comando/payload | Handler/efecto | Retorno y alcance del éxito |
|---|---|---|---|
| `init()` | `RADIO_INIT`, luego `GET_INFO` | reinicia estado; lee info | `DeviceInfo`; prueba diálogo, no RF |
| `set_phy()` | `SET_PHY <BHI>` | selecciona perfil/estado | `None` tras ACK; no prueba sintonía/selector externo |
| `set_channel()` | `SET_CHANNEL B` | actualiza canal/comandos | ACK de control |
| `set_power()` | `SET_POWER b` | guarda potencia | ACK; no mide dBm físico |
| `configure_prop()` | 18 B LE | muta configuración SmartRF | ACK; no prueba modulación ni interoperabilidad |
| `set_adv_hop()` | booleano | hopping BLE adv | ACK de opción |
| `get_stats()` | sin payload | snapshot de métricas | `DeviceStats`; contadores, no instrumento |
| `start_rx()` | sin payload | marca evento RX | ACK antes de `RadioIF_startRx`; error tardío posible |
| `read_packets()` | stream | parsea RX_PACKET/ERROR | `Packet`/`RxStreamError` |
| `stop_rx()` | sin payload | marca stop; 3 retries host | ACK de parada lógica |
| `transmit()` | len+bytes+power | agenda TX_RAW | ACK de enqueue; no final RF |
| `transmit_frame()` | len+bytes | llama el mismo TX raw con potencia almacenada | no agrega un stack/frame de protocolo |
| `transmit_burst()` | len+bytes+count u16+interval u32 | scheduler finito | ACK de programación |
| `transmit_continuous()` | len+bytes+interval u32 | repite paquetes hasta stop | ACK de inicio lógico |
| `stop_transmit()` | vacío | limpia raw/burst/continuous/jam | ACK; no mide energía residual |
| `tx_cw()` | primero `SET_POWER`, luego `TX_CW` | `CMD_TX_TEST`, carrier | ACK indica command handle válido |
| `tx_prbs()` | power + mode 1/2 | PRBS-15/32 | ACK; espectro requiere instrumento |
| `tx_test_stop()` | vacío | cancela/flush test | idempotente |
| `start_jam()/stop_jam()` | canal/power/duración | sesión repetida, guards/cooldown | experimental; ACK no prueba interferencia |
| `random_bytes()` | `CMD_RANDOM`, longitud 1..240 | TRNG del CC | bytes; sólo prueba el chip si corre HIL |
| `aes_encrypt()/aes_decrypt()` | `CMD_AES_ECB/CTR/CBC`, op+key+IV+datos | AES one-shot | bytes; límites 16/192/200 B según modo |
| `aes_ccm_*()/aes_gcm_*()` | `CMD_AES_CCM/GCM`, key+nonce/IV+AAD+datos/tag | AEAD | bytes/tag o `CryptoError`; no gestiona claves/protocolo |
| `sha256()` | `CMD_SHA256`, hasta 240 B | hash one-shot | digest de 32 B |
| `ecdsa_sign()/ecdsa_verify()` | `CMD_ECDSA_*`, curva+claves+hash/firma | firma/verificación | firma 64 B o booleano; soporte de curva depende del firmware |
| `ecdh()` | `CMD_ECDH`, curva+privada+peer public | secreto compartido | 32 B; no implementa un handshake superior |

**Hallazgo de código:** `transmit(packet)` usa por defecto `power_dbm=-128`. El firmware no trata `-128` como sentinel “usar potencia configurada”: `RadioIF_resolveTxPowerValue` busca la entrada exacta y, si no existe, toma la potencia más cercana, previsiblemente la mínima de la tabla. En cambio `transmit_frame()` usa `s_tx_power_dbm` establecido por `SET_POWER`. Hasta confirmar intención, conviene pasar potencia explícita a `transmit()`.

---

## 7. Protocolo host FeralRF

### 7.1 Frame y transporte

```text
antes de COBS:
[CMD/RSP:1][SEQ:1][LEN:2 little-endian][PAYLOAD:0..255][CRC16:2 little-endian]

en el cable:
COBS(frame) + 0x00
```

CRC16-CCITT: polinomio `0x1021`, inicial `0xFFFF`, sobre header+payload. El máximo pre-COBS es 261 bytes. No existe fragmentación. La implementación Python (`protocol.py`) y C (`protocol.c`) comparten algoritmo y formato.

### 7.2 IDs actuales

| Grupo | IDs |
|---|---|
| configuración | `01 CMD_RADIO_INIT`, `02 CMD_SET_CHANNEL`, `03 CMD_SET_POWER`, `04 CMD_SET_PHY`, `05 CMD_GET_INFO`, `06 CMD_GET_STATS`, `07 CMD_SET_ADV_HOP`, `08 CMD_SET_PROP_CONFIG` |
| RX | `10 CMD_RX_START`, `11 CMD_RX_STOP` |
| TX paquetes | `20 CMD_TX_RAW`, `21 CMD_TX_CONTINUOUS`, `22 CMD_TX_BURST`, `23 CMD_TX_FRAME`, `24 CMD_TX_STOP` |
| jamming | `30 CMD_JAM_CONTINUOUS`, `33 CMD_JAM_STOP`; `31 JAM_REACTIVE` y `32 JAM_PATTERN` están reservados/pendientes |
| test | `55 CMD_TX_CW`, `56 CMD_TX_PRBS`, `57 CMD_TX_TEST_STOP` |
| crypto | `59 CMD_RANDOM`, `5A..5E CMD_AES_*`, `5F CMD_SHA256`, `60 CMD_ECDH`, `61/62 CMD_ECDSA_*` |
| respuestas | `80 RSP_ACK`, `81 RSP_ERROR`, `90 RSP_RX_PACKET`, `93 RSP_STATS`, `94 RSP_INFO`, `95..9C` respuestas crypto |

Los IDs BLE stack retirados (`09`, `0B`, `40–54`, respuestas `A0–B2`) no deben reutilizarse según `protocol.h`.

### 7.3 ACK — Acknowledgement

`ACK` significa que el firmware validó el payload y aceptó/aplicó la parte síncrona del comando. Para `TX_RAW/BURST/CONTINUOUS` suele emitirse después de copiar estado y marcar un evento, antes de la llamada RF real. Para `RX_START`, ocurre antes de que `DataTask` invoque `RadioIF_startRx`.

```text
ACK = comando aceptado por CommandProcessor
ACK ≠ operación RF terminada
ACK ≠ onda emitida/recibida
ACK ≠ frecuencia, potencia o modulación medidas
```

CW/PRBS son algo más fuertes: el handler llama `RadioIF_runTxTest` antes de ACK y verifica que se obtuvo un command handle, pero tampoco demuestra la señal físicamente.

### 7.4 ERROR síncrono y asíncrono

Errores síncronos conservan el `SEQ` del comando: comando/payload/frame/estado inválido, RF no listo o init fallido. Errores asíncronos usan seq 0; hosts aceptan además `0xFF` histórico. Casos actuales: fallo tardío de RX start, fallo tardío TX_RAW y host busy. No todos los fallos tardíos generan evento: burst/continuous pueden detenerse silenciosamente.

**Trazas representativas.** `Radio.transmit()` construye `CMD_TX_RAW`, `_send_command()` asigna `SEQ`, `protocol.encode_frame()` añade longitud/CRC/COBS y Serial lo entrega por Bridge/UART; `CommandProcessor` valida, `ControlTask` agenda y `RadioIF_transmit()` llama al RF Driver. El `RSP_ACK` puede volver antes de este último paso. En sentido inverso, un callback RF deja una entrada RX, `DataTask` la convierte en `RSP_RX_PACKET` con `SEQ=0`, `OutputIF` la encuadra y el lector Python la materializa como `Packet`; no responde a un comando pendiente.

---

## 8. Arquitectura RX

1. Python envía `RX_START`.
2. `CommandProcessor` rechaza si hay TX/jam activo; si no, marca `TASK_EVENT_CONTROL_RX_START` y responde ACK.
3. `DataTask_poll` llama `RadioIF_startRx` en RF task.
4. `RadioIF` cierra sesión TX, reinicia la cola, configura backend BLE/IEEE/prop y postea RX.
5. callback TI marca `RF_EventRxEntryDone`/overflow.
6. `RadioIF_poll` interpreta la cola del modo activo y encola `RadioIF_RxPacket`.
7. `LLManager` clasifica BLE o deja crudo.
8. `DataTask` emite hasta ocho paquetes por poll; `OutputIF` los encola.
9. Python produce objetos `Packet`.

| Campo | Origen / matiz |
|---|---|
| `timestamp_us` | RAT ticks / 4; útil para correlación, no timestamp PC absoluto |
| `channel` | estado seleccionado, no medición independiente de frecuencia |
| `rssi_dbm` | byte appended por RF Core; no calibración de laboratorio |
| `lqi` | correlation IEEE; 0 en propietario; semántica variable por PHY |
| `crc_ok` | status del backend; frames CRC erróneo se cuentan y descartan |
| `data` | bytes crudos limitados; máximo emitido 239 B por metadata |
| LL meta | clasificación BLE simple; no decodificación completa |

Posibles pérdidas: data queue RF, cola lógica RadioIF, límite ocho/poll, `PacketQueue` host de 32 y UART sin flow control. `rx_drop`/`rx_overflow` ayudan, pero no contabilizan todas las capas. RX crudo tampoco descifra seguridad ni ensambla automáticamente MAC/red/aplicación.

---

## 9. Arquitectura TX

### 9.1 `TX_RAW`

Entrada: 1..125 bytes y potencia `int8`. `ControlTask_onTxRaw` copia los bytes, marca pending y ACK. Después RF task llama `RadioIF_transmitRaw`, que selecciona BLE advertising raw, IEEE 802.15.4 o propietario. Si falla, emite `RSP_ERROR` seq 0. El host que deja de leer después del ACK puede no observarlo.

### 9.2 `TX_FRAME`

El nombre no implica framing superior. `CommandProcessor` valida `len+bytes`; `ControlTask_onTxFrame()` llama directamente `ControlTask_onTxRaw(payload, ..., s_tx_power_dbm)`. La diferencia real frente a RAW es principalmente el payload wire (sin byte power) y uso de la potencia almacenada. La construcción concreta de comandos RF del PHY ocurre igualmente en `RadioIF`.

### 9.3 `TX_BURST`

`count` es u16 1..65535; `interval_us` es u32. Al aceptar, guarda copia, contador restante, potencia actual y `next_due`; ACK significa “programado”. Cada `DataTask_poll` envía a lo sumo una iteración cuando vence la fecha. Después de una transmisión exitosa decrementa; al llegar a cero limpia estado. Si una falla, limpia inmediatamente y **no notifica host**.

Por tanto, `scheduled=N` en `ota_tx_burst.py` sólo significa que la API recibió ACK, no que se observaron N emisiones.

### 9.4 `TX_CONTINUOUS`

Es repetición indefinida de un paquete con intervalo; no una portadora continua. Ejecuta una TX por vencimiento hasta `TX_STOP` o fallo. Si `interval_us=0`, intenta una por vuelta de polling, todavía separada por overhead de software/driver. Para energía RF continua se usa CW.

### 9.5 `TX_STOP`

Limpia pending raw/burst/continuous, estados de potencia por operación, flags y sesión jam. No llama específicamente a cancelar una TX one-shot ya bloqueada en `RF_runCmd`; su efecto principal es sobre operaciones programadas entre iteraciones.

### 9.6 CW y PRBS

- **CW — Continuous Wave:** portadora sin modulación de datos, corre hasta `TX_TEST_STOP`.
- **PRBS — Pseudo-Random Binary Sequence:** patrón pseudoaleatorio modulado para evaluar espectro/enlace. HEAD ofrece PRBS-15 (`mode=1`) y PRBS-32 (`mode=2`). El docstring histórico del header menciona PRBS-9, pero el código y Python dicen 15/32; BLE DTM PRBS-9 no se expone.

El firmware crea `CMD_TX_TEST`, `TRIG_NEVER`, lo postea y conserva handle para cancelación. Un analizador de espectro/power meter es necesario para frecuencia, máscara y potencia.

---

## 10. Conceptos PHY necesarios

- **RF:** energía electromagnética en radiofrecuencia.
- **carrier / portadora:** frecuencia central sobre la que se impone información.
- **PHY:** reglas físicas para frecuencia, modulación, rate, sincronización y representación de bits/símbolos.
- **canal:** índice que mapea a una frecuencia/segmento dentro de una tecnología.
- **modulación:** forma de variar portadora para representar símbolos.
- **bitrate:** bits/s; **symbol rate:** símbolos/s. En modulaciones multi-bit no son necesariamente iguales.
- **deviation:** desplazamiento de frecuencia en FSK alrededor de la portadora.
- **bandwidth:** ancho espectral ocupado/aceptado; `rx_bw` en presets es un valor de registro, no Hz autoexplicativos.
- **preamble:** patrón inicial para detección/sincronía.
- **sync word/access address:** patrón que identifica inicio/tipo de tráfico esperado.
- **packet/frame:** “packet” suele ser unidad genérica; “frame” una unidad estructurada de una capa. FeralRF no usa siempre los nombres con una distinción fuerte.
- **CRC/FCS:** detección de errores; CRC válido no autentica el emisor.
- **RSSI:** estimación de potencia recibida; no sustituye un medidor calibrado.
- **TX power:** nivel solicitado en dBm; la tabla/PA/ruta/antena determinan el valor real.

Familias implementadas:

- BLE LE 1M, 2M y Coded S=2/S=8 como PHY raw. Bluetooth define LE 1M, LE 2M y LE Coded, donde S controla redundancia ([Bluetooth SIG overview](https://www.bluetooth.com/wp-content/uploads/2023/02/2301_5.4_Tech_Overview_FINAL.pdf)).
- IEEE 802.15.4 2.4 GHz O-QPSK, canales 11–26. IEEE define PHY y MAC para redes de baja tasa ([IEEE 802.15.4](https://standards.ieee.org/ieee/802.15.4/12701/)).
- propietario 2/4-(G)FSK, MSK y OOK en bandas configurables soportadas por el silicio/configuración.

GFSK filtra pulsos antes de FSK para reducir ocupación; OOK conmuta presencia/amplitud de portadora; 4-FSK usa cuatro estados de frecuencia y transporta más bits por símbolo. Estas definiciones son contexto RF, no prueba de que todos los parámetros de cada preset sean conformes a una norma.

---

## 11. Protocolos y tecnologías nombradas

| Tecnología | Nivel/contexto | Parte relevante | Qué implementa FeralRF | Qué no implementa | Equipo/tráfico plausible |
|---|---|---|---|---|---|
| BLE | PHY+Link Layer+stack | PHY advertising/data | RX raw 4 PHY, hopping adv y TX raw; clasificación PDU | GAP/GATT/SMP/conexión pública actual | beacons, teléfonos; observación raw limitada |
| IEEE 802.15.4 | PHY+MAC | PHY 2.4 y frames raw | RX/TX raw canal 11–26 | análisis MAC completo, seguridad/red | sensores Zigbee/Thread; bytes/FCS/RSSI |
| Zigbee | red/aplicación sobre 802.15.4 | comparte PHY/MAC | sólo frames crudos y KillerBee adapter | NWK/APS/ZCL, joins, keys | bombillas/coordinadores con parser externo |
| Thread | IPv6 mesh sobre 802.15.4 | comparte PHY | captura/inyección raw posible | 6LoWPAN, MLE, seguridad, routing | border router/nodos con herramientas externas |
| Matter | aplicación sobre IP/Thread/Wi-Fi | indirecta por Thread | nada específico | commissioning, fabric, clusters | sólo tráfico 802.15.4 subyacente, cifrado |
| 6LoWPAN | adaptación IPv6 | bytes dentro de 802.15.4 | nada específico | compresión/fragmentación | parser host externo |
| Wireless M-Bus | PHY/MAC metering | presets S/T/C/N | configuración PHY y bytes raw | stack EN 13757, cifrado/decodificación | medidores compatibles; interop no actual validada |
| Wi-SUN FAN | stack mesh IP sobre SUN PHY | FSK 902/915 | siete presets carrier/rate | FAN, RPL, seguridad/certificación | nodos Wi-SUN/SDR; sólo PHY raw |
| Amazon Sidewalk | red propietaria BLE/Sub-G/LR | Sub-G FSK | presets 50/250 k | auth/framing/stack; LR/LoRa | dispositivo Sidewalk/SDR con autorización |
| MIOTY | TS-UNB LPWAN | 868/396 baud | preset conservado | PHY nativo funcional y stack | estación/nodo MIOTY; hoy esperado no capturar |
| propietario | depende del diseño | FSK/GFSK/OOK/MSK/4FSK | parámetros y bytes raw | semántica del fabricante | sensores/remotos conocidos o reverse engineering |

Un preset Wi-SUN o Sidewalk sólo hace que parámetros locales se parezcan a una portadora esperada; no genera headers, autenticación, hopping, temporización de stack ni certificación. BLE stack sí existió históricamente, pero fue retirado el 20 de julio de 2026; Sniffle quedó como alternativa recomendada para funciones BLE superiores.

---

## 12. Presets

Un preset es un diccionario Python pasado a `configure_prop()`: `frequency_hz`, `mod_type`, `symbol_rate`, `deviation`, `rx_bw`, `sync_word` y opcionalmente `format_conf`. Es configuración de PHY/driver, no un parser ni un generador de protocolo.

### 12.1 Inventario actual (31)

| Grupo | Presets | Frecuencia/modulación | Estado defendible |
|---|---|---|---|
| 433 MHz | `gfsk_433_50k`, `gfsk_433_10k`, `fsk_433_50k`, `ook_433_4k8`, `ook_433_2k4`, `msk_433_50k`, `4fsk_433_50k`, `4gfsk_433_50k` | 433.92 MHz; 2/4FSK, GFSK, MSK, OOK | implementados; control actual; históricamente marginal; OOK 433 0/10 histórico |
| 868 genérico | `gfsk_868_50k`, `gfsk_868_100k`, `ook_868_4k8`, `msk_868_50k`, `4fsk_868_50k`, `4gfsk_868_50k` | 868 MHz | implementados; OTA histórica favorable, no repetida en campaña actual |
| W-MBus | `wireless_mbus_s_868`, `_t_868`, `_c_868`, `_n_169_2k4`, `_n_169_4k8` | 868.3/868.95/169.45 MHz GFSK | S/T/C OTA histórica; N no cerrada; no stack |
| 915/902 genérico | `gfsk_915_50k`, `gfsk_902_50k` | 915/902.2 MHz GFSK | implementados; control actual |
| propietario 2.4 | `gfsk_2440_250k`, `gfsk_2440_50k` | 2440 MHz GFSK | experimental; evidencia histórica contradictoria, modulado por verificar independientemente |
| Sidewalk | `sidewalk_915_fsk_50k`, `_250k` | 915 MHz FSK | carrier/preset experimental; 250k BW corregido tras smoke histórico; sin stack |
| Wi-SUN | `wisun_915_fsk_50k`, `_100k`, `_150k`, `_200k`, `_300k` | 902.2 MHz FSK | carrier/preset experimental; F29 reportó marcadores; sin FAN |
| MIOTY | `mioty_868_tsunb` | 868 MHz, FSK 396 baud | incompleto/pending; 0/10 histórico; probable custom CPE necesario |

`rx_bw` y `deviation` no deben presentarse como valores físicos autoevidentes: comentarios de `radio.py` los llaman registros/valores, y la traducción en `RadioIF_setPropConfig` modifica campos/overrides SmartRF. La conformidad con un estándar requiere comparar cada bit de configuración y medir RF.

**Evidencia experimental del Vault:** campaña actual: 27 objetivos no-OOK/MIOTY registrados `PASS-control`, 18 con logs individuales y nueve sólo en resumen. No es PASS-RF. Historia del repositorio: 16 presets reportados 10/10 OTA el 28 de abril, F29 70/70 en siete presets el 3 de mayo, MIOTY 0/10; faltan logs crudos/hashes suficientes para promoverlos a validación actual.

---

## 13. Todos los ejemplos y scripts propios

Hay **27 scripts Python** bajo `python/examples/` y un orquestador Bash. La tabla no cuenta scripts del submódulo TI porque son software tercero.

| Script | Propósito / entradas principales | API y operación | Tipo / relevancia / qué prueba un PASS |
|---|---|---|---|
| `killerbee_sniff.py` | port, canal, timeout, count, PCAP | `KillerBeeFeralRF`, sniff IEEE | demo integración; PASS/frames prueba flujo adapter, no Zigbee completo |
| `release_gate_multi_phy.py` | port y retries/params por PHY | lanza smokes por subprocess | gate; hereda límites de cada hijo |
| `smoke_phase2.py` | port, PHY, canal, power | init/set/RX start-stop | smoke control; PASS sólo ACK/lifecycle |
| `smoke_phy4_ieee154.py` | canal 11–26, duración | RX IEEE y packets | smoke RX; con paquetes supera ACK, pero fuente/interop requieren control |
| `smoke_prop_phase1.py` | preset, power, duración, TX hex, auto-reset | configure/RX/TX | smoke control; su PASS puede no observar RF; OOK riesgoso |
| `smoke_tx_phase1.py` | PHY/canal/power/packet | `transmit` RAW | smoke; PASS=ACK, no aire |
| `smoke_tx_frame_phase1.py` | PHY/canal/power/frame | `transmit_frame` | smoke; FRAME no es stack |
| `smoke_tx_burst_phase1.py` | count/interval | `transmit_burst` | PASS=scheduled/ACK, no count físico |
| `smoke_tx_continuous_phase1.py` | interval/run time | continuous+stop | PASS=ACK inicio/stop, no energía observada |
| `smoke_ota_txrx.py` | dos ports; PHY o preset; count/min | TX markers + RX match | OTA simétrico; sí observa enlace entre dos FeralRF, no instrumento independiente |
| `smoke_f17_emulation.py` | dos ports, count, skips | helpers IEEE/Sub-G/OOK | cross-validation de firmas; no emulación completa de dispositivo |
| `smoke_f29_subg_915.py` | dos ports/count/window | siete presets Wi-SUN/Sidewalk | marker PHY; excluye MIOTY; no stack |
| `lab/canary_regression.py` | port/PHY/canal/reporte | RX ventanas+stats monotónicas | smoke/soak corto; no RF calibrada |
| `lab/demo_emulate_ook_garage.py` | personalidad/count/interval/power | `emulate_ook`, reset final | demo peligrosa/regulada; payload parecido, no clon completo |
| `lab/demo_emulate_subg_sensor.py` | personalidad/interval/power | `emulate_sub1ghz` | demo de payload/preset, no identidad/interoperabilidad garantizada |
| `lab/demo_mioty_listen.py` | preset/duración | prop RX | stub explícito; esperado incompleto |
| `lab/demo_sidewalk_subg.py` | preset/duración | prop RX | stub PHY, no auth/stack/LR |
| `lab/demo_wisun_scan.py` | preset/duración | prop RX | stub PHY, no FAN/decodificación |
| `lab/ota_rx_probe.py` | port/PHY/canal/duración/marker | RX y cuenta | helper de observación FeralRF |
| `lab/ota_tx_frame.py` | port/PHY/canal/power/repeat | TX_FRAME repetido | helper; PASS=ACKs, receptor separado requerido |
| `lab/ota_tx_burst.py` | port/count/interval | TX_BURST | `scheduled=N` no confirma N emisiones |
| `lab/smoke_f22_tx_test.py` | TX/RX ports por defecto Linux | CW/PRBS/stop y RX BLE | lab; CW infiere interferencia; PRBS sólo ACK sin analizador |
| `lab/smoke_f25_crypto.py` | port | TRNG/AES/SHA/ECDH/ECDSA | HIL contra vectores/oráculo; no RF |
| `lab/smoke_f9_phy_matrix_ota.py` | dos ports/cycles/count | cambia PHY sin reset + markers | regresión OTA entre misma implementación |
| `lab/smoke_jam_phase1.py` | port/PHY/canal/power/duración | start/stop jam | experimental; PASS=ACK, no interferencia medida |
| `lab/sweep_phy4_ieee154.py` | rango canal/dwell | RX por canales 11–26 | exploración de actividad; no analizador de espectro |
| `lab/test_pa_characterization.py` | dos ports, sweep, PA prompts, CSV | burst+RSSI receptor | experimento; otro CatSniffer no es power meter calibrado |
| `run_validation_baseline.sh` | port/rx-port/filter | resetea y orquesta smokes/OTA | histórico; no correr sin Shell explícito por KI-15 |

### 13.1 Supuesto de puertos peligroso

`Radio._get_shell_port`, `smoke_ota_txrx.py`, `smoke_f17_emulation.py`, `smoke_f29_subg_915.py`, `lab/smoke_f9_phy_matrix_ota.py`, `lab/smoke_f22_tx_test.py` y el Python embebido de `run_validation_baseline.sh` derivan Shell como `Bridge+2`. La evidencia KI-15 demuestra que no es portable a la enumeración Windows observada. Los scripts pueden abrir otro puerto, fallar o actuar sobre otra interfaz/dispositivo. Catnip agrupa interfaces por descriptores/serie/ubicación y es el modelo más robusto.

### 13.2 Personalidades de emulación

`emulation/ieee154_device.py`, `sub1ghz_device.py` y `ook_device.py` definen dataclasses y payload builders para beacons/data-poll, sensores/W-MBus y controles OOK. “Emulate” significa reproducir una firma de bytes y parámetros suficiente para un experimento definido; no implementa temporización, estado, seguridad ni aplicación completa del equipo nombrado.

---

## 14. Tests

| Archivo | Cobertura real |
|---|---|
| `test_protocol.py` | COBS, CRC y frames host |
| `test_commands_contract.py` | builders sólo payload; ID sólo en header; baud |
| `test_enums_no_collisions.py` | IDs pendientes no colisionan |
| `test_radio_seq.py` | wrap 0..254 y reserva `0xFF` |
| `test_radio_strict_responses.py` | init/INFO y payload corto con FakeSerial |
| `test_async_error_surfacing.py` | errores seq 0/FF y buffering host |
| `test_read_one_packet.py` | un packet, timeout y skip de error async |
| `test_props.py` | schema/rangos/serialización e inventario F29 |
| `test_tx_test.py` | IDs/payloads/wrappers CW/PRBS/stop simulados |
| `test_crypto.py` | validación API/payload/respuesta mock crypto |
| `test_crypto_vectors.py` | vectores calculados con librería host; no chip |
| `test_emulation.py` | forma/firmas de payloads de personalidades |
| `test_killerbee_integration.py` | adapter con `FakeRadio` |
| `test_killerbee_dispatch.py` | hooks/dispatch KillerBee con mocks |

No hay tests C unitarios del firmware en este árbol. `pytest` normal es host-only; el marker `hardware` está configurado, pero los archivos actuales inventariados se apoyan principalmente en fakes/mocks. CI puede demostrar que contratos Python siguen coherentes, no que el HEX arranque, que una antena esté seleccionada ni que se emita RF.

```text
unit test PASS      = lógica host bajo entradas simuladas
hardware PASS       = protocolo/firmware respondió en una placa
OTA PASS            = receptor observó RF compatible
instrumented PASS   = instrumento independiente midió la magnitud reclamada
interop PASS        = un tercero conforme entendió/aceptó el protocolo
```

---

## 15. Sistema de build

`firmware/cc1352/CMakeLists.txt` requiere CMake 3.20+, `arm-none-eabi-gcc/g++/objcopy/size`, target por defecto `CC1352P7`, Cortex-M4F hard-float y `USE_TIRTOS=ON`. Selecciona subfamilia `cc13x2x7_cc26x2x7`, define `DeviceFamily_CC13X2X7` y usa `linker/cc1352p7.ld`; existe alternativa CC1352P con otro linker.

El build compila aplicación, patches RF, `ti_sysbios_config.c`, `ti_drivers_config.c`, `ti_radio_config_433.c` y enlaza librerías precompiladas RF/drivers/driverlib/SYSBIOS. El submódulo Git público no contiene todas las librerías precompiladas requeridas; se necesita instalación completa TI mediante `TI_SDK_FULL` o copia equivalente. CMake puede crear un symlink dentro del SDK para compatibilidad GCC 13+, efecto lateral que debe considerarse.

Artefactos: `feralrf_cc1352.elf`, `.hex`, `.bin`, `.map`; README exige flashear `.hex` con Catnip y advierte que `.bin` produjo fallos de boot. SysConfig/SmartRF aportan estructuras C, patches y overrides; algunos ficheros son generados/derivados y no deben editarse como si fueran lógica normal.

Python usa setuptools (`pyproject.toml`), versión `0.3.0`, Python ≥3.9, `pyserial`, `pyserial-asyncio` y `cobs`; extras `dev` y `killerbee`. Instalación típica: `pip install -e ".[dev]"` desde `python/`.

---

## 16. Relación con Catnip

Catnip y FeralRF ocupan capas distintas:

| Catnip | FeralRF |
|---|---|
| aplicación CLI de PC | firmware CC + API Python |
| descubre y agrupa Bridge/LoRa/Shell | usa principalmente Bridge; Shell sólo reset |
| identifica placa/metadata | INFO FeralRF reporta versión/caps/serial constante |
| flashea/restaura RP2040 y CC | no se autoflashea |
| usa bootloader CC por Bridge y control por Shell | protocolo runtime propio COBS+CRC |
| controla ruta SX1262 oficial | no usa SX1262 |

Catnip selecciona roles por texto de interfaz, número de interfaz, serie y ubicación, con fallback; por ello puede saber que COM87 es Shell aunque Bridge sea COM88. `Radio.reset_device()` sólo suma dos. La solución conceptual es reutilizar agrupación por identidad/descriptor o aceptar Shell explícito; el presente documento no implementa el cambio.

Catnip puede instalar/restaurar imágenes y después salir; durante runtime, `feralrf.Radio` habla directamente al endpoint FeralRF. No deben presentarse como el mismo producto ni atribuir a Catnip la semántica de los comandos FeralRF.

---

## 17. Historia Git e intención de implementación

La historia es extensa y contiene WIP, experimentos y resultados sin logs anexos. La tabla resume linajes que siguen afectando HEAD.

| Feature/componente | Introducido | Cambios importantes | Implementación actual | Estado |
|---|---|---|---|---|
| protocolo/Python base | `69fe69f`, `451f18e` (2026-02-16) | contratos `2e12a4d`; IDs consolidados `ef4f2f2` | COBS+CRC, Radio sync | current |
| tareas/colas | `5a30716`, `4d73b78`, `d2a985a` | TI-RTOS two-task `3ea5f44`; queue 32 `5c8b561` | UART/RF tasks | current |
| RX multi-PHY | `3bdd61b`, `b7e4189`, `372b1e1` | TI-RTOS/prebuilt SDK `9f14254`; switching fixes | BLE/IEEE/prop | current con límites |
| TX_RAW | `3e41b31` | working RX/TX `dcbfe84`; error async `f121505`, seq0 `2aac924` | enqueue+async error | current |
| TX_BURST | `4e4246c` | scheduler conservado | fallo tardío silencioso | current/problemático |
| TX_CONTINUOUS | `fb41335` | scheduler conservado | repetición de paquetes | current |
| TX_FRAME | `c388dbd` | hoy alias de raw con power almacenado | no stack framing | current |
| jamming | `a3821ec`..`ab4019f` | guards lab; shared handle fix `e910fd5` | repetición TX/CW-like | experimental |
| presets/OOK | `a941995` | OOK fixes `6850713`, `f471ac6`; 433 persistent `bd0cca5` | 31 presets | current/mixto |
| W-MBus/MSK/4FSK | `04881c7`, `4676f6d`, `b089f71` | validación histórica en commits | presets raw | current, no stack |
| test CW/PRBS | `d5329b1`..`492136d` | lazy-open fix `5804abc` | CW, PRBS15/32 | current |
| crypto | `2a68c54`..`60b74f5` | fixes bounds/driver `3b1100f` | comandos 59–62 | current |
| async errors | `f121505` | drop bug `d3a69d6`, seq conflict `2aac924/a6b3f5c` | buffer host y `RxStreamError` | current/parcial |
| Wi-SUN/Sidewalk | `c45114b`, `2595276` | BW 250k `10fc47a` | presets+demos | experimental PHY-only |
| MIOTY | `5bb5ae7` | marcado pending `6e34ef2` tras 0/10 | preset retenido | incomplete |
| emulación | `be8eab1`, `aab23a5`, `9845212` | smoke `f831664` | helpers PHY | experimental/demo |
| KillerBee | serie `e01859f`..`c9e1428`, merge `338a630` | reset default off `e02d127` | adapter | current, HW/interop pendiente |
| BLE stack/GATT | abril–mayo, numerosos F8/F20/F21 | retirado `52adcbd`..`785a093` (2026-07-20) | sólo PHY+LL classifier; residuos internos | removed/historical |
| RP2040 dentro del repo | `e8af382` | no mantenido; eliminado `ebdfcf3` | dependencia externa stock | removed |

**Git muestra intención, no éxito:** asuntos como “12/12 PASS”, “16 presets 10/10” o “all validated” explican por qué existen fixes/presets, pero sin artefacto, hash y log no sustituyen evidencia actual.

### 17.1 Evolución destacada

El proyecto nació con protocolo, firmware y Python en febrero; añadió RX/TX, burst/continuous/frame y jamming en días. En abril incorporó configuración propietaria, OOK y migró a TI-RTOS7/SDK 8.30. La dificultad dominante fue el ciclo de vida RF: handles múltiples, `RF_close`, estructuras corrompidas y 433. Eso explica el handle 433 persistente y el switching especial actual.

Después creció hacia BLE central/GATT/follower y periférico. Los commits documentan NOSYNC, timing, fixes y pruebas, pero el 20 de julio se eliminó el stack completo, manteniendo PHY raw y `ll_manager`; estudiar esos commits es útil como historia, no como API vigente. El 21 de julio se acotó el repositorio al CC1352 y se retiró RP2040 local.

La documentación final intentó corregir invariantes RF; aun así, las contradicciones `RF_open` 433 y `RF_runCmd(FS)` en jam permanecen. Esto demuestra por qué el código enlazado tiene precedencia.

---

## 18. Actual frente a obsoleto

| Elemento | Clasificación | Evidencia breve |
|---|---|---|
| protocolo COBS/CRC y API `Radio` | CURRENT | Python+C handlers/tests |
| PHY 0–7, RX raw | CURRENT | `PhyManager`, `RadioIF`, campaña control/RX IEEE |
| TX RAW/FRAME/BURST/CONTINUOUS | CURRENT pero semántica async problemática | handlers/scheduler; TX RF actual no validada |
| CW/PRBS | CURRENT | handlers+TI command; instrumento pendiente |
| crypto | CURRENT | API/firmware; campaña actual no HIL |
| jamming | CURRENT but experimental | comandos 30/33; discrepancia FS y sin validación de interferencia |
| KillerBee adapter | CURRENT | paquete/tests mocks; RF/interop actual pendiente |
| presets 868/915/433 | CURRENT, estado RF mixto | código+historia+campaña control |
| MIOTY | PARTIALLY implemented | preset, demo stub, 0/10 histórico |
| spectrum scan | DOCUMENTATION/HELPER-ONLY | builders/model sin enum/handler/API E2E |
| JAM_REACTIVE/PATTERN | DISABLED/PENDING | IDs pendientes 31/32, modelos host sin handler |
| BLE GAP/GATT/central/peripheral/attacks | REMOVED | commits de retiro 2026-07-20 |
| `tx_queue.c` BLE data queue | HISTORICAL residue | compilado, sin caller público actual |
| prototipos BLE follower/central en `radio_if.h` | HISTORICAL/unclear | no comandos públicos tras retiro |
| `main.c` NoRTOS | ALTERNATE/legacy | CMake permite OFF, no baseline |
| `startup/main_rtos.c` | HISTORICAL/unlinked | no está en `ALL_SOURCES` |
| firmware RP2040 local | REMOVED | commit `ebdfcf3`; stock externo requerido |
| RSA/high PA/IEEE 802.15.4g completo | PLANNED/unsupported | sin comando/ruta pública completa |

---

## 19. Aplicaciones reales

### 19.1 Usos demostrados o con evidencia actual

**Observación IEEE 802.15.4.** En un laboratorio con un dispositivo Zigbee en canal 25, FeralRF inicia RX, entrega frames crudos, RSSI/LQI/CRC/timestamp y Python los registra. La campaña observó 41/43/43 paquetes en tres ventanas. Permite estudiar actividad y bytes; no atribuye con certeza cada frame sin captura simultánea ni descifra Zigbee.

**Control/configuración multi-PHY.** Una placa recorrió ocho PHY y 27 presets con ACK/lifecycle. Es útil para enseñar el pipeline y detectar regresiones de control; no acredita ondas.

### 19.2 Usos técnicamente plausibles

- ingeniería inversa de sensores Sub-G conocidos, comparando frecuencia/modulación/sync y payloads;
- experimentación con TX raw/burst/continuous bajo autorización y receptor independiente;
- captura para decodificadores host/Wireshark/KillerBee;
- caracterización funcional de cambios de PHY y recuperación;
- test CW/PRBS con analizador de espectro y atenuación adecuados;
- comparación de aceleradores crypto con vectores y oráculos host.

### 19.3 Potencial futuro

- spectrum scan E2E, protocolos superiores, MIOTY con patch CPE, high-PA/routing probado, mejor descubrimiento/reset y telemetría de completion TX.

FeralRF no reemplaza un SDR general, VSA/analizador de espectro, generador RF calibrado, power meter, sniffer certificado, pila Zigbee/BLE/Wi-SUN ni equipo de conformidad.

---

## 20. Escenarios concretos de uso

### 20.1 Observar IEEE 802.15.4 / Zigbee

1. Seleccionar `PHY.IEEE_802_15_4`, canal 11–26.
2. `start_rx()` y consumir `Packet`.
3. Guardar bytes y metadatos; entregar `data` a parser 802.15.4/Zigbee externo.
4. Correlacionar con un equipo fuente u observador independiente.

Se aprende ocupación, timing, RSSI aproximado, CRC y estructura raw. No se obtienen automáticamente redes, claves o ZCL.

### 20.2 Observar advertising BLE

Seleccionar BLE 1M/2M/Coded y, para advertising, hopping 37–39. `LLManager` identifica tipos básicos. Un teléfono/beacon puede generar tráfico. Sin stack retirado, FeralRF no sigue una conexión ni hace GATT; Sniffle es la herramienta externa indicada por el proyecto para BLE superior.

### 20.3 Investigar sensor Sub-G propietario

Con autorización y conocimiento aproximado, elegir frecuencia, modulación, rate, deviation, BW y sync; capturar crudo y comparar repeticiones. Un SDR puede descubrir parámetros y validar que FeralRF está sintonizado. Los presets son puntos de partida, no autodetección.

### 20.4 Experimentación TX

- RAW: bytes + potencia por llamada.
- FRAME: mismos bytes por backend, potencia almacenada.
- BURST: N paquetes espaciados.
- CONTINUOUS: repetición hasta stop.

Usar receptor/instrumento, atenuación y entorno autorizado. Registrar ACK, `RxStreamError`, conteo recibido y condición de hardware por separado.

### 20.5 Modos de prueba

CW sirve para frecuencia/potencia/portadora; PRBS para señal modulada y ocupación. Confirmarlos requiere analizador o receptor apropiado. Inferir CW por caída de paquetes BLE puede mostrar interferencia, pero no mide máscara/potencia y no debe realizarse fuera de un recinto autorizado.

---

## Referencias de procedencia

### Repositorio FeralRF

- `../FeralRF/README.md`
- `../FeralRF/docs/{ARCHITECTURE.md,protocol.md,PYTHON_API.md,VALIDATION_MATRIX.md}`
- `../FeralRF/python/feralrf/{radio.py,protocol.py,commands.py,enums.py,presets.py,_responses.py}`
- `../FeralRF/python/feralrf/{emulation,integrations}/`
- `../FeralRF/python/examples/`, `../FeralRF/python/tests/`
- `../FeralRF/firmware/cc1352/{CMakeLists.txt,include,src,syscfg,linker}/`
- Git de `FeralRF@0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`.

### Evidencia secundaria del Vault

- `04 - FeralRF/Arquitectura FeralRF.md`
- `04 - FeralRF/Matriz de capacidades.md`
- `04 - FeralRF/Protocolo y API Python.md`
- `04 - FeralRF/Pruebas y evidencia existente.md`
- `05 - Evaluación/Auditoría técnica de validación FeralRF - EV ejecutadas.md`
- `05 - Evaluación/FeralRF - Matriz de pruebas.md`
- notas de hardware, firmware RP2040 y Catnip citadas por sección.

### Contexto externo oficial

- [TI CC1352P7 datasheet](https://www.ti.com/lit/ds/symlink/cc1352p7.pdf)
- [TI SimpleLink Proprietary RF User's Guide — PHY Configuration](https://software-dl.ti.com/simplelink/esd/simplelink_cc13xx_cc26xx_sdk/latest/exports/docs/proprietary-rf/proprietary-rf-users-guide/rf-core/phy-configuration.html)
- [TI SimpleLink Academy — RF Driver](https://software-dl.ti.com/lprf/simplelink_academy/modules/prop_01_basic/prop_01_basic.html)
- [Raspberry Pi RP2040 specifications](https://www.raspberrypi.com/products/rp2040/specifications/)
- [Raspberry Pi RP2040 datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf)
- [IEEE 802.15.4 overview](https://standards.ieee.org/ieee/802.15.4/12701/)
- [Bluetooth SIG — Bluetooth Core 5.4 technical overview](https://www.bluetooth.com/wp-content/uploads/2023/02/2301_5.4_Tech_Overview_FINAL.pdf)

---

## 21. Glosario

| Término | Expansión inglesa | Explicación breve en español |
|---|---|---|
| ACK | Acknowledgement | confirmación de aceptación; aquí no prueba RF completa |
| API | Application Programming Interface | interfaz de funciones para programas |
| ASK | Amplitude Shift Keying | modulación por amplitud |
| BLE | Bluetooth Low Energy | tecnología Bluetooth de bajo consumo |
| bandwidth | — | ancho de frecuencias ocupado/aceptado |
| bitrate | — | bits transmitidos por segundo |
| carrier | — | portadora RF central |
| CDC | Communications Device Class | clase USB usada como puertos seriales virtuales |
| CLI | Command Line Interface | programa manejado por comandos de terminal |
| COBS | Consistent Overhead Byte Stuffing | codificación que reserva `00` como delimitador |
| CRC | Cyclic Redundancy Check | verificación matemática de errores accidentales |
| CW | Continuous Wave | portadora continua sin paquetes |
| dBm | decibels relative to 1 milliwatt | unidad logarítmica de potencia |
| DUT | Device Under Test | equipo bajo prueba |
| FCS | Frame Check Sequence | campo de verificación al final de una trama |
| frame | — | unidad estructurada de una capa/protocolo |
| front-end | — | red RF entre transceptor y antena: switch/filtros/PA, etc. |
| FSK | Frequency Shift Keying | símbolos representados por cambios de frecuencia |
| GFSK | Gaussian Frequency Shift Keying | FSK con filtrado gaussiano |
| GPIO | General-Purpose Input/Output | pin digital programable |
| Hz/kHz/MHz/GHz | hertz y múltiplos | ciclos por segundo; unidades de frecuencia |
| IEEE | Institute of Electrical and Electronics Engineers | organización de estándares técnicos |
| LQI | Link Quality Indicator | indicador de calidad/correlación del receptor |
| MCU | Microcontroller Unit | procesador con memoria y periféricos integrados |
| MSK | Minimum Shift Keying | FSK de fase continua con índice específico |
| OOK | On-Off Keying | presencia/ausencia de portadora representa símbolos |
| OTA | Over The Air | prueba mediante propagación RF real |
| packet | — | conjunto de bytes recibido/transmitido |
| PA | Power Amplifier | amplificador de potencia RF |
| PHY | Physical Layer | capa que define señal, modulación, rate y canal |
| PRBS | Pseudo-Random Binary Sequence | secuencia pseudoaleatoria para test |
| RF | Radio Frequency | radiofrecuencia y subsistema de radio |
| RF Core | Radio Core | procesador/subsistema interno TI que ejecuta comandos radio |
| RSSI | Received Signal Strength Indicator | estimación de potencia recibida |
| RTOS | Real-Time Operating System | sistema operativo con tareas/temporización embebida |
| RX | Receive / Receiver | recepción/receptor |
| SDK | Software Development Kit | código, headers, librerías y herramientas del fabricante |
| SoC | System on Chip | sistema con CPU y subsistemas integrados |
| symbol | — | estado de modulación; puede representar uno o más bits |
| sync word | Synchronization Word | patrón para reconocer inicio/flujo esperado |
| TI-RTOS/SYSBIOS | Texas Instruments Real-Time OS | kernel usado por el firmware predeterminado |
| TRNG | True Random Number Generator | generador aleatorio físico del chip |
| TX | Transmit / Transmitter | transmisión/transmisor |
| UART | Universal Asynchronous Receiver/Transmitter | enlace serial entre RP2040 y CC1352P7 |
| USB | Universal Serial Bus | enlace físico/lógico entre PC y RP2040 |
| USB CDC | USB Communications Device Class | función USB que el SO presenta como COM/tty |
| wire protocol | — | representación exacta de mensajes en el enlace |

---

## 22. Limitaciones y problemas conocidos

| Problema | Subsistema/evidencia | Impacto / workaround / estado |
|---|---|---|
| KI-15 Shell=`Bridge+2` | `radio.py`, varios scripts; reproducido COM88→COM90 vs COM87 | reset/API y baseline inseguros; usar identidad/Shell explícito; actual |
| ACK anticipado | CommandProcessor/ControlTask | falsos PASS de TX/RX si sólo se observa ACK; consumir async y usar observador |
| burst/continuous fallan en silencio | `control_task.c:373-405` | host desconoce aborto; actual |
| RX_START falla después de ACK | `data_task.c` | leer `RxStreamError`; actual |
| cola host 32, un comando pendiente | packet/host task | drops/busy bajo carga; sin backpressure UART |
| PHY/band switching histórico | Git/matrices; resets usados en campaña | reproducibilidad HEAD sin reset pendiente |
| OOK lock | código/docs/presets | reset requerido; reset API afectado por KI-15 |
| 433 marginal | resultados históricos variables; antena/ruta | control PASS no elimina fallo OTA; caracterizar con antena/instrumento |
| MIOTY 0/10 | preset comments/commit `6e34ef2` | no soporte nativo demostrado; custom CPE pendiente |
| selector RF externo U2/CTF | hardware vs SysConfig | banda física no establecida por código solamente |
| potencia `-128` en RAW | código actual | probablemente clamp a mínimo; pasar power explícito |
| docs `RF_open` contradictorias | `main_rtos.c` vs ARCHITECTURE | estudiar código enlazado |
| regla `CMD_FS` no universal | jam usa `RF_runCmd` | riesgo específico experimental |
| versión/serial | FW 1.0.0, paquete 0.3.0, docs “v2.0”, serial constante | no usar INFO como identidad única |
| límites 125/239/255 | Control/Data/Protocol | no fragmentación; payloads mayores no soportados |
| RF regulada/jamming | legal/lab | sólo entorno autorizado/aislado |

---

## 23. Qué se ha validado realmente

La fuente secundaria principal es `05 - Evaluación/Auditoría técnica de validación FeralRF - EV ejecutadas.md` (auditoría 2026-10-05). No se generó evidencia nueva para esta Wiki.

| Nivel | Evidencia actual defendible |
|---|---|
| host/control | init/info/stats favorable pero serie formal parcial; reconnect 5/5; RX start/stop; rechazo RX+TX; PHY 8/8 control |
| configuración | 27/27 presets reportados control, auditabilidad individual 18/27 |
| RX RF | IEEE 802.15.4 canal 25: 41/43/43 paquetes, atribución limitada y sin PCAP completo |
| TX RF actual | no ejecutada: EV-12 prepared/not tested |
| OTA actual | no repetida para matrices multi-PHY/presets |
| instrumentación RF | no evidencia actual de frecuencia/potencia/espectro |
| interoperabilidad de stack | no validada; sólo raw/presets |
| reset | API bloqueada por KI-15; ruta manual 1/1 |

Historia documental previa reporta OTA BLE/IEEE/Sub-G/presets, CW interference, crypto y otros conteos entre abril y mayo de 2026. Es útil como antecedente y motivación de código, pero carece de suficiente vínculo a binario/log/placa para sustituir la campaña actual. La frase académicamente defendible es: **“baseline de control ampliamente recorrido y una ruta RX IEEE 802.15.4 validada físicamente en la condición descrita”**, no “FeralRF validado completamente”.

---

## 24. Preguntas técnicas abiertas

| Pregunta | Por qué importa | Incertidumbre / dónde investigar |
|---|---|---|
| ¿Qué controla realmente U2/CTF por banda? | determina ruta/antena/PA | esquema+firmware RP2040+medición GPIO/RF |
| ¿Qué HEX exacto está flasheado? | vincula experimento a fuente | hash/manifest de build y Catnip metadata |
| ¿Switching sin reset es estable en HEAD? | lifecycle multi-PHY | `radio_if.c`, prueba controlada F9 sin resets |
| ¿Cómo reportar completion por TX? | ACK no basta | ControlTask/OutputIF; diseñar evento, no asumirlo |
| ¿Cuántos drops produce carga sostenida? | confiabilidad RX | queues, UART, stats y generador RF |
| ¿`-128` debía significar “actual”? | potencia RAW predeterminada | blame/history/API docs y medición |
| ¿Por qué jam viola regla FS? | posible hang/riesgo experimental | `RadioIF_startJamSession`, TI RF behavior |
| ¿Qué prototipos BLE residuales son alcanzables? | evitar confundir histórico con feature | linker/call graph de `radio_if.*`, `tx_queue.c` |
| ¿Parámetros W-MBus/Wi-SUN son conformes? | preset vs protocolo | norma/SmartRF export + dispositivo tercero |
| ¿Puede MIOTY usar custom CPE viable? | cierra preset pendiente | SmartRF Studio/TI patches/medición TS-UNB |
| ¿KillerBee preserva FCS/timestamps correctamente E2E? | análisis real | adapter, patch y Wireshark con hardware |
| ¿Cómo descubrir Shell de forma robusta? | reset seguro multi-dispositivo | reutilizar agrupación Catnip por serial/location |

---

## 25. Ruta de aprendizaje recomendada

1. **Separación física.** Leer §§1–2 y `01 - Hardware/RP2040, CC1352P7 y SX1262 - Quién hace qué.md`. Resultado: saber por qué FeralRF corre en CC y aún necesita RP.
2. **Camino PC→radio.** Leer §3, `python/feralrf/protocol.py`, `firmware/cc1352/src/host_if_task.c` y `protocol.c`. Dibujar un frame.
3. **Contrato y evidencia.** Leer §§7 y 23. Memorizar `ACK ≠ RF`. Revisar `protocol.h` y `command_processor.c`.
4. **API Python.** Leer §6 y `radio.py` en orden: conexión, `_read_response`, init/config, RX, TX, test y crypto.
5. **Scheduler firmware.** Leer §5 y `control_task.c` + `data_task.c`; seguir primero `RX_START`, luego `TX_RAW`, después burst.
6. **RF Driver.** Leer `radio_if.c` por regiones: lifecycle/switch, backends RX, TX, test y jam; contrastar con documentación TI enlazada.
7. **PHY y protocolos.** Leer §§10–12; separar silicio, preset, PHY, MAC y stack.
8. **Ejemplos.** Usar §13 como índice; leer smokes de control antes de OTA/lab. Marcar cuáles usan `Bridge+2`.
9. **Tests/build.** Leer §§14–15, `pyproject.toml` y `CMakeLists.txt`; entender qué no prueba CI.
10. **Historia.** Leer §§17–18 y después `git show` de los commits de una feature; terminar siempre comprobando HEAD.
11. **Aplicación de laboratorio.** Leer §§19–20 y §22; elegir un escenario con observador adecuado y autorización RF.
12. **Huecos.** Cerrar con §24; cada pregunta apunta a una zona de código/evidencia concreta.

### Resumen defendible en un minuto

FeralRF sustituye el firmware del CC1352P7, no el del RP2040. Python envía frames COBS+CRC por Cat-Bridge; el RP2040 sólo los puentea por UART; el Cortex-M4F valida comandos y TI RF Driver gobierna el RF Core. El sistema ofrece RX/TX crudo multi-PHY, configuración propietaria, modos CW/PRBS, crypto y un adapter KillerBee. Presets con nombres Zigbee/W-MBus/Wi-SUN/Sidewalk representan, como máximo, la parte PHY configurada: no son stacks. Los ACK de varias operaciones sólo prueban aceptación/scheduling. La evidencia actual es fuerte para control y para una recepción IEEE 802.15.4 concreta; TX OTA, instrumentación, interoperabilidad y varias rutas problemáticas siguen pendientes. La historia explica decisiones como el handle 433 persistente y también muestra que el stack BLE superior fue eliminado, por lo que la fuente actual —no los commits antiguos ni el marketing— define qué existe hoy.
