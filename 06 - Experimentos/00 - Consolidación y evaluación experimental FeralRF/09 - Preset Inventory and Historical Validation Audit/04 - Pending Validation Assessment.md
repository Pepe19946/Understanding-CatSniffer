# Fase 1 — Evaluación de validación pendiente

## Decisión por categoría

| Categoría | Presets | Justificación y siguiente acción |
|---|---:|---|
| A — Adecuadamente cubierto para el alcance raw actual | 14 | P09, P10, P13–P15, P20, P21, P24–P26, P28–P31 tienen L2 exacto. Reusar evidencia; no repetir sólo para aumentar PASS. |
| B — Positivo histórico que exige repetición controlada | 0 | Ningún preset queda aquí: los positivos débiles de 433 están excluidos; OOK 868 ya fue repetido el 22-oct y falló/bloqueó. |
| C — No suficientemente probado pero elegible | 0 | No queda una entrada válida, dentro del scope y sin ensayo: las pendientes tienen fallo o bloqueo técnico. |
| D — Fallo previo no resuelto | 1 | P11 `ook_868_4k8`: claim 10/10 histórico frente a OP-MO-06 0/10 peer y cleanup fallido. Antes de otra OTA debe investigarse el lifecycle y observar TX físicamente. |
| E — Bloqueado por implementación/driver | 6 | P12/P18/P19 usan modType reservado; P22/P23 caen a 868; P27 carece de TS-UNB y produjo 0/10 raw. No ejecutar OTA en la implementación actual. |
| F — Excluido/bloqueado por hardware o scope | 10 | P01–P08 son 433 y están fuera de la campaña; P16/P17 son 169 y no son defendibles en este hardware. |

El total es 31 y las categorías son mutuamente exclusivas. Los tres presets reservados de 433 se asignan a F por la exclusión de banda, aunque además tengan bloqueo de driver.

## Configuraciones alrededor de 915 MHz

- A 915.000 MHz solicitados: `gfsk_915_50k`, `sidewalk_915_fsk_50k` y `sidewalk_915_fsk_250k`, cada uno 10/10 en E08/E11/E12.
- A 902.200 MHz solicitados: `gfsk_902_50k`, los cinco `wisun_915_fsk_*`; 10/10 por preset, y `wisun_915_fsk_50k` además 60/60 recovery y 10/10 en dirección inversa RF-02.
- El sufijo `915` de Wi-SUN no coincide con el valor del diccionario; no debe reportarse como prueba a 915 MHz.
- Todos son L2 raw entre CatSniffer, no medida independiente de frecuencia ni interoperabilidad Wi-SUN/Sidewalk.

## Qué no repetir

- Los 14 categoría A no necesitan una nueva corrida de marcadores idénticos. Una repetición futura sólo se justifica si añade identidad binaria, secuencias únicas, dirección/placa distinta, control ambiental o instrumentación.
- P12/P18/P19 no deben probarse: aceptar 4/5/6 no satisface el contrato TI.
- P22/P23 no deben probarse: existe riesgo de operar a 868 MHz cuando se solicita 2440 MHz.
- P16/P17 no deben probarse en este hardware.
- P01–P08 no deben probarse en la campaña actual de 433 excluida.
- P27 no debe repetirse como “MIOTY”; otra prueba raw 396-baud no respondería la pregunta de TS-UNB.

## Prioridades técnicas para una fase posterior

1. Identidad reproducible source→binario→dispositivo; hoy `1.0.0`/`FERALRF1` no distingue builds.
2. Corregir/diagnosticar metadata propietaria y el contrato de respuesta asíncrona/`RX_STOP`.
3. Para OOK 868, inspeccionar completion TX, segundo-TX timeout, estado RX y recuperación; sólo después considerar una repetición aislada con analizador.
4. Medir frecuencia, potencia, modulación/ancho ocupado y estado de switch para al menos un control 868 y uno 902.2/915.
5. Si el proyecto requiere protocolos nominales, definir framing/rol e interoperabilidad por protocolo antes de nuevas OTA.
6. Corregir o rechazar explícitamente 2.4 propietario y valores reservados; no validar alrededor del defecto.

Esto no es un runbook y no autoriza pruebas.

## Auditoría estática de scripts existentes

Clasificación sólo para la validación requerida; ningún script fue ejecutado.

