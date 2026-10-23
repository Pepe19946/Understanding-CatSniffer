# Alcance, snapshot de fuentes y metodología

Fecha de corte: 2026-10-22 (America/Mexico_City)  
Tipo de trabajo: auditoría estática; **sin apertura de puertos serie y sin operación RF**.

## 1. Objetivo y límite de autorización

Esta auditoría determina qué presets históricamente excluidos, bloqueados o fallidos pueden someterse a una validación físicamente significativa y qué falta en los demás. El preset se trata como una solicitud de parámetros, no como evidencia de un protocolo completo.

La autorización de esta fase cubre lectura de fuente, historial Git, SDK, documentación de hardware, scripts y evidencia preservada. No autoriza preflight serial, cambio de banda, reset, RX, TX ni instrumentación RF. Cualquier experimento queda detenido en el gate de aprobación individual descrito en `05 - Preparación de validación y plan experimental.md`.

## 2. Snapshot reproducible

| Elemento | Snapshot auditado |
|---|---|
| Repositorio primario | `FeralRF/` |
| Rama | `feature/hardware-validation` |
| HEAD | `fbff1b7f5077964418fb63a3fdf346a27c35a003` |
| Fecha/autor del commit | 2026-10-16 16:24:50 -0600, Jose Aguilar |
| Asunto | `docs: add second Sub-GHz campaign report and bilingual manuals` |
| SDK incorporado | SimpleLink Low Power F2 SDK `8.30.01.01` |
| Target de firmware | `DEVICE_VARIANT=CC1352P7`; familia de headers `cc13x2x7_cc26x2x7` |
| Vault | `Understanding-CatSniffer/`, rama `main`, HEAD `01144888db8aa1d7867859a785749e6a01ff5c4d` |

El árbol `FeralRF` ya estaba sucio antes de crear esta auditoría. Se preservaron sin modificación los siguientes elementos ajenos al trabajo:

- modificado: `python/examples/killerbee_sniff.py`;
- no versionados: `docs/NEXT_RF_VALIDATION_PLAN.md`, `docs/RF_VALIDATION_RUNBOOK.md`, `experimental_artifacts/`, `python/op13_rx_temp.py`, `python/op13_tx_temp.py`, `python/validation_artifacts/` y `validation_runs/`.

El directorio del Vault estaba limpio antes de añadir estos seis documentos. No se reescribieron las auditorías `00` ni `01`.

## 3. Autoridad de las fuentes

Se aplicó el siguiente orden:

1. **Fuente actual de FeralRF** para conducta de implementación.
2. **Headers del SDK 8.30.01.01 incorporado** para el contrato exacto de los comandos RF.
3. **Transcripts/JSONL y reportes con artefactos primarios** para resultados observados.
4. **Commits, matrices y documentos sin transcript** como claims históricos, no como prueba primaria.
5. **Documentación del fabricante** para capacidad del silicio, bandas y front-end.

Un ACK sólo prueba que el parser aceptó la longitud y encoló/guardó el comando cuando corresponda. No prueba que `RF_open`, `CMD_FS`, RX o TX hayan terminado ni que la señal solicitada haya existido en el conector de antena.

## 4. Fuentes principales inspeccionadas

### Implementación

- `python/feralrf/presets.py`
- `python/feralrf/radio.py` (`Radio.configure_prop`, `set_phy`, `start_rx`, `transmit`)
- `python/feralrf/commands.py` (`CommandBuilder.set_prop_config`)
- `firmware/cc1352/src/command_processor.c`
- `firmware/cc1352/src/control_task.c`
- `firmware/cc1352/src/radio_if.c`
- `firmware/cc1352/src/smartrf_prop_0.c`
- `firmware/cc1352/syscfg/ti_radio_config*.{c,h}`
- `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/.../rf_prop_cmd.h`

### Scripts

- `python/examples/smoke_ota_txrx.py`
- `python/examples/smoke_f29_subg_915.py`
- `python/examples/lab/demo_mioty_listen.py`
- `python/examples/smoke_prop_phase1.py`
- `experimental_artifacts/next_rf_validation/preflight.py`
- `experimental_artifacts/next_rf_validation/_common.py`
- `experimental_artifacts/next_rf_validation/rf05_ook_868.py`
- `docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16/experimental_artifacts/manual_preset_runner.py`

### Evidencia e historial

