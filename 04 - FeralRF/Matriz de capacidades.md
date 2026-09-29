# Capacidades FeralRF — Cómo leer afirmaciones con rigor

## En una frase

Una capacidad puede estar anunciada, implementada, cubierta por tests y físicamente probada en grados diferentes; esta matriz conserva esas categorías separadas.

## Modelo mental

Piensa en cuatro preguntas sucesivas: “¿se afirma?”, “¿existe un camino de código?”, “¿alguna prueba ejercita ese camino?” y “¿hay evidencia de radio/hardware real?”. Un sí temprano no garantiza los siguientes.

## Criterio

Baseline verificado nuevamente el `2026-09-28`: `FeralRF@0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`, Python `0.3.0`, firmware target predeterminado CC1352P7/TI-RTOS7.

- **Declarado:** aparece en README/docs.
- **FW:** existe handler y operación fuente alcanzable desde el protocolo actual.
- **Python:** existe API/transporte alcanzable.
- **Pruebas:** especifica si es codec/mock/host; no equivale a RF real.
- **Validación física:** `reportada` solo cuando `docs/VALIDATION_MATRIX.md` conserva fecha/condición/conteo. No fue repetida por Codex y, salvo la propia matriz, no hay logs crudos adjuntos.
- Un preset o ejemplo no constituye un stack de protocolo ni prueba de éxito.

## Matriz principal

