# Auditoría metodológica Codex — campaña OTA Sub-GHz, octubre de 2026

**Revisión:** 2, 2026-10-16 (America/Mexico_City)

**Motivo:** reaudit después de restaurar el manual y el informe experimental que durante la primera revisión se observaron vacíos.

**Alcance:** análisis estático de documentos, código, Git y registros ya existentes. No se abrió ningún puerto, no se ejecutó RF y no se modificaron manual, informe experimental, auditoría causal, scripts, firmware ni matriz oficial.

**Dictamen revisado:** **AUDITABLE A NIVEL DOCUMENTAL, INTERNAMENTE COHERENTE Y CON CUMPLIMIENTO SUSTANCIAL, PERO SÓLO PARCIALMENTE VERIFICADA CON EVIDENCIA PRIMARIA**.

## 1. Nota de revisión y verificación de fuentes restauradas

Ambos documentos fueron leídos directamente del disco; no se utilizaron resúmenes en caché como sustituto.

| Documento | Ruta exacta | Existe / legible | Bytes | Líneas | SHA-256 | Título y estado |
|---|---|---:|---:|---:|---|---|
| Manual | `FeralRF/MANUAL_FERALRF_SUBGHZ.md` | Sí / sí | 50.002 | 2.259 | `334F33999BFF6982DF402DD73A9154FA24A77B2658CD44A373D32A3C10DF24C0` | `Manual FeralRF Sub-GHz OTA Runbook`; contenido sustantivo, no placeholder |
| Informe | `Understanding-CatSniffer/06 - Experimentos/Pruebas OTA SubGHz/Auditoría final de experimentos FeralRF OTA SubGhz.md` | Sí / sí | 60.758 | 988 | `E4DB4C9CE59FC7E019BBF3D168B075C64FC5172739C3757EF40BD6C361315E8B` | `Informe técnico de validación OTA Sub-GHz — FeralRF`; contenido sustantivo, no placeholder |

El manual contiene quince secciones principales: modelo de estado, preparación del PC, estado Shell, cuatro modos de preparación, cleanup, control de un paquete, validación formal de diez paquetes, presets elegibles, lista de exclusión, comparación histórica, plantilla por preset, controles, criterios de aceptación, plantilla de resultado y checklist final. Sus encabezados están escritos como texto en negrita escapado (`**## ...**`), no como Markdown canónico, pero son legibles.

El informe contiene resumen, objetivos, fuentes/procedencia, plataforma, arquitectura, metodología, resultados, controles, incidentes, anomalías, contexto histórico, conclusiones y una matriz E01–E24. Está seguido en Git y coincide con el blob de `Understanding-CatSniffer` HEAD `19cca8cefb5bb7ab099eb03320b5aa93f3385554`; su último commit es ese mismo SHA (`docs: Auditorías de resultados obtenidos tras la segunda campaña de pruebas OTA SubGHz.`). El manual sigue **untracked** en `FeralRF` y no tiene procedencia Git.

El informe declara como evidencia primaria `Experimentos FeralRF OTA SubGHz.txt`, SHA-256 `1a6694430fd6f036437e7ce55501bba57e74e2e66deb1e547b0c8f68053d41d1` (`informe:102-120`). Ese TXT no existe en ninguno de los repositorios ni en el workspace. Por ello se aplican cuatro niveles de fuerza:

1. **Log crudo accesible:** seis checkpoints preliminares 3A–4A.
2. **Transcripción resumida y fechada:** E01–E24 en el informe restaurado.
3. **Agregado derivado:** totales recalculables desde E01–E24.
4. **No verificable:** campos por paquete, stdout/stderr, exit codes y cleanup de E01–E24 que no fueron reproducidos en el informe.

## 2. Dictamen ejecutivo actualizado

La restauración resuelve el principal impedimento de la primera auditoría: ahora sí se conocen las reglas del runbook y el informe presenta una cronología concreta de 24 intentos, con timestamps, presets, etiquetas abreviadas, ACK, recepciones y clasificaciones. La aritmética es coherente: 19 PASS + 3 BLOCKED no inducidos + 2 BLOCKED inducidos = 24; 19 × 10 = 190 objetos RX válidos según la implementación; E23 añade cinco, para 195 eventos reportados.

El informe soporta documentalmente que los trece presets tuvieron al menos una corrida 10/10 y que hubo cinco controles positivos completos y una repetición adicional. No obstante, la ausencia del TXT citado y de logs independientes de `manual_preset_runner.py` impide elevar esos resultados a verificación primaria corrida por corrida. La formulación defendible es:

> La campaña de trece presets está **documentada como completada y es auditable en consistencia y método**; no está **independientemente verificada en su totalidad desde stdout/stderr primario**.

El procedimiento cumplió los elementos funcionales centrales: roles COM fijos, sesiones frescas, PHY/preset simétricos, 0 dBm solicitado, RX antes de TX, diez solicitudes, marcadores, stats, observación de errores y clasificación separada de BLOCKED. No tuvo cumplimiento literal pleno: no se documentó el modo A/B/C/D por nombre ni la verificación de import en cada sesión; se omitió un control tras el noveno preset; B02 y B03 fueron reintentados antes del sentinel recomendado; las dos contrapruebas RESET se agruparon antes de un único control; se reutilizaron labels; el estado final y cleanup por corrida no son comprobables; y no se preservó en el workspace el TXT primario.

