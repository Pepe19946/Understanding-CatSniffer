# Auditoría causal Codex — comparación histórica OTA Sub-GHz

**Fecha:** 2026-10-16 (America/Mexico_City)  
**Método:** análisis estático de código, Git y registros existentes. No se abrió ningún puerto, no se modificó firmware ni Python y no se generaron observaciones RF nuevas.  
**Dictamen:** **NO SE ESTABLECIÓ UNA CAUSA RAÍZ DEFINITIVA**. Varias explicaciones procedurales simples quedan contradichas. La hipótesis causal más coherente es un estado previo anómalo del CC1352/RF Core/backend proprietary que el primer reset controlado pudo limpiar, pero sólo alcanza **PLAUSIBLE**. Una diferencia no ensayada —payload de seis bytes en todos los fallos frente a cuatro bytes en los éxitos verificables— también es **PLAUSIBLE** y constituye el A/B mínimo de mayor rendimiento.

## 1. Resumen ejecutivo

La campaña fallida identificable no es una colección vaga de recuerdos: el 8 de octubre de 2026 quedaron registrados 24 procesos/corridas proprietary, 240 retornos satisfactorios de `Radio.transmit()`, cero objetos RX entregados y cero `RxStreamError` expuestos. Incluyó 868, 902.2, 915 y 2440 MHz, ambos sentidos para controles representativos y una observación RX concurrente de 30 s. Es un **FAIL del criterio de entrega host-visible**, no prueba de 240 emisiones RF fallidas.

La comparación directa más fuerte es:

- 2026-10-08, `wisun_915_fsk_50k`, COM33→COM88, 902.2 MHz, FSK, 50 kbaud, 0 dBm, mismo diccionario de preset: 10 retornos TX, 0 objetos RX (`04_proprietary_extended_campaign.md:460-462`).
- 2026-10-15, checkpoints 3A, 3A-R, 3C, 3D, 3E y 4A, misma dirección/preset/parámetros: seis corridas de 10 objetos RX con el marcador, soportadas por logs crudos; por ejemplo 3A `checkpoint_3a_20261015_raw.log:1,80-81` y 4A `checkpoint_4a_20261015_raw.log:1,80-81`.

Las fuentes seguidas entre el baseline histórico `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea` y el checkout `ba27fbeb8c5683a31cca4339982cadeaaa406444` tienen **los mismos blobs Git** para `radio.py`, `presets.py`, `command_processor.c`, `control_task.c`, `data_task.c` y `radio_if.c`. Por tanto, no hay una regresión o corrección de fuente rastreable en ese intervalo que explique el cambio 0/10→10/10. Los binarios instalados, sin embargo, no tienen hash/build ID y no pueden equipararse al código.

Hallazgos causales principales:

1. **Estado/lifecycle previo del CC1352 — PLAUSIBLE.** El tramo fallido no hizo reset; 3A comenzó con reset verificado de ambos CC y funcionó. Los éxitos posteriores sin reset no refutan que ese primer reset limpiara un estado atrapado. Sí refutan que sea obligatorio resetear antes de cada preset. El power-cycle 4A demuestra que el éxito actual no depende sólo del estado RAM acumulado desde 3A.
2. **Longitud/contenido del payload — PLAUSIBLE.** Todas las corridas fallidas preservadas usaron marcadores de seis bytes; F29 histórico y los seis éxitos actuales usan `DEADBEEF`, cuatro bytes. El código auditado declara longitud variable hasta 255 bytes, de modo que no hay defecto estático evidente; falta el A/B de 4 vs. 6 bytes bajo una condición idéntica.
3. **Ruta eléctrica U2/CTF/antena — PLAUSIBLE, débilmente localizada.** El estado Shell es sólo cache. No obstante, 3D, 3E y 4A entregaron 10/10 con `Band: 0`, mientras el histórico fallido había verificado `Band: 1`. Esto contradice que el mero comando lógico `band2` sea necesario y reduce el peso de una explicación simple de selección lógica; no determina la posición física de U2.
4. **Binario instalado distinto — INSUFFICIENT EVIDENCE.** Es posible porque faltan hashes, pero no se identificó cambio de implementación proprietary en Git ni cambio funcional RP2040 entre v3.1.0.0 y v3.1.0.1.

No existe una hipótesis clasificada **CONFIRMED ROOT CAUSE** ni una causa técnica que alcance **STRONGLY SUPPORTED**. La única conclusión fuertemente soportada de H10 es metodológica: ACK TX significa comando encolado, no emisión RF.

## 2. Experimentos históricos identificados

### 2.1 Campaña de cero entrega relevante

