# FeralRF desde la PC — Protocolo y API Python

> Verificado nuevamente el `2026-09-28` contra `FeralRF@0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`; no se ejecutó código ni hardware.

## En una frase

La API `feralrf.Radio` convierte una llamada Python en una trama binaria, la envía por [[USB CDC - Cómo CatSniffer aparece ante la PC|Cat-Bridge]] y recibe una respuesta o evento generado por el firmware FeralRF en CC1352P7.

## Modelo mental por capas

`aplicación → Radio API → comando/payload → COBS+CRC → puerto serie → RP2040 passthrough → parser CC → operación RF`

Cada capa resuelve un problema distinto. La API expresa intención; el codec delimita y protege el mensaje; Cat-Bridge lo transporta; el firmware CC lo interpreta; el driver TI opera la radio. Una respuesta ACK puede confirmar que una orden fue aceptada sin demostrar que la transmisión RF posterior terminó con éxito.

## 1. Transporte

La API Python se conecta al puerto **Cat-Bridge** y usa al RP2040 únicamente como passthrough. El transporte físico visto por el CC1352P7 es UART0 921600, 8N1, sin flow control (`firmware/cc1352/include/config.h`, `src/host_if.c`). Cat-LoRa no participa. Cat-Shell solo aparece en `Radio.reset_device()` y en el flasheo externo mediante Catnip.

`Radio.list_devices()` enumera pyserial y acepta VID `0x1209`, `0x2E8A`, `0x2341`, `0x1A86` o `0x10C4`. Para VID `0x1209`, exige que description+product contenga `Bridge`; no filtra PID, no agrupa tres CDC y no comprueba el protocolo. `Radio()` sin puerto toma el primero.

Comparado con Catnip, el descubrimiento es más simple: no usa número de interfaz USB ni agrupa por serial. `_get_shell_port()` supone que Shell es el número final del Bridge + 2.

## 2. Frame y codec

Fuente dual concordante: `python/feralrf/protocol.py` y `firmware/cc1352/src/protocol.c`.

```text
sin COBS:
┌─────────┬───────┬────────────┬──────────────┬────────────┐
│ ID (1)  │SEQ (1)│ LEN LE (2) │ PAYLOAD 0..N │ CRC LE (2) │
└─────────┴───────┴────────────┴──────────────┴────────────┘
                         N <= 255

wire = COBS(frame) + 0x00
CRC16-CCITT(frame sin CRC), poly 0x1021, init 0xFFFF
```

- Tamaño máximo pre-COBS: 261 bytes.
- No existe capa de fragmentación/reensamblado.
- Campos multibyte: little-endian.
- Requests/comandos: IDs menores de `0x80`.
- Responses/eventos: `0x80..0xFF`.

## 3. Modelo request/response/evento

`Radio._send_command()` incrementa SEQ, reserva `0xFF`, codifica y escribe. `_read_response()` lee hasta `0x00`, decodifica COBS, verifica CRC/longitud y espera un ID esperado.

- ACK/ERROR/INFO/STATS síncronos reflejan SEQ del request.
- RX_PACKET usa contador firmware propio y omite `0xFF`.
- Errores asíncronos usan SEQ 0; Python conserva compatibilidad con 0xFF.
- `_pending_async` guarda eventos asíncronos que aparecen mientras se espera otra respuesta.
- Un frame malformado/CRC inválido se descarta y la API continúa hasta timeout.
- No existe reconexión automática durante streaming. `init()` sí reabre y reintenta hasta tres veces.

### Respuestas principales

| ID | Nombre | Semántica |
|---:|---|---|
| `0x80` | ACK | comando aceptado; no siempre operación RF terminada |
| `0x81` | ERROR | error de payload/frame/estado/RF |
| `0x90` | RX_PACKET | evento asíncrono con packet+metadata |
| `0x93` | STATS | 16 bytes base, 36 con contadores LL |
| `0x94` | INFO | versión 1.0.0, capabilities y serial fijo |
| `0x95..0x9C` | crypto | resultados TRNG/AES/SHA/ECDH/ECDSA |

`protocol.h` define además `ERR_RF_NOT_READY=0x07`, que falta en la lista de errores de `docs/protocol.md`.

## 4. Capas Python

```mermaid
flowchart LR
    E[Ejemplos / KillerBee / app] --> H[Radio API + presets]
    H --> B[CommandBuilder / response parsing]
    B --> P[protocol.py<br/>COBS + CRC16]
    P --> S[pyserial Cat-Bridge]
    S --> RP[RP2040 passthrough]
    RP --> F[firmware protocol.c<br/>command_processor.c]
    F --> C[ControlTask / DataTask]
    C --> R[RadioIF / TI RF driver]
```

