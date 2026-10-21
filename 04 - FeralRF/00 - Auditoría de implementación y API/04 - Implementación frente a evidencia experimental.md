# Implementación frente a evidencia experimental

## Regla de comparación

La fuente auditada describe `FeralRF@fbff1b7`; la evidencia de Fase I describe binarios identificados genéricamente como CC `1.0.0` y RP2040 `3.1.0.1`, sin hash binario. Una coincidencia entre mecanismo y síntoma aumenta la explicación técnica, pero no demuestra que el binario histórico contuviera exactamente HEAD.

Fuentes experimentales principales: [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/01 - Informe consolidado de evaluación experimental|informe consolidado]], [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/02 - Matriz histórica de experimentos y ejecuciones|matriz histórica]], [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/03 - Capacidades demostradas y catálogo de presets|capacidades y presets]] y [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/04 - Evidencia, procedencia y preguntas abiertas|procedencia]].

## Matriz declarada–implementada–observada

| Capacidad | Implementación actual | Evidencia Fase I | Evaluación |
|---|---|---|---|
| Init/info/stats | handlers completos; info constante | respuestas válidas | Implementado y demostrado; provenance insuficiente |
| IEEE RX | backend TI, metadata y cola | paquetes OTA válidos | Implementado y demostrado en condiciones ensayadas |
| IEEE TX RAW/FRAME | mismo TX raw con dos comandos | recepción peer positiva | Implementado y demostrado; FRAME no implica MAC |
| RX_STOP | ACK agendado, tráfico asíncrono intercalable | timeout con 3/4 `0x90` inesperados | Defecto/síntoma confirmado; causa final inconclusa |
| Prop FSK/GFSK | config genérica + RX/TX raw | actividad 10/10 en 902.2 MHz | Ruta RF demostrada; metadata y modulación nominal no plenamente verificadas |
| Metadata prop | parser con orden erróneo y canal residual | canal 25 y RSSI incoherente | Defecto apoyado por fuente + experimento |
| BURST/CONT | scheduler sin completion/conteo | peer recibió menos que solicitado | Implementado con limitaciones; no prueba número emitido |
| Presets | parámetros host hacia config genérica | 13 presets con actividad 10/10 | Configuración/actividad, no interoperabilidad de protocolo |
| CW/PRBS | post de comandos TI | ACK observado | Implementado, pero radiación/espectro no verificados |
| Banda | Cat-Shell cambia GPIO RP2040 | Sub-GHz funcionó en proximidad | Control lógico ejecutado; ruta eléctrica no certificada |
| Reset | Cat-Shell inferido por puerto | selección COM incorrecta una vez | Limitación de API confirmada experimentalmente |
| KillerBee | adapter + patch/dependencia externa | falló antes de hardware | Inconcluso para RF; defecto de integración/packaging |
| OOK | código especial y restricciones | no ejecutado | Implementado parcialmente, no validado |
| Criptografía | drivers y contrato dedicado | no cubierta | Implementado pero no validado experimentalmente |
| Jamming | sesión básica BLE/IEEE | no cubierta suficientemente | Experimental/no validado |
| Spectrum | no hay ruta extremo a extremo | no cubierta | Ausente, no “no validado” |

## RX_STOP y respuestas `0x90`

Fase I registró `Response timeout (ignored 3 unexpected response(s), last=0x90)` y después cuatro. `0x90` es `RSP_RX_PACKET`. La fuente establece:

1. `RX_STOP` llega como comando síncrono y su ACK se encola.
2. Ya pueden existir paquetes en data queue TI, cola RadioIF (8) o cola de salida (32).
3. `DataTask` asigna a cada RX una secuencia rotativa no reservada.
4. `_read_response()` descarta el RX con secuencia distinta de la esperada; sólo preserva generalmente secuencia cero.
5. `stop_rx()` cuenta/ignora RX y reintenta.

Esto explica por qué el host reportó `0x90` inesperados y perdió esos paquetes. No explica de forma concluyente la ausencia del ACK. Hipótesis compatibles incluyen cola de salida saturada (la respuesta se descarta), acceso concurrente no protegido a `PacketQueue`, pérdida de bytes en el ring RP2040, ejecución con otro binario o una interacción temporal todavía no instrumentada. Ninguna se eleva a causa raíz.

El resultado histórico debe leerse como “stop no obtuvo su ACK dentro del contrato host bajo carga RX”, no como prueba de que el CC nunca procesó `RX_STOP`.

## Metadata propietaria

Fase I vio paquetes a frecuencia solicitada 902.2 MHz con `channel:25` y RSSI de signos/magnitudes poco plausibles. La fuente aporta dos explicaciones directas e independientes:

- **RSSI/CRC/timestamp:** `RadioIF_startSub1ghzRfBackend()` pide RSSI, timestamp y status en ese orden; `RadioIF_processPropPackets()` los indexa status, RSSI, timestamp. La referencia del SDK y su fuente Sidewalk colocan `header,payload,CRC,RSSI,timestamp,status`. El parser está desplazado.
- **channel 25:** `pkt.channel = s_channel`; la frecuencia propietaria no actualiza `s_channel`, y seleccionar PHY con canal cero preserva el canal anterior. El valor es estado lógico residual.

Por tanto, la interpretación de Fase I se cualifica: esos campos no caracterizan de forma confiable la señal propietaria actual. La observación OTA aún apoya recepción, pero RSSI, CRC filtering y timestamp requieren nueva evidencia tras corregir el contrato en una fase futura.

## `SET_PROP_CONFIG` y presets

La respuesta positiva de 13 presets demuestra que los comandos fueron aceptados y que existió actividad RF detectable en la configuración ensayada. No demuestra que cada nombre de preset corresponda a su modulación/protocolo nominal.

El handler sólo valida longitud, escribe estructuras y ACK. En especial, `mod_type=4/5/6` es reservado en el SDK actual. Por ello los resultados de presets asociados a MSK/4-FSK/4-GFSK no deben presentarse como validación física de esas modulaciones. Pueden reflejar comportamiento residual/default del RF core u otra configuración aceptada; se necesitaría captura de espectro o receptor independiente que decodifique el modo esperado.

La actividad W-MBus-like tampoco demuestra Wireless M-Bus: no hay stack, state machine ni decodificación del protocolo.

## BURST y CONTINUOUS

El código concuerda con que ambos comandos reciban ACK y produzcan actividad. También muestra por qué el conteo de un peer no es una medida exacta de lo solicitado:

- ejecución de una unidad por poll/deadline;
- TX RF bloqueante dentro de la secuencia;
- fallo interno termina la operación sin evento;
- no hay contador emitido ni completion;
- el peer puede perder tramas por RF independientemente de la emisión.

Fase I puede sostener “se observó actividad repetida” y el count recibido, pero no “firmware emitió exactamente N” ni “falló después de M”. La semántica de API no ofrece esa evidencia.

## TX, modos de prueba y ACK

RAW/FRAME cuentan con confirmación OTA peer y son las pruebas más fuertes de TX funcional. El ACK por sí solo sólo fue aceptación diferida. CW/PRBS llegan más lejos antes del ACK —abren/ajustan/postean—, pero todavía no prueban salida física ni características espectrales. La conclusión de Fase I de “control aceptado” es correcta; cualquier afirmación de potencia/forma de onda sería excesiva.

## Banda y reset

La fuente RP2040 confirma que `band1/2/3` sólo conmuta CTF1/2/3 y que `boot/exit` manipula boot/reset/baudrate. La fuente CC1352P7 no conoce esos comandos. Esto coincide con los procedimientos manuales de Fase I y desmiente cualquier interpretación en que `set_phy()` seleccione el front-end externo.

La actividad Sub-GHz cercana apoya que alguna ruta física útil estuvo activa; no identifica eléctricamente la tabla GPIO↔conector/antena. La selección incorrecta COM90/COM87 valida la crítica al cálculo `bridge+2`.

## KillerBee

Fase I no alcanzó hardware: la dependencia declarada no resolvió y la integración requiere un patch/fork. La fuente confirma un adapter plausible, pero también una pérdida de errores por `read_one_packet()`, suposiciones de FCS y ausencia de resultados de inject. La clasificación correcta es:

- packaging/entorno: limitación demostrada;
- adapter: implementado con contrato incompleto;
- sniff/inject OTA mediante KillerBee: no verificado;
- radio CC: no debe declararse defectuoso a partir de RF-04.

## Historia y documentación

El historial contiene una retirada deliberada de BLE stack (`785a093`, `d98e1ce`), mejoras a async errors (`d3a69d6`, `a6b3f5c`) y ajustes de port discovery (`5e0f9d5`). Esto muestra que documentos o experimentos de revisiones anteriores pueden describir superficies ya eliminadas. Las notas que llaman “stack 802.15.4 en Python” o “Wireless M-Bus estable” exceden el código actual: hay raw PHY/framing/presets, no pilas completas.

El parser propietario defectuoso proviene históricamente de `b27f6b38` (con ajuste de longitud en `384001f5`) según `git blame`; no es un cambio local no rastreado. Aun así, no se conoce el commit exacto del binario probado.

## Resultado de la correlación

Fase I y fuente concuerdan en las capacidades base: init, IEEE RX/TX y proprietary raw existen. La fuente aporta límites que el ACK y la actividad OTA no podían establecer. Los dos hallazgos experimentales mejor explicados son metadata propietaria y el origen de los `0x90`; sólo el primero tiene una violación determinista suficiente para llamarlo causa del valor reportado. El timeout final de `RX_STOP`, el conteo exacto TX y la ruta física de banda siguen necesitando instrumentación.