Fuentes primarias documentales seguidas en Git:

- `FeralRF/docs/EXPERIMENT_REPORT_2026-10-12/experiments/03_proprietary_gfsk_868_915.md`, idéntico por SHA-256 al espejo del Vault.
- `FeralRF/docs/EXPERIMENT_REPORT_2026-10-12/experiments/04_proprietary_extended_campaign.md`, idéntico por SHA-256 al espejo del Vault.
- Ambos entraron en `ba27fbeb...` el 2026-10-12. Los prompts/stdout incrustados son evidencia de ejecución; las tablas y explicaciones retrospectivas no se elevan al mismo nivel.

Condiciones compartidas registradas:

| Campo | Registro histórico |
|---|---|
| Fecha | Primer grupo: fecha/hora individual ausente; consolidado 2026-10-08 (`03...md:23`). Segundo grupo: 2026-10-08 (`04...md:16`). |
| Hardware | CatSniffer V3 #1 COM33/34/35 y #2 COM88/86/87 (`03...md:54-70`). Identidad física exacta/revisión no cerrada. |
| Roles | Principal COM33 TX→COM88 RX; controles inversos COM88→COM33. |
| RP runtime | COM33 v3.1.0.0/`8eaa84c`; COM88 v3.1.0.1/`c0cd5a4` (`03...md:72-77`). Son strings/asociaciones, no hashes del UF2. |
| CC runtime | `1.0.0`, caps 7, serial lógico común `FERALRF1` durante controles concurrentes (`04...md:36-52`). |
| Host | FeralRF `0178721c...`, paquete 0.3.0; Python 3.14.7 consta en la auditoría de procedencia. No hay `pip freeze` por proceso. |
| Helper | `Invoke-FeralPresetOta`, Python inline en PowerShell (`03...md:203-244`). No existe artefacto standalone/hash independiente. |
| Preparación | `band1→band2` y luego `Band: 1` para Sub-GHz; sin reset automático ni flashing (`03...md:177-201`; `04...md:54-69`). |
| Flujo | `connect`; `init`; PHY 7/canal 0; mismo preset en ambos; potencia TX 0 dBm; `start_rx`; lectura negativa 3 s; 10 TX separados 0.1 s; lectura positiva 5 s; stop/disconnect con excepciones suprimidas. |
| Resultado | 24 corridas, 240 retornos TX, 0 objetos proprietary entregados, 0 errores async expuestos (`04...md:764-782`). Exit codes no preservados. |

### 2.2 Inventario de las 24 corridas

Todos los marcadores siguientes contienen **seis bytes**. Cada corrida registró 10 retornos TX y 0 hits; donde la salida no contiene `RX_ITEM`, el helper observó lista positiva vacía, no sólo un marcador incorrecto.

| ID | Preset / parámetros solicitados | Dirección / marcador | Estado lógico | Calidad del registro |
|---|---|---|---|---|
| HF-01 | `gfsk_868_50k`; 868 MHz, mod 1, 50k, dev 100, BW 0x52, sync 0x930B51DE | 33→88; `b10133008801` | `Band:1` | comando y resumen; stdout literal inicial ausente (`03...md:287-307`) |
| HF-02–03 | mismo | 88→33; `b10288003301`, repetido | `Band:1` | prompts/stdout literales, 0/10 dos veces (`03...md:356-386`) |
| HF-04 | `gfsk_915_50k`; 915 MHz, mod 1, 50k, dev 100, BW 0x52, sync 0x930B51DE | 33→88; `b11133008801` | `Band:1` heredado | prompt/stdout literal (`03...md:390-418`) |
| HF-05–06 | `4fsk_868_50k`; 868 MHz, mod 5, 50k | 33→88 `b201...02`; 88→33 `b202...02` | `Band:1` | salida preservada (`04...md:92-125`); mod 5 es reservado en el setup TI auditado |
| HF-07 | `gfsk_2440_50k`; 2440 MHz, mod 1, 50k | 33→88; `b30133008803` | `Band:0` | salida preservada (`04...md:126-170`) |
| HF-08–09 | mismo | 33→88 `b311...03`; 88→33 `b312...03` | `Band:0` | controles concurrentes; HF-09 tiene timestamps completos (`04...md:172-281`) |
| HF-10 | `gfsk_2440_250k`; 2440 MHz, mod 1, 250k, dev 320, BW 0x59 | 33→88; `b32133008804` | `Band:0` | salida preservada (`04...md:313-331`) |
| HF-11 | `gfsk_868_100k`; 868 MHz, mod 1, 100k, dev 150, BW 0x56 | 33→88; `b40133008805` | `Band:1` | salida preservada (`04...md:355-379`) |
| HF-12 | `msk_868_50k`; 868 MHz, mod 4, 50k | 33→88; `b41133008806` | `Band:1` | salida preservada; mod 4 reservado en el setup TI auditado |
| HF-13 | `4gfsk_868_50k`; 868 MHz, mod 6, 50k | 33→88; `b42133008807` | `Band:1` | salida preservada; mod 6 reservado |
| HF-14–16 | W-MBus S: 868.3/32.768k; T/C: 868.95/100k; mod 1; sync 0x543D0000 | 33→88; `b501...08`, `b511...09`, `b521...0a` | `Band:1` | tres salidas preservadas (`04...md:381-406`) |
| HF-17 | `gfsk_902_50k`; 902.2 MHz, mod 1, 50k | 33→88; `b6013300880b` | `Band:1` | salida preservada (`04...md:408-423`) |
| HF-18–19 | Sidewalk raw FSK: 915 MHz, 50k/250k, mod 0 | 33→88; `b611...0c`, `b621...0d` | `Band:1` | dos salidas preservadas (`04...md:424-454`) |
| HF-20–24 | Wi-SUN raw FSK: 902.2 MHz, 50/100/150/200/300k, mod 0 | 33→88; `b701...0e` a `b741...12` | `Band:1` | cinco salidas preservadas (`04...md:455-486`) |

