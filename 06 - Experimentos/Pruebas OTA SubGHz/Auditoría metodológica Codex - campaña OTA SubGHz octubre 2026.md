# Auditoría metodológica Codex — campaña OTA Sub-GHz, octubre de 2026

**Fecha de auditoría:** 2026-10-16 (America/Mexico_City)  
**Alcance:** revisión estática y de solo lectura. No se abrieron puertos serie, no se transmitió RF y no se modificaron firmware, scripts, matriz oficial ni el reporte experimental primario.  
**Dictamen:** **PARCIALMENTE AUDITABLE / NO DEMOSTRADA COMO CAMPAÑA COMPLETA**.

## 1. Dictamen ejecutivo

Existe evidencia primaria suficiente para confirmar **seis ejecuciones de recuperación del 15 de octubre**, todas sobre `wisun_915_fsk_50k`, COM33→COM88: 60 ACK de encolado TX, 60 objetos RX que contienen `DE AD BE EF`, aumento de `rx_ok` de 0 a 10 por corrida, ausencia de excepciones fatales y limpieza satisfactoria. Los resúmenes están en `FeralRF/experimental_artifacts/checkpoint_{3a,3a_r,3c,3d,3e,4a}/*raw.log`; por ejemplo, 3A en líneas 1 y 80–81, y 4A en líneas 1 y 80–81.

No se puede confirmar el agregado declarado de **13 presets nuevos, 19 ejecuciones, 190 paquetes, tres BLOCKED no inducidos y dos BLOCKED inducidos con RESET**. No se encontraron logs de `manual_preset_runner.py`, inventario de etiquetas, comandos de invocación, exit codes ni registros de recuperación de esas 18 ejecuciones. Además:

- `FeralRF/MANUAL_FERALRF_SUBGHZ.md` existe pero tiene **0 bytes**, está sin seguimiento y no tiene historial Git.
- `Understanding-CatSniffer/06 - Experimentos/Pruebas OTA SubGHz/Auditoría final de experimentos FeralRF OTA SubGhz.md` existe pero tiene **0 bytes**, está sin seguimiento y no tiene historial Git.
- Los `.log` están excluidos por `FeralRF/.gitignore:52`; sólo sobreviven localmente los logs de 3A–4A.

Por tanto, no es posible auditar el reporte primario ni demostrar conformidad literal con el manual designado. La secuencia propuesta recuperable en `FeralRF/SUBGHZ_PROVENANCE_AUDIT.md:425-493` se usa abajo **sólo como sustituto provisional**, no como si fuera el manual ausente.

Hay una segunda limitación material: el código fuente actual de RX propietario parece interpretar en orden incorrecto los metadatos anexados por el RF Core. `RadioIF_processPropPackets()` trata los seis bytes finales como `status, RSSI, timestamp` (`firmware/cc1352/src/radio_if.c:1719-1735`), mientras la configuración habilita RSSI, timestamp y status (`:1639-1641`) y el SDK local documenta ese orden posterior al frame. El mismo código usa el primer byte —compatible con RSSI— para decidir CRC (`:1724-1725`). Esto explica directamente, si el binario instalado corresponde a esa fuente, los RSSI aleatorios, timestamps sospechosos y los 60 `crc_ok=True`. En consecuencia, **los marcadores recibidos son evidencia fuerte de entrega peer-to-peer, pero “CRC-good” no queda validado de manera independiente**.

La campaña es utilizable como baseline comparativo **condicional**: el subconjunto 3A–4A es una referencia fuerte para “se entregaron objetos RX con el marcador bajo estas dos unidades y este estado host” y contrasta legítimamente con campañas de cero objetos entregados. No es todavía un baseline forense completo de 19 ejecuciones, de los 13 presets, de exactitud de metadatos, de forma de onda/frecuencia/potencia ni de equivalencia binaria entre campañas.

## 2. Alcance y fuentes

### Fuentes primarias disponibles

