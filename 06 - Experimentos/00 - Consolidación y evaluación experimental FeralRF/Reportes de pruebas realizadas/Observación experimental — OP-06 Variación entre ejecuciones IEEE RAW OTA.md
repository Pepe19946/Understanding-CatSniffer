### Observación experimental — OP-06: variación entre ejecuciones IEEE RAW OTA

**Estado del procedimiento:** PASS con observación de reproducibilidad.

Durante la validación de OP-06 se realizaron dos ejecuciones con CatSniffer #2 como transmisor (`COM33`) y CatSniffer #3 como receptor (`COM88`), utilizando IEEE 802.15.4, canal 25, potencia TX solicitada de 0 dBm y marcador hexadecimal `A10133008801`.

Antes de transmitir, se realizó un control negativo de cinco segundos. El receptor detectó nueve paquetes, pero ninguna coincidencia con el marcador (`marker_hits=0`).

| Parámetro                         | Ejecución 1 | Ejecución 2 |
| --------------------------------- | ----------- | ----------- |
| Duración RX                       | 60 s        | 90 s        |
| Solicitudes TX                    | 10          | 10          |
| `TX_RAW ACK`                      | 10          | 10          |
| Paquetes RX totales               | 109         | 151         |
| Paquetes con CRC válido reportado | 109         | 151         |
| Coincidencias del marcador        | 9           | 10          |
| Resultado del script RX           | FAIL        | PASS        |
| `RX_STOP ACK`                     | Sí          | Sí          |

**Interpretación:**

La primera ejecución confirmó recepción OTA funcional, pero no alcanzó el mínimo de diez coincidencias exigido por el procedimiento. La segunda ejecución cumplió el criterio de aceptación con diez coincidencias válidas.

No se ha demostrado que aumentar la ventana RX de 60 a 90 segundos haya causado la mejora. La diferencia también podría estar relacionada con la sincronización entre los procesos, las condiciones RF u otros factores no controlados.

Además, como las diez solicitudes TX utilizaron el mismo marcador, `marker_hits=10` no permite establecer una correspondencia individual entre cada solicitud de transmisión y cada paquete recibido.

**Conclusión:** OP-06 se considera PASS por la segunda ejecución, conservando el resultado inicial de 9/10 como observación experimental. No existe evidencia suficiente para atribuir la coincidencia faltante a un defecto del firmware.

**Ajustes recomendados para Codex:**

1. Incorporar ambas ejecuciones al registro experimental, sin sustituir el resultado 9/10 por el posterior 10/10.
2. Aclarar que `marker_hits` representa coincidencias recibidas, no una medición directa de Packet Delivery Ratio (PDR).
3. Explicar que `TX_RAW ACK` confirma la aceptación de la solicitud por el firmware, pero no constituye por sí solo una medición RF.
4. Conservar `--min-hits 10` como criterio del procedimiento.
5. Registrar como mejora futura una variante con identificadores de secuencia por paquete para correlacionar TX y RX.

No se recomienda modificar el firmware basándose únicamente en estos resultados.
