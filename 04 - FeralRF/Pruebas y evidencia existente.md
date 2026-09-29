# Evidencia FeralRF — Qué demuestran realmente las pruebas

## En una frase

Los tests actuales demuestran principalmente contratos y lógica host; la matriz documental reporta corridas físicas, pero no conserva suficiente artefacto crudo para tratarlas como reproducción independiente del baseline actual.

## Modelo mental

Hay una escalera de evidencia: codec aislado → API con mocks → integración de software → dispositivo real → RF entre equipos → medición instrumental. Cada peldaño responde preguntas distintas. Esta nota evita usar la palabra “probado” sin decir en cuál.

## 1. Qué se inspeccionó

Baseline `FeralRF@0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`, verificado nuevamente el `2026-09-28`. No se ejecutaron tests: hacerlo podía crear `.pytest_cache`, `__pycache__` o artefactos dentro del repositorio read-only. La clasificación se basa en fuente, workflows, ejemplos y registros documentales.

## 2. Suite `python/tests/`

| Categoría | Archivos | Qué demuestra | Qué no demuestra |
|---|---|---|---|
| Codec | `test_protocol.py` | COBS, CRC16, build/parse y rechazo de CRC/longitud en Python | identidad con firmware ejecutado ni UART real |
| Contrato/IDs | `test_commands_contract.py`, `test_enums_no_collisions.py` | payload separado de ID, baud default, no colisiones declaradas | handler firmware operativo |
| Sesión/respuestas | `test_radio_strict_responses.py`, `test_radio_seq.py` | init mock, INFO corto, SEQ y filtros | enumeración/timeout/reconnect físicos |
| Streaming/error | `test_async_error_surfacing.py`, `test_read_one_packet.py` | FakeSerial RX_PACKET y errores seq 0/FF | throughput, drops o callbacks TI |
| TX test | `test_tx_test.py` | métodos forman comandos y validan argumentos | portadora/PRBS RF |
| Prop/presets | `test_props.py` | schema, rangos y serialización de presets | que parámetros SmartRF sean correctos u OTA |
| Crypto API | `test_crypto.py` | IDs, validaciones y payload/respuesta simulada | acelerador TI real |
| Vectores crypto | `test_crypto_vectors.py` | vectores conocidos mediante biblioteca host `cryptography` | ejecución del mismo vector en CC1352 |
| Emulación | `test_emulation.py` | payloads/personalidades y presets host | emulación funcional de protocolo/dispositivo |
| KillerBee | `test_killerbee_integration.py`, `test_killerbee_dispatch.py` | adapter, capabilities y dispatch con FakeRadio; opcionalmente clases KillerBee reales | sniff/inject/jam on-air |

No se encontraron pruebas unitarias C del firmware ni HIL dentro de `python/tests/`. El marker `hardware` está declarado en `pyproject.toml`, pero los archivos actuales inspeccionados usan mocks; los flujos físicos están en `examples/`.

## 3. CI

`.github/workflows/build.yml`:

- instala y ejecuta pytest para Python 3.9–3.12;
- ejecuta black/isort/flake8; mypy está permitido fallar;
- el job firmware tiene `continue-on-error: true`;
- el job firmware solo ejecuta `cmake ..` y no invoca el build.

Por tanto, CI puede probar la lógica Python, pero no prueba que el firmware enlace ni que corra. La presencia del workflow no es evidencia física.

`.github/workflows/release.yml` empaqueta/publica Python. El texto del release menciona firmware assets, pero el workflow no compila ni carga artefactos firmware.

## 4. Ejemplos y su naturaleza

| Grupo | Rutas | Clasificación |
|---|---|---|
| Control/smoke de un equipo | `smoke_phase2.py`, `smoke_phy4_ieee154.py`, `smoke_prop_phase1.py`, `smoke_tx_*` | smoke manual con hardware; no gate automatizado por CI |
| OTA dos equipos | `smoke_ota_txrx.py`, `smoke_f29_subg_915.py`, `lab/smoke_f9_phy_matrix_ota.py` | validación de marcadores on-air si se ejecuta |
| Orquestación | `run_validation_baseline.sh`, `release_gate_multi_phy.py` | workflow/gate manual; resetea entre pasos |
| Test modes/crypto | `lab/smoke_f22_tx_test.py`, `lab/smoke_f25_crypto.py` | harness de banco; los archivos no contienen resultados ejecutados |
| Emulación/demos | `smoke_f17_emulation.py`, `lab/demo_emulate_*`, `demo_{wisun,sidewalk,mioty}_*` | demostración de payload/preset, no prueba de stack completo |
| KillerBee | `killerbee_sniff.py`, `docs/TESTING-ON-LINUX.md` | ejemplo/runbook; documentación dice que RF real sigue pendiente |
| Jamming | `lab/smoke_jam_phase1.py` | operación experimental; ACK no prueba interferencia |
| Caracterización | `lab/test_pa_characterization.py`, `sweep_phy4_ieee154.py` | procedimiento de laboratorio, no resultados preservados |