| Capacidad | Declarado | FW | Python | Pruebas actuales | Ejemplo | Validación física | Evidencia | Límites |
|---|---|---|---|---|---|---|---|---|
| Descubrimiento/sesión | sí, estable | RADIO_INIT/INFO | `Radio.list_devices/connect/init` | fake serial para init; contrato baud | múltiples smoke | reportada: control baseline 18/18 | `radio.py`, `command_processor.c`, matriz §1–2 | autodetección VID amplia; primer puerto; no probe |
| Info/version/capabilities | sí | INFO 1.0.0/caps 0x07 | `DeviceInfo` | `test_radio_strict_responses.py` mock | smoke phase2 | incluida en control reportado | `ControlTask_getInfoPayload`, `Radio.init` | versiones docs 2.0/Python 0.3.0 difieren |
| Stats/diagnóstico RX | sí | GET_STATS 16/36 B | `get_stats()` | parsing indirecto/contratos; sin firmware test | smoke/release scripts | reportada a nivel control | `send_stats`, `Radio.get_stats` | RF debug interno no expuesto |
| Config PHY/canal/potencia | sí, estable | handlers + `RadioIF_set*` | métodos `set_*` | payload/enum/host mocks parciales | casi todos los smoke | reportada control y OTA por PHY | `command_processor.c`, `radio_if.c` | ACK no verifica selector externo U2 |
| RX continuo multi-PHY | sí | RF callbacks, queue, `RSP_RX_PACKET` | `start_rx/read_packets/stop_rx` | fake frames, async error y `read_one_packet` | baseline/OTA/probe | reportada con marcadores para varios PHY | `data_task.c`, `radio.py`, matriz §2/4 | hasta 239 B; drops posibles; sin reconnect |
| RSSI/LQI/CRC/timestamp/LL meta | sí | emitidos en RX_PACKET | objeto `Packet` | parsing con FakeSerial | sniff/KillerBee | reportada solo como parte de RX, no calibración | `DataTask_emitRxPacket`, `read_packets` | precisión/calibración no conservada |
| TX_RAW/TX_FRAME | sí, estable | handler/control/TI RF | `transmit`, `transmit_frame` | builders/contrato; sin prueba RF automatizada | `smoke_tx_*`, OTA | reportada OTA por PHY/presets | handlers 20/23, matriz | ACK es aceptación, no TX_DONE |
| TX_BURST/CONTINUOUS/STOP | sí | implementados | métodos públicos | contrato básico; no dispositivo | smoke dedicados | matriz reporta PASS IEEE/control; evidencia por comando ambigua | IDs 21/22/24, `control_task.c` | fallos tardíos burst/continuous no generan evento visible |
| Test TX CW/PRBS | sí, estable | IDs 55–57, `RadioIF_runTxTest` | `tx_cw/prbs/test_stop` | `test_tx_test.py` mock | `lab/smoke_f22_tx_test.py` | reportada 2026-04-29 como wire-level | matriz §6 | no análisis espectral preservado; comentario PRBS inconsistente |
| IEEE 802.15.4 raw | sí, estable | SmartRF IEEE RX/TX | PHY + raw API | host/KillerBee mocks | `smoke_phy4_ieee154.py` | reportada 10/10 OTA | matriz §2/4 | no stack Zigbee/Thread/Matter |
| Zigbee/Thread/Matter/6LoWPAN | declarado “ride on PHY” | solo PHY 802.15.4 | raw frames + KillerBee adapter | adapter mocked | KillerBee sniff/runbook | no para stacks; solo PHY reportado | README, integration | parsing/MAC/security/stack no implementados |
| BLE PHY raw 1M/2M/Coded | sí | RF backend BLE + LL classifier | PHY enum/raw RX/TX | host packet tests, no radio | baseline OTA | reportada: 8–10/10 o 10/10 | matriz §2/4 | captura cruda; no stack BLE |
| BLE advertisement hopping | sí | SET_ADV_HOP/driver cadence | `set_adv_hop()` | contrato indirecto | quick start/scripts | no evidencia específica separada | handler 07, `radio_if.c` | passive raw hopping, no active scan |
| BLE active scan/conexión/GATT | docs dicen retirado | no handler actual; quedan restos internos no alcanzables | no API | no tests actuales de dispositivo | ejemplos históricos eliminados | pendiente/no | protocol §9, architecture §2 | no debe llamarse soporte BLE completo |
| Propietario Sub-1 GHz FSK/GFSK/MSK/OOK/4FSK | sí | `SET_PROP_CONFIG`, SmartRF/patches | `configure_prop`, presets | schema/wire de presets | smoke prop/OTA/lab | reportada por banda; 433 marginal y OOK433 FAIL | `presets.py`, `radio_if.c`, matriz §5 | presets no son stacks; OOK exige reset |
| Propietario 2.4 GHz | README experimental | config/ruta generic | presets `gfsk_2440_*` | schema preset | smoke prop/OTA | matriz reporta GFSK2440 10/10, contradice README | README vs matriz §2/5 | requiere revalidación HEAD y selector RF |
| Wireless M-Bus S/T/C | README estable | solo radio configurable | presets | schema preset | baseline/OTA | reportada 10/10 OTA | matriz §2/5 | no parser/stack W-MBus |
| Wireless M-Bus N 169 | declarado preset | generic prop | presets 169 | schema | baseline enumera | pendiente/no OTA | matriz §5/14 | compatibilidad RF/antena sin validar |
| Wi-SUN | experimental | solo carrier FSK generic | presets NA-1 | schema preset | demo/smoke F29 | reportada 70/70 compartida para 7 presets | matriz §9 | no FAN stack |
| Amazon Sidewalk | experimental | solo FSK generic | presets FSK | schema preset | demo/smoke F29 | reportada dentro de 70/70 | matriz §9 | no Sidewalk stack; LR/SX1262 ausente |
| MIOTY | planned | no TS-UNB nativo | preset retenido | schema confirma existencia, no función | demo listen | reportada FAIL 0/10 | `presets.py`, matriz §9 | requiere CPE patch custom |
| Crypto TRNG/AES/SHA/ECDH/ECDSA | README estable | TI crypto drivers y handlers | métodos `Radio` | mocks + vectores software de referencia | `lab/smoke_f25_crypto.py` | reportada 9/9 hardware, 2026-04-30 | `crypto_engine.c`, tests, matriz §7 | logs/resultados crudos no adjuntos; RSA ausente |
| RSA | planned | no | no | no | no | pendiente | README/matriz | ausente |
| Emulación PHY-level | sí | reutiliza TX/config | helpers `emulation/` | payload/schema host | demos F17 | reportada 7/7 wire-level; 433/OOK experimental | matriz §8 | no autenticación ni stack completo |
| Jamming continuo | experimental | IDs 30/33 + sesión RF | `start_jam/stop_jam` | adapter mock; política payload | smoke lab | pendiente con criterio real; ACK solamente | protocol/matriz | no prueba de interferencia; reactivo/pattern ausentes |
| Spectrum/RSSI scan | pending | no ID/handler | solo builder/dataclasses huérfanos | no data path | no funcional | pendiente | `_spectrum.py`, `_responses.py`, `commands.py` | no implementado end-to-end |
| KillerBee IEEE 802.15.4 | declarado | reutiliza IEEE raw | adapter sniff/set/inject/jam | unit/mock + real package con FakeRadio | `killerbee_sniff.py`, runbook | pendiente explícitamente | `integrations/killerbee.py`, docs | requiere parche externo a KillerBee; no prueba RF integrada |
| Reset/recovery CC | sí | RP no FeralRF | `reset_device()` usa Shell boot/exit | mock | validation scripts | matriz reporta OOK recovery PASS; otro doc reporta reset rompe init | `radio.py`, matriz §10, killerbee doc | `bridge+2` frágil; contradicción de banco |
| Firmware update | README remite a Catnip | no updater | no updater | no | comando documentado | no evidencia FeralRF propia | README | dependencia externa Catnip |
| SX1262/LoRa | explícitamente fuera del firmware CC | no | no | no | no | no | presets comment/ausencia | FeralRF ignora Cat-LoRa y SX1262 |

