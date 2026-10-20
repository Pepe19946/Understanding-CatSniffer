# RF-03 — Validación de transición IEEE 802.15.4 ↔ Sub-GHz

**Fecha:** 2026-10-20
**Resultado:** BLOCKED
**Código de salida:** 3
**Condición metodológica:** Ejecución autorizada fuera de la secuencia original del RUNBOOK.

## 1. Objetivo

Evaluar la transición entre IEEE 802.15.4 y Sub-GHz sin reiniciar el CC1352P7.

Secuencia prevista:

1. IEEE 802.15.4, canal 25.
2. Sub-GHz, preset `wisun_915_fsk_50k`, frecuencia solicitada 902.2 MHz.
3. Retorno a IEEE 802.15.4, canal 25.

Cada etapa debía transmitir diez paquetes y recibir al menos nueve con el marcador correspondiente y CRC válido reportado.

## 2. Contexto metodológico

RF-03 se ejecutó mediante excepción autorizada porque RF-01 permanecía en BLOCKED.

RF-02 había terminado en PASS funcional.

La excepción permitió obtener evidencia adicional, pero no modificó los criterios de aceptación.

## 3. Hardware y entorno

**Transmisor:**

* CatSniffer #2.
* Cat-Bridge: COM33.
* Cat-Shell: COM35.

**Receptor:**

* CatSniffer #3.
* Cat-Bridge: COM88.
* Cat-Shell: COM87.

**Firmware RP2040:**

* Versión: `v3.1.0.1`.
* Git: `c0cd5a4 (clean)`.

**Firmware CC1352P7:**

* Versión reportada: `1.0.0`.
* Capabilities: `7`.

**Entorno:**

* Windows.
* Python 3.14.7.
* Biblioteca FeralRF instalada desde el repositorio local.

## 4. Preflight

El preflight previo terminó en PASS, código de salida 0.

Se confirmó:

* Enumeración correcta de interfaces USB.
* Respuesta de Cat-Shell en ambas placas.
* Respuesta de Cat-Bridge en ambas placas.
* Contadores iniciales en cero.
* Desconexión correcta de ambas interfaces.

El preflight no constituye validación de operaciones RF activas.

## 5. Procedimiento

Se ejecutó:

```powershell
python .\experimental_artifacts\next_rf_validation\rf03_cross_phy_switch.py `
    --tx-port COM33 `
    --rx-port COM88 `
    --tx-shell COM35 `
    --rx-shell COM87 `
    --min-received 9 `
    2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '03_rf03_cross_phy_switch.jsonl')

$LASTEXITCODE
```

El script registró `cc1352_reset_allowed=false` y una inicialización por dispositivo.

## 6. Resultados de la primera etapa

**Etapa:** `IEEE_INITIAL`
**Canal:** 25
**Marcador:** `C3032501`
**Potencia solicitada:** 0 dBm

| Métrica                               | Resultado |
| ------------------------------------- | --------: |
| Intentos de transmisión               |        10 |
| ACK de transmisión                    |        10 |
| Paquetes recibidos                    |        14 |
| Paquetes con marcador esperado        |        10 |
| Paquetes con marcador y `crc_ok=true` |        10 |
| Paquetes sin marcador                 |         4 |
| Errores de observación                |         0 |
| Confirmación de `RX_STOP`             |        No |
| Etapa completada                      |        No |

Los diez paquetes experimentales reportaron RSSI de aproximadamente −63 dBm.

## 7. Anomalía

Durante la detención de recepción se registró:

`RX_STOP_CLEANUP_ERROR: TimeoutError('Response timeout (ignored 4 unexpected response(s), last=0x90)')`

Posteriormente:

`FATAL: TimeoutError('Response timeout (ignored 3 unexpected response(s), last=0x90)')`

El script terminó con:

* `completed_stages=[]`.
* `recovery_required=true`.
* `result=BLOCKED`.
* Código de salida 3.

Ambas interfaces registraron `DISCONNECT_OK`.

La desconexión del puerto no demuestra que el receptor haya abandonado correctamente su estado de recepción.

## 8. Etapas no ejecutadas

### `SUBGHZ_MIDDLE`

**NOT RUN.**

No se ejecutó la transición hacia Sub-GHz porque la primera etapa no completó su cierre.

### `IEEE_RETURN`

**NOT RUN.**

No se ejecutó el retorno a IEEE 802.15.4.

## 9. Comparación experimental

RF-01 y RF-03 presentan el mismo patrón observable:

* Recepción de 10/10 paquetes IEEE con marcador esperado.
* Error durante `RX_STOP`.
* Resultado global BLOCKED.

El control de recuperación de RF-01 reprodujo el mismo comportamiento después de un power-cycle.

En contraste, RF-02, configurado como Sub-GHz, recibió 10/10 paquetes y completó `RX_STOP` correctamente.

Esta comparación identifica una diferencia funcional reproducible entre las ejecuciones realizadas, sin establecer su causa raíz.

## 10. Interpretación técnica

Existe evidencia positiva de recepción IEEE 802.15.4 en el canal 25.

Sin embargo, el experimento no consiguió completar la primera etapa debido a un timeout durante la detención de RX.

No se obtuvo evidencia sobre la transición IEEE → Sub-GHz ni sobre el retorno Sub-GHz → IEEE.

La anomalía es compatible con el patrón observado en RF-01, pero no permite atribuir el problema a un componente o función específicos.

## 11. Conclusión

**RF-03: BLOCKED.**

La recepción IEEE inicial funcionó conforme al umbral de paquetes, pero el cierre de esa etapa falló.

La funcionalidad de transición entre PHY permanece **sin validar**, no demostrada como defectuosa.

No se investigó el código ni se realizaron modificaciones.

## 12. Evidencia

* Preflight: `validation_runs/next_rf_20261020_091040/03a_preflight_rf03.jsonl`.
* Experimento: `validation_runs/next_rf_20261020_091040/03_rf03_cross_phy_switch.jsonl`.
* RUNBOOK: `docs/RF_VALIDATION_RUNBOOK.md`, sección 7.
* Salida de PowerShell del 2026-10-20.

## 13. Seguimiento posterior

Durante una fase posterior de investigación:

* Examinar el comportamiento de `RX_STOP` en IEEE 802.15.4.
* Compararlo con el cierre satisfactorio observado en Sub-GHz.
* Determinar el significado de las respuestas inesperadas identificadas como `0x90`.
* Validar la transición entre PHY después de resolver o caracterizar el bloqueo.

Estas actividades quedan fuera del alcance de la campaña experimental actual.