- Estado Git y archivos del checkout `FeralRF`.
- `FeralRF/experimental_artifacts/checkpoint_3a` a `checkpoint_4a`: scripts y logs locales del 15 de octubre.
- `FeralRF/experimental_artifacts/checkpoint_3a/checkpoint_3a_single_run.py`, SHA-256 `485E3989D448F752B4E2F22A4EDE8C40A2C88447CB39C76E5E4BBB351074C842`.
- `FeralRF/experimental_artifacts/manual_preset_runner.py`, SHA-256 `1F5D0862872773A575C7F355F20571171C1BA37C5B0ED75B837C46204EAB91A2`.
- `FeralRF/python/feralrf/{radio.py,commands.py,presets.py}`.
- `FeralRF/firmware/cc1352/src/{command_processor.c,control_task.c,data_task.c,radio_if.c}`.
- `CatSniffer-Firmware/RP2040/catsniffer/src/{main.c,shell_commands.c}` para separar Bridge/Shell y control de CTF.

### Fuentes secundarias, no tratadas como prueba independiente de ejecución

- `FeralRF/SUBGHZ_PROVENANCE_AUDIT.md`.
- `FeralRF/SUBGHZ_RECOVERY_AUDIT.md`.
- `FeralRF/SUBGHZ_RECOVERY_RUN.md`.
- Documentos históricos del Vault y `FeralRF/docs/VALIDATION_MATRIX.md`.

### Evidencia solicitada pero ausente

- Contenido del manual y del reporte primario (ambos archivos vacíos).
- Logs crudos de las 13 corridas nuevas y de las cinco BLOCKED.
- Comandos exactos, orden temporal, etiquetas y exit codes de esa fase.
- Manifest/hashes de los binarios RP2040 y CC1352 instalados.
- Registro de antenas, geometría, revisión/BOM de las placas y alimentación.
- Medición eléctrica de GPIO8/9/10 o CTF1/2/3 y verificación de la posición real de U2.

## 3. Procedencia del repositorio y firmware

### FeralRF

- Rama: `feature/hardware-validation`.
- HEAD: `ba27fbeb8c5683a31cca4339982cadeaaa406444`, commit del 2026-10-12 14:20:55 -06:00, `docs: add FeralRF hardware validation results and test matrix`.
- No hay modificaciones de archivos seguidos. Son **untracked** el manual vacío, las tres auditorías Sub-GHz, `manual_preset_runner.py` y todos los helpers de checkpoints. Los logs son ignorados por Git.
- `docs/VALIDATION_MATRIX.md` está seguido y sin cambios; su último commit específico es `7ad0a1abb9b8efaa25126cf561d931ccf7b36ee4` (2026-07-22). No contiene la campaña de octubre; correctamente no fue actualizado por esta auditoría.

El HEAD antecede a los helpers y a la campaña. Por tanto, citar sólo `ba27fbeb` **no identifica el procedimiento ejecutado**: también se necesitan los hashes de los scripts no seguidos y de cada log.

### Host Python y scripts

Los checkpoints 3A–4A insertan correctamente `FeralRF/python` en `sys.path` (`checkpoint_3a_single_run.py:17-20`). En cambio, `manual_preset_runner.py:17-20`, al estar un nivel más arriba, calcula `parents[2]` como la raíz del workspace e inserta `catsniffer-feralrf/python`, que no existe. En el entorno inspeccionado la importación funciona porque `feralrf 0.3.0` está instalado en modo editable y apunta a `FeralRF/python`; eso no fue impreso ni congelado en cada corrida manual. La identidad exacta del módulo cargado durante esas corridas queda **NO VERIFICADA**.

### RP2040 Cat-Bridge, Cat-Shell y switch RF

El repositorio `CatSniffer-Firmware` está en `v3.x`, HEAD `c0cd5a45e019dbd14ed11d039aacb13f300e5731`, sin cambios seguidos. En el código:

- Cat-Bridge CDC0 reenvía bytes USB↔UART CC1352 y contabiliza overrun/drop (`RP2040/catsniffer/src/main.c:202-260`). No interpreta comandos FeralRF.
- Cat-Shell ejecuta `boot`, `exit`, `band1`, `band2` y `status` (`shell_commands.c:193-218,235-267`).
- `boot`/`exit` actúan sobre RESET/BOOT y baudrate del **CC1352**, no reinician el RP2040 (`main.c:345-359,404-425`).
- `band2` llama `change_band(SUBGIG_1)`; el RP2040 programa CTF1/2/3 a 0/0/1 (`main.c:379-402`). `status` muestra el estado cacheado, no una lectura eléctrica de U2.

### CC1352

El CC1352 procesa `RADIO_INIT`, `SET_PHY`, `SET_POWER`, `SET_PROP_CONFIG`, `RX_START`, `RX_STOP` y `TX_RAW` en `firmware/cc1352/src/command_processor.c:101-223`. La configuración y RF real se realizan después en `control_task.c`, `data_task.c` y `radio_if.c`.

La respuesta de `TX_RAW` confirma que el comando fue validado y **encolado**: `ControlTask_onTxRaw()` fija el evento (`control_task.c:249-259`) y `command_processor.c:210-223` envía ACK. La operación RF ocurre luego en `ControlTask_processTxRaw()` (`control_task.c:332-357`); un fallo posterior se comunica como error asíncrono. Del mismo modo, `RX_START` recibe ACK antes de que `DataTask_poll()` ejecute `RadioIF_startRx()` (`data_task.c:115-124`). Un ACK no es observación RF.

Los dispositivos sólo reportaron la identidad genérica CC `1.0.0`, capacidades 7 y serial lógico `FERALRF1`. No hay ELF/BIN/HEX, hash de lectura ni build ID que una el binario instalado a `ba27fbeb`; `SUBGHZ_PROVENANCE_AUDIT.md:230-232,337-348` reconoce la misma brecha.

## 4. Procedimiento reconstruido

La secuencia efectivamente implementada por `manual_preset_runner.py` es:

1. Recibe `preset`, marcador hexadecimal de cuatro bytes y etiqueta mediante `sys.argv`; usa puertos fijos COM33 TX y COM88 RX (`:23-40`). No descubre puertos ni impide etiquetas duplicadas.
2. Copia el preset e imprime el diccionario solicitado (`:52-65`).
3. Crea dos sesiones, ejecuta `init()` TX y RX (`:67-83`). `RADIO_INIT` también reinicia métricas en CC (`control_task.c:211-223`), por lo que los stats iniciales cero no prueban ausencia de historia anterior.
4. Selecciona `PHY.PROPRIETARY_GFSK`, canal solicitado 0, y aplica el preset a ambos (`:85-99`). La frecuencia efectiva proviene luego de `configure_prop`, no del nombre del enum.
5. Fija 0 dBm en TX y obtiene stats iniciales (`:101-111`).
6. Solicita RX, espera 0.3 s, solicita diez TX con 0.1 s de pausa host, espera 1 s y sólo entonces drena RX durante 2 s (`:113-180`). RX permanece activo, así que el orden es válido para diez tramas, pero no es lectura host concurrente y depende del buffering/contadores.
7. Drena el canal TX durante 0.5 s para errores asíncronos (`:182-206`). La API conserva esos errores si llegan mientras espera otro ACK (`python/feralrf/radio.py:285-367`).
8. Detiene RX, obtiene stats finales y desconecta en `finally` (`manual_preset_runner.py:208-261`).
9. Clasifica y devuelve 0 sólo para `PASS-RF`; cualquier otro resultado devuelve 2 (`:263-300`).

`boot`/`exit`, Shell `status`, `band2`, verificación lógica/eléctrica, controles positivos, recuperación y RESET físico **no forman parte del runner**. Deben tener registros externos correlacionados con la etiqueta; no se encontraron para la fase de 13 presets.

### Diferencias frente a `checkpoint_3a_single_run.py`

