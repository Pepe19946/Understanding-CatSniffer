# Auditoría de API Python y contrato firmware

## Superficie pública

`feralrf/__init__.py:L41-L59` exporta `Radio`, `Packet`, `DeviceInfo`, `DeviceStats`, `RxStreamError`, `PHY`, `Command`, `Response`, marcadores de estabilidad, presets y excepciones. `Packet` contiene timestamp, channel, RSSI firmado, LQI, CRC, data y metadata BLE opcional; `DeviceInfo` no identifica de forma única el build; `DeviceStats` combina métricas RF generales y LL opcionales (`radio.py:L18-L67`).

Los métodos operativos públicos encontrados son:

- ciclo de vida: `list_devices`, `connect`, `reset_device`, `disconnect`, `init`;
- configuración: `set_phy`, `set_channel`, `set_power`, `configure_prop`, `set_adv_hop`, `get_stats`;
- RX/TX: `start_rx`, `stop_rx`, `read_packets`, `read_one_packet`, `transmit`, `transmit_frame`, `burst`, `continuous`, `stop_transmit`;
- test: `tx_cw`, `tx_prbs`, `tx_test_stop`;
- criptografía: random, AES ECB/CCM/GCM, SHA-256, ECDSA y ECDH;
- experimental: `start_jam`, `stop_jam`.

La clasificación no es internamente consistente. `Radio.STABLE_METHODS` (`radio.py:L89-L106`) omite métodos públicos necesarios como `connect`, `disconnect`, `list_devices`, `read_one_packet`, test TX y criptografía, mientras `docs/PYTHON_API.md:L17-L55` los presenta dentro de la API estable y `STABLE_COMMANDS` sí incluye test/crypto. Spectrum se marca pendiente aunque no tiene Command/handler; jamming reactivo/pattern también está pendiente. Emulación y `KBCatSniffer` son módulos importables pero no exports top-level.

## Contrato de trama y correlación

| Campo | Python | Firmware | Evaluación |
|---|---|---|---|
| command | `uint8` | `uint8` | alineado |
| sequence | `uint8`, host usa 0…254 | eco en respuestas síncronas; RX genera 0…254 | colisión conceptual entre async y síncrono |
| length | `uint16` little-endian | parser con máximo 255 | Python no impone el máximo global |
| CRC | CCITT, init `0xFFFF`, little-endian | mismo algoritmo | alineado |
| framing | COBS + `0x00` | mismo | alineado |

`_send_command()` incrementa secuencia y omite `0xFF`. `_read_response()` acepta la secuencia esperada, trata algunos `seq=0` como asíncronos y reconoce ERROR 0/FF, pero descarta respuestas inesperadas con secuencia no cero. `DataTask` asigna a `RX_PACKET` una secuencia rotativa 0…254. En consecuencia, un RX asíncrono no se preserva salvo el caso ocasional `seq=0`. Éste es un desacuerdo de contrato confirmado.

## Matriz de comandos

“ACK-estado” significa que sólo se modificó/aceptó estado lógico; “ACK-agendado” que la acción RF ocurre posteriormente; “resultado” que la operación devuelve datos específicos.

