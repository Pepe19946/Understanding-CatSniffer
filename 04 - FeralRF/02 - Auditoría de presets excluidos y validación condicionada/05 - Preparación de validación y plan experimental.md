# Preparación de validación y plan experimental

## 1. Matriz de readiness

Los estados describen si vale la pena ejecutar, no un resultado PASS/FAIL.

| Preset | Razón de exclusión | Compatibilidad HW | Readiness FW | Prueba requerida / prerrequisito | Riesgo | Decisión | Confianza |
|---|---|---|---|---|---|---|---|
| `gfsk_433_50k` | ruta/antena y evidencia contradictoria | chip sí; PCB v3 no probado | ruta 433 existe | esquema/BOM v3 + medición de ruta + antenas 433 | emisión mal adaptada, falso FAIL | **BLOCKED BY HARDWARE EVIDENCE** | media-alta |
| `gfsk_433_10k` | igual | igual | igual | igual + config SmartRF a 10 kbaud | igual | **BLOCKED BY HARDWARE EVIDENCE** | media |
| `fsk_433_50k` | igual | igual | FSK válido | igual | igual | **BLOCKED BY HARDWARE EVIDENCE** | media-alta |
| `ook_433_4k8` | OOK lock + ruta 433 | chip sí; PCB no probado | patch existe | hardware 433 probado + recuperación OOK | lock y emisión mal adaptada | **BLOCKED BY HARDWARE EVIDENCE** | alta |
| `ook_433_2k4` | igual | igual | patch existe | igual + config 2.4 kbaud validada | igual | **BLOCKED BY HARDWARE EVIDENCE** | alta |
| `msk_433_50k` | `mod_type=4` reservado | chip plausible; PCB incierto | sin ruta MSK | implementación oficial distinta | estado RF indefinido | **UNSUPPORTED BY CURRENT DRIVER CONTRACT** | alta |
| `4fsk_433_50k` | `mod_type=5` reservado | chip plausible; PCB incierto | sin ruta 4-FSK | implementación oficial distinta | estado RF indefinido | **UNSUPPORTED BY CURRENT DRIVER CONTRACT** | alta |
| `4gfsk_433_50k` | `mod_type=6` reservado | chip plausible; PCB incierto | sin ruta 4-GFSK | implementación oficial distinta | estado RF indefinido | **UNSUPPORTED BY CURRENT DRIVER CONTRACT** | alta |
| `ook_868_4k8` | lock/recovery y falta de evidencia reciente | chip/U4 plausibles; confirmar placas/antenas | patch y packet path existen | identidad, entorno autorizado, power-cycle y control compuesto | lock del RF Core | **CONDITIONAL** | alta |
| `msk_868_50k` | `mod_type=4` reservado | HW plausible | sin ruta MSK | implementación oficial distinta | estado RF indefinido | **UNSUPPORTED BY CURRENT DRIVER CONTRACT** | alta |
| `4fsk_868_50k` | `mod_type=5` reservado | HW plausible | sin ruta 4-FSK | implementación oficial distinta | estado RF indefinido | **UNSUPPORTED BY CURRENT DRIVER CONTRACT** | alta |
| `4gfsk_868_50k` | `mod_type=6` reservado | HW plausible | sin ruta 4-GFSK | implementación oficial distinta | estado RF indefinido | **UNSUPPORTED BY CURRENT DRIVER CONTRACT** | alta |
| `wireless_mbus_n_169_2k4` | banda fuera de P7 + ruta 433 incoherente | no compatible/establecido | incorrecta | hardware distinto + nueva configuración | transmisión fuera de diseño | **BLOCKED BY HARDWARE EVIDENCE** | alta |
| `wireless_mbus_n_169_4k8` | igual | igual | incorrecta | igual | igual | **BLOCKED BY HARDWARE EVIDENCE** | alta |
| `mioty_868_tsunb` | no existe TS-UNB | 868 plausible | faltan bursts/timing/reensamble | stack/patch + base station | falso claim de protocolo | **BLOCKED BY IMPLEMENTATION** | alta |
| `gfsk_2440_250k` | fallback a 868 + overrides erróneos | chip/U4 plausibles | frecuencia efectiva incorrecta | corregir ruta 2.4 y medir frecuencia | transmitir en banda no pretendida | **BLOCKED BY IMPLEMENTATION** | alta |
| `gfsk_2440_50k` | igual | igual | igual | igual | igual | **BLOCKED BY IMPLEMENTATION** | alta |