- El helper original fija preset y marcador, y delega reset/banda a pasos externos (`:1-5,23-28`).
- Su exit code es 0 siempre que no haya excepción fatal, incluso con 0 RX, TX incompletos o errores asíncronos (`:250-263`). El 0 del proceso no equivale a PASS-RF.
- El runner manual añade validación de argumentos y clasificación, pero conserva el mismo flujo de radio y tiene el error de ruta de importación descrito arriba.
- El runner manual clasifica `FAIL` siempre que haya cero marcadores (`:284-287`), incluso si hubo 0 ACK, excepciones TX o errores asíncronos. Eso contradice el criterio recuperable de “FAIL-DELIVERY tras setup válido” (`SUBGHZ_PROVENANCE_AUDIT.md:475-491`): esos casos deberían ser BLOCKED o INCONCLUSIVE.
- Una excepción de desconexión se registra pero no afecta la clasificación; tampoco se exige que stats y conteo concuerden. Diez hits con más paquetes extra pueden pasar, y once hits caen en INCONCLUSIVE.
- 3E sí aplicó una ventana negativa de 3 s y una positiva de 5 s; el runner manual no incluye control negativo por corrida.

## 5. Matriz de cumplimiento del procedimiento

Como el manual designado está vacío, esta matriz contrasta la evidencia con la secuencia requerida por el encargo y con el método propuesto en `SUBGHZ_PROVENANCE_AUDIT.md:425-493`. No puede llamarse una verificación exhaustiva del manual real.