Se revisaron los documentos 02, 03, 04, 07 y 08 de la consolidación experimental, las dos auditorías previas, la campaña Sub-GHz del 15–16 de octubre, el runbook preparado el 19 de octubre y los commits F10, F26 y F29/F29.b. Los commits `4970392`, `6138178`, `f5863c3` y `6e34ef2` contienen claims útiles, pero no aportan por sí solos stdout/JSONL primario de sus OTA.

### Hardware y referencias externas

- `hardware/CatSniffer.pdf` y `hardware/CatSniffer.kicad_sch`.
- Datasheet de Johanson `0900PC15A0036`, cuya banda declarada es 862–928 MHz y 2.4–2.5 GHz.
- Página y datasheet del CC1352P7 de TI.
- Página MIOTY de TI, documentación de MIOTY Alliance y catálogo de implementaciones de referencia.

El esquema local es revisión 1.0 (2021) y rotula `CC1352P1F3RGZT`, mientras los preflights históricos describen placas v3 con CC1352P7. Por tanto, el esquema es evidencia de topología y componentes de una revisión, no identificación suficiente del hardware conectado. U4 se declara como `0900PC15A0036`, el switch principal U2 como `RFSW8006QTR7`, y el selector lo controla el RP2040 mediante Cat-Shell, no el CC1352P7.

Referencias de fabricante:

- TI CC1352P7: https://www.ti.com/product/CC1352P7
- TI MIOTY: https://www.ti.com/tool/MIOTY
- Johanson 0900PC15A0036: https://www.johansontechnology.com/datasheets/0900PC15A0036/0900PC15A0036.pdf
- MIOTY Alliance, recursos: https://developers.mioty-alliance.com/build/

## 5. Método de traza

Para cada familia se reconstruyó:

`PROP_PRESETS` → `Radio.configure_prop()` → `<IBIHBIH>`/`SET_PROP_CONFIG` → `command_processor.c` → `RadioIF_setPropConfig()` → selección de `RF_Mode`/setup/overrides → `CMD_FS` + `CMD_PROP_RX` o `CMD_PROP_TX`.

También se separó el momento de configuración del de ejecución. `SET_PROP_CONFIG` muta estructuras y responde ACK. RX comienza más tarde con `RadioIF_startRx()`. TX se acepta primero en `ControlTask_onTxRaw()` y se ejecuta después en `ControlTask_processTxRaw()`; un fallo físico sólo puede aparecer como error asíncrono. Esta separación explica por qué PASS-CTRL no equivale a PASS-RF.

## 6. Taxonomía aplicada

Los estados de readiness son: `READY FOR CONTROLLED TEST`, `CONDITIONAL`, `BLOCKED BY IMPLEMENTATION`, `BLOCKED BY HARDWARE EVIDENCE`, `UNSUPPORTED BY CURRENT DRIVER CONTRACT` e `INCONCLUSIVE`.

Los resultados experimentales, sólo si hay ejecución, son: `PASS`, `PARTIAL`, `FAIL`, `BLOCKED`, `NOT RUN` e `INCONCLUSIVE`. En esta fase todos los experimentos propuestos permanecen `NOT RUN`.

La confianza se asigna como alta cuando coincide fuente actual + contrato SDK o artefacto primario; media cuando la conclusión depende de documentación de revisión/hardware incompleto; baja cuando sólo existe un claim sin evidencia primaria.

## 7. Resultado ejecutivo de la auditoría estática

- El catálogo actual tiene 31 presets.
- Catorce presets FSK/GFSK de 868–915/902.2 MHz tienen evidencia reciente de entrega raw entre CatSniffers; esto no prueba sus protocolos nominales.
- `ook_868_4k8` es el único excluido que queda como candidato RF inmediato, pero condicionado a identidad, banda, antena, recuperación eléctrica y aprobación individual.
- `mod_type=4/5/6` contradice el comando TI usado: esos valores están reservados.
- 169.45 MHz queda fuera de las bandas especificadas para CC1352P7 y además cae por la ruta firmware de 433 MHz.
- Los dos presets propietarios de 2.4 GHz caen a 868 MHz en la ruta de ejecución actual y carecen de setup/overrides 2.4 dedicados.
- `mioty_868_tsunb` configura un paquete FSK genérico; no implementa telegram splitting, hopping/temporización TS-UNB, reensamble ni interoperabilidad con base station.