No hay ningún preset excluido clasificado `READY FOR CONTROLLED TEST`. Sólo OOK 868 es `CONDITIONAL` con prerrequisitos resolubles sin cambiar el firmware.

## 2. Gate común obligatorio

Antes de abrir cualquier COM o ejecutar preflight:

1. aprobación explícita del operador para **un solo experimento identificado**;
2. identificación física de ambas placas y revisión de hardware;
3. asociación explícita Bridge/Shell, sin aritmética Bridge+2;
4. confirmación de antena/ruta/banda y entorno regulatorio o apantallado;
5. disponibilidad de retiro total de alimentación;
6. registro de HEAD, estado sucio, Python/import y binario/firmware instalados;
7. directorio nuevo de evidencia y espacio para stdout/stderr/JSONL/exit code;
8. ausencia de OOK desde el último power-cycle.

Si un requisito falta, el resultado es `BLOCKED` o `NOT RUN`; no se abre el puerto para “ver qué pasa”.

## 3. Experimento candidato único: RF-05/OOK-868

Estado actual: **PLANNED — NOT AUTHORIZED — NOT RUN**.

### Objetivo estrecho

Determinar si dos CatSniffer con la misma revisión y ruta 868 conocida intercambian paquetes propietarios OOK a 868 MHz/4.8 kbaud, y si un power-cycle completo restaura un control RF conocido. No valida un protocolo OOK externo, potencia, espectro ni conformidad regulatoria.

### Hardware y condiciones

| Campo | Requisito |
|---|---|
| Placas | dos CatSniffer v3 identificados; históricamente #2 y #3 |
| MCU | CC1352P7 confirmado en ambas |
| Bridge/Shell | confirmar en el momento; valores históricos COM33/COM35 y COM88/COM87 no se presuponen |
| Antena/carga | antenas verificadas para 868 MHz dentro de caja RF apantallada; alternativa conducida sólo con fixture y atenuación calculados, nunca coaxial directo |
| Banda/switch | Cat-Shell `band2` confirmado en ambas; anotación de que status no prueba posición eléctrica |
| Frecuencia | 868000000 Hz |
| Potencia solicitada | 0 dBm |
| Fuente | rama/HEAD indicados en §1; registrar cualquier cambio y hash de scripts |
| Recuperación | acceso físico para retirar todas las fuentes >=15 s antes y después |

### Infraestructura existente

- preflight: `experimental_artifacts/next_rf_validation/preflight.py`;
- etapa OOK: `experimental_artifacts/next_rf_validation/rf05_ook_868.py`;
- control de recuperación: `experimental_artifacts/next_rf_validation/rf01_ieee_channel_switch.py --only-control`;
- logging JSONL: `_common.py`.

No se propone un sweep ni un script nuevo.

### Procedimiento propuesto

Los nombres COM que siguen son los históricos y deben sustituirse si la identificación aprobada produce otros. La aprobación del experimento debe abarcar explícitamente el preflight, la etapa OOK y el control de recuperación.

1. Crear directorio de corrida y guardar `git rev-parse HEAD`, `git status --short`, versión de Python, ruta importada de `feralrf` y hashes SHA-256 de los tres scripts.
2. Retirar toda alimentación de ambas placas >=15 s; reconectar; no abrir otro proceso serial.
3. Ejecutar preflight, que abre los cuatro puertos pero no transmite:

```powershell
python .\experimental_artifacts\next_rf_validation\preflight.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --antenna-confirmed --recovery-confirmed 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '00_preflight.jsonl')
$preflightExit = $LASTEXITCODE
```

