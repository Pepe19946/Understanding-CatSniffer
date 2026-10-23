# Informe experimental — MIOTY-labeled FSK y OOK

**Fecha:** 2026-10-22, America/Mexico_City  
**Directorio de evidencia:** `python/validation_artifacts/mioty_ook_20261022_133716/`  
**Rama:** `feature/hardware-validation`  
**Commit:** `f2744e400f743df30499e20f03f156f08ad15bd8`  
**Commit auditado previo:** `fbff1b7f5077964418fb63a3fdf346a27c35a003`

## 1. Setup

- Dos CatSniffer con CC1352P7, conexiones USB independientes y setup 868 MHz autorizado por el operador.
- Potencia solicitada en todas las corridas: 0 dBm; no se usó high-PA.
- Unidad A, USB `E661A89753565932`: Bridge COM33, LoRa COM34 no usado, Shell COM35.
- Unidad B, USB `E661A89753656A35`: Bridge COM88, LoRa COM86 no usado, Shell COM87.
- Dirección OTA: Unidad A TX → Unidad B RX.
- Python 3.14.7; FeralRF Python 0.3.0; pyserial 3.5; pyserial-asyncio 0.6; cobs 1.2.2.
- RP2040: `v3.1.0.1`; CC1352 reportado: `feralrf_cc1352 (custom)`, versión lógica `1.0.0`, capabilities `0x07`, serial lógico `FERALRF1`.

La identidad lógica del CC1352 no permite determinar hash ni provenance del binario instalado. La revisión/identidad física y la autorización de antena/ruta/entorno fueron confirmadas por el operador, no medidas por el software.

## 2. Fuente y procedimiento

La implementación RF entre el commit auditado y HEAD no cambió; HEAD añadió documentación. Se siguió el runbook movido a `Understanding-CatSniffer/06 - Experimentos/MIOTY_OOK_VALIDATION_RUNBOOK.md` mediante su harness embebido, con puertos explícitos y sin `Radio.reset_device()`.

El primer wrapper de OP-MO-03 seleccionó accidentalmente el bloque Python incorrecto y terminó con `KeyError` antes de abrir puertos; no fue una corrida RF. Se corrigió la extracción con un patrón literal y guards de contenido. No se modificó ningún parámetro experimental.

## 3. Operaciones ejecutadas

| Operación | Preset/acción | TX ACK | Markers únicos | Cleanup | Exit | Resultado |
|---|---|---:|---:|---|---:|---|
| OP-MO-01 | Identificación Bridge/Shell | N/A | N/A | Puertos cerrados | 0 | PASS |
| OP-MO-02 | `RADIO_INIT`/info/stats | N/A | N/A | Disconnect | 0 | PASS |
| OP-MO-03 | `gfsk_868_50k` baseline | 10/10 | 10/10 | RX_STOP ACK | 0 | PASS raw |
| OP-MO-04 | `mioty_868_tsunb` raw | 10/10 | 0/10 | RX_STOP ACK | 1 | FAIL raw criterion |
| OP-MO-05 | GFSK post-MIOTY | 10/10 | 10/10 | RX_STOP ACK | 0 | PASS raw |
| OP-MO-06 | `ook_868_4k8` | 1/10 | 0/10 | RX_STOP error | 1 | BLOCKED/FAIL cleanup |
| OP-MO-07a | Power-cycle, re-ID y preflight | N/A | N/A | Disconnect | 0 | PASS control |
| OP-MO-07b | GFSK post-OOK | 10/10 | 10/10 | RX_STOP ACK; TX stats timeout | 0 | PASS raw; recuperación PARTIAL |

## 4. MIOTY-labeled FSK

`mioty_868_tsunb` solicitó 868000000 Hz, FSK, 396 baud, deviation 1, `rx_bw=0x4A` y sync `0x930B51DE`. Ambas placas aceptaron configuración y diez solicitudes TX. El receptor no informó paquetes: `rx_ok=0`, cero markers y cero secuencias únicas. No hubo errores asíncronos y RX_STOP recibió ACK.

**Conclusión actual:** `FAIL_RAW_DELIVERY_CRITERION` para “Generic proprietary FSK raw delivery using the mioty_868_tsunb preset” en esta corrida y setup. La causa es inconclusa: no hubo medición de TX completion, frecuencia, desviación, espectro ni airtime físico.

No se evaluaron MIOTY telegram splitting, TS-UNB coding/interleaving, burst scheduling, time/frequency distribution, reassembly ni interoperabilidad. El resultado no demuestra incompatibilidad del CC1352P7 con MIOTY.

