# RF-01 — IEEE 802.15.4 Channel Switching

**Fecha:** 2026-10-20
**Resultado final:** BLOCKED
**Motivo:** Timeout reproducible durante `RX_STOP`.

## 1. Objetivo

Validar la comunicación OTA IEEE 802.15.4 entre dos CatSniffer v3 y el cambio de canal 25 → 20 → 25 dentro de una misma sesión.

## 2. Entorno

* Repositorio: FeralRF.
* Rama: `feature/hardware-validation`.
* HEAD registrado en la revisión estática: `fbff1b7f5077964418fb63a3fdf346a27c35a003`.
* Estado del árbol de trabajo: contiene modificaciones y archivos sin seguimiento; el HEAD no identifica por sí solo todos los scripts ejecutados.
* TX: CatSniffer #2, Cat-Bridge COM33, Cat-Shell COM35.
* RX: CatSniffer #3, Cat-Bridge COM88, Cat-Shell COM87.
* Hardware reportado: CatSniffer v3, RP2040 + CC1352P7.
* Firmware RP2040: `v3.1.0.1`, Git `c0cd5a4 (clean)`.
* Firmware CC1352P7 reportado: `1.0.0`.
* PHY: IEEE 802.15.4.
* Canal del control: 25.
* Potencia solicitada: 0 dBm.
* Antenas: confirmadas por el operador.
* Selección RF: `band1` mediante Cat-Shell.

## 3. Ejecución original

**Inicio UTC:** 15:19:55
**Etapa:** `CONTROL_CH25`
**Marcador:** `C1012501`

Resultados:

* 10 intentos TX.
* 10 ACK de transmisión.
* 10 paquetes recibidos con marcador.
* 10 paquetes con `crc_ok=true` reportado por firmware.
* 3 paquetes adicionales sin marcador.
* 0 errores durante la observación.
* Timeout durante `RX_STOP`.
* `RESULT=BLOCKED`.
* Código de salida: 3.

Las etapas de desajuste de canal, canal 20 y retorno al canal 25 no se ejecutaron.

## 4. Recuperación

Se realizó la recuperación física de los dispositivos.

Un intento intermedio de preflight quedó bloqueado porque los CatSniffer estaban desconectados. El operador identificó y corrigió esta condición.

Tras reconectarlos, el preflight terminó con `RESULT=PASS` y código de salida 0.

La recuperación confirmó comunicación funcional mediante Cat-Shell y Cat-Bridge, sin transmisiones RF.

## 5. Control positivo posterior a la recuperación

**Inicio UTC:** 15:41:05
**Etapa:** `RECOVERY_CONTROL_CH25`
**Marcador:** `C10125F0`
**Modo:** `--only-control`

Resultados:

* 10 intentos TX.
* 10 ACK de transmisión.
* 10 paquetes recibidos con marcador.
* 10 paquetes con `crc_ok=true` reportado por firmware.
* 2 paquetes adicionales sin marcador.
* 0 errores durante la observación.
* RSSI de paquetes experimentales: aproximadamente −57 a −58 dBm.
* Timeout durante `RX_STOP`.
* `RESULT=BLOCKED`.
* Código de salida: 3.

## 6. Comparación

Las dos ejecuciones confirmaron recepción OTA de 10/10 paquetes experimentales en el canal 25.

Ambas terminaron con timeout durante `RX_STOP`.

Las excepciones registraron respuestas inesperadas cuyo último identificador fue `0x90`.

El fallo se reprodujo después de la recuperación física y de un preflight satisfactorio.

## 7. Evaluación frente a los criterios de aceptación

**Criterios satisfechos parcialmente:**

* Configuración inicial del canal 25.
* Inicio de RX.
* Recepción OTA de al menos 9/10 paquetes experimentales.
* CRC válido según el indicador reportado por firmware.
* Ausencia de errores durante la observación.

**Criterios no satisfechos:**

* Confirmación de `RX_STOP`.
* Finalización completa de la primera etapa.
* Cambio de canal 25 → 20.
* Validación del desajuste de canales.
* Retorno al canal 25.
* Finalización satisfactoria del experimento completo.

## 8. Conclusiones

1. Existe evidencia positiva de comunicación OTA IEEE 802.15.4 en el canal 25 entre ambos CatSniffer.
2. La recepción del marcador experimental se reprodujo satisfactoriamente después de recuperar los dispositivos.
3. El procedimiento de detención de RX falló en dos ejecuciones.
4. No se ha identificado la causa raíz del timeout.
5. No se ha demostrado el cambio de canal dentro de una misma sesión.
6. RF-01 permanece clasificado como **BLOCKED**.

Los ACK TX no constituyen evidencia independiente de transmisión RF. El indicador `crc_ok` procede del firmware del receptor, no de una verificación CRC externa.

## 9. Evidencias

* Ejecución original: `validation_runs/next_rf_20261020_091040/01_rf01_ieee_channel_switch.jsonl`
* Preflight de recuperación: `validation_runs/next_rf_20261020_091040/01b_preflight_recovery.jsonl`
* Control de recuperación: `validation_runs/next_rf_20261020_091040/01c_rf01_recovery_control.jsonl`

## 10. Seguimiento para Codex

Auditar estáticamente el recorrido de `RX_STOP`, identificando:

* Procesador que ejecuta cada componente.
* Interfaz de comunicación entre host, RP2040 y CC1352P7.
* Construcción y envío de la solicitud de detención.
* Interpretación de respuestas y eventos asíncronos.
* Origen y significado del identificador `0x90`.
* Condiciones que producen `Response timeout`.
* Tratamiento de errores y limpieza del estado RX.
* Diferencias entre el comportamiento esperado y el observado.

No modificar firmware ni scripts hasta identificar una causa sustentada en código.

**PENDIENTE DE CONSOLIDACIÓN — REQUIERE AUDITORÍA POSTERIOR.**


{
  "event": "OBSERVATION_END",
  "stage": "RECOVERY_CONTROL_CH25",
  "crc_good_marker_count": 10,
  "marker_match_count": 10,
  "packet_count": 12,
  "error_count": 0
}


