# RF-01 — Nueva evidencia de switching IEEE 802.15.4

**Fecha:** 2026-10-21
**Experimento:** RF-01 — IEEE Channel Switching
**Resultado global:** `BLOCKED`
**Avance funcional:** Cambio de canal `25 → 20 → 25` demostrado en una ejecución, con recepción OTA de marcadores válidos.

## 1. Configuración

| Elemento              | Configuración                                                              |
| --------------------- | -------------------------------------------------------------------------- |
| TX                    | CatSniffer #2 — Cat-Bridge COM33 / Cat-Shell COM35                         |
| RX                    | CatSniffer #3 — Cat-Bridge COM88 / Cat-Shell COM87                         |
| PHY                   | `IEEE_802_15_4`                                                            |
| Banda lógica          | `band1` (2.4 GHz) en ambos dispositivos                                    |
| Potencia solicitada   | 0 dBm                                                                      |
| Transmisiones         | 10 por etapa                                                               |
| Criterio de recepción | ≥9 marcadores con CRC válido                                               |
| Evidencia primaria    | `validation_runs/manual_20261021_130015/01_rf01_ieee_channel_switch.jsonl` |

El preflight previo terminó en `PASS`.

## 2. Resultados observados

| Etapa                | TX/RX | Marcadores CRC válidos | `RX_STOP` | Resultado                      |
| -------------------- | ----- | ---------------------: | --------- | ------------------------------ |
| `CONTROL_CH25`       | 25/25 |                  10/10 | ACK       | PASS                           |
| `MISMATCH_TX25_RX20` | 25/20 |                   0/10 | ACK       | PASS — control negativo        |
| `MATCHED_CH20`       | 20/20 |                  10/10 | ACK       | PASS                           |
| `RETURN_CH25`        | 25/25 |                   9/10 | Timeout   | Recepción PASS; cierre fallido |

**Observación principal:** el receptor obtuvo marcadores válidos después de cambiar al canal 20 y nuevamente después de regresar al canal 25.

El control negativo también fue satisfactorio: con TX en el canal 25 y RX en el 20 no se observaron marcadores.

## 3. Diferencias frente a la evidencia histórica

Referencia: `02 - Matriz histórica de experimentos y ejecuciones.md`, registros `RF-01-A` y `RF-01-R` del 20 de octubre de 2026.

| Aspecto                    | 20 de octubre   | 21 de octubre                |
| -------------------------- | --------------- | ---------------------------- |
| Control inicial, canal 25  | 10/10           | 10/10                        |
| Primer `RX_STOP`           | Timeout         | ACK                          |
| Control negativo TX25/RX20 | No alcanzado    | 0/10                         |
| Comunicación en canal 20   | No evaluada     | 10/10                        |
| Regreso al canal 25        | No evaluado     | 9/10                         |
| Último punto alcanzado     | Control inicial | Recepción de la cuarta etapa |
| Resultado global           | BLOCKED         | BLOCKED                      |

En la ejecución histórica, el timeout inicial impidió evaluar el switching. En la ejecución actual se completaron las cuatro etapas de recepción previstas.

**Diferencia relevante:** ya no se dispone únicamente de evidencia de comunicación en el canal 25; existe evidencia experimental de reconfiguración de canal y recuperación de comunicación al regresar al canal original.

## 4. Incidencia pendiente

En `RETURN_CH25`, después de recibir 9/10 marcadores válidos, se produjo:

`TimeoutError('Response timeout (ignored 4 unexpected response(s), last=0x90)')`

El valor `0x90` corresponde a una respuesta asíncrona de recepción de paquetes.

* **Observado:** no se obtuvo `RX_STOP ACK` en la cuarta etapa.
* **Hipótesis:** la gestión de respuestas asíncronas o el estado de RX podría contribuir al timeout.
* **No demostrado:** que el firmware CC1352P7 haya fallado físicamente al detener RX o que el cambio de canal haya causado el problema.

## 5. Conclusión

**RF-01 permanece `BLOCKED` según los criterios del procedimiento**, porque no obtuvo todos los ACK de `RX_STOP`.

No obstante, la ejecución proporciona nueva evidencia positiva de:

1. Comunicación IEEE 802.15.4 en canal 25.
2. Ausencia de marcadores al desalinear TX/RX entre canales 25 y 20.
3. Comunicación IEEE 802.15.4 en canal 20 después de la reconfiguración.
4. Recuperación de comunicación en canal 25 sin reinicio intermedio.

La principal brecha observada se concentra ahora en la finalización fiable de RX, no en una incapacidad demostrada para cambiar de canal.

**Límites:** una sola ejecución no demuestra estabilidad del switching; tampoco se realizó una medición RF independiente.

**Recuperación posterior:** ambos CatSniffer fueron desconectados de la alimentación durante al menos 15 segundos y reconectados. Pendiente verificar el preflight posterior a la recuperación.