4. Continuar sólo si el preflight identifica exactamente ambas placas/firmwares esperados y retorna 0. La procedencia del binario CC1352 debe registrarse como limitación si no puede vincularse a HEAD.
5. Obtener aprobación inmediata final para RF-05. Ejecutar una sola vez:

```powershell
python .\experimental_artifacts\next_rf_validation\rf05_ook_868.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --marker-hex C5050001 --min-received 9 --operator-approved --initial-power-cycle-confirmed 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '05_rf05_ook_868.jsonl')
$ookExit = $LASTEXITCODE
```

6. Tras cualquier resultado, no emitir otro comando RF en esa sesión. Cerrar procesos, conservar log y retirar toda alimentación >=15 s.
7. Reconectar y repetir preflight. Si falla, detener y clasificar recuperación fallida.
8. Ejecutar sólo el control IEEE fresco:

```powershell
python .\experimental_artifacts\next_rf_validation\rf01_ieee_channel_switch.py --only-control --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --min-received 9 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '05b_rf05_recovery_ieee_control.jsonl')
$recoveryExit = $LASTEXITCODE
```

### Respuestas esperadas y aceptación

| Etapa | Control esperado | Criterio |
|---|---|---|
| Configuración | ACK de PHY/config/power; warnings OOK registrados | sólo evidencia de control |
| TX | 10 ACK + cero errores asíncronos | necesario, no suficiente |
| RX | >=9/10 paquetes con `C5050001` y `crc_ok=True` | evidencia peer raw |
| Cleanup | `RX_STOP` y disconnect reportados | si falla, detener/power-cycle |
| Recuperación | power-cycle documentado + preflight + IEEE control PASS | obligatorio para PASS compuesto |

Resultado compuesto:

- `PASS`: etapa OTA satisface todos los criterios **y** recuperación/control posterior PASS.
- `PARTIAL`: recepción OOK positiva, pero falta o falla una parte no destructiva de la recuperación/control.
- `FAIL`: configuración completa, controles basales suficientes, pero cero/menos de nueve marcadores o errores RF atribuibles.
- `BLOCKED`: identidad, banda, antena, puertos, entorno, init o baseline no permiten una ejecución interpretable.
- `INCONCLUSIVE`: evidencia/estado no permite atribución, por ejemplo binario desconocido o cleanup ambiguo.

El script RF-05 retorna 2 incluso si su etapa OTA pasa, porque deliberadamente deja el resultado en `PARTIAL` hasta el control externo. No debe reinterpretarse ese código como FAIL automático.

### Evidencia a conservar

- aprobación y alcance exactos;
- fotos/identidad/revisión y antenas;
- inventario de puertos y respuesta completa de Shell/Bridge;
- HEAD, dirty state, hash de scripts, identidad de firmware;
- stdout/stderr JSONL íntegro de cada etapa;
- `$preflightExit`, `$ookExit`, `$recoveryExit`;
- configuración completa y potencia solicitada;
- todos los ACK, errores asíncronos y paquetes RX;
- hora y duración de ambos power-cycles;
- cleanup y estado final;
- limitación de no usar instrumentación RF independiente.

## 4. Configuraciones sin experimento candidato

- MIOTY: una OTA raw no responde la pregunta de interoperabilidad.
- 2.4 propietario: el firmware transmitiría por la configuración fallback de 868; ensayo inseguro y no interpretable.
- 169 MHz: fuera de rango y ruta incoherente.
- MSK/4-FSK/4-GFSK: valores reservados; prohibido experimentar/remapear.
- 433: falta establecer front-end de la revisión exacta y antena; primero corresponde una revisión documental/medición pasiva del hardware, no TX.

## 5. Gate de decisión

Esta auditoría se detiene aquí. La frase “autorizar RF-05/OOK-868” debe recibirse explícitamente y sólo autoriza el paquete compuesto descrito arriba. No autoriza otra frecuencia, preset, retry, sweep ni cambio de firmware.

