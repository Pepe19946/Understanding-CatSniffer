# Procedimientos Sub-GHz y experimentos condicionados

## Prerrequisitos comunes Sub-GHz

Complete OP-01 de [[05 - Manual práctico de uso de FeralRF]]. Verifique antenas y autorización para la frecuencia configurada. Antes de una receta Sub-GHz ordinaria, seleccione `band2` por los Shell explícitos; la campaña del 12-oct forzó `band1`→`band2`:

```powershell
python -c "import serial,time; s=serial.Serial('COM35',115200,timeout=.5); s.write(b'band1\r\n'); time.sleep(.3); s.write(b'band2\r\n'); time.sleep(.3); print(s.read_all().decode(errors='replace')); s.close()"
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=.5); s.write(b'band1\r\n'); time.sleep(.3); s.write(b'band2\r\n'); time.sleep(.3); print(s.read_all().decode(errors='replace')); s.close()"
```

La recuperación del 15-oct también obtuvo 10/10 bajo estados lógicos `Band: 0`; por tanto `Band: 1` no quedó demostrado como condición causal necesaria en enlace cercano. Aun así, una repetición ordinaria debe seleccionar explícitamente la ruta correcta, registrar la respuesta y no llamarla medición eléctrica.

Un preset con nombre Wi-SUN, Sidewalk o Wireless M-Bus sólo aporta parámetros PHY/raw. No demuestra framing, stack, seguridad ni interoperabilidad externa.

---

## OP-12 — Comprobar por control un preset GFSK 868

### 1. Objetivo

Configurar `gfsk_868_50k`, recorrer RX start/stop y solicitar un TX RAW en una sola placa.

### 2. Evidencia histórica y estado

**VALIDADO EN LAS CONDICIONES INDICADAS (control).** EV-11 ejecutó el comando dos veces con ACK y cero paquetes locales. No fue una prueba OTA. Es uno de 27 presets aceptados por control; nueve sólo quedaron resumidos.

### 3. Equipo necesario

Una CatSniffer V3, antena/ruta 868 MHz y Bridge histórico `COM88`. No se requiere peer porque el objetivo es sólo control.

### 4. Antes de abrir las terminales

Prepare entorno, identifique `COM88/COM87`, seleccione `band2` y confirme autorización. No use esta receta como prueba de radiación o enlace.

### 5. Disposición de terminales

**Terminal 1 — control de preset / COM88.** Patrón A; el script RX y TX en secuencia, no simultáneamente.

### 6. Ejecución paso a paso