| Capa | Código | Papel |
|---|---|---|
| Transporte/codec | `protocol.py`, pyserial dentro de `radio.py` | serial, COBS, CRC, frame |
| Comandos | `commands.py`, `enums.py` | IDs y payloads |
| Sesión/API | `radio.py:Radio` | estados, timeouts, errores, objetos Packet/Info/Stats |
| Presets/features | `presets.py`, `emulation/`, `_jamming.py`, `_spectrum.py` | configuración compuesta; spectrum solo estructura sin data path |
| Integración | `integrations/killerbee.py` | adaptador IEEE 802.15.4 sobre `Radio` |
| Aplicaciones | `examples/` | smoke, OTA, laboratorio y demos |

## 5. Comandos implementados actuales

| Grupo | IDs/operaciones |
|---|---|
| Sesión/config | `RADIO_INIT 01`, `SET_CHANNEL 02`, `SET_POWER 03`, `SET_PHY 04`, `GET_INFO 05`, `GET_STATS 06`, `SET_ADV_HOP 07`, `SET_PROP_CONFIG 08` |
| RX | `RX_START 10`, `RX_STOP 11` |
| TX | `TX_RAW 20`, `TX_CONTINUOUS 21`, `TX_BURST 22`, `TX_FRAME 23`, `TX_STOP 24` |
| Jamming experimental | `JAM_CONTINUOUS 30`, `JAM_STOP 33` |
| Test TX | `TX_CW 55`, `TX_PRBS 56`, `TX_TEST_STOP 57` |
| Crypto | `RANDOM 59`, AES `5A..5E`, SHA256 `5F`, ECDH `60`, ECDSA `61..62` |

Spectrum no tiene IDs en `Command`, handler firmware ni método `Radio`; quedan builders/dataclasses huérfanos. JAM_REACTIVE/PATTERN solo tienen IDs pendientes, no implementación.

## 6. Traza A — información de dispositivo

1. `Radio.init()` llama `connect()` y manda `Command.RADIO_INIT`.
2. `build_frame(0x01, seq, b"")` produce COBS+CRC y pyserial lo escribe por Cat-Bridge.
3. RP2040 pasa bytes a UART; `HostIFTask_poll()` detecta `0x00` y difiere el frame a RF task.
4. `CommandProcessor_processEncodedFrame()` decodifica y `handle_command()` llama `ControlTask_onRadioInit()`.
5. `OutputIF_sendResponse(RSP_ACK, seq)` encola la respuesta.
6. Python recibe ACK, envía `GET_INFO 0x05` y firmware usa `ControlTask_getInfoPayload()`.
7. `Radio.init()` construye `DeviceInfo` con firmware `1.0.0`, capabilities `0x07` y serial `FERALRF1` representado como hex.

## 7. Traza B — configuración RF

`Radio.set_phy(PHY.IEEE_802_15_4, channel=11)` → `CommandBuilder.set_phy()` payload `<BHI>` → `CMD_SET_PHY 0x04` → `handle_command()` → `ControlTask_onSetPhy()` → `PhyManager_select()` → `RadioIF_setPhy()` ajusta canal/config SmartRF → ACK → Python actualiza `_phy/_channel`.

El ACK indica que se aceptó la configuración interna. No confirma que U2/CTF externo del CatSniffer esté alineado con la banda.

`Radio.configure_prop()` sigue el mismo patrón con payload de 18 bytes y `RadioIF_setPropConfig()`. OOK deja el backend bloqueado y requiere reset.

## 8. Traza C — recepción y streaming

1. `Radio.start_rx()` envía `RX_START 0x10`.
2. `handle_command()` verifica que no haya TX, llama `ControlTask_onRxStart()` y responde ACK.
3. En la RF task, `DataTask_poll()` consume el evento y llama `RadioIF_startRx()`.
4. `RadioIF` abre/selecciona el cliente RF, postea FS/RX y recibe callbacks `RF_EventRxEntryDone`/overflow.
5. `RadioIF_poll()` drena la queue TI; `DataTask_emitRxPacket()` crea `RSP_RX_PACKET`.
6. Python `read_packets()` decodifica timestamp, channel, RSSI, LQI, CRC, data y metadata LL en `Packet`.