El RESET CC y `band2` **no son requisitos universales del manual**. El modo A preferido prohíbe ambos (`manual:187-233`); los modos B–D son controles opcionales. La preparación descrita por el informe equivale aproximadamente al modo D, pero no registra ese nombre ni todos sus tiempos. La medición eléctrica CTF/U2 y un receptor RF independiente quedan fuera del PASS funcional; sólo serían obligatorios para afirmar ruta física, frecuencia, potencia o forma de onda.

## 3. Procedencia de repositorio, scripts y firmware

### Estado actual y reutilización de la auditoría previa

- `FeralRF`: rama `feature/hardware-validation`, HEAD `ba27fbeb8c5683a31cca4339982cadeaaa406444`, detrás de `origin` por un commit; no hay cambios en archivos seguidos. Manual, auditorías Sub-GHz y `experimental_artifacts/` son untracked.
- `manual_preset_runner.py`: 9.877 bytes, SHA-256 `1F5D0862872773A575C7F355F20571171C1BA37C5B0ED75B837C46204EAB91A2`.
- `checkpoint_3a_single_run.py`: 8.624 bytes, SHA-256 `485E3989D448F752B4E2F22A4EDE8C40A2C88447CB39C76E5E4BBB351074C842`.
- Esos hashes coinciden con los registrados en la primera auditoría. No se observaron cambios seguidos en el host Python o firmware CC relevantes; por tanto, se conservan los hallazgos de implementación previos.
- El informe restaurado sí está seguido en el Vault; corrige la afirmación previa de que era un archivo untracked.

El HEAD identifica la fuente seguida, pero no los helpers untracked ni los binarios instalados. Los strings RP `v3.1.0.1`, CC `feralrf_cc1352 (custom)` y API `1.0.0`/capabilities 7 no constituyen correspondencia binario↔commit (`informe:185-201`). Siguen faltando hashes/build IDs de RP2040 y CC1352.

### Capas de ejecución preservadas

- **Host Python:** `Radio.init`, `set_phy`, `set_power`, `configure_prop`, `get_stats`, `start_rx`, `stop_rx`, `read_packets` y `transmit` están en `FeralRF/python/feralrf/radio.py` (funciones alrededor de líneas 399, 455, 480, 490, 553, 599, 609, 636 y 725).
- **Cat-Bridge RP2040:** CDC0 reenvía PC↔CC1352; `CatSniffer-Firmware/RP2040/catsniffer/src/main.c:202-260`.
- **Cat-Shell RP2040:** `boot`, `exit`, `band1`, `band2` y `status`; `shell_commands.c:193-218,235-267`.
- **CC1352:** comandos en `FeralRF/firmware/cc1352/src/command_processor.c:101-223`; ejecución posterior en `control_task.c`, `data_task.c` y `radio_if.c`.
- **Switch RF:** `band2` solicita `SUBGIG_1` y el RP pretende CTF1/2/3 = 0/0/1 (`main.c:379-402`). `status` sólo informa estado cacheado, no mide GPIO o U2.

## 4. Procedimiento reconstruido

### Preparación reportada

El informe describe: repositorio y `PYTHONPATH`; rama/HEAD/Python; enumeración USB; `status` en COM35/COM87; `boot`→`exit`; `band2`; segundo `status` con `Band:1`; control inicial; adaptación y comprobación sintáctica del runner (`informe:328-375`). Esto es compatible con el **modo D** del manual (`manual:559-601`), aunque no se registró como tal ni se reproducen los intervalos 0,5/0,3/≥3,5 s exigidos por ese modo.

### Ejecución por preset

`manual_preset_runner.py:23-40,52-300` recibe preset/marcador/label, fija COM33→COM88, crea objetos, inicializa TX/RX, configura `PROPRIETARY_GFSK` canal 0 y el mismo diccionario, solicita 0 dBm, toma stats, inicia RX, espera 0,3 s, hace diez `transmit()` separados 0,1 s, espera 1 s, drena RX 2 s, observa errores TX 0,5 s, detiene RX, obtiene stats y desconecta. Este flujo coincide con `informe:377-410` y el timing formal del manual (`manual:955-985,1809-1845`).

### Cronología y controles

- E01 control inicial.
- P01–P03; E05 control.
- P04–P06; E09 BLOCKED; E10 control de recuperación.
- P07–P10; E15 repetición BLOCKED; E16 repetición PASS; E17 control.
- E18 P11 BLOCKED; E19 reintento PASS; P12–P13.
- E22 RESET RX; E23 RESET TX; E24 control final.

Los seis logs crudos 3A, 3A-R, 3C, 3D, 3E y 4A son **anteriores y distintos**: empiezan entre 16:52 y 18:57 UTC. E01 comienza a las 21:08 UTC (`informe:879-904`). No deben contarse como los cinco controles y la repetición de E01–E24; son evidencia primaria adicional de recuperación sobre `wisun_915_fsk_50k`.