| Requisito | Tipo | Estado | Evidencia / discrepancia |
|---|---|---|---|
| Manual y criterios disponibles antes de ejecutar | obligatorio documental | **NOT VERIFIED** | Archivo de 0 bytes, untracked, sin historial. Impide probar conformidad literal. |
| Rama, HEAD, estado y hash del helper al inicio | obligatorio de trazabilidad | **PARTIALLY VERIFIED** | HEAD y hashes recuperables a posteriori; helper manual no seguido y sin manifest por corrida. |
| Python, paquete y dependencias fijados | obligatorio de reproducibilidad | **PARTIALLY VERIFIED** | Actualmente Python 3.14.7 y editable 0.3.0; no hay `pip freeze`/`__file__` por corrida manual. |
| Identificación Bridge/Shell por USB, no aritmética | obligatorio | **VERIFIED** para 3A–4A; **NOT VERIFIED** para las 13 | #2 COM33/35 y #3 COM88/87 constan en recovery; runner sólo fija Bridges. |
| `status` inicial y contadores | obligatorio | **PARTIALLY VERIFIED** | Presente en checkpoints; ausente para la fase manual. |
| Preparación inicial `boot`→0.5 s→`exit`→0.3 s→≥3.5 s | obligatorio para la condición inicial reconstruida | **PARTIALLY VERIFIED** | Evidencia directa completa en 3A-R (`reset_band.log:1-24`); no para los 13 nuevos. |
| `band2` en ambas unidades | obligatorio si se declara Sub-GHz por Shell | **PARTIALLY VERIFIED** | 3A-R y 3C tienen respuestas y `Band:1`; 3D–4A recibieron con `Band:0`; fase manual sin logs. |
| Verificación lógica posterior a `band2` | obligatorio | **PARTIALLY VERIFIED** | Directa en 3A-R/3C; `status` sólo prueba estado software. |
| Verificación eléctrica GPIO/CTF/U2 | obligatoria para afirmar posición del switch; recomendada para delivery funcional | **NOT VERIFIED** | No hubo medida. El propio log etiqueta la evidencia como `LEVEL 2 / SOFTWARE STATE ONLY`. |
| Antena/carga, distancia, orientación y alimentación | obligatorio de comparabilidad RF | **NOT VERIFIED** | No existe registro de sesión recuperable. |
| Revisión PCB/BOM y rework de cada unidad | obligatorio de baseline forense | **NOT VERIFIED** | V3/CC1352P7 está soportado, pero la equivalencia con PCB/BOM del repo es desconocida (`SUBGHZ_PROVENANCE_AUDIT.md:234-247`). |
| Binarios RP/CC ligados a hashes | obligatorio de baseline forense | **NOT VERIFIED** | Versiones genéricas no cierran source→build→artifact→flash. |
| Sesiones frescas por corrida | obligatorio | **VERIFIED por código**; ejecución manual **NOT VERIFIED** | El runner crea objetos nuevos; faltan logs que prueben cada invocación. |
| Selección PHY y preset idéntico en TX/RX | obligatorio | **VERIFIED** en 3A–4A; **NOT VERIFIED** en las 13 | El runner lo implementa y debería imprimir diccionario; salidas ausentes. |
| Inspección de preset, frecuencia, mod, rate, sync | obligatorio | **PARTIALLY VERIFIED** | Valores fuente y plan recuperables; valores de ejecución sólo para `wisun_915_fsk_50k`. |
| Potencia solicitada 0 dBm | obligatorio | **VERIFIED** en 3A–4A; **NOT VERIFIED** en las 13 | Solicitud/ACK no es medición de potencia. |
| Stats inicial/final | obligatorio | **VERIFIED** en 3A–4A; **NOT VERIFIED** en las 13 | `rx_ok=10` final en seis logs; deriva del mismo parser, no es corroboración independiente de CRC. |
| RX antes de TX y esperas 0.3/0.1/1/2 s | obligatorio | **VERIFIED** por script y logs 3A–4A; manual **NOT VERIFIED** | La fase manual carece de stdout. |
| Conservar Packet, marcador, errores async y fatales | obligatorio | **PARTIALLY VERIFIED** | Completo en cinco helpers base; 3E no añadió ventana TX async (`SUBGHZ_RECOVERY_RUN.md:1318`). Sin logs manuales. |
| RX stop y desconexión | obligatorio | **VERIFIED** en 3A–4A; **NOT VERIFIED** en las 13/BLOCKED | El runner puede declarar PASS pese a fallo de disconnect. |
| Control positivo periódico durante 13 presets | obligatorio de deriva | **NOT VERIFIED** | Hay seis corridas anteriores del sentinel, pero no cronología que pruebe cadencia intercalada. |
| Recuperación después de fallo/reset | obligatorio cuando ocurre bloqueo | **NOT VERIFIED** | No hay registros de las cinco BLOCKED ni de la siguiente corrida. |
| Dos contra-pruebas con RESET físico | experimento diagnóstico opcional salvo que el manual lo eleve | **NOT VERIFIED** | La afirmación carece de registro. 4A es un power-cycle documentado, no evidencia de esas dos ejecuciones. |
| Receptor/analizador independiente | validación RF independiente fuera del alcance funcional peer-to-peer | **NOT APPLICABLE** al PASS de entrega entre dos CatSniffer; **NOT VERIFIED** para frecuencia/modulación/potencia | No debe presentarse el resultado como certificación de waveform o protocolo. |
| Archivo crudo inmutable por etiqueta, comando y exit code | obligatorio de trazabilidad | **DEVIATION** | Sólo 3A–4A; los logs manuales no están. Etiquetas duplicadas no pueden descartarse. |

El trabajo no terminado que impide declarar la campaña completa dentro de su alcance funcional es principalmente **documental y de trazabilidad**: manual/reporte vacíos, ausencia de logs de los 13 presets/BLOCKED, falta de cronología de controles y recuperaciones, y falta de identidad binaria. No hace falta repetir hardware para reparar evidencia que ya exista: primero debe recuperarse y preservarse el material original. La verificación eléctrica o un receptor independiente sólo son necesarias si se quieren ampliar las conclusiones a switch/frecuencia/modulación/potencia o resolver la contradicción histórica.

## 6. Auditoría de resultados y clasificaciones

### Ejecuciones verificables