RX admite hasta 239 bytes emitidos por la restricción de payload. Una falla al iniciar el backend sucede después del ACK y se reporta como `RSP_ERROR` asíncrono/`RxStreamError`.

## 9. Traza D — transmisión

`Radio.transmit(packet, power)` → `CommandBuilder.tx_raw()` → `CMD_TX_RAW 0x20` → `handle_command()` → `ControlTask_onTxRaw()` copia máximo 125 bytes y agenda evento → ACK inmediato → `ControlTask_processTxRaw()` → `RadioIF_transmitRaw()` → ruta BLE/IEEE/proprietary y comando TI RF.

**Semántica importante:** el ACK precede a la operación RF. Si TX_RAW falla, firmware emite `RSP_ERROR` asíncrono. `Radio.transmit()` retorna al recibir ACK y no espera un evento de éxito RF. En burst/continuous, una falla posterior limpia el estado sin respuesta asíncrona equivalente visible. No existe `TX_DONE`.

## 10. Eventos y metadata RX

Payload `RSP_RX_PACKET`:

```text
timestamp_us:u64 | channel:u8 | rssi:i8 | lqi:u8 | crc_ok:u8 |
data_len:u8 | data | ll_kind:u8 | ll_type:u8 | ll_flags:u8
```

La clasificación LL (`UNKNOWN/ADV/SCAN/CONNECT/DATA`) solo aporta semántica real para BLE crudo. No constituye un stack BLE ni confirma conexión/GATT. Para IEEE/proprietary, se entregan paquetes crudos.

## 11. Manejo de error, estado y límites

- `ERR_INVALID_CMD/PAYLOAD/FRAME/FRAME_TOO_LONG/STATE/RF_INIT_FAILED/RF_NOT_READY` existen en código.
- Solo un frame de comando puede quedar diferido; otro recibe ERROR busy asíncrono.
- Cola host de salida: 32 frames; al llenarse se descarta sin backpressure al PC.
- TX de control: máximo efectivo 125 bytes; BLE ADV máximo 31; IEEE máximo 125.
- RX: máximo 239 bytes emitidos.
- RX y TX son mutuamente excluyentes; jam también bloquea RX.
- `stop_rx()` reintenta; `stop_jam()` reintenta y cae a TX_STOP.
- `reset_device()` deriva Shell por `port+2`, manda `boot` y `exit`, espera y reinicializa.
- No hay protocol negotiation ni comprobación de versión compatible antes de comandos.
- No hay firmware update en la API.

## 12. Contradicciones del contrato

1. `docs/protocol.md` se declara contrato actual, pero omite comandos test TX y crypto que sí están en `protocol.h`, `enums.py`, handlers y API.
2. El documento omite error `0x07 ERR_RF_NOT_READY`.
3. `docs/ARCHITECTURE.md` nombra firmware FeralRF v2.0; `GET_INFO` devuelve 1.0.0 y el paquete Python es 0.3.0.
4. `commands.py` conserva builders Spectrum sin enum/handler/método público: no constituyen soporte.
5. Comentarios y declaraciones BLE central/follower persisten en `radio_if.h/.c`, pero no son alcanzables desde protocolo/API actual después de retirar el stack.

## 13. Evidencia

- Host: `python/feralrf/{protocol.py,enums.py,commands.py,_responses.py,radio.py}`
- Features: `python/feralrf/{presets.py,emulation/,integrations/killerbee.py,_spectrum.py}`
- Firmware: `firmware/cc1352/{include/protocol.h,src/protocol.c,src/host_if_task.c,src/command_processor.c,src/control_task.c,src/data_task.c,src/output_if.c,src/radio_if.c}`
- Documentación contrastada: `docs/protocol.md`, `docs/PYTHON_API.md`, `docs/ARCHITECTURE.md`.

## Por qué esto importa después

Esta separación permite localizar una observación futura: un error CRC pertenece al framing; un timeout puede pertenecer al transporte o firmware; un ACK seguido de falla pertenece a la ejecución asíncrona. No deben colapsarse como “la radio no funciona”.

## Comprueba tu comprensión

1. ¿Qué añade COBS y qué añade CRC?
2. ¿Quién asigna e interpreta el ID de comando?
3. ¿Por qué `RSP_RX_PACKET` no necesita corresponder uno a uno con una solicitud?
4. ¿Qué demuestra y qué no demuestra el ACK de `TX_RAW`?

**Anterior:** [[Arquitectura FeralRF]]  
**Siguiente:** [[Matriz de capacidades]]