Los parámetros se verificaron contra `FeralRF/python/feralrf/presets.py:81-168,171-227,247-277`. El nombre `wisun_915_*` no cambia la frecuencia efectiva solicitada: es 902.2 MHz.

### 2.3 Ceros históricos excluidos de la comparación principal

- OOK 433 0/10 del 2026-04-08 (`docs/VALIDATION_MATRIX.md:45-71`) no es comparable: banda, modulación, front-end, antena y lifecycle de patches difieren; no hay log crudo.
- MIOTY 868 0/10 (`docs/VALIDATION_MATRIX.md:160-166`) corresponde a soporte TS-UNB incompleto; no es un control de GFSK/FSK estándar.
- F29 70/70 del 2026-05-03 es un claim exitoso, no un fallo. Su script histórico es recuperable y usaba `DEADBEEF` y reset por preset, pero no hay stdout crudo ni identidad de binarios. Sólo sirve como antecedente, no como observación independiente.

## 3. Procedencia experimental

### 3.1 Fuentes ausentes

- `FeralRF/MANUAL_FERALRF_SUBGHZ.md`: 0 bytes, SHA-256 del archivo vacío.
- `Understanding-CatSniffer/06 - Experimentos/Pruebas OTA SubGHz/Auditoría final de experimentos FeralRF OTA SubGhz.md`: 0 bytes, SHA-256 del archivo vacío.
- No hay logs del `manual_preset_runner.py` para las 13 corridas nuevas ni para las cinco BLOCKED declaradas. Por ello no se construyen pares 868/915 octubre-vs-histórico a partir de esos claims.

### 3.2 Repositorios y artefactos

| Elemento | Histórico fallido | Octubre exitoso verificable |
|---|---|---|
| FeralRF fuente host | `0178721c...` | checkout `feature/hardware-validation` en `ba27fbeb...` |
| Código relevante seguido | blobs idénticos en ambos commits para seis archivos host/CC principales | igual |
| Script | helper inline, sin hash separado | 3A SHA-256 `485E3989...74C842`; 3E SHA-256 `AF11BAAB...1AF1C`; archivos untracked |
| Logs | transcripciones incrustadas en reporte seguido; no stdout externo | seis `.log` locales ignorados por `.gitignore:52` |
| RP runtime | COM33 v3.1.0.0; COM88 v3.1.0.1 | ambos reportan v3.1.0.1 |
| Diferencia RP rastreada | `8eaa84c`→`c0cd5a4` cambia únicamente la salida `fw_version` para añadir la placa (`b672ffd`) | no cambio en bridge, band, reset o GPIO |
| CC runtime | `1.0.0`, caps 7, `FERALRF1` | mismo identificador genérico |
| Hash RP/CC instalado | ausente | ausente |
| Unidades físicas | COM/PnP sugieren continuidad | USB seriales de recuperación `E661A89753565932` y `E661A89753656A35`; la equivalencia anterior no es absoluta |

La ausencia de build IDs impide afirmar que el mismo código CC se ejecutó en ambas fechas. También impide atribuir el cambio a firmware.

## 4. Corridas comparables históricas y de octubre