| ID | Condición | TX ACK de encolado | RX con marcador | Stats RX final | Errores/fatal | Resultado auditable |
|---|---|---:|---:|---:|---|---|
| 3A | reset/band documentados secundariamente; `Band:1` post | 10 | 10 | 10/0/0/0 | 0/ninguna | entrega peer 10/10 verificada |
| 3A-R | reset y `band2` verificados en log | 10 | 10 | 10/0/0/0 | 0/ninguna | repetición 10/10 verificada |
| 3C | sin reset; `band2` | 10 | 10 | 10/0/0/0 | 0/ninguna | entrega 10/10 verificada |
| 3D | `band1`, `Band:0` | 10 | 10 | 10/0/0/0 | 0/ninguna | entrega 10/10 verificada; invalida asumir que el estado lógico `band2` era necesario en corta distancia |
| 3E | `Band:0`, flujo legacy, negativo 0 | 10 | 10 | 10/0/0/0 | sin error observado; TX async pendiente desconocido | entrega 10/10 verificada |
| 4A | power-cycle ~15 s, `Band:0` inicial | 10 | 10 | 10/0/0/0 | 0/ninguna | entrega 10/10 verificada |

Los seis `TEST_BEGIN` y `TEST_SUMMARY` están respectivamente en líneas 1 y 80 (3A, 3A-R, 3C, 3D y 4A) y líneas 1/76 (3E). Cada payload observado es `04 DE AD BE EF 33 C8`: byte de longitud, marcador y dos bytes de CRC incluidos por la configuración. Esto es evidencia mucho más fuerte que un ACK TX, aunque no constituye un receptor independiente.

La categoría documental “cinco controles positivos + una repetición” no está codificada en los logs. Sí se verifica el total de **seis corridas 10/10 del mismo sentinel**. Una repetición adicional 1/1 sólo aparece como afirmación secundaria en `SUBGHZ_PROVENANCE_AUDIT.md:76` y no se incluye.

### Trece presets planificados, no ejecuciones verificadas

El plan recuperable contiene exactamente trece presets nuevos: `gfsk_902_50k`; `wisun_915_fsk_{100k,150k,200k,300k}`; `gfsk_915_50k`; `sidewalk_915_fsk_{50k,250k}`; `gfsk_868_{50k,100k}`; `wireless_mbus_{s,t,c}_868` (`SUBGHZ_PROVENANCE_AUDIT.md:439-465`). Los parámetros fuente están en `python/feralrf/presets.py:81-136,171-185,205-227,247-277`. Esto verifica el **plan y los diccionarios**, no que se ejecutaran.

Conversión comprobada:

- `0x930B51DE` = **2,466,992,606**.
- `0x543D0000` = **1,413,283,840**.

Los tres presets Wireless M-Bus usan `0x543D0000` (`presets.py:113-136`). No es posible cotejar el número supuestamente discrepante del reporte porque el reporte encontrado está vacío. Cualquier decimal distinto de 1,413,283,840 para ese sync word es incorrecto. Además, el nombre Wireless M-Bus sólo identifica parámetros raw; no se probó framing/interoperabilidad W-MBus.

### Evaluación de las afirmaciones agregadas

| Afirmación | Dictamen |
|---|---|
| 13 presets nuevos probados | **NOT VERIFIED**: plan presente, logs ausentes. |
| 5 controles positivos + 1 repetición | **PARTIALLY VERIFIED**: seis corridas sentinel 10/10, no su taxonomía/cadencia. |
| 19 ejecuciones OTA exitosas | **PARTIALLY VERIFIED**: 6 verificadas, 13 no. |
| 190 paquetes CRC-good con marcador | **NOT JUSTIFIED tal como está redactado**: 60 objetos con marcador constan; faltan 130 y el flag CRC propietario es cuestionable por el parser. |
| 3 BLOCKED no inducidas | **NOT VERIFIED**: sin stdout, excepción, fase ni recuperación. |
| 2 BLOCKED inducidas por RESET físico | **NOT VERIFIED**: sin registro primario. |
| Sin etiquetas duplicadas | **NOT VERIFIED**: el runner acepta cualquier string y no mantiene registro/constraint de unicidad. |

