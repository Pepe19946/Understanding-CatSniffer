# Fase 1 — Registro de fuentes

Fecha de corte: 2026-10-23. Las rutas `FeralRF/...` y `Understanding-CatSniffer/...` son relativas al workspace `catsniffer-feralrf/`.

| Fuente | Revisión/fecha | Tipo | Secciones/uso | Conclusiones dependientes |
|---|---|---|---|---|
| `FeralRF/python/feralrf/presets.py` | HEAD `f2744e4` | C source | `PROP_PRESETS` completo | Recuento 31, parámetros y nombres |
| `FeralRF/python/feralrf/radio.py` | HEAD | C source | `set_phy`, `set_channel`, `configure_prop`, TX/RX/reset | API, defaults, ACK y limitaciones lifecycle |
| `FeralRF/python/feralrf/commands.py` | HEAD | C source | `set_phy`, `set_prop_config` | Wire payload `<IBIHBIH>` |
| `FeralRF/python/feralrf/enums.py` | HEAD | C source | IDs PHY/command/response | Selección backend y comandos |
| `FeralRF/firmware/cc1352/src/command_processor.c` | HEAD | C source CC1352P7 | `CMD_SET_PHY`, `CMD_SET_CHANNEL`, `CMD_SET_PROP_CONFIG` | Parser acepta 16/18 bytes y ACK sin resultado RF |
| `FeralRF/firmware/cc1352/src/control_task.c` | HEAD | C source CC1352P7 | `ControlTask_onSetPhy`, TX pending/process | Separación aceptación/ejecución |
| `FeralRF/firmware/cc1352/src/radio_if.c` | HEAD | C source CC1352P7 | `RadioIF_applySub1ghzChannelConfig`, `RadioIF_setPropConfig`, parser PROP, OOK patches | Fallback 2440→868, 169 incoherente, format, rate, metadata, OOK lock |
| `FeralRF/firmware/cc1352/include/radio_if.h` | HEAD | C header | `RadioIF_PropConfig` | Unidades declaradas frente a encoding real |
| `FeralRF/firmware/cc1352/CMakeLists.txt` | HEAD | C build | target/device/SDK | Target CC1352P7 |
| `FeralRF/firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/.../rf_prop_cmd.h` | submodule `5b31d0a`, SDK 8.30.01.01 | C SDK primario | `modType`, deviation step | 0/1/2 válidos; 4/5/6 reservados; paso 250 Hz teórico |
| `FeralRF/python/pyproject.toml` | HEAD | C manifest | `[project]` | Paquete 0.3.0 |
| `FeralRF/docs/VALIDATION_MATRIX.md` | histórico hasta 2026 | H/R matriz | baseline abril, F29, presets | Claims antiguos; no primarios equivalentes |
| `FeralRF/docs/HARDWARE_VALIDATION_RESULTS/VALIDATION_MATRIX_CURRENT.md` | corte 2026-10-22; audita `fbff1b7` | S/C matriz canónica | PROP/PRE rows | Estado implementación/evidencia previa |
| `FeralRF/docs/HARDWARE_VALIDATION_RESULTS/EXPERIMENTAL_EVIDENCE_REFERENCE.md` | 2026-10-22 | S/R registro | campañas y procedencia | Reconciliación, límites de primarios |
| `FeralRF/docs/HARDWARE_VALIDATION_RESULTS/PENDING_VALIDATION_ASSESSMENT.md` | 2026-10-22 | S/C | readiness | Bloqueos y prioridades |
| `FeralRF/docs/HARDWARE_VALIDATION_RESULTS/PROJECT_STATUS_REPORT.md` | 2026-10-22 | S | estado de proyecto | Separación raw/protocolo |
| `FeralRF/docs/HARDWARE_VALIDATION_RESULTS/FUNCTIONAL_GAPS.md` | 2026-10-22 | S/C | GAP-05/13/14/15/20/21 | Reserved mods, OOK, 2.4, 169, protocolos |
| `FeralRF/docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-12/` | 6–12 oct | P/R | `execution_report`, experimentos 01–05, CSV/matriz | IEEE y OCT-PROP 0/240; no duplicar copia bajo HARDWARE_VALIDATION_RESULTS |
| `FeralRF/docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16/SUBGHZ_RECOVERY_RUN.md` | 15-oct | P/R largo | checkpoints 0–4A | 60/60 recovery y estados/reset/band |
| `FeralRF/docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16/VALIDATION_MATRIX_261016` | 15–16 oct | P/R matriz | P01–P13, E01–E24 | 13 presets nuevos, ACK/peer/incidentes exactos |
| `FeralRF/docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16/SUBGHZ_PROVENANCE_AUDIT.md` | 2026-10-16 | R/S | procedencia | Distinguir claim, script y primary artifact |
| `FeralRF/docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16/experimental_artifacts/` | 15–16 oct | P scripts/log helpers | checkpoints y `manual_preset_runner.py` | Procedimiento exacto, markers, limitaciones |
| `FeralRF/python/validation_artifacts/mioty_ook_20261022_133716/EXPERIMENT_REPORT_MIOTY_OOK_20261022.md` | 22-oct; HEAD `f2744e4` | P/R | informe integral | MIOTY 0/10, OOK 0/10, recuperación GFSK |
| `FeralRF/python/validation_artifacts/mioty_ook_20261022_133716/STAGE_C_REPORT.md` … `STAGE_G_FINAL_REPORT.md` | 22-oct | P/R por etapa | OP-MO-03…07 | Payloads únicos, ACK, cleanup y timeout stats |
| `FeralRF/python/validation_artifacts/mioty_ook_20261022_133716/OP-MO-*.txt` | 22-oct | P transcript | salida cruda | Inspectabilidad de resultados actuales |
| `Understanding-CatSniffer/06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/00 - Índice, alcance y método.md` | hasta 21-oct | S | método y 193 unidades | Regla de conteo previa |
| `.../01 - Informe consolidado de evaluación experimental.md` | hasta 21-oct | S/R | evolución y conclusiones | Historia 70/70→0/70→recovery |
| `.../02 - Matriz histórica de experimentos y ejecuciones.md` | hasta 21-oct | S/R | tabla 193 | Inventario sin doble conteo |
| `.../03 - Capacidades demostradas y catálogo de presets.md` | hasta 21-oct | S/C | catálogo compacto | Cobertura raw y gaps |
| `.../04 - Evidencia, procedencia y preguntas abiertas.md` | hasta 21-oct | S/register | índice de fuentes | Procedencia y duplicados |
| `.../05 - Manual práctico de uso de FeralRF.md` | actualizado 21-oct | Manual mixto | OP-01–03 | Identificación/control; procedimiento ≠ ejecución |
| `.../06 - Procedimientos IEEE, TX y modos de prueba.md` | actualizado 21-oct | Manual mixto | OP-04–11 | IEEE/TX; histórico vs reconstruido |
| `.../07 - Procedimientos Sub-GHz y experimentos condicionados.md` | no validado como revisión | Manual/propuesta | OP-12–16 y catálogo | Cobertura declarada; no evidencia de ejecución |
| `.../08 - Validación reciente del manual frente a evidencia histórica.md` | 21-oct | P/R/S | comparación OP/RF | Resultados recientes IEEE y bloqueos |
| `Understanding-CatSniffer/06 - Experimentos/EXPERIMENT_REPORT_2026-10-20/` | 20-oct | P/R | RF-01–RF-04 | RF-02 reverso; bloqueos RF-01/03/04 |
| `Understanding-CatSniffer/06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/Reportes de pruebas realizadas/` | 21-oct | P/R | nueve reportes | Validación manual, switching, BURST/CONTINUOUS, recovery |
| `Understanding-CatSniffer/04 - FeralRF/02 - Auditoría de presets excluidos y validación condicionada/` | 22-oct | C/S/R | documentos 00–06 | Inventario 31, MIOTY, OOK, HW/driver y readiness |
| `Understanding-CatSniffer/05 - Evaluación/EV-11 — Control de presets propietarios.md` y auditoría EV | antes de 8-oct | P/S | EV-00…EV-14 | L1 control, 27 presets, límites |