| ID / API | Builder Python | Handler CC1352P7 | Respuesta real | Evidencia / evaluación |
|---|---|---|---|---|
| `01 RADIO_INIT` / `init` | vacío | `ControlTask_onRadioInit` | ACK tras reset lógico | Demostrado; no reejecuta `RadioIF_init` |
| `02 SET_CHANNEL` | `u8` | almacena canal | ACK-estado | Sin rango/confirmación RF |
| `03 SET_POWER` | `i8` enmascarado | almacena potencia | ACK-estado | Valor efectivo no reportado |
| `04 SET_PHY` | `phy,u8 channel` | valida enum, cambia estado | ACK-estado | Open/start posterior |
| `05 GET_INFO` | vacío | valores constantes | `INFO` | Demostrado; identidad no única |
| `06 GET_STATS` | vacío | métricas RadioIF/LL | `STATS` | Demostrado; omite transporte/output queue |
| `07 SET_ADV_HOP` | booleano | almacena flag | ACK-estado | Sólo contexto BLE adv residual |
| `08 SET_PROP_CONFIG` | 16/18 bytes | decodifica y muta structs | ACK-estado | Sin validación/resultado del driver |
| `10 RX_START` | vacío | agenda start | ACK-agendado; ERROR async si falla | Demostrado con limitaciones |
| `11 RX_STOP` | vacío | agenda stop | ACK-agendado | Timeouts históricos; RX intercalado |
| `20 TX_RAW` / `transmit` | len + data | copia a slot | ACK-agendado; ERROR async en raw | OTA demostrado |
| `21 TX_CONTINUOUS` | len+data+interval | activa scheduler | ACK-agendado | Sin progreso/completion/fallo async |
| `22 TX_BURST` | len+data+count+interval | activa scheduler | ACK-agendado | Sin conteo/completion |
| `23 TX_FRAME` | len + data | misma ruta raw | ACK-agendado | OTA demostrado; nombre no añade MAC |
| `24 TX_STOP` | vacío | borra estados/sesión jam | ACK | Sin resultado/conteo |
| `30 JAM_CONTINUOUS` / `start_jam` | canal/duración | inicia sesión BLE/IEEE | ACK si start interno acepta | Experimental, no validado en Fase I |
| `33 JAM_STOP` | vacío | detiene sesión | ACK | fallback Python a TX_STOP |
| `55 TX_CW` | potencia | abre/FS/postea test | ACK si handle válido | ACK observado, RF no medido |
| `56 TX_PRBS` | potencia | abre/FS/postea test | ACK si handle válido | Igual |
| `57 TX_TEST_STOP` | vacío | detiene test | ACK | Estado final no consultable |
| `59 CMD_RANDOM` | longitud | TRNG | `RSP_RANDOM` | Implementado, no validado aquí |
| `5A CMD_AES_ECB` | modo/clave/data | driver AES | `RSP_AES` | Tamaños básicos validados |
| `5B CMD_AES_CCM` | clave/nonce/AAD/data/tag | driver AESCCM | `RSP_AES_CCM` | Python excede potencialmente payload 255 |
| `5C CMD_AES_CTR` | modo/clave/contador/data | driver AESCTR | `RSP_AES` | Implementado; sujeto al máximo global |
| `5D CMD_AES_CBC` | modo/clave/IV/data | driver AESCBC | `RSP_AES` | Implementado; sujeto al máximo global |
| `5E CMD_AES_GCM` | clave/nonce/AAD/data/tag | driver AESGCM | `RSP_AES_GCM` | Python excede potencialmente payload 255 |
| `5F CMD_SHA256` | data | driver SHA2 | `RSP_SHA256` | Implementado |
| `60 CMD_ECDH` | curve/keys | ECDH P-256/25519 | `RSP_ECDH` | Implementado |
| `61 CMD_ECDSA_SIGN` | curve/key/hash | ECDSA P-256 | `RSP_ECDSA_SIG` | Python también permite curve25519 |
| `62 CMD_ECDSA_VERIFY` | curve/pub/hash/sig | ECDSA P-256 | `RSP_ECDSA_VERIFY` | Mismatch de curva |

No se encontraron IDs vigentes en el enum Python sin un caso equivalente en el dispatcher C. Tampoco se encontró un comando C actual sin enum Python dentro de esta tabla. Las brechas están principalmente en semántica, resultados y capacidades fuera del protocolo, no en números huérfanos. Los builders spectrum/promiscuous sí están huérfanos porque ni siquiera tienen IDs actuales.

## Semántica de respuestas y errores

`RSP_ACK` confirma diferentes hitos según el comando. Para setters confirma almacenamiento; para RX/TX diferidos confirma aceptación; para test confirma scheduling; nunca confirma OTA. `RSP_ERROR` lleva esencialmente un código de un byte; Python modela `context`, pero firmware normalmente no lo proporciona. El caller no recibe etapa, PHY, operación ni status TI.

Los fallos posteriores de TX raw y RX start pueden aparecer asíncronamente. Burst/continuous se cancelan silenciosamente si `RadioIF_transmitRaw()` falla. Los drops de `OutputIF` también son silenciosos. `read_packets()` puede exponer `RxStreamError`, pero una aplicación que espera una respuesta síncrona, usa `read_one_packet()` o sólo llama métodos TX no dispone de un canal uniforme de eventos.

La limpieza es también asimétrica: `disconnect()` cierra puerto, pero no garantiza `RX_STOP`/`TX_STOP`; `reset_device()` opera una segunda interfaz mediante supuestos; `RADIO_INIT` no reconstruye todo RadioIF. No existe context manager, consulta de estado ni reconciliación tras timeout.

## Evaluación de completitud A–D

### A. Exposición faltante

#### Control explícito de la integración RP2040