## 7. Discrepancias confirmadas

1. **Fuentes rectoras vacías.** El manual y reporte primario existen como archivos de 0 bytes; cualquier conclusión atribuida a su contenido no es auditable.
2. **Agregado sin evidencia cruda.** No existen en el workspace salidas de `manual_preset_runner.py`; 13 PASS y 5 BLOCKED no pueden reconstruirse.
3. **Scripts no ligados al commit.** El helper manual y los checkpoints son untracked; HEAD no basta para reproducirlos.
4. **Ruta de importación errónea.** `manual_preset_runner.py:17-20` no ancla `FeralRF/python`. Funciona hoy por instalación editable, dependencia no capturada por corrida.
5. **Semántica de clasificación incompleta.** Cero RX se marca FAIL incluso con setup/TX inválido; errores de cleanup pueden coexistir con PASS; exit code 2 no distingue PARTIAL/FAIL/BLOCKED.
6. **ACK TX no prueba TX RF.** Firmware ACKea el encolado antes de ejecutar el comando RF; el runner reduce el riesgo al drenar errores asíncronos, pero no crea observación independiente.
7. **`channel=37` pese a solicitar 0.** `RadioIF_setPhy()` sólo actualiza su `s_channel` si `channel != 0` (`radio_if.c:1944-1955`), por lo que conserva el canal anterior; `RadioIF_processPropPackets()` publica ese estado (`:1733`). Es metadata interna obsoleta, no prueba RX en canal BLE 37.
8. **LQI cero es deliberado.** La ruta propietaria fija `pkt.lqi = 0u` (`radio_if.c:1735`). No es una medición de calidad.
9. **RSSI/timestamp/CRC probablemente desalineados.** El parser usa offsets incompatibles con el orden de metadatos configurado (`radio_if.c:1719-1735`). Los RSSI de -127 a +126 dBm y deltas de timestamp de ~0.5 ms mientras el host espacia ~0.1 s son coherentes con lectura de bytes equivocados. Dado que el binario instalado no está ligado a esta fuente, la causalidad runtime se clasifica como **hipótesis fuertemente soportada por código**, no identidad binaria demostrada.
10. **Stats y CRC no son controles independientes.** `rx_ok` se incrementa después del mismo `crc_ok` parseado (`radio_if.c:1738-1751`); que ambos digan diez no valida dos veces el CRC.
11. **`band2` no quedó causalmente validado.** 3D, 3E y 4A entregaron 10/10 con `Band:0`. Eso no prueba que el switch no importe: puede haber ruta residual, acoplamiento de corta distancia o divergencia entre estado cacheado y U2. Sólo demuestra que la respuesta Shell no es una medición eléctrica.

## 8. Brechas y limitaciones de evidencia

- No puede saberse qué ocurrió en las 13 corridas nuevas, ni verificar preset, marcador, etiqueta, frecuencia solicitada, ACK, RX, stats, error, cleanup o exit code.
- No puede auditarse la cadencia de controles positivos ni si cada bloqueo fue seguido por la recuperación requerida.
- No se puede distinguir una etiqueta duplicada de una repetición intencional.
- La semántica `crc_ok` propietaria no es fiable hasta validar/corregir el layout y ligar el binario instalado al código.
- No existe prueba independiente de frecuencia, modulación, potencia, tasa de emisión o estado de U2. La evidencia demuestra compatibilidad entre los dos endpoints, no conformidad de waveform.
- Las versiones `v3.1.0.1` y CC `1.0.0` no sustituyen hashes de binarios.
- Las revisiones de PCB/esquemático del repositorio son ambiguas respecto de las unidades instaladas; tampoco se registraron antenas ni geometría.
- El reporte secundario de 1/1 post-4A y las auditorías previas no sustituyen stdout primario.