| Par | Campos equivalentes | Diferencias materiales | Comparabilidad |
|---|---|---|---|
| HF-20 vs 3A/3A-R | COM33→COM88; `wisun_915_fsk_50k`; PHY 7/canal 0; 902.2 MHz; mod 0; 50k; dev 100; BW 0x52; sync 0x930B51DE; 0 dBm; RX antes de TX; diez solicitudes | 6-byte vs 4-byte payload; sin reset vs reset verificado; helper/ventanas; RP COM33 version string; binarios/antenas/geometría desconocidos | **Mejor par paramétrico**, pero no A/B único |
| HF-20 vs 3E | mismos campos anteriores; además mismo patrón RX→negativa 3 s→10 TX/0.1 s→positiva 5 s; sin reset por esa ejecución | payload 6 vs 4; 3E hereda estado tras éxitos previos y `Band:0`; histórico `Band:1`; binarios/entorno físico no cerrados | **Mejor par de procedimiento host** |
| HF-20 vs 4A | mismos preset/dirección/parámetros; ambos sin reset/band después del límite inmediato | 4A parte de power-cycle completo y payload 4; histórico estado previo desconocido y payload 6 | bueno para descartar dependencia exclusiva de warm state actual |
| HF-17 vs 3A–4A | misma frecuencia, tasa, dev, BW y sync | GFSK mod 1 vs FSK mod 0; nombre/payload/estado difieren | útil, **no equivalente** |
| HF-01/HF-04/HF-14–19 vs “13 presets octubre” | nombres o familias presuntamente repetidos | logs actuales ausentes | **NO COMPARABLE como evidencia** |

No se dispone de una corrida actual raw de `gfsk_868_50k` o `gfsk_915_50k`. Las tablas de `PYTHON_API.md` y el reporte vacío no sustituyen ese registro.

## 5. Comparación lado a lado del flujo de comandos

| Paso | 8 de octubre, fallos | 15 de octubre, éxito 3A | Control 3E/4A | Evaluación causal |
|---|---|---|---|---|
| Descubrimiento USB | mapeo correcto documentado; enumeración literal inicial incompleta | PnP/MI_00–MI_04 y Shell explícitos | mismo | no indica cruce Bridge/Shell |
| Reset | ninguno | `boot` 0.5 s, `exit` 0.3 s, espera ≥3.5 s, respuesta verificada en ambas | 3E ninguno; 4A power-cycle ~15 s y ninguno posterior | reset inicial es diferencia compatible, no necesidad demostrada |
| Banda | `band1→band2`; `Band:1` | `band2`; `Band:1` | 3E/4A `Band:0`, sin comando | selección lógica no es requisito observado del éxito actual |
| Sesiones | objetos nuevos; `connect()` y `init()` | objetos nuevos; `init()` conecta | objetos nuevos | equivalente |
| PHY/preset | PHY 7 canal 0; mismo diccionario TX/RX | igual | igual | equivalente para HF-20 |
| Potencia | 0 dBm solicitados | 0 dBm | 0 dBm | equivalente, no medición física |
| RX | `start_rx()` ACK; luego negativa 3 s | ACK; espera 0.3 s | 3E negativa 3 s | RX se pidió antes de TX en todos |
| TX | 10 llamadas, pausa 0.1 s | 10 llamadas, pausa 0.1 s | igual | ACK sólo encolado |
| Payload | seis bytes únicos por corrida | `DEADBEEF`, cuatro bytes | cuatro bytes | diferencia sistemática no controlada |
| Lectura RX | después del TX, 5 s | espera 1 s y lee 2 s | 3E replica 5 s; 4A método actual | 3E contradice fallo inherente del helper viejo |
| Concurrencia | HF-09: RX activo desde 4.064 s antes del primer ACK hasta 24.690 s después del último | no necesaria | no | contradice consumo tardío como causa única |
| Stats/errors | stats sólo en controles: todos cero; sin `RX_ITEM`; sin async expuesto | before/after, 10 objetos; ventana async TX/RX | 3E no drena TX async; 4A sí | histórico localiza sólo “no entrega host” |
| Cleanup | excepciones `stop_rx`/`disconnect` suprimidas; exit code no guardado | errores de cleanup registrados | registrados | brecha documental, no causa del 0/10 actual |

## 6. Diferencias de implementación host y firmware

### 6.1 Host Python