- **Necesidad:** seleccionar/consultar ruta RF y resetear el CC de forma inequívoca.
- **Actual:** Cat-Shell implementa `band1/2/3`, `boot`, `exit`; Python sólo tiene `reset_device()` y un helper privado que deduce puerto.
- **Limitación demostrada:** Fase I seleccionó una vez COM90 cuando el shell real era COM87.
- **Responsabilidad futura:** API host de descubrimiento bridge/shell por identidad y operaciones explícitas; para status verificable se requiere ampliar RP2040/protocolo shell.
- **Compatibilidad:** mantener selección automática sólo como conveniencia, nunca como identidad.

#### Métricas de transporte y cola de salida

- **Necesidad:** distinguir pérdida RF de pérdida CC-UART/RP2040/USB.
- **Actual:** RP2040 y `PacketQueue` poseen contadores locales no expuestos por `GET_STATS`.
- **Cambio adicional:** requiere firmware/protocolo, no sólo wrapper Python.
- **Beneficio:** atribución de timeouts y drops.

No se clasifican como “exposición faltante” las rutinas BLE internas residuales: carecen de comando vigente y fueron retiradas deliberadamente de la superficie pública.

### B. Interfaz incompleta

| Área | Lo que falta para cumplir su propósito | Capa adicional necesaria |
|---|---|---|
| Identidad/versionado | build ID, commit/hash de firmware, variante y capacidades granulares | firmware + proceso de build |
| Lifecycle | mapeo explícito shell, estado conectado/inicializado/RF, recuperación tras timeout | Python; parte firmware/RP2040 |
| Config RF | validación y lectura de parámetros efectivos (frecuencia, potencia, modulación) | Python + firmware |
| RX | contrato uniforme de eventos, preservación durante llamadas síncronas, metadata válida, contadores por buffer | ambas + posiblemente RP2040 |
| TX | completion, fallo, número transmitido y motivo de stop en raw/burst/continuous | firmware/protocolo/Python |
| Crypto | tamaño total ≤255 y curvas por operación | principalmente Python |
| KillerBee | propagación de errores/metadatos y contrato de FCS/inyección | adapter y packaging |

### C. Contrato incorrecto o inconsistente

1. `RX_PACKET` usa secuencias no cero, mientras el host sólo preserva de forma general async `seq=0`.
2. `mod_type=4/5/6` se nombra MSK/4-FSK/4-GFSK, pero el SDK los reserva.
3. ECDSA host permite Curve25519 y firmware ECDSA no.
4. Python no limita el payload global a 255 y varios builders enmascaran en vez de rechazar valores.
5. `configure_prop()`/docs sugieren configuración, mientras ACK sólo prueba parseo y escritura en estructuras.
6. La lista programática de métodos estables no coincide con la documentación ni con comandos estables.
7. `serial=FERALRF1` parece identidad, pero es constante.
8. El extra de packaging KillerBee no representa por sí solo el entorno con patch requerido.

### D. Capacidad de proyecto faltante

| Capacidad | Base para considerarla faltante | Estado actual | No confundir con |
|---|---|---|---|
| Spectrum scan | marcada pending y existen stubs/types | sin ID, handler ni API extremo a extremo | una gráfica host vacía |
| Jamming reactivo/pattern | constantes/política pending | sin contrato firmware ejecutable | jamming continuo básico |
| Promiscuous/filter control | builder residual, necesidad de sniff controlable | sin enum/handler/API | RX raw fijo actual |
| Pilas de protocolos nombrados | claims/objetivos documentales deben aclararse | sólo PHY/raw/presets/emulación | interoperabilidad Zigbee/Thread/W-MBus/etc. |

Las pilas completas sólo deben llamarse “faltantes” si siguen siendo requisito del proyecto. El código actual parece deliberadamente enfocado en radio raw y retiró BLE stack; por ello la decisión de producto necesita aclaración antes de convertir esa ausencia en roadmap.

## Funcionalidad presente pero poco documentada

- La suite criptográfica es extensa y tiene responses dedicadas, pero no aparece en la clasificación programática de métodos estables.
- `read_one_packet()` existe específicamente para integraciones, aunque su pérdida de `RxStreamError` no está resaltada.
- `set_adv_hop()` permanece pese a la reducción de BLE a PHY/LL parcial; su alcance real es estrecho.
- Las tablas de potencia incluyen configuraciones high-PA, pero no hay API para conocer la resolución efectiva.
- El LL manager aún clasifica PDUs y mantiene estadísticas BLE, sin constituir una pila BLE.

## Conclusión de API

La API cubre el inventario básico de comandos, pero no modela adecuadamente una operación asíncrona de radio. Su abstracción predominante es “enviar orden y recibir ACK”; faltan resultados, estado observable, identidad y fronteras explícitas con RP2040. Las brechas críticas requieren cambios coordinados de firmware/protocolo, no una simple suma de métodos Python.