## 9. Aptitud como baseline histórico

### Apto, con alcance restringido

Usar 3A–4A como baseline de **entrega host-visible de marcador raw** bajo:

- dos unidades identificadas como #2 COM33/35 y #3 COM88/87;
- dirección COM33→COM88;
- preset host `wisun_915_fsk_50k` (902.2 MHz solicitado), 0 dBm solicitado;
- helper hash `485E3989...74C842` y checkout `ba27fbeb...`;
- seis condiciones de estado, incluida la reproducción legacy 3E.

Este subconjunto sí permite comparar de forma prudente “10 objetos peer con marcador” contra “0 objetos entregados” en las campañas antiguas. Deben compararse objetos/marcadores y errores, no el campo CRC ni RSSI/LQI/timestamp.

### No apto todavía

No usar el agregado 19/190 como baseline forense ni afirmar que los 13 presets pasaron hasta recuperar los logs. Tampoco usar octubre como prueba de que:

- el binario instalado corresponde a `ba27fbeb`;
- `band2` colocó U2 en una posición determinada;
- se emitió a frecuencia/potencia/modulación exactas;
- Wireless M-Bus, Wi-SUN o Sidewalk interoperan;
- cada paquete tuvo CRC válido según RF Core.

## 10. Tareas restantes priorizadas

1. **Recuperar, sin regenerar**, el contenido original del manual y reporte, y los stdout/stderr/exit codes de las 13 PASS y cinco BLOCKED. Preservar timestamps y hashes; no reconstruirlos desde memoria.
2. Crear un manifiesto inmutable que relacione etiqueta → comando → preset/marker → archivo crudo → exit code → estado previo/posterior → control/recovery. Esto también revelará duplicados.
3. Capturar o recuperar la identidad exacta del entorno usado (`python --version`, `feralrf.__file__`, paquete/dependencias) y los hashes de todos los helpers.
4. Corregir documentalmente el agregado: separar ACK/enqueued, Packet con marcador, CRC reportado y RF independiente; no recalcular resultados sin los registros originales.
5. Auditar de forma acotada el layout de metadata proprietary y determinar si el binario instalado contiene el mismo parser antes de seguir usando CRC/RSSI/timestamp.
6. Obtener build IDs/hashes RP y CC vinculados a cada USB serial. Es la brecha principal para comparación forense entre campañas.
7. Sólo si la investigación posterior necesita atribuir la causa al frente RF: documentar antenas/geometría/revisión y verificar GPIO/CTF/U2 o energía en J1. No es necesario repetir OTA para resolver primero las brechas documentales.

## 11. Preguntas para la investigación histórica posterior

1. ¿Dónde se guardaron los logs del runner manual y de las cinco BLOCKED, y sus timestamps/hash coinciden con las etiquetas del reporte?
2. ¿Qué identidad exacta de `feralrf.__file__`, dependencias y helper tuvo cada proceso manual?
3. ¿Los binarios RP/CC de octubre y de las campañas de cero paquetes eran realmente iguales o sólo compartían strings de versión?
4. ¿Los cero históricos tuvieron errores asíncronos, `rx_ok`, drops/overflow y cleanup conservados, o sólo TX ACK?
5. ¿Cuál fue el orden exacto de los 13 presets, controles y cinco bloqueos; hubo contaminación de estado o etiquetas repetidas?
6. ¿Qué operaciones de recuperación siguieron a cada BLOCKED y se verificó el primer intento posterior sin ocultar/reintentar el fallo?
7. ¿Las mismas placas, antenas, posición y alimentación participaron en las campañas antiguas y en octubre?
8. ¿Qué versión del parser proprietary estaba instalada en cada campaña y cómo debe reinterpretarse `crc_ok` sin alterar la evidencia cruda?
9. ¿Por qué la entrega funciona con estado lógico `Band:0`, y qué estado eléctrico real tuvo U2? Esta pregunta queda expresamente diferida; no se investiga aquí.