Un ejemplo es código ejecutable que requiere hardware; no es evidencia de que se ejecutó. La evidencia de ejecuciones reside solo en la narrativa de `VALIDATION_MATRIX.md` y algunos comentarios/commit subjects.

## 5. Lectura crítica de `VALIDATION_MATRIX.md`

Vocabulario original:

- `PASS`: “control path y/o OTA”;
- `marginal`: pasa con poco margen;
- `FAIL`: no funciona en el hardware usado;
- `experimental`: presente sin validación real.

Mapeo al proyecto:

| Estado original     | Categorías de este proyecto                                                               |     |
| ------------------- | ----------------------------------------------------------------------------------------- | --- |
| PASS control path   | implementado + resultado físico reportado, pero tipo de prueba puede ser solo request/ACK |     |
| PASS OTA con conteo | implementado + resultado físico RF reportado                                              |     |
| marginal            | físicamente probado según documento, resultado limitado                                   |     |
| FAIL                | prueba física negativa reportada, no ausencia de implementación                           |     |
| experimental        | implementado pero físicamente pendiente                                                   |     |
| preset              | Python implementado; no prueba de stack                                                   |     |

Fortalezas del registro: da fechas, dos placas para OTA, marcadores/conteos, fallos y condiciones marginales. Debilidades: no enlaza logs, fotos, instrumentos, firmware hash, versión RP2040, estado de selector RF ni commit exacto por corrida. El baseline completo es del 2026-04-08 y el propio documento pide repetirlo contra firmware actual.

## 6. Evidencia física reportada

| Fecha | Evidencia declarada | Cobertura | Reserva |
|---|---|---|---|
| 2026-04-08 | 18/18 control; OTA por PHY/preset | BLE raw, IEEE, Sub-1, proprietary, W-MBus, OOK | corrida anterior a HEAD; sin log adjunto |
| 2026-04-29 | smoke F22 | CW/PRBS/stop wire-level | no espectro/analizador conservado |
| 2026-04-30 | 9/9 smoke hardware | TRNG/AES/SHA/ECDH/ECDSA | no salida cruda adjunta |
| 2026-05-03 | 70/70 F29 | presets Wi-SUN/Sidewalk FSK | prueba carrier/preset, no stack |
| 2026-05-04 | 7/7 wire-level | emulación PHY payload | no equivalencia a dispositivo/stack |
| 2026-07-02 | comentario de banco | reset opt-in rompe init con RP stock | contradice recovery PASS general; condiciones incompletas |

No se encontró evidencia física preservada para KillerBee end-to-end, BLE active scan/GATT, spectrum, jamming efectivo, RSA, W-MBus N 169, High-PA o SX1262.

## 7. KillerBee

La integración actual contiene:

- `KillerBeeFeralRF`: `SNIFF`, `SETCHAN`, `INJECT`, `PHYJAM`, `FREQ_2400`;
- `pnext()` sobre `Radio.read_one_packet()`;
- inject que retira FCS y llama `transmit_frame()`;
- jamming que llama la capacidad experimental;
- parche externo `docs/killerbee-catsniffer.patch` para modificar KillerBee.

Las pruebas verifican el adapter y el dispatch con FakeRadio; incluso `test_killerbee_dispatch.py` usa el paquete real con radio falsa. `docs/killerbee-catsniffer.md` dice explícitamente que live sniff, injection, jam y key-capture requieren banco. Por ello el adapter está implementado, pero “compatibilidad KillerBee completa” no está físicamente validada.

## 8. Huecos que prepara Fase 5

1. Congelar hashes de RP2040, FeralRF HEX, Python y SDK para cada corrida.
2. Registrar placa TX/RX, revisión/P74, SO, puertos, antena, canal/frecuencia, potencia y estado CTF.
3. Separar control-path de OTA en resultados.
4. Guardar stdout/PCAP/fotos o trazas de instrumento por prueba.
5. Repetir capacidades reclamadas contra HEAD actual, no inferir continuidad desde abril.
6. Validar errores tardíos, colas, resets, reconexión y cambios de PHY.
7. Validar KillerBee real y no solo adapter con FakeRadio.

No se proponen correcciones en esta fase.

## Por qué esto importa después

La futura fase experimental debe comenzar donde termina la evidencia, no repetir ciegamente afirmaciones de README ni confundir tests unitarios con funcionamiento OTA.

## Comprueba tu comprensión

1. ¿Qué prueba un vector crypto ejecutado por una biblioteca host?
2. ¿Qué evidencia adicional convertiría un ejemplo en un resultado reproducible?
3. ¿Por qué `PASS control path y/o OTA` es una categoría ambigua?
4. ¿Qué necesita KillerBee para pasar de adapter probado a integración RF validada?

**Anterior:** [[Matriz de capacidades]]  
**Relacionado:** [[Fuentes FeralRF]], [[Estado y siguientes pasos]]
