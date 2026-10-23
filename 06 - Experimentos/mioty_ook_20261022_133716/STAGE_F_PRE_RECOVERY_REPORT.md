# Stage F — OP-MO-06 OOK, estado previo a recuperación

## Configuración

- TX: Unidad A, COM33.
- RX: Unidad B, COM88.
- Preset: `ook_868_4k8`, sin modificaciones.
- Configuración solicitada: 868000000 Hz, OOK (`mod_type=2`), 4800 baud, deviation 0, `rx_bw=0x4C`, sync `0x930B51DE`.
- Potencia solicitada: 0 dBm.
- Marker: `MO06O00`, secuencias 1–10.

## Resultado raw previo a recuperación

| Campo | Observación |
|---|---|
| Configuración aceptada | 2/2 |
| RX_START | ACK |
| TX ACK | 1/10, sólo secuencia 1 |
| TX secuencia 2 | Timeout |
| TX secuencias 3–10 | `CommandError('Transmit failed')` |
| Payloads coincidentes | 0 |
| Secuencias únicas | 0/10 |
| Errores TX contabilizados | 9 |
| RX_STOP | `CommandError('Stop RX failed')` |
| Stats posteriores | Omitidos por recuperación segura |
| Exit code | 1 |
| Clasificación | `BLOCKED_OR_CLEANUP_FAILED` |

## Interpretación

La configuración OOK fue aceptada en ambas placas y RX_START recibió ACK. Sólo la primera solicitud TX fue aceptada; no hubo evidencia peer de entrega raw. El timeout, los errores TX posteriores y el fallo de RX_STOP impiden cumplir el objetivo raw y dejan el lifecycle sin cierre lógico.

El resultado actual no contradice por sí solo el reporte histórico 10/10 porque las condiciones/binario exactos no están identificados y la causa del fallo actual no está establecida. Tampoco prueba ausencia de modulación OOK: no hubo instrumentación física y un ACK no acredita señal radiada.

La etapa OOK completa sigue abierta hasta realizar power-cycle físico, reidentificación, preflight y baseline GFSK posterior. No debe enviarse ningún otro comando a las placas en el estado actual.