- `Radio.init()` conecta, envía `RADIO_INIT` y obtiene info (`python/feralrf/radio.py:399-453`). No equivale a reset físico.
- `set_phy`, `set_power` y `configure_prop` esperan ACK (`radio.py:455-525`).
- `start_rx()` espera ACK (`radio.py:599-607`); `read_packets()` consume el stream hasta una ventana total (`:636-709`).
- Las respuestas asíncronas que llegan mientras otra llamada espera un ACK se guardan en `_pending_async` (`:123-133,278-393`). El helper histórico, no obstante, dependía además de buffers CC/UART/RP/USB mientras el objeto RX no leía.
- El control concurrente HF-09 elimina esa dependencia durante 30 s y aun así obtiene cero eventos (`04...md:231-281`).
- El helper 3E replica las ventanas históricas y obtiene 10/10 (`checkpoint_3e_legacy_single_run.py:118-177`; raw `:1,76-77`).

### 6.2 Cat-Bridge y Cat-Shell, RP2040

- CDC0/Bridge es reenvío USB↔UART CC1352 (`CatSniffer-Firmware/RP2040/catsniffer/src/main.c:202-260`).
- Cat-Shell ejecuta `boot`, `exit`, `band1`, `band2`, `band3` y `status` (`shell_commands.c:193-267`).
- `boot`/`exit` resetean/controlan el CC1352; no reinician el RP2040.
- `band2` ordena CTF 0/0/1, pero `status` muestra `catsniffer.band`, no lee GPIO/U2. `change_band()` retorna temprano si el cache ya coincide (`main.c:379-402`).
- En la campaña fallida se usaron Shell reales COM35 y COM87, no la regla errónea Bridge+2. H2 no explica los resultados.

### 6.3 CC1352 y RF Core

- `command_processor.c:101-223` valida y ACKea RADIO_INIT/config/RX/TX.
- `RX_START` se ACKea antes de que `DataTask_poll()` ejecute `RadioIF_startRx()` (`data_task.c:115-124`).
- `TX_RAW` se ACKea al copiar/encolar (`command_processor.c:210-223`; `control_task.c:249-259`); `RadioIF_transmitRaw()` corre después (`control_task.c:332-357`).
- El backend proprietary usa el mismo camino general para 868/902/915/2440, con setup/frecuencia derivados en `RadioIF_setPropConfig()` (`radio_if.c:2020-2170`) y TX en `RadioIF_transmitPropRaw()` (`:1757-1820`).
- Longitud: host codifica longitud explícita (`commands.py:57-60`); CC la parsea y copia; `pktLen=payload_len`; `bVarLen=1`; RX acepta hasta 255 (`radio_if.c:1631-1642,1757-1819`; `smartrf_prop_0.c:249-295`). El código no ofrece una razón obvia para rechazar seis bytes.
- El parser de metadata proprietary tiene un orden de offsets cuestionable y los stats dependen del mismo `crc_ok` (`radio_if.c:1706-1751`). Esto debilita CRC/RSSI/timestamp, pero no explica por sí solo que el histórico no entregara ningún objeto.

### 6.4 Historia dirigida

`git diff 0178721c... ba27fbeb...` es vacío para:

- `python/feralrf/radio.py`
- `python/feralrf/presets.py`
- `firmware/cc1352/src/{command_processor.c,control_task.c,data_task.c,radio_if.c}`

Los blobs son idénticos. Entre RP `8eaa84c` y `c0cd5a4`, el único cambio en `RP2040/catsniffer` añade `Board: v3 RP2040 CC1352P7` a `fw_version`; no cambia band/reset/bridge/CTF. Una regresión rastreable en estas fuentes queda contradicha.

## 7. Mecanismos candidatos

| Hipótesis | Clasificación | Alcance posible |
|---|---|---|
| H1 selección lógica/externa Sub-GHz incorrecta | **PLAUSIBLE** sólo para estado físico; la versión “faltó `band2`” está **CONTRADICTED** | podría bloquear Sub-GHz, no explica limpiamente 2440 sin fallo amplio de U2/ruta |
| H2 puertos Cat-Shell/Cat-Bridge incorrectos | **CONTRADICTED** | no explica la campaña |
| H3 comandos/argumentos incorrectos | **CONTRADICTED** para FSK/GFSK comparable; defecto confirmado sólo en mod 4/5/6 | explica como máximo HF-05/06/12/13 nominales, no el patrón común |
| H4 PHY/preset/frecuencia/sync incompatibles | **CONTRADICTED** para HF-20 vs 3A–4A | no explica mismo preset 0/10→10/10 |
| H5 RX no iniciado/ordenado | **CONTRADICTED** | no explica HF-09 ni la espera negativa de 3 s |
| H6 buffering/timing/consumo host | **CONTRADICTED** como causa única | interacción histórica residual no excluida |
| H7 estado CC1352/RF Core y reset | **PLAUSIBLE** | puede explicar el backend proprietary completo y el cambio tras 3A |
| H8 versión/regresión firmware | **INSUFFICIENT EVIDENCE** para binarios; regresión en fuente seguida **CONTRADICTED** | podría explicar todo si los binarios difirieran materialmente |
| H9 U2/front-end/antena/ruta física | **PLAUSIBLE** | puede impedir RF pese a ACK; localización no disponible |
| H10 interpretación de ACK/stats/cero RX | **STRONGLY SUPPORTED** como error de inferencia, no causa física | explica por qué “TX success” no localiza el fallo |
| H11 payload de seis vs cuatro bytes | **PLAUSIBLE** | podría explicar las 24 corridas si el binario/driver real fallaba por longitud |
| H12 distancia/orientación/antena | **WEAKLY SUPPORTED** | puede cambiar link budget, pero hubo un fallo a ~2 cm y otro más separado |

