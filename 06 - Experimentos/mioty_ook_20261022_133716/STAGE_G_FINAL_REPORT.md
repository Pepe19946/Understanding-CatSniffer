# Stage G — Control GFSK posterior al power-cycle OOK

## Resultado literal del runbook

| Campo | Observación |
|---|---|
| `band2` | Aceptado en ambas placas; `Band: 1` |
| Configuración GFSK | 2/2 |
| TX ACK | 10/10 |
| Payloads coincidentes | 10 |
| Secuencias únicas | 1–10, 10/10 |
| `crc_ok=True` reportado | 10/10 |
| Errores asíncronos/TX contabilizados | 0 |
| `RX_STOP` | ACK |
| RX stats | `rx_ok=10`, demás contadores 0 |
| TX stats posterior | Timeout |
| Exit code | 0 |
| Clasificación impresa | `PASS_RAW_STAGE` |

## Evaluación de recuperación

La entrega raw GFSK y el cleanup RX se restauraron después del power-cycle, por lo que existe evidencia positiva de recuperación RF funcional. No obstante, `TX_STATS_AFTER` agotó timeout. El harness imprime PASS porque no incorpora los errores de stats posteriores a `raw_criterion`.

Se conservan dos clasificaciones complementarias:

- **PASS_RAW_STAGE:** resultado literal y defendible para entrega peer GFSK y `RX_STOP`.
- **PARTIAL de recuperación completa:** el plano de control pasó antes del ensayo, pero la consulta TX posterior no cerró correctamente.

No se repitió la corrida y no se enviaron más comandos RF. Los RSSI positivos y LQI=0 son metadata defectuosa y no sustentan conclusiones físicas.

