# Stage C — OP-MO-03 control conocido

## Configuración

- TX: Unidad A, Bridge COM33, Shell COM35.
- RX: Unidad B, Bridge COM88, Shell COM87.
- Preset: `gfsk_868_50k`.
- Configuración solicitada: 868000000 Hz, GFSK, 50000 baud, deviation 100, `rx_bw=0x52`, sync `0x930B51DE`.
- Potencia solicitada: 0 dBm.
- Payload: prefijo `MO03C00` y secuencias 1–10.
- Umbral provisional: 9/10 secuencias únicas con `crc_ok=True`, sin errores y cleanup correcto.

## Resultado

| Campo | Observación |
|---|---|
| Configuración aceptada | 2/2 |
| TX ACK | 10/10 |
| Payloads coincidentes | 10 |
| Secuencias únicas | 1–10, 10/10 |
| `crc_ok=True` reportado | 10/10 |
| Errores asíncronos/TX | 0 |
| `RX_STOP` | ACK |
| RX stats | `rx_ok=10`, demás contadores 0 |
| Exit code | 0 |
| Clasificación del runbook | `PASS_RAW_STAGE` |

## Interpretación

Se observó entrega raw bidireccional en el sentido Unidad A→Unidad B para diez payloads GFSK propietarios únicos bajo la configuración solicitada. Esto establece que el setup actual puede servir como baseline antes del preset MIOTY-labeled.

No se midieron frecuencia radiada, potencia, desviación, ancho ocupado ni forma de onda. `TX_ACK` no es por sí mismo completion, aunque los diez payloads peer aportan evidencia directa de entrega. Los metadatos RSSI incluyen valores incoherentes (`+43`, `+46`, `+7 dBm`) y LQI fue siempre cero; esos campos no son válidos para caracterización física. Aunque `crc_ok=True` fue 10/10, se conserva la limitación conocida del parser proprietary.

## Cleanup y continuación

`RX_STOP` fue aceptado, no hubo errores asíncronos y ambos handles se cerraron. La continuación hacia OP-MO-04 es técnicamente segura, pero requiere autorización separada del operador. No se ejecutó MIOTY-labeled FSK en esta etapa.