## 8. Evidencia a favor y en contra por hipótesis

### H1/H9 — banda y ruta externa

**A favor:** `set_phy()` sólo configura el CC; U2 lo gobierna el RP2040. `Band` no es readback (`Sub-GHz en CatSniffer V3...:98-145`). Una U2 mal seleccionada permite ACK sin RF útil en J1. No hay medición CTF/U2/J1 en ninguna campaña.

**En contra:** histórico sí forzó `band1→band2` y reportó `Band:1`. Octubre entregó 10/10 con `Band:0` en 3D, 3E y después de power-cycle en 4A (`SUBGHZ_RECOVERY_RUN.md:1133-1176,1352-1386,1548-1582`). Además el histórico falló también en proprietary 2440 con `Band:0` y control concurrente.

**Mecanismo esperado:** U2 conecta la rama incorrecta o con alta pérdida; comandos CC siguen ACKeando. **No cubre por sí solo** todos los resultados salvo una avería/estado amplio de ruta. **Verificación discriminante:** leer CTF/U2 y energía en J1 en una sola condición, no repetir un barrido.

### H2 — interfaces incorrectas

**A favor:** herramientas antiguas con Bridge+2 serían incorrectas para COM88→COM87.  
**En contra:** la campaña fallida no usó esa aritmética para el helper; COM33/88 fueron Bridge y COM35/87 Shell correctos. Ambos CC respondieron a `init()`/info y ambos sentidos se probaron.  
**Conclusión:** **CONTRADICTED** como causa de las 24 corridas.

### H3/H4 — argumentos o configuración RF

**A favor:** mod 4/5/6 son reservados por TI en el setup auditado; esas cuatro corridas no validan MSK/4FSK/4GFSK. ACK de `configure_prop()` no certifica configuración efectiva.  
**En contra:** el mejor par usa mod 0 válido y el diccionario `wisun_915_fsk_50k` es byte-a-byte el mismo en los commits comparados. TX y RX recibieron el mismo preset. GFSK mod 1 válido también falló en varias bandas.  
**Conclusión:** no es causa común. Los nombres inválidos explican sólo la falta de validez de un subconjunto.

### H5 — RX no activo

**A favor:** firmware ACKea `RX_START` antes de ejecutar físicamente el arranque.  
**En contra:** cada helper dejó 3 s después del ACK; HF-09 mantuvo lector concurrente listo 4.064 s antes del primer ACK TX y 24.690 s después del último, con cero eventos. Octubre usa el mismo camino firmware y recibe.  
**Conclusión:** **CONTRADICTED** como error de orden host; un fallo interno del backend RX pertenece a H7/H8.

### H6 — buffers/timing

**A favor:** el helper normal no lee el puerto RX durante las diez llamadas TX; dependía de buffers intermedios.  
**En contra:** HF-09 consumió activamente; 3E reprodujo exactamente 3 s/10 TX/5 s y recibió 10/10. Los 326 bytes `ring_dropped` de COM88 eran previos/no crecieron, y el control inverso con RX COM33 también dio cero.  
**Conclusión:** **CONTRADICTED** como explicación única; no se excluye una interacción con un binario/estado distinto.

### H7 — reset y estado CC

**A favor:** todos los fallos del 8 de octubre carecieron de reset; el primer éxito 3A siguió reset verificado en ambos CC. Un reset destruye RAM/driver/RF Core de CC mientras `RADIO_INIT` sólo reinicializa el software previsto. El patrón “reset inicial→éxitos persistentes” es compatible con liberar un backend atrapado. Explicaría Sub-GHz y 2440 por compartir proprietary.  
**En contra:** no hubo A/B desde el mismo estado fallido. 3C, 3D y 3E funcionaron sin reset; 4A funcionó tras power-cycle y sin `boot/exit`. La fuente ya contenía el arreglo de handle persistente.  
**Conclusión:** **PLAUSIBLE**, candidato específico número uno, pero no prueba reset necesario ni causa exacta del estado.

