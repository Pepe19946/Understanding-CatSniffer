# Stage E — OP-MO-05 control posterior a MIOTY-labeled FSK

## Configuración

- TX: Unidad A, COM33.
- RX: Unidad B, COM88.
- Preset: `gfsk_868_50k`.
- Frecuencia solicitada: 868000000 Hz.
- Potencia solicitada: 0 dBm.
- Marker: `MO05C00`, secuencias 1–10.

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
| Clasificación | `PASS_RAW_STAGE` |

## Interpretación

El baseline propietario GFSK volvió a entregar 10/10 inmediatamente después de la corrida MIOTY-labeled 0/10. Por tanto, aquella corrida negativa no dejó el plano raw GFSK general inoperativo y su cleanup fue suficiente para esta transición concreta.

Esto no identifica la causa del 0/10 lento. Los RSSI reportados incluyen valores físicamente incoherentes de hasta +109 dBm y LQI=0; no son evidencia física válida. La frecuencia y potencia efectivas siguen sin medición independiente.

## Estado previo a OOK

No hubo error asíncrono, fallo de cleanup ni baseline pendiente. Técnicamente se cumplen los gates de software para considerar OP-MO-06, pero OOK requiere una autorización separada y power-cycle físico obligatorio inmediatamente después de la corrida.
