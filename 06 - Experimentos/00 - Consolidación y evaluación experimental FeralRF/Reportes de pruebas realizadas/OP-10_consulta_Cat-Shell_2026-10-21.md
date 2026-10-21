# OP-10 — Consulta posterior de Cat-Shell

**Fecha:** 2026-10-21  
**Dispositivo:** CatSniffer #2  
**Interfaz:** Cat-Shell, `COM35`, `115200 8N1`  
**Herramienta:** `python -m serial.tools.miniterm COM35 115200`  
**Comando:** `status`

## Respuesta observada (transcripción literal)

```text
Mode: 0, Band: 0, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes
```

## Alcance de la evidencia

- La consulta posterior reporta `Band: 0` (selección lógica `band1`) y firmware RP2040 `v3.1.0.1`.
- Los contadores mostrados son `uart_overrun=0` y `ring_dropped=0 bytes` **en el momento de la consulta**; no acreditan ausencia de pérdidas en todas las capas durante OP-10.
- **No acredita retrospectivamente** qué banda estaba seleccionada durante CW, PRBS15 o PRBS32. Las salidas de esas tres ejecuciones no contienen lecturas de Cat-Shell.
- `Radio: LoRa` es una etiqueta del estado reportado por Cat-Shell; no prueba modulación LoRa en las pruebas FeralRF.

**Estado de OP-10:** control de comandos ACK para CW/PRBS15/PRBS32 y TX_TEST_STOP; selección de banda durante las ejecuciones no documentada; salida RF física no medida.

**Fuente primaria:** salida `status` facilitada por el operador en la conversación; procedimiento de referencia: `06 - Procedimientos IEEE, TX y modos de prueba`, OP-10.