### H8 — firmware

**A favor:** no hay hashes/binarios CC ni manifests; versión `1.0.0` no distingue builds. COM33 pasó de RP v3.1.0.0 a v3.1.0.1 antes de recuperación.  
**En contra:** no hay diff relevante de FeralRF/CC entre `0178721c` y `ba27fbeb`; el diff RP sólo cambia texto de identificación. Ningún flash está documentado entre fallo y éxito.  
**Conclusión:** binario distinto **INSUFFICIENT EVIDENCE**; una regresión identificable en fuente está **CONTRADICTED**.

### H10 — interpretación

**A favor:** ACK TX se emite al encolar, antes de `RF_runCmd`; stats RX y `crc_ok` usan el mismo parser; cero stats no demuestra ausencia de energía.  
**En contra:** la ausencia de todo `RX_ITEM` y el control concurrente sí demuestran cero entrega host-visible en esas ventanas.  
**Conclusión:** **STRONGLY SUPPORTED** como límite interpretativo. No es una causa física y no invalida el 0-eventos observado.

### H11 — cuatro vs seis bytes

**A favor:** los 24 fallos usan marcadores de seis bytes; los seis éxitos raw verificables usan cuatro. El smoke F29 histórico también fija cuatro bytes (`smoke_f29_subg_915.py:67-105`). `manual_preset_runner.py:30-38` prohíbe otra longitud. La correlación cubre el patrón completo disponible.  
**En contra:** el código host/CC y las estructuras TI están diseñados para longitud variable y aceptan seis bytes. No existe excepción, ACK diferente ni A/B directo. Los binarios ejecutados no están ligados a esa fuente.  
**Mecanismo esperado:** defecto de longitud variable en TX, framing o RX del binario real; ACK de encolado seguiría siendo exitoso. **Conclusión:** **PLAUSIBLE**, candidato número dos, con confianza moderada-baja.

## 9. Evaluación causal ordenada

| Rango | Mecanismo | Estado | Por qué ocupa este lugar |
|---:|---|---|---|
| 1 | Estado previo CC1352/RF Core/backend proprietary limpiado por reset/power boundary | **PLAUSIBLE** | coincide temporalmente con la primera recuperación y explica Sub-GHz+2440; falta A/B desde el estado fallido |
| 2 | Defecto dependiente de payload de seis bytes | **PLAUSIBLE** | única diferencia de datos perfectamente correlacionada con 24 fallos vs éxitos verificables; fuente estática lo contradice parcialmente |
| 3 | Diferencia no registrada del binario CC/build | **INSUFFICIENT EVIDENCE** | podría explicar comportamiento con fuente idéntica; ninguna identidad binaria permite probarlo |
| 4 | Estado físico U2/front-end/antena distinto | **PLAUSIBLE** | mecanismo real posible y no medido; éxito con `Band:0` y fallos en 2440 reducen especificidad |
| 5 | Entorno físico/distancia/alimentación | **WEAKLY SUPPORTED** | condiciones incompletas, pero el histórico incluye al menos una prueba cercana y una más separada |

Ninguno supera **PLAUSIBLE** como causa. No es válido escoger reset, banda o antena por descarte narrativo.

## 10. Hallazgos confirmados y preguntas críticas