## Qué significa “validado físicamente” aquí

`docs/VALIDATION_MATRIX.md` registra:

- corrida completa OTA de dos placas el 2026-04-08 con conteos por PHY/preset;
- test modes el 2026-04-29;
- crypto 9/9 el 2026-04-30;
- presets F29 70/70 el 2026-05-03;
- condiciones marginales/fallos 433, OOK y MIOTY.

Esto es evidencia documental explícita de pruebas físicas y por eso se marca **reportada**. Sin embargo:

1. no hay logs crudos, hashes de binarios, commits exactos de cada corrida ni archivos de resultados adjuntos;
2. la matriz dice que la última corrida completa precede a HEAD y pide repetirla;
3. `PASS` mezcla “control path y/o OTA”, por lo que algunas filas no permiten separar ambas;
4. no se verificó que la placa probada fuera exactamente la unidad v3.1/P74 actual ni su estado CTF;
5. Codex no reprodujo estas pruebas.

## Conclusión de capacidades

El núcleo implementado es una API de **primitivas RF/packet-level** sobre la radio integrada CC1352P7: configuración, RX/TX, métricas, test TX y crypto. Los nombres Zigbee, Thread, W-MBus, Wi-SUN y Sidewalk describen PHY/presets o integración de host, no stacks completos. BLE queda como PHY raw. Spectrum, BLE active scanning/conexión/GATT, RSA, jamming reactivo/pattern y SX1262 no están implementados end-to-end en HEAD.

## Concepción errónea común

**“Si existe un preset llamado Wi-SUN o Sidewalk, FeralRF implementa ese protocolo.”** Un preset configura parámetros PHY; una pila completa también necesita MAC, estados, seguridad e interoperabilidad definidos por el protocolo.

## Comprueba tu comprensión

1. ¿Por qué un test con FakeSerial no valida transmisión RF?
2. ¿Qué diferencia hay entre IEEE 802.15.4 raw y Zigbee completo?
3. ¿Por qué una corrida física anterior a HEAD debe revalidarse?
4. ¿Qué significa que KillerBee esté implementado como adapter pero físicamente pendiente?

**Anterior:** [[Protocolo y API Python]]  
**Siguiente:** [[Pruebas y evidencia existente]]