## 5. OOK raw

`ook_868_4k8` solicitó 868000000 Hz, OOK, 4800 baud, deviation 0, `rx_bw=0x4C` y sync `0x930B51DE`. La configuración fue aceptada en ambas placas y RX_START recibió ACK. Sólo TX seq=1 recibió ACK; seq=2 agotó timeout, seq=3–10 devolvieron `CommandError`, no se recibieron markers y RX_STOP falló.

**Objetivo A — raw OTA:** FAIL/BLOCKED. No existe evidencia peer de entrega OOK en esta corrida. Un único ACK no prueba radiación OOK.

## 6. Recuperación OOK

Tras OP-MO-06 no se emitió otro comando. El operador retiró toda alimentación durante el intervalo solicitado y reconectó ambas placas. Los COM se reidentificaron, ambos Shell regresaron a `Band: 0`, preflight pasó y stats iniciales fueron cero.

El control GFSK final obtuvo 10/10 ACK y 10/10 secuencias, todas reportadas `crc_ok=True`; RX_STOP recibió ACK y RX stats informó `rx_ok=10`. Sin embargo, `TX_STATS_AFTER` agotó timeout.

**Objetivo B — recuperación:** PASS para restauración de entrega raw GFSK; PARTIAL para cierre completo del lifecycle/control debido al timeout posterior. El exit code 0 del harness no comprueba ese último punto.

## 7. Comparación con evidencia histórica

- MIOTY-labeled raw: el claim histórico registraba 0/10. La corrida actual también produjo 0/10, con 10/10 ACK y cleanup correcto. Son ejecuciones independientes; la coincidencia no establece causa ni valida TS-UNB.
- OOK 868 raw: el reporte histórico documentó 10/10 con recuperación declarada. La corrida actual obtuvo 1/10 ACK, 0/10 markers y fallo de RX_STOP. Ambos resultados deben coexistir; la diferencia de binario/condiciones no puede reconstruirse completamente.
- Recuperación: el power-cycle actual restauró preflight y entrega GFSK 10/10, pero quedó un timeout de stats TX posterior.

## 8. Observaciones confirmadas

1. Los baselines GFSK previo y posterior a MIOTY entregaron 10/10.
2. El preset MIOTY-labeled fue aceptado y sus diez TX recibieron ACK, pero el peer recibió 0/10.
3. La configuración OOK fue aceptada, pero el lifecycle falló durante la segunda solicitud TX y RX_STOP.
4. El power-cycle restauró acceso, estado inicial observable y entrega GFSK 10/10.
5. El metadata proprietary RSSI/LQI es incoherente; se observaron RSSI positivos imposibles y LQI cero.

## 9. Fallos e inconclusos

- FAIL: entrega raw MIOTY-labeled bajo el criterio de esta corrida.
- FAIL/BLOCKED: entrega raw OOK y cleanup lógico OOK.
- PARTIAL: recuperación completa post-OOK por timeout de `TX_STATS_AFTER`.
- INCONCLUSIVE: causa física/software del 0/10 MIOTY-labeled.
- INCONCLUSIVE: si el primer TX OOK generó señal radiada; no hubo marker ni instrumento.
- No establecidos: frecuencia/potencia/modulación efectivas, hashes de firmware y calidad física de enlace.

## 10. CRC y metadata

Los controles GFSK reportaron `crc_ok=True` en todos los payloads coincidentes. No se equipara este campo con validación CRC independiente debido al defecto conocido del parser proprietary. Los bytes coincidentes, secuencias únicas, CRC reportado, RSSI/LQI y stats se preservan por separado en los logs.

## 11. Recomendaciones

1. No repetir OOK sin instrumentación y tracing del estado TX/RX; localizar el timeout de la segunda transmisión y el `RX_STOP` error.
2. Corregir el harness para que `TX_STATS_AFTER_ERROR` afecte la clasificación de recuperación.
3. Investigar MIOTY-labeled con observación física de TX completion/airtime y parámetros RF antes de repetir el mismo OTA.
4. No presentar OP-MO-04 como MIOTY o TS-UNB.
5. Añadir identidad de artifact/firmware a `GET_INFO` para comparaciones reproducibles.

## 12. Cumplimiento

- No se modificaron Python, firmware, presets, scripts ni documentación canónica.
- No se instaló ninguna dependencia.
- No se ejecutó high-PA ni presets fuera de 868 MHz.
- No se usó reset por aritmética de COM.
- No se hizo staging ni commit.
- Los únicos archivos creados son evidencia bajo este directorio de validación.