## 5. Matriz de cumplimiento del manual

Definiciones: **VERIFIED** = evidencia primaria accesible o condición directamente comprobable; **PARTIALLY VERIFIED** = informe detallado/código, pero falta transcripción primaria; **NOT VERIFIED** = no hay evidencia accesible suficiente; **DEVIATION** = diferencia literal respecto del runbook; **NOT APPLICABLE** = no requerido para el alcance funcional declarado.

| Manual requirement | Required condition | Evidence from experimental report | Supporting implementation/log | Verdict | Notes |
|---|---|---|---|---|---|
| Identificar unidades y roles | #2 TX COM33/35; #3 RX COM88/87; sin aritmética | Tabla de roles e IDs USB (`informe:163-183`) | Logs 3A–4A confirman Bridge COM33/88 | **PARTIALLY VERIFIED** | Shell/USB de E01–E24 depende del TXT ausente |
| Preparar Python e importar el checkout | Directorio, `PYTHONPATH`, `feralrf.__file__` y assert (`manual:63-135`) | Directorio, Python 3.14.7 y `PYTHONPATH` declarados (`informe:122-139,328-346`) | Runner usa `parents[2]` y apunta a un `workspace/python` inexistente (`manual_preset_runner.py:17-20`) | **DEVIATION** | Funcionó por instalación editable; identidad cargada por corrida no quedó preservada |
| Registrar `status` inicial | Mode, Band, Radio, LoRa, FW y contadores (`manual:139-181`) | Se afirma consulta de ambos Shell; sólo parte de campos se resume | Status completos existen para checkpoints preliminares, no E01–E24 | **PARTIALLY VERIFIED** | Faltan salidas literales y counters de la campaña |
| Seleccionar y registrar modo A/B/C/D | Un modo explícito antes de probar (`manual:185-601,2197`) | Se describe reset + banda, equivalente a D | No hay campo “Preparation mode” de E01 | **PARTIALLY VERIFIED** | No es válido convertir reset/band2 en obligación universal |
| `boot`/`exit` | Obligatorio sólo en modos B/D; respuestas y settling | `BOOT`/`PASSTHROUGH` reportados (`informe:328-350`) | Funciones RP/CC confirmadas por código | **PARTIALLY VERIFIED** | No hay transcript ni tiempos accesibles |
| `band2` y verificación lógica | Obligatorio sólo en C/D; `SUB-GHz Band` y `Band:1` | Confirmado inicialmente en ambos (`informe:263-285`) | RP pretende CTF 0/0/1 | **PARTIALLY VERIFIED** | El 16-oct y post-RESET carecen de salida literal (`informe:688-698`) |
| Verificación eléctrica del switch | Sólo necesaria para afirmar GPIO/U2/ruta física | El informe declara que no se midió | Manual delimita expresamente la evidencia (`manual:531-557`) | **NOT APPLICABLE** | **NOT VERIFIED** para una afirmación eléctrica, que el informe no hace |
| Objetos `Radio` frescos e `init()` | Ambos endpoints en cada corrida | Flujo descrito para cada ejecución | Implementado en `manual_preset_runner.py:67-83` | **PARTIALLY VERIFIED** | Código sí; ejecución individual depende del TXT |
| PHY/preset simétricos | `PROPRIETARY_GFSK`, canal 0 y mismo diccionario | Metodología y tabla P01–P13 (`informe:377-450`) | `manual_preset_runner.py:85-99`; presets en `presets.py:81-136,171-185,205-277` | **PARTIALLY VERIFIED** | Los trece diccionarios del informe coinciden con fuente actual |
| Inspección independiente del preset | Imprimir y comparar antes de invocar (`manual:1847-1857,2201`) | `TEST_BEGIN` habría impreso el diccionario; no se documenta impresión independiente | Runner imprime al empezar, no antes | **NOT VERIFIED** | No afecta la simetría implementada, sí la trazabilidad requerida |
| Potencia y stats iniciales | 0 dBm solicitado; stats TX/RX | Metodología y matriz resumen | `manual_preset_runner.py:101-111` | **PARTIALLY VERIFIED** | ACK/petición no mide potencia física; stats literales ausentes |
| RX antes de TX y timing | 0,3 s; 10 TX; 0,1 s; 1 s; lectura 2 s; TX async 0,5 s | Secuencia exacta (`informe:377-410`) | `manual_preset_runner.py:113-206` | **PARTIALLY VERIFIED** | Implementación verificada; timestamps por evento no accesibles |
| Diez TX y sin retry automático | Una corrida por proceso, diez solicitudes | E01–E24 distingue intentos y repeticiones | Runner no reintenta (`:123-144,298-304`) | **PARTIALLY VERIFIED** | Las repeticiones son procesos separados; faltan comandos/exit codes |
| Marcador y label trazables | Marcador único por preset/run; label identificable (`manual:1499-1545,1773-1807`) | Marcadores P01–P13; labels repetidos reconocidos (`informe:671-686`) | Runner no impone unicidad | **DEVIATION** | E14–E16 y E21–E23 reutilizan respectivamente el mismo label; unicidad de marcador en repeticiones no es verificable |
| Registrar Packet, CRC, marker y errores | Todos los eventos y `RxStreamError` | Totales y anomalías agregadas, no eventos literales | Runner los emite (`:151-206`) | **PARTIALLY VERIFIED** | 190/195 no son recontables sin TXT |
| Clasificar PASS/PARTIAL/FAIL/BLOCKED | PASS formal 10/10; FAIL sólo tras setup normal; fatal = BLOCKED | E09/E15/E18 y E22/E23 correctamente separados de FAIL | Runner prioriza fatal (`:263-299`) | **PARTIALLY VERIFIED** | E15 falla al construir `Radio` fuera del `try`, por lo que BLOCKED es clasificación documental, no necesariamente salida del runner |
| Cleanup | `stop_rx`, desconectar y conservar errores (`manual:605-631,2247-2249`) | Informe afirma finalización correcta como criterio, sin detalle por run | `finally` existe (`runner:225-261`) | **PARTIALLY VERIFIED** | Construcción está fuera del `try`; errores de disconnect no invalidan PASS |
| Control inicial y final | Sentinel al inicio y cierre (`manual:1861-1879`) | E01 y E24 PASS | Cronología del informe | **PARTIALLY VERIFIED** | Resumen fechado, sin stdout primario |
| Control tras cada tres presets | Calendario **recomendado** | Tras 3 y 6; después se ejecutaron cuatro (#7–#10) (`informe:512-518`) | E05/E10/E17 | **DEVIATION** | No es requisito obligatorio; el informe reconoce la desviación |
| Control tras BLOCKED | Recomendado inmediatamente (`manual:1871`) | B01→E10 sí; B02→E16 y B03→E19 reintentan primero | E09–E19 | **DEVIATION** | El PASS posterior no restaura retrospectivamente el orden recomendado |
| Control tras cada RESET | Recomendado después de cualquier experimento RESET (`manual:1873`) | E22 y E23 seguidos por un solo E24 | E22–E24 | **DEVIATION** | Adecuado como recuperación final; no hay sentinel entre contrapruebas |
| Reconexión/power boundary | Si se declara cold start: retirar toda energía ≥10 s, reenumerar, status (`manual:667-713`) | Se reporta desconexión/reconexión USB antes de E15–E17 | No hay duración, identidad ni status literal | **PARTIALLY VERIFIED** | No puede llamarse power-cycle completo documentado |
| RESET físico contraprueba | Diagnóstico opcional, no requisito del PASS de 13 presets | E22/E23 y sus excepciones descritas | Sin log crudo | **PARTIALLY VERIFIED** | **NOT APPLICABLE** para decidir si la campaña principal se completó |
| Estado final | Restaurar o registrar estado lógico/power (`manual:2251-2255`) | No hay status final literal | Ningún log E24 accesible | **NOT VERIFIED** | No invalida paquetes ya observados; limita reproducibilidad |
| Preservación de evidencia | Resultado por run con configuración, stats, errores y cleanup (`manual:2037-2175`) | Informe conserva resumen y hash del TXT | TXT y logs de runner ausentes; `.log` ignorados por Git | **DEVIATION** | Principal limitación restante |
| Sólo presets elegibles | Excluir OOK/MIOTY/169/433/MSK/4FSK/4GFSK | P01–P13 están en lista elegible | Coinciden con `manual:1549-1683` | **VERIFIED** | Verificado a nivel de inventario del informe |
| No sobregeneralizar protocolo/RF | PASS raw funcional, no interoperabilidad/espectro | Límites explícitos (`informe:44-60,82-94,295-322`) | Manual `:1997-2033` | **VERIFIED** | “CRC-good” debe seguir leyéndose como valor reportado por firmware |
| Antenas y revisión hardware | No es requisito expreso del runbook | Antenas declaradas; distancia y medición ausentes (`informe:203-219`) | Sin revisión PCB/BOM por unidad | **NOT APPLICABLE** | Sigue siendo brecha para comparación forense física, no para PASS funcional |

No falta una medición de hardware obligatoria dentro del alcance funcional declarado. Sí quedan sin demostrar requisitos documentales/operativos del checklist: identidad exacta del import, modo/timing completo, preset impreso independientemente, cleanup por corrida, estado final y registro primario íntegro.

## 6. Auditoría de resultados y clasificaciones

### Reconciliación del agregado

| Claim | Recuento independiente desde E01–E24 | Fuerza de evidencia | Dictamen |
|---|---:|---|---|
| 13 presets nuevos | P01–P13, cada uno con un PASS 10/10 | Tabla detallada del informe; sin stdout | **Internamente verificado / primariamente no verificado** |
| 5 controles positivos | E01, E05, E10, E17, E24 | Cronología y tabla de controles | **Internamente verificado** |
| 1 repetición adicional | E16 | Cronología; E15 fue BLOCKED | **Internamente verificado** |
| 19 PASS | 13 + 5 + 1 = 19 | Derivado de la cronología | **Aritméticamente correcto** |
| 190 objetos con marcador y `crc_ok=True` | 19 × 10 = 190 | Agregado; eventos no accesibles | **Correcto si la transcripción es fiel; no recontable** |
| 3 BLOCKED no inducidos | E09, E15, E18 | Punto de fallo y excepción resumidos | **Consistente** |
| 2 BLOCKED inducidos | E22 RX reset, E23 TX reset | Resumen; E23 añade 5 paquetes | **Consistente** |
| 195 eventos totales reportados | 190 + 5 de E23 | Agregado | **Aritméticamente correcto; no recontable** |

Los parámetros P01–P13 coinciden con `PROP_PRESETS` actual, incluidas frecuencias efectivamente solicitadas: los `wisun_915_fsk_*` usan 902,2 MHz; los Sidewalk-named usan 915 MHz; los Wireless M-Bus-named usan 868,3/868,95 MHz. Esto valida la tabla contra fuente, no la señal física ni la versión exacta ejecutada.

La corrección Wireless M-Bus del informe es correcta: `1413283840 = 0x543D0000`; `0x543C0000 = 1413218304` (`informe:633-659`). Se detectó, además, un error distinto en el manual restaurado: `manual:107-113` dice que `2466996702` es `0x930B51DE`; en realidad `0x930B51DE = 2466992606` y `2466996702 = 0x930B61DE`. Los logs crudos 3A y `presets.py` usan `2466992606`. Este error documental no cambia las corridas que obtuvieron el valor desde `PROP_PRESETS`, pero debe corregirse antes de usar el bloque “Expected reference dictionary” como comparación literal.

Las clasificaciones BLOCKED están metodológicamente justificadas: E09 y E18 fallaron antes de RX/TX por `get_stats`; E15 no abrió COM33; E22/E23 fueron interrumpidas por RESET. No deben recodificarse como FAIL-RF. E23 conserva correctamente sus cinco entregas sin convertirse en PASS/PARTIAL porque la comparación fue intencionalmente interrumpida.

Los labels duplicados están transparentemente reconocidos y los IDs documentales E01–E24 resuelven la referencia dentro del informe. No resuelven la ambigüedad de archivos crudos inexistentes ni cumplen retroactivamente la política de labels/marcadores únicos.

## 7. Hallazgos de implementación que permanecen válidos

1. **ACK TX no es transmisión RF verificada.** `command_processor.c:210-223` responde después de encolar; `ControlTask_processTxRaw()` ejecuta después (`control_task.c:332-357`). La recepción peer con marcador es evidencia funcional más fuerte, pero no medición RF independiente.
2. **Import path incorrecto en el runner.** `manual_preset_runner.py:17-20` sube dos niveles desde `experimental_artifacts` y añade `catsniffer-feralrf/python`, inexistente. La ejecución dependió de un editable install no fijado en el log accesible.
3. **Clasificación incompleta.** El runner sí exige diez ACK y cero errores async para PASS, pero marca cero hits como FAIL aun con TX incompleto si no hubo fatal (`:275-289`); ignora errores de cleanup al clasificar y agrupa PARTIAL/FAIL/BLOCKED en exit code 2. La construcción de `Radio` ocurre fuera del `try` (`:67-76`).
4. **`channel=37` no representa el canal solicitado.** `RadioIF_setPhy()` conserva `s_channel` cuando `channel == 0` (`radio_if.c:1944-1955`) y el parser lo publica. El informe hace bien en no interpretarlo como canal RF.
5. **LQI = 0 es hardcoded** en proprietary (`RadioIF_processPropPackets`, `radio_if.c:1735`), no una medición.
6. **RSSI/timestamp/CRC probablemente están desalineados.** `RadioIF_processPropPackets()` interpreta los seis bytes finales como status/RSSI/timestamp (`radio_if.c:1719-1735`), mientras la configuración solicita RSSI, timestamp y status (`:1639-1641`). Esto es una hipótesis fuertemente soportada por la fuente actual, no identidad runtime demostrada porque falta el hash del binario.
7. **Stats y CRC no son controles independientes.** `rx_ok` se incrementa desde el mismo `crc_ok` parseado (`radio_if.c:1738-1751`). Por ello “190 CRC-good” significa 190 flags reportados por la implementación, no 190 CRC recalculados independientemente.

El informe trata adecuadamente `channel`, LQI, RSSI y timestamp como anomalías abiertas (`informe:605-631`) y limita frecuencia, modulación, potencia y protocolos (`:44-60,82-94`). Su terminología “paquetes válidos” sólo es aceptable bajo esa definición explícita de validez funcional/firmware-reportada.

## 8. Limitaciones restantes

- Falta el TXT primario cuyo hash cita el informe y faltan stdout/stderr/exit codes por E01–E24. Esto impide revisar cada `TEST_BEGIN`, `RX_PACKET`, stats, excepción y cleanup.
- Manual y helpers siguen fuera de Git; no existe manifest contemporáneo que ligue E-ID→comando→hash de script→log→exit code.
- El manual parece construido a partir de la recuperación y no tiene historia Git; no puede demostrarse que fuera un protocolo preregistrado antes de E01.
- No hay hashes/build IDs de los binarios instalados ni equivalencia exacta firmware↔commit.
- No se identificó revisión PCB/BOM/rework por unidad, distancia/orientación exacta, alimentación ni geometría. El operador declaró antenas compatibles; no hubo caracterización.
- No se midieron CTF/U2, frecuencia, potencia, desviación o forma de onda. Son límites de atribución física, no fallos del alcance funcional.
- La semántica de CRC/RSSI/timestamp requiere resolver el layout y vincularlo al binario antes de usar esos campos como métricas físicas.

## 9. Comparación con el dictamen metodológico anterior

### A. Hallazgos resueltos por la restauración

- El manual y el informe ya no están vacíos.
- Se puede distinguir qué era obligatorio, recomendado, opcional y externo al alcance.
- P01–P13, E01–E24, parámetros, timestamps, controles, bloqueos, labels duplicados y limitaciones están documentados.
- El agregado 13/5/1/19/190/3/2 es internamente reconciliable.
- `band2` y `Band:1` iniciales, antenas declaradas, incidentes y control post-RESET ya tienen soporte documental.
- La discrepancia Wireless M-Bus puede cotejarse y la conversión del informe es correcta.

### B. Hallazgos que permanecen válidos

- Los seis checkpoints crudos sólo prueban sus propias corridas.
- ACK≠emisión RF; peer packet≠validación instrumental; `crc_ok` no es independiente.
- Import path, semántica de clasificación/cleanup y metadata proprietary siguen problemáticos.
- Faltan binarios instalados, medición del switch y condiciones físicas completas.
- Los 13 presets no tienen logs primarios accesibles y la trazabilidad es inferior a la de 3A–4A.

### C. Correcciones a la primera auditoría

| Antes | Después de restaurar |
|---|---|
| Manual/reporte vacíos y no auditables | Ambos sustantivos; informe además seguido en Git |
| Trece presets eran sólo un plan | El informe documenta trece PASS; sigue faltando el transcript primario |
| Seis raw podían representar “cinco controles + repetición” | Son checkpoints preliminares distintos por timestamp; E01–E24 comienza después |
| Reset y `band2` tratados como posibles requisitos generales | Son modos opcionales; Mode A los omite expresamente |
| Verificación eléctrica considerada potencialmente requerida | No aplica al PASS funcional; sólo a una afirmación física |
| Labels duplicados no podían determinarse | El informe los identifica y añade E01–E24, aunque la desviación persiste |
| Dictamen “no demostrada como campaña completa” | Campaña **documentada como completa**, pero **no verificada primariamente en su totalidad** |

## 10. Impacto sobre la auditoría causal histórica

La conclusión causal principal **no cambia**: no se estableció una causa raíz definitiva y RESET, banda, antena o firmware no deben declararse causa confirmada.

Cambios dirigidos que sí merece la auditoría causal:

- **Corregir fuentes ausentes:** manual e informe ya están disponibles; sólo sigue ausente el TXT/logs primarios.
- **Expandir comparaciones:** el informe permite pares documentales históricos↔octubre para `gfsk_868_50k`, `gfsk_868_100k`, `gfsk_915_50k`, Sidewalk-named y Wireless M-Bus-named. Su fuerza es menor que HF-20↔3A–4A porque no hay stdout actual.
- **Fortalecer moderadamente H11 (4 vs 6 bytes):** el runner exige marcadores de cuatro bytes (`manual_preset_runner.py:30-38`) y el informe documenta éxito en las trece configuraciones. La correlación se amplía, pero sigue sin A/B idéntico ni logs primarios; permanece **PLAUSIBLE**, no causa confirmada.
- **RESET:** trece presets sin reset entre corridas fortalecen que no es necesario resetear cada preset. La preparación inicial sí incluyó `boot/exit`, por lo que no prueba que un reset inicial fuera irrelevante ni causal.
- **Banda:** se fortalece que hubo selección lógica inicial, pero el estado del 16-oct/post-RESET y U2 físico siguen abiertos.
- **Recuperación:** E22/E23→E24 documenta recuperación en una sesión nueva; no recuperación automática. B02/B03 muestran que reintento/control funcionó, pero el orden se apartó del runbook.

Se recomienda un **addendum dirigido** a la auditoría causal para corregir sus secciones de disponibilidad y comparabilidad; no es necesario repetirla completa. Su ranking causal y “causa raíz no establecida” siguen defendibles.

## 11. Conclusiones y acciones mínimas

1. La campaña de trece presets es ahora **auditable documentalmente** y su resumen es internamente correcto.
2. El cumplimiento del runbook es **sustancial, no pleno**. Las desviaciones de controles/recuperación son respecto de un calendario recomendado; no invalidan automáticamente los PASS precedentes. Las brechas obligatorias más importantes son trazabilidad primaria, verificación del import, registro de modo/estado, preset independiente, cleanup y estado final.
3. La conclusión funcional defendible es entrega host-visible de marcadores entre dos CatSniffer bajo trece diccionarios solicitados. No es validación independiente de CRC, frecuencia, modulación, potencia, protocolo o switch eléctrico.
4. Como baseline histórico, octubre es **apto con niveles de evidencia**: 3A–4A para comparación primaria exacta; E01–E24 para ampliar comparaciones documentales. No se deben mezclar ambos conjuntos ni presentar los 190 como recontados desde logs accesibles.
5. Acciones mínimas, sin repetir hardware: recuperar el TXT por su hash; preservar logs/exit codes y crear un manifest E01–E24; registrar hashes de binarios/entorno si ya existen; y corregir el decimal erróneo `2466996702` del manual. Sólo una atribución física futura justificaría mediciones CTF/U2 o RF.

## 12. Addendum de cierre por evidencia primaria (2026-10-16)

Este addendum revisa únicamente las conclusiones que dependían de la ausencia del transcript original. La evidencia de implementación, las limitaciones de provenance y las reservas sobre validación RF independiente de las secciones anteriores se conservan salvo corrección expresa.

### 12.1 Verificación del transcript incorporado

| Propiedad | Resultado |
|---|---|
| Ruta exacta | `Understanding-CatSniffer/06 - Experimentos/Pruebas OTA SubGHz/Experimentos FeralRF OTA SubGHz.md` |
| Acceso directo | Archivo existente y legible desde disco |
| Tamaño | 466.343 bytes |
| Líneas | 8.777 |
| SHA-256 | `1a6694430fd6f036437e7ce55501bba57e74e2e66deb1e547b0c8f68053d41d1` |
| Comparación con el TXT citado | Coincidencia exacta; no sólo equivalencia normalizada |
| Formato | UTF-8 sin BOM, finales CRLF; cambió la extensión, no los bytes |
| Cobertura | Contiene las 24 ejecuciones E01–E24 del 15–16 de octubre |

El archivo conserva los comandos, salidas y eventos originales, pero también el diálogo y análisis retrospectivos de la sesión. Para el recuento se trataron como evidencia de ejecución sólo las líneas de eventos con timestamp emitidas por los helpers y las salidas de comandos preservadas; las explicaciones posteriores no se contaron como eventos nuevos. La preparación está documentada en las líneas 52–70, 132–166, 328–336, 401–466, 507–543, 576–632 y 676–702. La secuencia E01–E24 ocupa las líneas 1242–8772.

### 12.2 Reconciliación independiente de E01–E24

| Conjunto | IDs | Resultado derivado del transcript |
|---|---|---|
| Trece presets elegibles | E02, E03, E04, E06, E07, E08, E11, E12, E13, E14, E19, E20, E21 | 13 PASS |
| Controles positivos | E01, E05, E10, E17, E24 | 5 PASS |
| Repetición adicional | E16 | 1 PASS |
| Bloqueos no inducidos | E09, E15, E18 | 3 BLOCKED |
| Bloqueos con RESET inducido | E22, E23 | 2 BLOCKED |
| Total | E01–E24 | 19 PASS, 5 BLOCKED; 0 PARTIAL y 0 FAIL |

En los 19 PASS hay exactamente 190 `TX_ACK` y 190 `RX_PACKET`; los 190 objetos llevan el marcador esperado y `crc_ok=True`. En cada PASS, las estadísticas RX pasan de `rx_ok=0` a `rx_ok=10`, permanecen en cero `rx_crc_err`, `rx_drop` y `rx_overflow`, no hay error asíncrono ni excepción fatal, y constan `RX_STOP_ACK` y desconexión completa de ambos extremos. E01 usa el helper de checkpoint y no emite `TEST_CLASSIFICATION`, pero sus diez ACK, diez paquetes conformes, estadísticas, ausencia de errores y cleanup satisfacen los criterios de PASS del manual.

Los eventos BLOCKED también concuerdan con el informe:

- E09: timeout en `TX get_stats`, sin TX ni RX.
- E15: fallo al abrir COM33 durante `tx.init()`, sin TX ni RX.
- E18: timeout en `RX get_stats`, sin TX ni RX.
- E22: diez ACK TX y cero paquetes entregados después del RESET físico del RX; excepción serie y cleanup RX degradado.
- E23: cinco ACK y cinco paquetes conformes antes del RESET físico del TX; después, cinco excepciones de intento TX y excepción fatal asíncrona.

Por tanto, el transcript contiene 195 objetos `RX_PACKET` en total: 190 pertenecen a ejecuciones PASS y cinco a E23 BLOCKED. También contiene 205 ACK TX: 190 en PASS, diez en E22 y cinco en E23. Esta distinción respalda exactamente el agregado del informe y evita inferir recepción a partir de ACK.

No se hallaron discrepancias entre los timestamps de `TEST_BEGIN`, el orden E01–E24 y la matriz del informe. La conversión raw `1413283840 = 0x543D0000` es correcta; algunas explicaciones retrospectivas del transcript escriben `0x543C0000`, pero no alteran el valor configurado en los eventos ni la corrección ya aplicada en el informe.

### 12.3 Cumplimiento que cambia con la evidencia primaria

| Requisito previamente abierto | Dictamen final | Evidencia y reserva |
|---|---|---|
| Identidad TX/RX, COM y grupos USB | VERIFIED | Enumeración y asignación preservadas en líneas 132–166; TX COM33/35 y RX COM88/87. |
| Estado inicial, `boot`/`exit`, tiempos y contadores | VERIFIED | Comandos y respuestas en líneas 328–543. |
| `band2` y verificación lógica `Band:1` | VERIFIED para la preparación inicial | Comandos y respuestas en líneas 576–702. No consta la salida literal de una nueva selección el 16-oct o después de RESET; Mode A no la exige por preset. |
| PHY, preset, potencia y orden RX→TX | VERIFIED | Eventos `TEST_BEGIN`, `CONFIGURE`, `START_RX`, ACK y lectura de paquetes para los 19 PASS. |
| Diez envíos, marcadores, flags CRC y estadísticas | VERIFIED como observación de firmware/host | 190/190 en PASS. No equivale a CRC, frecuencia o modulación verificados con instrumentación independiente. |
| Errores, excepciones y cleanup por corrida | VERIFIED | Eventos primarios y tracebacks preservados para PASS y BLOCKED. |
| Evidencia primaria E01–E24 | VERIFIED | Transcript de bytes idénticos al archivo citado por el informe. |
| Inspección independiente del preset | PARTIALLY VERIFIED | Hay impresión directa de `PROP_PRESETS` para 12/13 presets; falta la impresión independiente de `wireless_mbus_t_868`, aunque E20 registra los parámetros realmente solicitados. |
| Identidad del paquete Python importado | NOT VERIFIED | No consta `feralrf.__file__` ni equivalente. Sigue vigente el problema de ruta de `manual_preset_runner.py`; el transcript sí registra `PYTHONPATH` en la sesión inicial. |
| Cadencia de controles y sentinela inmediato | DEVIATION | Se probaron P7–P10 antes de E17; E15→E16 y E18→E19 reintentaron antes del control conocido; E22/E23 comparten un único control posterior E24. El manual presenta esta cadencia como recomendada, no como criterio formal de PASS. |
| Labels y marcadores únicos | DEVIATION | E14–E16 y E21–E23 reutilizan labels; las repeticiones reutilizan marcadores. Los IDs E01–E24 permiten desambiguar, pero no eliminan la desviación. |
| Estado Shell final y provenance binaria | PARTIALLY VERIFIED / NOT VERIFIED | No se preservó un status final equivalente ni hashes de binarios instalados; tampoco revisión/BOM completa de hardware. |

Corrección expresa a la sección anterior: E15 no falló al construir `Radio` fuera del `try`. El traceback primario (líneas 6595–6603) sitúa el fallo en `tx.init()`/`connect()` y el runner lo capturó, emitió `TEST_FATAL_EXCEPTION`, `BLOCKED` y exit code 2. La construcción fuera del `try` sigue siendo un riesgo teórico del helper, pero no explica E15.

La acción anterior “recuperar el TXT por su hash” queda resuelta por el archivo Markdown byte-idéntico. También queda resuelta la afirmación de que sólo seis checkpoints tenían evidencia primaria. Permanecen abiertas la identidad del import, los hashes de firmware instalado, las condiciones físicas completas y las desviaciones de cadence/labels.

### 12.4 Dictamen de cierre funcional

**Dictamen final: CLOSED WITH LIMITATIONS.**

Los trece presets elegibles alcanzaron al menos una ejecución que cumple los criterios funcionales formales del manual: diez ACK TX, diez objetos RX con marcador esperado y `crc_ok=True`, incremento coherente de `rx_ok`, ausencia de errores asíncronos/fatales y cleanup completo. Los controles E01, E05, E10, E17 y E24, junto con la recuperación documentada de los bloqueos, son suficientes para sostener la conclusión funcional, aunque no siguieron íntegramente la cadencia recomendada.

| Ámbito | Estado | Alcance de la conclusión |
|---|---|---|
| OTA funcional entre dos CatSniffer | CLOSED WITH LIMITATIONS | Entrega host-visible de los marcadores para los trece diccionarios solicitados. |
| Caracterización RF física | Abierto, fuera del cierre funcional | Frecuencia radiada, potencia, modulación y estado eléctrico de U2 requieren instrumentación independiente si se desean afirmar. |
| Corrección de metadata firmware | Abierto | `channel=37`, `lqi=0`, RSSI y timestamps anómalos, y semántica de `crc_ok` requieren análisis/corrección separados. |
| Causa histórica de recepción cero | Abierto | El transcript fortalece comparaciones, pero no establece causa raíz. |
| Interoperabilidad de protocolos nominales | No validada | Los nombres de preset no demuestran conformidad ni interoperabilidad Wireless M-Bus, Wi-SUN o Sidewalk. |

No queda ningún experimento físico obligatorio bajo el alcance funcional declarado por el manual. Las tareas restantes de reproducibilidad son preservar este transcript y su hash, registrar `feralrf.__file__`, entorno y hashes de binarios instalados cuando estén disponibles, y usar IDs/markers únicos en campañas futuras. La validación de metadata necesita trabajo de firmware; las afirmaciones RF físicas necesitan analizador/instrumentación, pero ninguna es condición para cerrar esta fase funcional.

La auditoría causal histórica no necesita repetirse. Sí requiere un addendum dirigido para: retirar la brecha de “transcript octubre ausente”, ampliar las comparaciones con los trece PASS primarios y reforzar —sin elevar a causa confirmada— la observación sobre payloads de cuatro bytes. Su conclusión de que no existe una causa raíz histórica definitiva permanece sin cambios.