| Pregunta | Respuesta basada en evidencia |
|---|---|
| 1. ¿Qué se hizo distinto? | Reset verificado al comienzo de 3A; payload 4 en vez de 6 bytes; helper/logging/ventanas; COM33 con RP 3.1.0.1; checkpoints aislados; entorno físico/binarios no equivalentes. 3E elimina el helper/ventanas como diferencia causal suficiente. |
| 2. ¿Scripts con argumentos/interfaces correctos? | Sí para las corridas FSK/GFSK comparables: Bridge COM33/88, Shell COM35/87, preset y dirección correctos. HF-01 carece de stdout literal y faltan exit codes. Mod 4/5/6 no representa las modulaciones nominales. |
| 3. ¿Se verificó banda Sub-GHz? | Lógicamente sí: transición y `Band:1`; eléctricamente no. |
| 4. ¿CC correcto con ruta externa incorrecta? | Sí, es posible: comandos CC y U2 son dominios distintos. ACK tampoco prueba que la configuración CC efectiva fuera correcta. |
| 5. ¿El helper pudo perder paquetes? | No como explicación única: HF-09 leyó concurrentemente y 3E hizo funcionar el helper legacy. Interacciones con otro binario/estado siguen abiertas. |
| 6. ¿RX estaba en modo previsto antes de TX? | A nivel host, sí. Físicamente, sólo existe ACK diferido; no hay readback de operación RF. La espera/concurrencia hace improbable un simple problema de orden. |
| 7. ¿Reset por preset era requerido? | No. 3C–4A muestran éxitos sin reset por preset. Un reset inicial pudo limpiar un estado histórico, pero no se demostró. |
| 8. ¿Octubre contradice explicaciones anteriores? | Sí: contradice lectura tardía como causa única, `band2` lógico obligatorio y reset antes de cada corrida. Debilita puertos, preset y regresión de fuente. |
| 9. ¿Diferencia más fuerte? | Como mecanismo: estado CC/backend previo limpiado al inicio, **PLAUSIBLE**. Como diferencia controlable no ensayada: payload 6 vs 4 bytes, también **PLAUSIBLE**. |
| 10. ¿Qué falta para una conclusión definitiva? | Binarios/build IDs; A/B 4/6 bytes; estado CTF/U2/J1; entorno físico; y, si existen, stdout original completo/exit codes del 8 y logs de los presets del 15–16. |

Hallazgos confirmados:

- El histórico registró cero objetos host, no sólo cero coincidencias de marcador.
- Los ACK no prueban emisión.
- El mejor preset/par de puertos sí es paramétricamente equivalente salvo payload y estados no registrados.
- No hubo cambios de fuente relevante entre los commits citados.
- El método legacy puede entregar 10/10 actualmente.
- `Band:1` no es necesario para el enlace cercano observado actualmente.

Preguntas no resueltas:

- ¿Qué imágenes exactas RP/CC estaban instaladas en cada fecha?
- ¿El binario ejecutado transmite/recibe correctamente seis bytes por proprietary?
- ¿Cuál era el estado físico de U2/CTF y la energía en J1?
- ¿Hubo operación previa capaz de dejar el RF Core/driver en un estado que `RADIO_INIT` no limpiara?
- ¿Fueron exactamente las mismas unidades, antenas, alimentación y geometría?

## 11. Pruebas discriminantes mínimas, sólo si se autoriza una fase futura

Antes de nuevo hardware, recuperar y hashear cualquier log/binario ya existente. Si la evidencia sigue ausente, el conjunto mínimo —no ejecutado en esta auditoría— es:

1. **A/B de longitud en un solo preset:** desde una condición inicial común y sin cambiar banda/configuración, `wisun_915_fsk_50k` con `DEADBEEF` (4 bytes) y con un marcador histórico exacto de 6 bytes, una corrida de diez cada uno. Registrar TX async, todos los objetos RX y stats. Esto discrimina H11 con mínimo cambio.
2. **Estado inicial vs reset:** sólo si se logra reproducir 0/10, preservar ese estado, repetir una vez tras reset CC explícito y verificado sin cambiar payload, geometría ni banda. Esto prueba H7; repetir resets rutinarios sin un brazo previo no sirve.
3. **Sólo si persiste 0/10:** medir CTF/U2 y energía correlacionada en J1 para una solicitud. Esto separa backend TX de ruta externa. No se necesita barrer presets.
4. Capturar build ID/hash RP y CC antes de cualquier alteración. Sin ello el resultado no cerrará H8.

## 12. Conclusiones y limitaciones

La campaña del 8 de octubre ejecutó pruebas OTA significativas: no estuvo bloqueada antes de comenzar y el RX fue solicitado antes de TX. Su resultado defendible es cero entrega host-visible en 24 corridas. No permite concluir cero emisiones físicas.

La campaña del 15 demuestra que el mismo par lógico de puertos y el mismo preset `wisun_915_fsk_50k` pueden entregar paquetes. Sus controles niegan que el fracaso anterior se explique únicamente por lectura tardía, falta de `band2`, ausencia de reset por preset, puertos intercambiados o un cambio rastreable de parámetros/fuente.

La explicación causal más consistente es un estado previo no capturado del CC1352/backend que el límite reset/power pudo remover, seguida por la diferencia de longitud de payload. Ambas siguen siendo hipótesis. La ruta U2 y los binarios instalados permanecen brechas importantes, pero no tienen evidencia suficiente para ocupar el lugar de causa.

La conclusión final es por tanto **causa raíz no establecida**. La comparación sí reduce sustancialmente el espacio causal y define dos A/B mínimos capaces de resolverlo sin otra campaña extensa.
