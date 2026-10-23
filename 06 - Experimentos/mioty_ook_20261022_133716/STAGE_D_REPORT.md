# Stage D — OP-MO-04 MIOTY-labeled generic FSK

## Configuración

- TX: Unidad A, Bridge COM33.
- RX: Unidad B, Bridge COM88.
- Preset: `mioty_868_tsunb`, sin modificaciones.
- Configuración solicitada: 868000000 Hz, FSK, 396 baud, deviation 1, `rx_bw=0x4A`, sync `0x930B51DE`.
- Tasa efectiva teórica documentada: aproximadamente 395.203 baud.
- Potencia solicitada: 0 dBm.
- Payload: prefijo `MO04M00` y secuencias 1–10.
- Separación: 1.0 s; observación posterior: 5.0 s.

## Resultado

| Campo | Observación |
|---|---|
| Configuración aceptada | 2/2 |
| RX_START | ACK |
| TX ACK | 10/10 |
| Payloads coincidentes | 0 |
| Secuencias únicas | 0/10 |
| `crc_ok=True` reportado | 0 |
| Errores asíncronos/TX | 0 |
| `RX_STOP` | ACK |
| RX stats final | `rx_ok=0`, demás contadores 0 |
| Exit code | 1 |
| Clasificación del runbook | `FAIL_RAW_DELIVERY_CRITERION` |

## Interpretación

El plano de control aceptó el preset en ambas placas y aceptó diez solicitudes `TX_RAW`. El receptor no informó paquetes ni marcadores durante la ventana definida. Esto es un resultado negativo para la pregunta estrecha: entrega raw FSK propietaria con la configuración exacta de `mioty_868_tsunb` en este setup y esta ejecución.

La causa no queda establecida. Los ACK no demuestran TX completado ni señal radiada y no hubo instrumentación para medir frecuencia, desviación, espectro o airtime. El resultado no demuestra que el CC1352P7 sea incompatible con MIOTY y no fue una prueba TS-UNB. Tampoco evaluó splitting, coding/interleaving, scheduling, reassembly o interoperabilidad.

El resultado reproduce la dirección del claim histórico 0/10 de marker raw, pero constituye evidencia actual independiente y debe conservarse por separado.

## Cleanup y continuación

`RX_STOP` fue aceptado, no hubo errores asíncronos y los handles se cerraron. Es técnicamente válido ejecutar OP-MO-05 para comprobar que el baseline GFSK sigue operativo, pero no se hará sin continuar el proceso supervisado.