Comando histórico exacto, desde `FeralRF\python`:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python'
python examples\smoke_prop_phase1.py --port COM88 --preset gfsk_868_50k --power 0
```

Equivalente desde la raíz: `python .\python\examples\smoke_prop_phase1.py --port COM88 --preset gfsk_868_50k --power 0`.

### 7. Salida esperada

Salida histórica: `SET_PROP_CONFIG ACK`, `SET_POWER ACK`, `RX_START ACK`, `packets=0`, `RX_STOP ACK`, `TX_RAW ACK`, stats y `PROP PRESET SMOKE PASS`.

### 8. Interpretación

PASS del smoke significa comandos aceptados y cleanup; `TX_RAW ACK` no prueba emisión. Cero paquetes era esperable en un único equipo y no es fallo RX.

### 9. Limpieza

El script detiene RX antes de TX y desconecta al final. Espere su terminación natural.

### 10. Fallos y recuperación

Si falla `RX_STOP`, no continúe al TX/preset siguiente. Si `SET_PROP_CONFIG` da ACK pero luego aparece error async, conserve ambos: el ACK no lo invalida ni lo oculta.

### 11. Límites de la conclusión

No valida 868 MHz efectivos, 50 ksymbol/s, GFSK, potencia, peer delivery ni un protocolo superior.

### 12. Fuentes

[[../../05 - Evaluación/EV-11 — Evidencia de control en 868 MHz]]; [[../../05 - Evaluación/EV-11 — Control de presets propietarios]]; `FeralRF/python/examples/smoke_prop_phase1.py`.

---

## OP-13 — Reproducir el control OTA Sub-GHz de recuperación #2→#3

### 1. Objetivo

Configurar `wisun_915_fsk_50k` en ambas placas y enviar diez marcadores `DEADBEEF` desde #2/COM33 a #3/COM88.

### 2. Evidencia histórica y estado

**VALIDADO EN LAS CONDICIONES INDICADAS.** Checkpoints 3A, 3A-R, 3C, 3D, 3E y 4A obtuvieron 60/60 marcadores CRC-good con cleanup. El comando siguiente es un **equivalente actual reconstruido** porque el helper fue archivado; no es una nueva ejecución.

### 3. Equipo necesario

Dos CatSniffer V3: TX #2 `COM33/35`, RX #3 `COM88/87`; antenas adecuadas para 902,2 MHz. Potencia solicitada 0 dBm. En 4A hubo power-cycle previo de ~15 s.

### 4. Antes de abrir las terminales

Prepare PowerShell y `PYTHONPATH`. Seleccione `band2` en ambas para una repetición ordinaria. Confirme que el script archivado existe y observe que sus puertos están fijos; no lo use si su mapeo actual difiere.

### 5. Disposición de terminales

**Terminal 1 — script coordinado TX #2 / RX #3.** Patrón C; el helper crea ambos `Radio`, inicia RX primero y transmite después. No abra dos terminales.

### 6. Ejecución paso a paso

1. Desde la raíz FeralRF, confirme preset y ruta del helper:

   ```powershell
   python -c "from feralrf import PROP_PRESETS; print(PROP_PRESETS['wisun_915_fsk_50k'])"
   Test-Path '.\docs\OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16\experimental_artifacts\manual_preset_runner.py'
   ```

2. Ejecute una sola vez:

   ```powershell
   python -u '.\docs\OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16\experimental_artifacts\manual_preset_runner.py' wisun_915_fsk_50k DEADBEEF manual-replay-wisun-50k
   $LASTEXITCODE
   ```

El helper fija TX=`COM33`, RX=`COM88`, diez llamadas, 0 dBm, RX antes de TX y desconexión de ambos.

### 7. Salida esperada

El resultado histórico, no una promesa de repetición, fue `TX_ATTEMPT_ACK` 10/10, diez `RX_PACKET` con marcador, `RX_STOP_ACK`, dos desconexiones, `TEST_SUMMARY ... peer_crc_good_marker=10` y `TEST_CLASSIFICATION ... PASS-RF`; exit 0.

### 8. Interpretación

Los markers soportan entrega raw entre CatSniffer. `frequency_hz=902200000` es configuración solicitada; `channel=37`, LQI 0 y RSSI mixto aparecieron en recuperación y no son metadatos calibrados.

### 9. Limpieza

El helper detiene RX y desconecta ambos en `finally`. Si termina PASS, no necesita reset automático. Conserve stdout desde `TEST_BEGIN` a `TEST_END`.

### 10. Fallos y recuperación

Ante excepción/cleanup incompleto, no reintente: cierre, quite toda alimentación ≥15 s, reenumere y vuelva a preflight. El script archivado no acepta argumentos de COM; un mapeo diferente requiere otro artefacto existente, como OP-14, no edición del helper.

### 11. Límites de la conclusión

No demuestra Wi-SUN, 902,2 MHz físicos, potencia, modulación o alcance. El éxito coexistió históricamente con campañas 0/70 y 0/240 cuya causa sigue abierta.

### 12. Fuentes

[[../Pruebas OTA SubGHz/Experimentos FeralRF OTA SubGHz]]; `FeralRF/docs/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-16/{SUBGHZ_RECOVERY_RUN.md,experimental_artifacts/manual_preset_runner.py}`.

### Variaciones OTA documentadas, no recetas independientes

| Preset/familia | Frecuencia configurada | Resultado estrecho 15–16 oct | Límite |
|---|---:|---:|---|
| `gfsk_868_50k`, `gfsk_868_100k` | 868 MHz | 10/10 | raw; GFSK física no instrumentada |
| `wireless_mbus_s/t/c_868` | 868,3/868,95 MHz | 10/10 | no stack/interoperabilidad W-MBus |
| `gfsk_915_50k` | 915 MHz | 10/10 | raw entre dos CatSniffer |
| `gfsk_902_50k` | 902,2 MHz | 10/10 | raw |
| `sidewalk_915_fsk_50k/_250k` | 915 MHz | 10/10 | no Sidewalk |
| `wisun_915_fsk_50k` | 902,2 MHz | 60/60 recovery | no Wi-SUN |
| `wisun_915_fsk_100k/_150k/_200k/_300k` | 902,2 MHz | 10/10 | no FAN/RPL/seguridad |

Cambiar el nombre del preset no garantiza el mismo resultado con otra frecuencia, antena, hardware o estado inicial.

---

## OP-14 — Ejecutar RF-02 Sub-GHz inverso #3→#2

### 1. Objetivo

Reproducir el script coordinado que transmitió diez marcadores `C2028802` desde CatSniffer #3 a #2 con `wisun_915_fsk_50k`.

### 2. Evidencia histórica y estado

**VALIDADO EN LAS CONDICIONES INDICADAS.** RF-02 terminó PASS/exit 0, 10/10 markers CRC-good, `RX_STOP` ACK y dos desconexiones. Se ejecutó por excepción porque RF-01 seguía BLOCKED.

### 3. Equipo necesario

TX #3: Bridge/Shell `COM88/87`; RX #2: `COM33/35`; antenas para 902,2 MHz. RP2040 histórico `v3.1.0.1 c0cd5a4`; CC reportó `1.0.0`; Python 3.14.7.

### 4. Antes de abrir las terminales

Prepare entorno, ejecute preflight fresco y confirme COM/antenas. Defina directorio de log:

```powershell
$rfRunDir = Join-Path (Resolve-Path '.\validation_runs') ('manual_' + (Get-Date -Format 'yyyyMMdd_HHmmss'))
New-Item -ItemType Directory -Path $rfRunDir | Out-Null
python .\experimental_artifacts\next_rf_validation\preflight.py --tx-port COM88 --rx-port COM33 --tx-shell COM87 --rx-shell COM35 --antenna-confirmed --recovery-confirmed 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '02a_preflight_rf02.jsonl')
$LASTEXITCODE
```

### 5. Disposición de terminales

**Terminal 1 — script coordinado TX #3/RX #2.** Patrón C; no abra RX y TX por separado.

### 6. Ejecución paso a paso

Comando histórico exacto:

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

El script selecciona `band2`, inicializa, configura ambos peers, inicia RX, transmite diez veces, observa, detiene y desconecta.

### 7. Salida esperada

Evidencia histórica: `BAND_SELECT_OK`×2, `INIT_OK`, diez `TX_ATTEMPT_ACK`, diez `RX_PACKET`, `STAGE_SUMMARY ... crc_good_marker_count:10 ... rx_stop_ok:true ... cleanup_errors:0`, `RESULT PASS`; exit 0.

### 8. Interpretación

10/10 prueba entrega host-visible raw y cleanup en esa corrida. `channel:25` pese a canal solicitado 0 y RSSI `-32, 63, 106, ...` son anomalías; no los use como medidas.

### 9. Limpieza

Espere `DISCONNECT_OK` para TX y RX y `RESULT`. Conserve JSONL. No necesita otra terminal ni Ctrl+C.

### 10. Fallos y recuperación

Sin `RESULT`, con exit distinto de 0 o cleanup error: no repita; cierre procesos, power-cycle ≥15 s, reenumere y preflight. No convierta la excepción metodológica histórica en permiso automático para saltar gates futuros.

### 11. Límites de la conclusión

No demuestra frecuencia/potencia reales, exactitud de metadatos, Wi-SUN ni fiabilidad universal. Los scripts recientes son artefactos locales sin seguimiento en HEAD `fbff1b7`.

### 12. Fuentes

[[../EXPERIMENT_REPORT_2026-10-20/RF-02 — Validación OTA Sub-GHz inversa]]; `FeralRF/experimental_artifacts/next_rf_validation/rf02_subghz_reverse.py`; `FeralRF/validation_runs/next_rf_20261020_091040/02_rf02_subghz_reverse.jsonl`.

---

## OP-15 — Preparación KillerBee RF-04 bloqueada

### 1. Objetivo

Documentar el intento de habilitar captura pasiva IEEE con KillerBee y el punto previo al hardware donde se bloqueó.

### 2. Evidencia histórica y estado

**EXPERIMENTO BLOQUEADO.** El extra pidió `killerbee>=3.0.0`, pip no encontró distribución compatible en el entorno y no se abrió hardware ni se produjo PCAP.

### 3. Equipo necesario

Para el plan no alcanzado: CatSniffer #2 Bridge `COM33`, antena 2,4 GHz, red Zigbee autorizada canal 25. El bloqueo real sólo necesitó Windows/Python 3.14.7.

### 4. Antes de abrir las terminales

No instale ni invente una fuente alternativa como parte de esta receta. La siguiente línea se conserva como evidencia histórica, no como recomendación repetible.

### 5. Disposición de terminales

**Terminal 1 — preparación de entorno.** Patrón A. Nunca se llegó a una terminal de captura.

### 6. Ejecución paso a paso

Intento histórico exacto:

```powershell
python -m pip install -e '.\python[killerbee]'
```

Comprobaciones históricas posteriores, sin RF:

```powershell
python --version
python -m pip show killerbee
python -c "import importlib.util; print(importlib.util.find_spec('killerbee'))"
python -m pip config list
```

El comando de captura planeado, **no ejecutado**, era:

```powershell
$rfRunDir = Join-Path (Resolve-Path '.\validation_runs') ('manual_' + (Get-Date -Format 'yyyyMMdd_HHmmss'))
New-Item -ItemType Directory -Path $rfRunDir | Out-Null
python .\python\examples\killerbee_sniff.py --port COM33 --channel 25 --duration 60 --count 10 --timeout 100 --min-valid-crc 10 --pcap (Join-Path $rfRunDir '04_rf04_zigbee_passive.pcap')
```

### 7. Salida esperada

La salida real fue `No matching distribution found for killerbee>=3.0.0`; `pip show` no encontró paquete y `find_spec` devolvió `None`.

### 8. Interpretación

La causa operativa identificada es dependencia ausente. No es un FAIL del adaptador, RX IEEE, CC1352P7 ni PCAP.

### 9. Limpieza

No hubo recurso serial ni RF que detener. La instalación fallida no produjo una captura; no se generó PCAP.

### 10. Fallos y recuperación

La recuperación correcta es detenerse hasta disponer de un entorno KillerBee oficialmente reproducible. No se documenta instalación alternativa porque no fue ensayada.

### 11. Límites de la conclusión

No se ejecutaron `KillerBeeFeralRF`, `pnext()` ni `sniffer_off()`; no se validó DLT, CRC ni cierre del adaptador.

### 12. Fuentes

[[../EXPERIMENT_REPORT_2026-10-20/RF-04 — Captura pasiva Zigbee mediante KillerBee]]; [[../EXPERIMENT_REPORT_2026-10-20/RF-04 — Preparación de captura pasiva Zigbee]]; `FeralRF/python/examples/killerbee_sniff.py` (interfaz actual inspeccionada).

---

## OP-16 — OOK 868 aislado RF-05

### 1. Objetivo

Preservar el procedimiento gated previsto para `ook_868_4k8`, sin presentarlo como operación validada.

### 2. Evidencia histórica y estado

**PROCEDIMIENTO RECONSTRUIDO — NO EJECUTADO.** RF-05 quedó `NOT RUN`: RF-01/RF-03 y gates de recuperación no estaban satisfechos. El comando existe en el runbook y script local, pero no es evidencia de éxito.

### 3. Equipo necesario

Dos CatSniffer V3, antenas/ruta 868 MHz, TX `COM33/35`, RX `COM88/87`, posibilidad real de retirar toda alimentación y autorización inmediata del operador.

### 4. Antes de abrir las terminales

**NO-GO salvo que se cumplan todos los gates:** RF-01…03 PASS en una campaña futura; RF-04 terminado/omitido; antenas 868 confirmadas; power-cycle completo disponible; aceptación de perder la sesión. La campaña histórica no los cumplió.

### 5. Disposición de terminales

**Terminal 1 — script coordinado OOK.** Patrón C. No abra segundo proceso. Este layout es documental; no autoriza su ejecución.

### 6. Ejecución paso a paso

Secuencia prevista por el runbook, no ejecutada:

1. Cerrar todos los procesos seriales, retirar USB y toda fuente de ambas placas ≥15 s.
2. Reconectar, ejecutar OP-01 y este preflight:

   ```powershell
   $rfRunDir = Join-Path (Resolve-Path '.\validation_runs') ('manual_' + (Get-Date -Format 'yyyyMMdd_HHmmss'))
   New-Item -ItemType Directory -Path $rfRunDir | Out-Null
   python .\experimental_artifacts\next_rf_validation\preflight.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --antenna-confirmed --recovery-confirmed 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '05a_preflight_rf05.jsonl')
   $LASTEXITCODE
   ```

3. Obtener aprobación explícita inmediatamente antes del comando.
4. Sólo entonces, comando gated:

   ```powershell
   python .\experimental_artifacts\next_rf_validation\rf05_ook_868.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --marker-hex C5050001 --min-received 9 --operator-approved --initial-power-cycle-confirmed 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '05_rf05_ook_868.jsonl')
   ```

5. Sin importar resultado, no enviar otro comando RF; power-cycle total ≥15 s, preflight y este único control IEEE:

   ```powershell
   python .\experimental_artifacts\next_rf_validation\rf01_ieee_channel_switch.py --only-control --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --min-received 9 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '05b_rf05_recovery_ieee_control.jsonl')
   $LASTEXITCODE
   ```

### 7. Salida esperada

No existe salida experimental. El script está diseñado para diez ACK, ≥9 markers CRC-good, stop/cleanup y `PARTIAL` hasta completar power-cycle final y control IEEE.

### 8. Interpretación

Incluso una etapa OTA satisfactoria no sería PASS compuesto sin recuperación y control posterior. Los flags de aprobación son gates de software, no prueba de que el operador hizo el power-cycle.

### 9. Limpieza

Después de cargar OOK, no cambie PHY ni resetee por software. Cierre, quite **toda** alimentación ≥15 s, reenumere, preflight y control IEEE.

### 10. Fallos y recuperación

Ante hang, error o cero markers: clasifique FAIL/BLOCKED según etapa, power-cycle completo y no reintente OOK en esa campaña. Si el control IEEE posterior se bloquea, la recuperación no está demostrada.

### 11. Límites de la conclusión

RF-05 no prueba OOK, 868 MHz, entrega, patches ni recuperación; sólo preserva un plan condicionado. El claim OOK histórico anterior tampoco sustituye esta ejecución ausente.

### 12. Fuentes

`FeralRF/docs/RF_VALIDATION_RUNBOOK.md` §9; `FeralRF/experimental_artifacts/next_rf_validation/rf05_ook_868.py`; [[02 - Matriz histórica de experimentos y ejecuciones#RF-05]].

