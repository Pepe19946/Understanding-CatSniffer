# RF-02 — Validación OTA Sub-GHz inversa

**Fecha:** 2026-10-20
**Resultado funcional:** PASS
**Código de salida:** 0
**Condición metodológica:** Ejecutado mediante excepción autorizada al RUNBOOK.

## 1. Objetivo

Evaluar la comunicación OTA Sub-GHz en dirección CatSniffer #3 (TX) → CatSniffer #2 (RX).

La prueba busca comprobar la configuración de ambos dispositivos, transmisión de paquetes, recepción del marcador esperado y cierre correcto de la sesión.

## 2. Contexto metodológico

El RUNBOOK original exige RF-01 PASS antes de ejecutar RF-02.

RF-01 permanece en BLOCKED debido a un timeout reproducible durante `RX_STOP`.

Se autorizó expresamente ejecutar RF-02 fuera de la secuencia original para ampliar la cobertura funcional de la campaña.

Esta excepción no modifica los criterios de aceptación de RF-02 ni permite declarar RF-01 como satisfactorio.

## 3. Hardware y entorno

**Transmisor:**

* CatSniffer #3.
* Cat-Bridge: COM88.
* Cat-Shell: COM87.

**Receptor:**

* CatSniffer #2.
* Cat-Bridge: COM33.
* Cat-Shell: COM35.

**Firmware RP2040, ambas placas:**

* Versión: `v3.1.0.1`.
* Git: `c0cd5a4 (clean)`.

**Firmware CC1352P7, ambas placas:**

* Versión reportada: `1.0.0`.
* Capabilities: `7`.

**Entorno:**

* Windows.
* Python 3.14.7.
* Biblioteca FeralRF desde el repositorio local.

El preflight previo a RF-02 terminó en PASS, con código de salida 0.

## 4. Configuración solicitada

| Parámetro           | Valor                         |
| ------------------- | ----------------------------- |
| PHY                 | `PROPRIETARY_GFSK`            |
| Preset              | `wisun_915_fsk_50k`           |
| Frecuencia          | 902200000 Hz (902.2 MHz)      |
| Canal solicitado    | 0                             |
| Potencia solicitada | 0 dBm                         |
| Marcador            | `C2028802`                    |
| Intentos TX         | 10                            |
| Umbral mínimo       | 9 paquetes válidos reportados |

La frecuencia y la potencia corresponden a valores de configuración, no a mediciones instrumentales.

## 5. Procedimiento ejecutado

Se ejecutó:

```powershell
python .\experimental_artifacts\next_rf_validation\rf02_subghz_reverse.py `
    --tx-port COM88 `
    --rx-port COM33 `
    --tx-shell COM87 `
    --rx-shell COM35 `
    --marker-hex C2028802 `
    --min-received 9 `
    2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '02_rf02_subghz_reverse.jsonl')

$LASTEXITCODE
```

El script seleccionó `band2` en ambas placas, inicializó los dispositivos, configuró la radio, inició recepción y realizó diez intentos de transmisión.

Posteriormente observó los paquetes recibidos, detuvo la recepción y cerró ambas conexiones.

## 6. Resultados observados

| Métrica                    | Resultado           |
| -------------------------- | ------------------- |
| Selección lógica Sub-GHz   | ACK en ambas placas |
| Inicialización             | PASS                |
| Configuración              | ACK                 |
| Inicio RX                  | ACK                 |
| Intentos TX                | 10                  |
| ACK TX                     | 10                  |
| Paquetes RX                | 10                  |
| Coincidencias de marcador  | 10                  |
| Paquetes con `crc_ok=true` | 10                  |
| Errores RX                 | 0                   |
| Errores TX                 | 0                   |
| `RX_STOP`                  | ACK                 |
| Errores de cleanup         | 0                   |
| Desconexiones              | 2/2 correctas       |
| Resultado final            | PASS                |
| Código de salida           | 0                   |

El contador `rx_ok` del receptor pasó de 0 a 10.

## 7. Anomalías registradas

### 7.1. Campo de canal

Aunque el experimento solicitó canal 0 y frecuencia 902.2 MHz, los diez eventos `RX_PACKET` reportaron `channel: 25`.

No se determinó si ese campo representa el canal RF efectivo, un valor residual o un metadato con otra interpretación.

**Estado:** pendiente de investigación posterior.

### 7.2. Valores RSSI

Los valores RSSI reportados fueron:

`-32, 63, 106, 5, 64, -72, 39, -74, -22, -123 dBm`

La presencia de valores positivos y las variaciones observadas impiden utilizarlos como mediciones confiables de calidad de enlace sin validación adicional.

**Estado:** pendiente de investigación posterior.

## 8. Interpretación técnica

La prueba aporta evidencia positiva de comunicación entre los dos CatSniffer en la configuración experimental Sub-GHz solicitada.

Se observaron diez paquetes con el marcador esperado, diez ACK de transmisión y cierre correcto de la sesión.

Los resultados satisfacen los criterios funcionales automatizados de RF-02.

No obstante, la evidencia disponible no demuestra de manera independiente la frecuencia efectiva de radiación, la potencia transmitida ni la exactitud de todos los metadatos RF.

## 9. Conclusión

**RF-02: PASS funcional, ejecutado mediante excepción metodológica.**

La comunicación y el cierre de sesión funcionaron conforme a los criterios establecidos.

Se conservan dos anomalías de metadatos para evaluación posterior:

1. `channel: 25` en recepción pese a solicitar canal 0.
2. Valores RSSI inconsistentes.

No se investigaron causas raíz ni se modificó código.

## 10. Evidencia

* Preflight: `validation_runs/next_rf_20261020_091040/02a_preflight_rf02.jsonl`.
* Experimento: `validation_runs/next_rf_20261020_091040/02_rf02_subghz_reverse.jsonl`.
* RUNBOOK: `docs/RF_VALIDATION_RUNBOOK.md`, sección 6.
* Salida de PowerShell del 2026-10-20.