| Script | CLI/puertos/preset | Reset/overrides | ACK, payload y unicidad | Errores/cleanup | Clasificación |
|---|---|---|---|---|---|
| `python/examples/smoke_prop_phase1.py` | `--port`, `--preset`, power/duration/payload/min-packets | Preset exacto; `--auto-reset` opcional | Reporta ACK; hace RX local antes de un TX; no prueba peer ni unicidad | `finally disconnect`; `stop_rx`; auto-reset usa API de puerto derivado | **NOT SUITABLE** para L2; útil sólo L1/control |
| `python/examples/smoke_ota_txrx.py` | `--tx-port`, `--rx-port`, `--preset` o PHY | Resetea ambos con Shell=`Bridge+2`; no Shell explícito | Cuenta ocurrencias de `DEADBEEF`; default PASS con ≥1; marcador repetido | Ignora `RxStreamError`; suprime excepción de `stop_rx`; cleanup no verificable | **NOT SUITABLE** para campaña estandarizada |
| `python/examples/smoke_f29_subg_915.py` | TX/RX explícitos; lista fija de 7 presets; count/min/power/window | Reset Shell=`Bridge+2`; no override de frecuencia | 10 ACK implícitos y marker fijo; exige `crc_ok`; no secuencias únicas | Suprime `stop_rx`; continúa tras excepciones por preset; sin Shell explícito | **CONDITIONALLY SUITABLE** sólo para reproducir claim F29 con mapeo compatible; no setup Windows actual |
| `docs/.../manual_preset_runner.py` | preset/marker/label posicionales; **COM33/COM88 hard-coded** | Preset exacto; sin reset/band/overrides | Log JSONL-like; 10 ACK; marker fijo, no único | Observa errores async TX/RX; stop/cleanup explícito; clasificación puede contar duplicados | **CONDITIONALLY SUITABLE** en el par histórico y con limitación de unicidad |
| `python/examples/lab/ota_rx_probe.py` | RX port/PHY/channel/duration/marker/match/min-hits | Sin preset; receptor genérico | Verifica bytes por criterio configurable; no correlaciona secuencias salvo payload diseñado | Reporta hits/exit; depende de operador/segundo proceso | **SUITABLE WITH EXISTING CLI OPTIONS** para IEEE/manual, no configura presets Sub-GHz |
| `python/examples/smoke_tx_phase1.py` | TX port/PHY/channel/payload/power | Sin preset | Un `TX_RAW` ACK por proceso; no peer | No observa async posterior; disconnect | **CONDITIONALLY SUITABLE** como mitad TX bajo receptor coordinado |
| `python/examples/lab/ota_tx_frame.py` | TX port/PHY/channel/payload/count/interval | Sin preset | Reporta ACK por loop; `TX_FRAME` no construye MAC | No peer/async; disconnect | **CONDITIONALLY SUITABLE** para IEEE raw, no presets |
| `python/examples/lab/ota_tx_burst.py` | TX port/PHY/channel/payload/count/interval | Sin preset | ACK significa burst programado, no `count` emitido | No peer ni completion individual | **NOT SUITABLE** para conteo/PDR; sólo investigación del scheduler |
| `python/examples/smoke_tx_continuous_phase1.py` | TX port/PHY/channel/payload/interval/run | Sin preset | ACK start/stop; no mide paquetes ni cese físico | Intenta stop en `finally`, suprime fallo de cleanup | **NOT SUITABLE** para validación de delivery/stop físico |
| `experimental_artifacts/next_rf_validation/*.py` | Puertos Bridge y Shell explícitos; flags/gates | Soportan preflight, band y escenarios RF-01/02/03/05 | JSONL, markers, criterios y errores async | Cleanup y estados BLOCKED; diseñados para campañas concretas | **SUITABLE WITH EXISTING CLI OPTIONS** sólo para sus escenarios archivados; no runner genérico de 31 presets |
| Harness de `MIOTY_OOK_VALIDATION_RUNBOOK.md` | Puertos/preset/marker por variables; secuencias únicas | Sin `reset_device()`; power-cycle explícito para OOK | Distingue ACK, payload y seq 1–10; preserva metadata | Detectó cleanup y stats; pero OP-MO-07b imprimió PASS pese a timeout `TX_STATS_AFTER` | **CONDITIONALLY SUITABLE**; corregir criterio antes de reutilizar |

### Limitaciones transversales de scripts

- `Radio.reset_device()` y runners que calculan Shell como Bridge+2 son inseguros con COM88/COM87.
- Un `transmit()` que retorna normalmente acredita ACK del comando, no completion RF; errores posteriores pueden ser asíncronos.
- Un marker fijo permite atribución al tipo de corrida, pero no a cada intento. Para PDR se necesitan nonce/contador único y deduplicación.
- `crc_ok`, RSSI, canal y timestamp propietarios no deben ser criterios físicos independientes hasta corregir/verificar el parser.
- Cleanup debe afectar la clasificación final. Suprimir excepciones de `stop_rx()` hace el resultado no apto para la campaña requerida.
- La función temporal `Invoke-FeralPresetOta` usada el 8-oct no es un CLI instalado; sólo sus artefactos/reportes permiten reconstrucción.

## Condiciones previas a cualquier OTA futura

Identidad de placas/firmware, Bridge/Shell por USB y no por aritmética; revisión/antena/ruta adecuadas; autorización de banda/potencia/entorno; ausencia de sesión OOK desde el último power-cycle; payloads únicos; logging completo de stdout/stderr/exit; observación de errores asíncronos; cleanup exigido; y, cuando la afirmación sea física, instrumentación independiente. Cumplir esto habilita una decisión posterior, no constituye autorización en esta fase.

## Corrección: validación no es diagnóstico

“No ejecutar OTA” en las categorías E/F significa **no usar la implementación actual como prueba positiva del claim nominal**. No prohíbe un diagnóstico seguro y deliberado. P06–P08/P12/P18/P19 admiten diagnóstico de contrato/rechazo sin afirmar emisión MSK/4-(G)FSK; P22/P23 admiten caracterización con analizador para detectar el fallback; P27 conserva valor como control negativo. P16/P17 sólo son diagnosticables en un diseño RF 169 MHz adecuado. P11 requiere banco controlado por su lifecycle OOK.

La matriz `03` es la autoridad para las columnas independientes “elegibilidad V” y “elegibilidad D”. Ninguna elegibilidad diagnóstica autoriza emisión ni elimina requisitos regulatorios.

## Changelog

- 2026-10-23: se reemplazó la lectura absoluta “no probar” por elegibilidad separada de validación y diagnóstico; no cambió la evidencia histórica.