## Duplicados reconciliados

- La copia de OCT-12 bajo `docs/HARDWARE_VALIDATION_RESULTS/` no se cuenta además del directorio canónico `docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-12/`.
- `SUMMARY.md`, `README.md`, matrices CSV/Markdown y documentos del Vault que describen la misma corrida son referencias secundarias, no nuevos experimentos.
- REC-3A es E01 y no se suma dos veces; E02–E24 aportan 23 unidades adicionales en la consolidación anterior.
- RF-05 del 20-oct fue NOT RUN y no cuenta como ejecución; OP-MO-06 del 22-oct sí es una ejecución OOK distinta.
- El primer wrapper OP-MO-03 que terminó en `KeyError` antes de abrir puertos no es una corrida RF.

## Limitación del registro

Las rutas y fechas permiten reproducir la auditoría documental, pero los artifacts instalados en hardware histórico no tienen hashes suficientes. Una fuente de reporte puede sostener una observación documentada sin ser telemetría primaria; esa distinción se conserva en la matriz.

## Fuentes añadidas para la corrección taxonómica

| Fuente | Tipo | Uso |
|---|---|---|
| TI `rf_prop_cmd.h` del SDK 8.30.01.01 local | Primaria local | Contrato de `CMD_PROP_RADIO_DIV_SETUP`: 0/1/2 y valores reservados |
| TI CC1352P7 datasheet/product page | Primaria fabricante | Capacidad de silicio 2-(G)FSK, 4-(G)FSK, MSK y OOK; rangos publicados |
| TI Proprietary RF User's Guide, PHY Configuration y SysConfig RF design | Primaria fabricante | Setup sub‑GHz/2.4 y diseño especial 169 MHz |
| TI SmartRF Studio release history | Primaria fabricante | Configuración CC1352P7 169 MHz N‑mode y perfiles caracterizados |
| Wi‑SUN Alliance FAN Technology Overview y PHY PICS | Primaria consorcio | SUN FSK, tasas, índices, FEC y separación PHY/stack |
| ETSI TS 103 357‑2 V2.1.1 | Norma primaria | TS‑UNB/telegram splitting y framing MIOTY |
| OMS Specifications Vol. 1/2 | Primaria consorcio | Relación con EN 13757‑4 y modos W‑M‑Bus |
| AWS IoT Wireless Sidewalk documentation | Primaria proveedor | Capas BLE/FSK/LoRa y necesidad de stack Sidewalk |
| CSA Zigbee Specification | Primaria consorcio | Separación IEEE 802.15.4 PHY/MAC de NWK/APS/ZDO/aplicación Zigbee |

Las URLs, versiones, fecha de consulta y limitaciones de acceso se registran en la Fase 2, documento `09 - Phase 2 Source Register.md`.

## Changelog

- 2026-10-23: se añadieron fuentes primarias externas usadas para corregir la taxonomía; no se sustituyeron ni eliminaron fuentes históricas.
