# Auditoría técnica de validación FeralRF - EV ejecutadas

Evaluación técnica consolidada y canónica. Auditoría documental del 7 de octubre de 2026; no es una nueva sesión experimental. Incluye todos los registros disponibles hasta EV-14 y conserva íntegra al final la auditoría del 5 de octubre, cuyo corte llegaba a EV-11. Fuentes principales: [[FeralRF - Wiki técnica integral]], [[Arquitectura FeralRF]], [[Matriz de capacidades]], [[Protocolo y API Python]], [[FeralRF - Guía de validación experimental]], [[FeralRF - Matriz de pruebas]], [[Registro de validación FeralRF]] y los EV enlazados abajo.

## Dictamen de alcance

La documentación demuestra un sistema de control host–firmware funcional en varios casos concretos: cinco reconexiones entre procesos, exclusión RX/TX con error 0x05, ocho selecciones PHY y aceptación declarada de 27 presets (18 con salidas individuales). Hay recepción local IEEE repetida y TX IEEE RAW/FRAME observado por una segunda placa. El observador reportó 99 registros coincidentes CRC-válidos con CONTINUOUS a intervalo cero; apoyan actividad repetida, sin establecer el conteo físico exacto. Estos resultados permiten afirmar funcionamiento parcial de esas rutas, bajo las condiciones registradas.

No permiten afirmar validación integral de FeralRF multi-PHY, de los 27 presets por aire, de los protocolos superiores, de los límites/crypto actuales, del cese físico de TX, ni de estabilidad prolongada. EV-12 muestra una anomalía de conteos del observador en los casos de intervalos positivos ensayados; la semántica de repetición/conteo del DUT es INCONCLUSIVE; EV-12/13 muestran timeouts de RX_STOP; EV-04 demuestra selección incorrecta de Shell en el mapa inicial. La causa de la anomalía de repetición y de los timeouts permanece abierta. EV-14 prueba tres casos host y una transición mínima, pero no reproduce un fallo RF físico.

## Alcance, disponibilidad y jerarquía de evidencia

Se leyeron completos los 25 Markdown de las tres carpetas, incluyendo la extensión `.MD` y las variantes con `#`/`##` en el nombre. Se siguieron y leyeron las notas de firmware RP2040, Catnip CLI y arquitectura de CatSniffer v3.1 fuera de estas carpetas para interpretar dependencias; no se reorganizaron. Se inspeccionaron referencias de estado/inventario fuera del alcance; sus conclusiones antiguas no son prueba actual.

La evidencia disponible en el Vault consiste en texto, comandos, stdout, tracebacks, tablas históricas y análisis estáticos previos. No se encontraron archivos adjuntos de logs crudos, PCAP, imágenes RF, binarios o medidas independientes en las tres carpetas. Las rutas `C:\Users\Support\...`, repositorios de firmware, scripts, tests y enlaces externos mencionados no fueron reejecutados ni verificados contra hardware en esta auditoría. Hay repositorios hermanos en el disco, pero no se inspeccionaron como una nueva fuente de implementación: presencia no demuestra correspondencia con el firmware cargado. No se verificaron fuentes web.

Se distinguen cuatro estratos de fuente, independientes del modelo A–F: (1) intención/contrato en Wiki/arquitectura/guía, (2) implementación reportada por análisis estático con commit de referencia, (3) observación experimental literal o narrada en EV, (4) interpretación/hipótesis de esta auditoría. Las salidas literales sustentan mejor la reproducción que un resumen; mocks sustentan contrato host, no RF; datos históricos de validación oficial sin artefactos/commit no se trasladan al montaje actual. No se usa Git como prueba experimental.

## Evidencia, estado, confianza y procedencia

Modelo canónico: [[FeralRF - Matriz de pruebas#Modelo de evidencia A–F]]. A: RF física independiente; B: comportamiento directo del dispositivo; C: control/API (mocks identificados); D: fuente/documentación; E: inferencia/hipótesis; F: dimensión no evaluada. El nivel no es PASS/FAIL, confianza ni procedencia. Un observador FeralRF físico separado aporta A para el transmisor aunque comparta implementación; entrega local ambiental del DUT aporta B. D no acredita que el binario cargado ejecute la fuente referenciada.

Corrección aprobada del segundo pase: se preserva la evidencia experimental; cambian únicamente interpretación, estados acotados, clasificación y prioridades. EV-05 tiene crc_ok=True en el primer paquete de las tres corridas, no en todos los paquetes. Un umbral receptor FAIL puede coexistir con una semántica DUT INCONCLUSIVE.

## Organización canónica y preservación

| Función | Documento canónico | Regla |
|---|---|---|
|Definición general|[[FeralRF - Wiki técnica integral]]|Enmienda actual distingue definición y corte histórico|
|Arquitectura/contrato/capacidades|[[Arquitectura FeralRF]], [[Protocolo y API Python]], [[Matriz de capacidades]]|Análisis estático previo conservado; cobertura actual enlazada|
|Guía de intención/procedimientos|[[FeralRF - Guía de validación experimental]]|Precedencia operativa y cambios explícitos, nunca comandos antiguos asumidos universales|
|Cobertura y reconciliación de 38 ítems|[[FeralRF - Matriz de pruebas]]|Estados acotados y fuente de evidencia por fila|
|Cronología y evolución|[[Registro de validación FeralRF]]|Fechas explícitas y dependencias; identidad por épocas|
|Juicio técnico, prioridades y próximos tests|Este documento|Hallazgos separados de hipótesis y propuestas|
|Registros experimentales|EV-03/04/05/06/10/11/12/13/14 canónicos|17 secciones, alcance/confianza y notas originales íntegras|
|Complementos|EV-11 bloques 433/868; EV-12 preliminar|Mismo ID, fuentes complementarias/históricas; sin A/B ni doble conteo|
|Fuentes e historia|[[Pruebas y evidencia existente]] y [[Fuentes FeralRF]]/firmware/hardware/herramientas|No se confunde referencia con artefacto realmente disponible|

Los25 archivos se conservan. La consolidación reúne cobertura e interpretación en un canónico por EV-11/12, con enlaces a los registros de origen completos; no elimina archivos fuente. Se corrigieron nombres truncados, `#` iniciales, doble punto y `.MD`. Cada EV tiene 17 secciones: contexto; objetivo; capacidad; condiciones; esperado; ejecución; observado; evidencia; comparación; interpretación; límites; resultado; confianza; preguntas; seguimiento; trazabilidad; original. Las condiciones ausentes se indican desconocidas, no se infieren de ejemplos de la guía. Los originales de EV, matriz, registro y auditoría se mantienen literales en bloques de texto, con nombre previo y SHA-256; las notas de definición/guía/fuentes conservan su cuerpo previo y reciben una enmienda explícita.

## Modelo de capacidades reconstruido

FeralRF reemplaza la aplicación del CC1352P7, no el firmware del RP2040 ni el SX1262. El host usa `feralrf.Radio`; el puente RP2040 expone tres CDC y transmite UART a921600, 8N1, sin flow control. Shell a115200 es una dependencia de recuperación/control externo; los comandos de banda/CTF del RP2040 no son comandos de la API FeralRF. La documentación apunta al target P7/x7 y SDK 8.30.01.01 con TI-RTOS7/SYSBIOS. V2/P1/SAMD21 no tiene target FeralRF explícito; compartir familia CC13xx no prueba compatibilidad.

El contrato comprende selección de ocho PHY (BLE1M, BLE2M, Coded S8, Coded S2, IEEE, sub 868, sub 915 y propietario genérico), configuración de canal/potencia y parámetros propietarios, RX con metadatos, RAW/FRAME, BURST, CONTINUOUS de paquetes, CW/PRBS15/32 y stops. Los presets cubren varias bandas/modulaciones y nombres de protocolos; PHY7 no obliga a que todos sean GFSK. W-MBus/Wi-SUN/Sidewalk, Zigbee/Thread/Matter/6LoWPAN describen compatibilidad de PHY/raw o requieren herramientas/pilas externas; no acreditan una pila superior en FeralRF.

La trama documentada antes de COBS tiene ID1/SEQ1/LENLE2/payload hasta 255/CRC16LE2 (máximo 261 bytes), delimitador 00, CRC, polinomio 0x1021/init 0xFFFF, sin fragmentación. Las respuestas síncronas correlacionan SEQ; los errores asíncronos usan 0 y compatibilidad FF. RX_PACKET tiene contador propio según arquitectura/API; una frase de Wiki que atribuye SEQ0 a todos los eventos se conserva como discrepancia pendiente. Límites efectivos documentados: TX125, RX emitido 239, BLE AdvData 31, SHA/TRNG240. No se demostraron las fronteras por hardware actual.

La documentación estática describe ACK previo a completitud de TX y ausencia de TX_DONE; FRAME alias RAW con potencia almacenada; BURST count u16/interval u32 y cancelación sin error explícito en algunas rutas; CONTINUOUS repite paquetes, no CW. Los dos tasks y colas estáticas, drenaje de 8 paquetes RF/poll y 4 salidas, profundidad output 32 y pérdida sin backpressure son limitaciones documentadas; no son automáticamente la causa de los síntomas actuales. Estadísticas RX en cero no reflejan necesariamente todos los descartes de salida.

Crypto documentada: TRNG, AES128 (ECB/CTR/CBC/CCM/GCM según API), SHA256, ECDH P256/Curve25519 y ECDSA P256; Curve25519 no implica ECDSA y RSA está pendiente. Integración KillerBee depende de paquete/parche externo. BLE passive advertising/channel hop existe documentado, pero no está validado por la simple selección PHY. Jamming continuo es experimental; reactive/pattern sólo IDs reservados. Spectrum/RSSI scan no tiene ruta end-to-end completa. MIOTY/TSUNB requiere soporte CPE que un preset no aporta. RSA/AIS/802.15.4g/HighPA están pendientes o limitados; DIO29 no enrutado y +5/+14 son preguntas de especificación/medición, no una potencia certificada. GAP/GATT/conexión/active scan BLE fueron retirados deliberadamente el 20-7-2026.

## Discrepancias y autoridad por pregunta

| Fuentes en desacuerdo / qué dicen | Autoridad y razón | Resolución experimental / residual |
|---|---|---|
|Auditoría 5-10, matriz previa, Wiki con corte 5-10: EV-12/13/14 pendientes; notas posteriores: ejecución|El EV literal posterior acredita ejecución; plan/auditoría acreditan intención/corte anterior|EV-12 tiene fecha 6-10;13/14 ejecutados según salidas y dependencias, día desconocido; no implica validación completa|
|Guía mapa COM88 DUT/COM31 peer; EV-12 COM33 DUT/COM88 RX tras sustitución|Mapa del EV es autoridad para esa ejecución; guía describe montaje previo|Épocas reconciliadas; identidad física y binarios no resueltos|
|Preliminar EV-12 PASS4/4; informe OTA fallos de conteo|Ambos tienen distintos observables; usar OTA para afirmar RF|ACK ocurrió; repetición no satisface criterios positivos ensayados; no hay contradicción de las salidas|
|Tres EV-11 mismos títulos;868 cuerpo 6/8 y luego 8/8; cierre 27/27|Contenido y agregados del registro definen bloques/revisión, no IDs nuevos|Seis 433+ocho 868+dos 169+nueve 902/915+dos 2440; sólo 18 stdout individuales|
|Auditoría histórica atribuye EV-05 a tráfico Zigbee; formal EV-05 sin bytes ni peer controlado|Salidas formales son autoridad para observación; atribución requiere prueba adicional|RX local sí; emisor/stack no demostrado. EV-14 bytes no repara capturas anteriores|
|EV-04/diagramas GPIO15 frente a overlay GPIO3/reset GPIO2 y referencias PCB|Fuentes estáticas de revisión específica deben identificar binario/placa; prueba funcional no mide pines|Reset manual observado; revisión/pines exactos permanecen sin reconciliar|
|Arquitectura/API: RX SEQ contador; Wiki: eventos SEQ0|Contrato/código de referencia reportado es más preciso para RX; sin nueva inspección no se decide conducta universal|Mocks EV-14 sólo errores SEQ0/FF; no resuelven SEQ real RX|
|Wiki no método público `get_info`; guía lo menciona en prosa|Métodos documentados de API y llamadas efectivas a `init()` fijan lo realmente usado|INITincluyeGET_INFO; invocación pública separada no verificada en esta auditoría|
|RF_open descrito genéricamente lazy; variantes 433 y excepciones jam/runCmd descritas en otras notas|Distinguir rutas concretas del análisis estático; no universalizar comentario arquitectónico|No prueba de que esa ruta cause anomalía de intervalos|
|Tarjetas de la guía vs procedimientos: EV-21 1M/Coded 10/10 y 2M8/10 vs todas≥8/10;EV-41 diez vs tres ciclos;EV-44 burst 100vs40;EV-46 init+RF vs init|La guía establece precedencia del procedimiento operativo para ejecución futura; ambas intenciones preservadas|Ningún resultado anterior se “aprueba” cambiándole retrospectivamente el umbral|
|EV-12 plan de control 01020304 vs operativo marcadores/twopeer/conteos; fase actual varía umbrales 1→40/5/2|EV describe ejecución real; criterio revisado se identifica explícitamente|Resultados min 1 conservados como débiles; conteo fuerte incumplido en casos documentados|
|Guía wrapper F22 cambia reset;helper potencia+5;EV-13 adaptado 0|Comando efectivo es autoridad de condición; wrapper no prueba cambio de potencia|EV-13 no ejecutó F22 directo; auditar ambos puertos y potencia antes de reutilizar|
|Plan preliminar de instrumento sin establecer;EV-13 declara disponible pero diferido|Declaración posterior establece disponibilidad reportada, no medición|Modelo/calibración desconocidos; no afirmar indisponibilidad absoluta ni cese medido|
|FeralRF v2.0, FW GET_INFO 1.0.0, Python 0.3.0|Son componentes/constantes distintas; serial FERALRF1 no identidad USB|Registrar todas y hashes; no concluir incompatibilidad sólo por versiones distintas|
|Fuentes de herramientas main bc80979/3.3.2.1 vs guía fix/CLI_control 126f13b/3.3.3.0|Cortes/ramas distintos; EXE e import local pueden no ser mismo entorno|Preflight reporta EXE3.3.3.0; dependencia `rich` ausente en checkout; no defecto RF demostrado|
|Capacidad 2440 pending README vs 10/10 histórico; OOK/433/MIOTY fallos vs control EV-11|Historia y control son dimensiones distintas; nueva OTA requerida|Dos2440 control no resuelven la contradicción RF; OOK/MIOTY ni se probaron actual|
|422 passed/1 skip del baseline host y 9/9 crypto histórico vs necesidad HIL actual|Son antecedentes, no nuevas medidas actuales;EV-14 sólo 3 mocks reportados|No sumarlos como tests físicos actuales ni como rerun de esta auditoría|

La nota externa al alcance `Estado y siguientes pasos` contiene un corte anterior que decía que no había pruebas físicas; permanece sin reorganizar y no debe interpretarse como estado de esta campaña. Nombres `.txt` en la auditoría antigua son referencias de exportación sin archivo `.txt` presente; las referencias actuales apuntan a los Markdown canónicos. Los enlaces de recursos son referencias de origen, no evidencia de consulta o ejecución nueva.

## A. Alcance actualmente respaldado

| Afirmación permitida | Base | Nivel | Estado, alcance y confianza |
|---|---|---|---|
|El host reabre e inicializa entre cinco procesos sin error registrado|[[EV-03 — Reconexión limpia entre procesos]]|C|PASS; COM88, cinco procesos, no unplug/misma instancia; High|
|Boot/exit manual recuperó INIT sin USB cycle; selector COM90≠87|[[EV-04 — Reset y reinicialización]]|B/C|Una secuencia manual; selector FAIL; API reset no ejecutado; global NOT FULLY VALIDATED. High secuencia/Medium generalización|
|RXIEEE25 entrega paquetes en tres ventanas30s; primer paquete CRC-válido en cada una|[[EV-05 — Primera observación RF IEEE]], EV-12/14|B local; A observador12|EV-05 PASS local. High entrega/Medium atribución; sin bytes completos, correlación independiente del emisor ni rendimiento RF|
|Solicitud TX tras RX_START fue rechazada0x05 y control reanudó|[[EV-06 — Exclusión RX y TX y recuperación de estado]]|C|PASS rechazo/recovery; no ausencia RF medida; High|
|Ocho secuencias PHY fueron aceptadas con reset entre filas|[[EV-10 — Matriz de PHY por control]]|C|PASS control; no transición física ni RF de ocho PHY; High|
|Control reportado en 27 presets,18 con transcripción y nueve sólo resumen|[[EV-11 — Control de presets propietarios]] y complementos|C|PARTIAL consolidado; High18 y menor auditabilidad nueve; RF no validada|
|Marcadores asociados a RAW/FRAME IEEE fueron recibidos por segunda placa|[[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]]|A/C|PASS entrega OTA acotada, IEEE25/0dBm configurado; negativo RAW; conteo exacto no establecido/completitud/frecuencia/potencia. High entrega|
|CONT0 reportó 99 registros coincidentes CRC-válidos; casos positivos ensayados reportaron un match|EV-12|A/C; E conteo/causa|PARTIAL; actividad repetida apoyada, conteo físico exacto no establecido; semántica DUT INCONCLUSIVE. High registros/Low causa|
|CW/PRBS15/32 y dos stops idle aceptados|[[EV-13 — CW PRBS y TX_TEST_STOP por control]]|C|PARTIAL global; PASS control0dBm/0,3s host; High ACK, onda/cese sin medir|
|Transición mínima BLE1M→IEEE sin reset entregó paquetes|[[EV-14 — Eventos RF asíncronos y firma RX]]|B/C|PARTIAL; tres declaradas/un stdout; Medium global; no matriz lifecycle|
|Tres casos SEQ/error host cumplen contrato mock|EV-14|C host/mock|PASS subconjunto FakeSerial; no error RF físico; High|
|Sin error ni coincidencia exacta8e89be en ventanas registradas|EV-14|B|Observación negativa acotada; INCONCLUSIVE para eliminación del fallo; High registro/Low exclusión|

Se demuestra un enlace físico limitado de paquetes entre dos endpoints FeralRF, sin certificar interoperabilidad externa ni todas las propiedades IEEE. El resultado global de EV-14 sigue NOT FULLY VALIDATED para error RF real/firma; sus subsets positivos se mantienen.

## B. Alcance pendiente o sólo indirecto

La [[FeralRF - Matriz de pruebas]] enumera todas las capacidades, incluidos RX/TX de BLE2M/Coded, sub 868/915,433,169,2440 y modulaciones; advertising/hopping; presets efectivos; interoperabilidad WMBus/Wi-SUN/Sidewalk/emulación; límites wire/API; crypto HIL; KillerBee; cese CW/PRBS/CONT; cambio de estado, bandas y reinit; soak/colas/SEQ wrap; errores RF reales; formato de flasheo y compatibilidad de placas. Nada de esto se cierra con ACK o un preset presente. Scans/reactive/pattern/RSA/AIS/802.15.4g/MIOTY tienen implementación o contrato insuficientes y deben distinguirse de funcionalidades implementadas pero no evaluadas. BLE stack retirado queda fuera del alcance actual.

## C. Problemas confirmados por la evidencia disponible

La severidad expresa impacto del observable, no localización de causa. El defecto localizado es la selección Shell. Los demás síntomas no prueban un defecto de implementación RF concreto.

| Tipo / problema | Evidencia y EV | Impacto práctico | Repetición disponible | Severidad / confianza |
|---|---|---|---|---|
|Defecto host localizado: selectorCOM90 frente a ShellCOM87|EV-04 consulta y asociación manual|Reset automático no fiable en ese mapa; manual explícito disponible|Una consulta y un ciclo; reset API sobreCOM90 no ejecutado|Alta para automatización/High local|
|Fallo de confirmación extremo a extremo: RX_STOP timeout|EV-12:9 respuestas inesperadas; EV-13:5/4; sólo último ID 0x90 preservado|Host no obtuvo respuesta exitosa correlacionada dentro del timeout; estado RF desconocido|Varias sesiones, con 99 matches o2/11 paquetes reportados; carga interna desconocida|Alta para lifecycle/High timeout; causa no localizada|
|Anomalía experimental: umbral receptor BURST incumplido|EV-12:40/25000 y5/250000 reportan1; host abierto3s también1|No sirve todavía como generador de conteo conocido; requisito físico DUT INCONCLUSIVE|Varias condiciones; falta contador físico independiente|Alta para validación/High observable|
|Anomalía experimental: CONT positivos ensayados reportan1 frente a99 registros coincidentes CONT0|EV-12:CONT1 y250000; 25000 sóloBURST|Cadencia/conteo físico no demostrado; no defecto scheduler confirmado|CONT1 tres declaradas/un stdout|Alta para validación/High registros; Low causa|

Limitación de evidencia, separada de defecto de producto: nueve presets sólo resumidos, hashes/fechas/lineaje/capturas incompletos. Debilitan reproducción, comparación y enlace al código, sin invalidar automáticamente las observaciones locales. Error documental corregido: el primer paquete es crc_ok=True en las tres salidas EV-05.

No son fallos del producto los errores de quoting/import, e.code=None del harness, marcador FRAME incompatible, FAIL esperado del negativo RAW, dependencias KillerBee/rich ausentes o umbral BLE exploratorio no alcanzado. Los issues RF históricos permanecen históricos.

### Cadena causal de RX_STOP

host request → serialization → transport → firmware handler → RF state transition → completion/error generation → response transport → host correlation → observed timeout.

| Etapa | Evidencia disponible | Punto no establecido en el intento fallido |
|---|---|---|
|Solicitud host|C: llamada/STEP/traceback stop_rx|Tiempos por intento y registro completo de retries|
|Serialización|D: ruta de API descrita|Bytes exactos, ID/SEQ/CRC de la solicitud|
|Transporte hacia dispositivo|C: otros comandos funcionaron|Entrega de esa solicitud STOP al firmware|
|Handler firmware|D: ruta documentada|Entrada efectiva al handler|
|Transición RF|Sin evidencia directa del caso fallido|Estado/handle/backend después del request|
|Generación completitud/error|ACK en otros intentos; contrato D|ACK/error generado en este intento|
|Transporte de respuesta|C: diagnóstico host de respuestas inesperadas|Stream bidireccional completo y enqueue/dequeue/drop|
|Correlación host|C: timeout y último ID 0x90|Existencia de ACK con otro SEQ, descartado o tardío|
|Resultado|C: no respuesta exitosa correlacionada dentro del timeout|Resultado físico de RX_STOP|

La evidencia se interrumpe antes de localizar el request fallido y reaparece en el diagnóstico host. Último ID 0x90 corresponde a RX_PACKET según contrato D, sin identificar todos los frames inesperados. Pueden ser eventos previos almacenados, no RX físico posterior. Totales2/11 o99 no miden tasa instantánea, backlog ni ocupación de colas. No se confirma fallo físico, pérdida de ACK, defecto firmware ni problema de correlación.

## D. Hipótesis pendientes y pruebas discriminantes

| Hipótesis | Evidencia a favor | Evidencia en contra / alternativa | Lo que falta | Experimento discriminante |
|---|---|---|---|---|
|Reloj usado por scheduler no avanza coherentemente con TI-RTOS|CONT0 reportó 99 coincidencias; positivos ensayados reportaron1; análisis previo sobre `ControlTask_getTimeUs`/SysTick|No se leyó reloj interno; backend puede abortar o el observador puede perder paquetes|now/next_due/remaining/retornos RF evento efectivo y reloj RTOS|Diagnóstico 12 instrumentado antes de cambiar la fuente de reloj|
|Scheduler/backend cancela o falla después del primer TX sin aviso host|ACK programa 40/5, observador ve1; cancelación silenciosa documentada estáticamente|CONT0 reporta99 registros; no log de fallo/cancelación real|Retornos y contador RF completado; errores asíncronos capturados|Comparar intervalos con log del scheduler/RF y medición independiente|
|Observador compartido o ruta RX causa conteo insuficiente|Sin conteo RF independiente calificado; observador separado con misma implementación|99 registros CONT0 y host abierto3s debilitan explicaciones simples; positivos espaciados reportan1|Tercer receptor/captura RF y roles invertidos|Separar conteo emitido/recibido bajo mismas condiciones|
|ACKRX_STOP perdido/descartado por cola de salida o correlación host/SEQ|Timeout con ID 0x90 inesperado, límites de colas documentados|También pocos paquetes reportados, sin carga interna medida; EV-14STOP ACK; no evidencia de enqueue/dequeue/wire|Trama TX/RX serial cruda, SEQ, CRC, timestamps y estado FW|Saber si ACK se generó/envió/recibió pero descartó; luego carga controlada|
|Backend no aplica realmenteRX_STOP/TX_STOP|Timeout o ACK no tiene medición física de cese|ACK observado en otras corridas; ausencia ACK no prueba recepción RF continua|Estado RF interno y prueba física post stop|RX: estado/handle/callback nuevo bajo estímulo verificado; TX: cese por observación RF independiente|
|Estado de CTF/U2 o GPIO explica limitaciones de bandas/potencia|Dependencia RP2040 documentada fuera de API y antecedentes 433/unidad débil|No medida CTF ni falla OTA actual multibanda; no puede inferirse defecto de ruta|Revisión/overlay/estado band y medición RF|Comparar estado CTF registrado y frecuencia/potencia por preset|
|Quedan fallos intermitentes de cambio PHY/reinit o firma sintética|Deadlocks/firma históricos|EV-14 transición mínima sin error/firma; no inevitabilidad|Ciclos completos/errores estimulados/bytes wire y binario fijado|EV-41/42/46 y EV-14 con control de estado, sin extrapolar una ventana|
|Potencia por defecto RAW −128 y clamping modifica RF|Valor documentado y posible clamping del backend|EV-12 potencia explícita 0; no potencia medida|Contrato default y medición conducida|Validar defaults por API y medida después de fijar montaje|

## E. Preocupaciones de arquitectura e implementación

La programación periódica y la observabilidad de completitud son centrales para RAW/BURST/CONT y herramientas de carga. La divergencia ACK/conteo hace prioritario medir el recorrido completo solicitud→scheduler→comando RF→finalización→señal. La documentación sobre SysTick no basta para cambiar el reloj sin validar que el fallo está allí. Antes de corregir, fijar binarios e instrumentar estados/retornos con coste y perturbación registrados.

Los timeouts STOP requieren separar cuatro capas: recepción de la orden por firmware, cambio de estado RF, generación/salida del ACK, y correlación del host. La profundidad de colas y ausencia de backpressure justifican estudiar pérdida, pero los pocos reportes no miden carga interna ni excluyen saturación; ésta tampoco es una explicación confirmada. El contrato actual no debe prometer completitud RF a partir de ACK; un nuevo evento/counter sólo sería una mejora propuesta que debe especificarse y verificarse.

El reset depende de la asociación de interfaces USB y revisión del puente; una suma COM fija no satisface esa asociación. El frontend CTF y los pines HighPA no pertenecen al mismo plano de control que la selección PHY del CC; documentar y verificar ese estado es necesario para interpretar bandas/potencia, sin declarar una falla física aún no medida. Workarounds históricos de `RF_close`, `RF_runCmd` y OOK son riesgos de lifecycle reportados, no causas actuales probadas.

### Clasificación del análisis de código

| Hallazgo | Clasificación aprobada | Relación experimental pendiente |
|---|---|---|
|ACK previo a trabajo RF diferido|D: comportamiento reportado en fuente referenciada|EV-12/14 son compatibles; medir orden handler/backend/wire en build trazado|
|Cancelación backend BURST/CONT sin aviso|D: ruta reportada; riesgo arquitectónico|EV-12 no prueba que ocurrió; registrar retorno RF y razón de cancelación|
|SysTick/timebase no progresa como supone scheduler|E: hipótesis runtime apoyada por descripción D|EV-12 requiere now/next_due, invocaciones y referencia temporal independiente|
|Colas limitadas y STATS incompletos|D: limitación reportada; riesgo|Ningún EV prueba pérdidas responsables; trazar generación/entrega/drop/wire|
|ACK RX_STOP perdido o mal correlacionado|E|Trama request/ACK con SEQ y etapas firmware del intento fallido|
|Backend no detuvo RX|E|Handle/estado/callback fresco bajo estímulo externo verificado|
|Potencia RAW default−128/resolución|D/E: código descrito e implicación física|EV-12 usó potencia explícita; medir default vs explícito|
|CTF/GPIO/revisión, RF_close/OOK/jam|D dependencias/riesgos históricos; E causalidad actual|Reproducción localizada con build/estado conocidos|

No se releyó código externo ni se vinculó el binario cargado. “Confirmado por fuente” significa en el análisis de referencia (D), no causa experimental confirmada. Los tres mocks EV-14 no prueban RX_PACKET backlog con una respuesta STOP real. Ninguna causa raíz RF se da por confirmada.

## F. Debilidades del método

Faltan manifests por corrida y hashes del binario/API/bridge; serial constante confunde identidad; fechas iniciales y sustitución física incompletas. Numerosos ensayos miden sólo aceptación, con reset entre filas; eso valida poco el estado continuo. El smoke no captura todos los eventos/completitud; leer stats después de INIT puede reiniciar métricas. Contadores cero con carga pequeña no establecen pérdida cero.

EV-05 carece de bytes y correlación concurrente del emisor, que era un suplemento; conserva PASS+B local y CRC del primer paquete en tres corridas; EV-12 usa receptor mismo firmware y umbrales 1 originalmente insuficientes para repetición; nueve presets sólo tienen resumen. Corridas narradas no siempre tienen stdout individual. TX_STOP y CW/PRBS carecen de criterio físico medido. La comparación CW/BLE ambiental no tiene baseline controlado adecuado. Falta control negativo parametrizado de EV-05, baseline bidireccional, tasa emitida efectiva, distribución temporal y repetición definida antes de concluir.

Tests Host, históricos oficiales y ejecución local física se deben informar por separado. Criterios nuevos pueden revelar limitaciones del criterio anterior, pero no cambiar qué acciones se realizaron ni crear replicaciones. Bloqueos de dependencia y ausencia de implementación son estados de alcance, no fallos RF.

## G. Debilidades documentales y trazabilidad

Se corrigieron encabezados inexistentes/truncados, nombres con `#`, duplicidad de IDs inrol y matrices con estados anteriores. Se establecieron canónicos y enlaces. Permanecen discrepancias técnicas de SEQRX,GPIO/revisiones,potencia/defaults y métodos API que requieren inspección del binario real o código de referencia. La guía conserva tarjetas y procedimientos de épocas distintas; la enmienda explícita permite conocer precedencia sin borrar evolución. Los registros normalizados acotan atribuciones Zigbee, frecuencia/modulación efectiva y causa del scheduler a la evidencia.

La integridad textual se verifica contra una copia anterior de los 25 notas, incluyendo el Wiki que ya era un archivo sin seguimiento Git. Las referencias `.txt` y títulos anteriores dentro de transcripciones son texto histórico, no enlaces activos nuevos. Recursos externos no verificados se marcan como tales. El inventario detallado y mapa de nombres al final permiten rastrear cualquier cambio.

## H. Alcance y límites del proyecto

Actualmente aparece viable como plataforma de control y RF raw con algunas rutas IEEE y BLE locales y amplia selección/control PHY. Esa viabilidad es condicional a los montajes, estados y configuración registrados. No demuestra precisión RF, robustez temporal, interoperabilidad total, escalabilidad, capacidades crypto actuales ni compatibilidad de otras placas. Las funcionalidades retiradas/pendientes no deben mezclarse con las rutas que sí existen. Las dos anomalías operativas principales y la selección Shell incorrecta impiden convertir un smoke aceptado en una garantía de funcionamiento general.

## Ajustes priorizados

La revisión aprobada elimina asignaciones P0 incondicionales. No se demuestra invalidación global de evidencia. P1 puede designar un defecto importante o un vacío que limita interpretar una capacidad central; no significa causa conocida. Validar → localizar → corregir → revalidar.

| Prioridad / categoría | Problema / evidencia | Capacidad y motivo | Ajuste / beneficio | Validación posterior / confianza |
|---|---|---|---|---|
|P1 metodología|Endpoints/interfaces actuales y roles insuficientemente fijados; sustituciónEV-12/serial constante|Atribución de futuras pruebas|Verificar asociación USB/placa y registrar despliegue actual; evitar ambigüedad|Discovery y asociación funcional por placa; High vacío|
|P1 host/API|SelectorCOM90≠COM87; EV-04|Recovery/automatización, defecto localizado; manual disponible|Corregir discovery por identidad/interfaz, no suma COM; preservar workaround|Selección correcta y ciclos de recovery por enumeración; High local|
|P1 RF/metodología|Observador/conteo físico no calificados; EV-12|Distinguir emisión de entrega; generador aún no fiable para pérdidas RX|Calificar observación con requests individuales, negativos y roles; comparar captura RF/serial|Correspondencia eventos/capturas y pérdidas/duplicados; High vacío|
|P1 investigación firmware/RF|Repetición no demostrada en positivos ensayados; EV-12|BURST/CONT y generación de carga, semántica DUT INCONCLUSIVE|Reproducir y trazar scheduling/retornos antes de elegir corrección|Caso original y discriminante revalidados; High síntoma/Low causa|
|P1 protocolo/API/integración|RX_STOP no obtiene respuesta exitosa correlacionada en ciertos intentos; EV-12/13|Confirmación lifecycle importante|Wire bidireccional/SEQ, handler, estado RF, generación/salida ACK; localizar|Stops con estímulo controlado y evidencia por etapa; High timeout/Low causa|
|P1 RF/lifecycle|TX_STOP/TX_TEST_STOP sólo ACK; EV-12/13|Cese físico central no medido|Medir actividad/energía relativa a request/respuesta; distinguir backlog|Cese bajo criterio explícito y montaje calificado; High vacío|
|P1 contrato/metodología|ACK, completitud y conteo confundidos por harness|Validación de TX/errores|Precisar semánticas/criterios; evento TX_DONE/counter sería diseño posterior, no corrección ya demostrado|Criterios acotados y compatibilidad si se implementa; High gap|
|P2 procedencia histórica|Hashes/fechas/lineaje/manifests faltantes|Reproducción/regresión|Recuperar donde sea posible, marcar incógnitas, sin fabricar ni retirar observaciones|Artefacto+despliegue asociados; High vacío|
|P2 evidencia|Nueve presets902/915 sólo resumen; EV-11|Auditabilidad individual|Recuperar transcripciones o repetir con nueva procedencia conservando historia|Nueve registros individuales y tabla sin doble conteo; High vacío|
|P2 RF/PHY|Presets/bandas no caracterizados OTA; EV-10/11, CTF/potencia/defaults|Cobertura y parámetros/interoperabilidad|Expandir después de observación calificada; registrar frontend; no asumir stack|Capturas/medidas por configuración; High gap|
|P2 firmware/protocolo/integración|Lifecycle, bounds, soak, SEQ, crypto yKillerBee sin cierre actual|Robustez e integración|Cobertura existente con generador fiable/dependencias/vectores; fixes sólo localizados|EV existentes41–47/15/29–31 según alcance; High gap|
|P2 contrato/documentación|SEQ/GPIO/API/potencia/criterios secundarios discrepantes|Esperados y vínculo fuente/build|Resolver con artefacto/inspección/prueba, preservar variantes históricas|Contrato vswire/despliegue y consistencia; High discrepancia|
|P3 documentación/usabilidad/producto|Naming, conveniencia yroadmap|Poco impacto en interpretación del núcleo actual|Polish y alcance pending/retirado explícito; sin roadmap nuevo|Revisión de enlaces/contrato; High documental|

Una validación de capacidad RX con BURST no calificado debe esperar conteo emitido conocido. Ese bloqueo de un experimento dependiente no es un P0 incondicional de la campaña. Implementar un nuevo evento de completitud no está justificado como reparación antes de localizar la necesidad.

### Impacto de identidad Shell y procedencia

EV-04 detuvo el resetAPI antes de ejecutarlo enCOM90. EV-10/11 utilizaron COM87 explícito; smoke presets auto_reset=no. EV-12 documenta roles/puertos explícitos; EV-13 adaptó harness yEV-14 evitó el helper completo con la suposición. La posibilidad de actuar sobre interfaz/unidad equivocada no demuestra que ocurrió. Catnip y efectoCOM87→respuestaCOM88 apoyan asociación funcional; FERALRF1 no identifica unidad. El cambio de rolesCOM88 y la sustitución limitan comparación de placas entre épocas, no borran el enlace OTA observado.

| Falta | Validez histórica | Reproducción | Comparación de regresión | Fiabilidad futura |
|---|---|---|---|---|
|Hash FW y despliegue|No borra observaciones; impide atribuir a implementación específica|Exactitud limitada|Comparación build limitada|Importante antes de diagnóstico ligado al código|
|Hash/versión API|Respuestas quedan evidenciadas|Host exacto no fijado|Atribución host limitada|Importante para correlación/lifecycle|
|Hash/revisión bridge|Ruta funcional observada|Reset/transporte no fijados|Comparación de interfaz limitada|Importante al localizar esas rutas|
|Lineaje de placa|Local válido; comparación entreEV incierta|Unidad no recuperable con certeza|Confusión hardware posible|Resolver endpoints actuales|
|Fecha exacta|Habitualmente no invalida observable|Contexto limitado|Importa si cambia despliegue|Registrar prospectivamente|

Hash de archivo no demuestra instalación: necesita registro de despliegue o enlace adecuado por readback. La recuperación histórica completa no es prerrequisito de cada nueva observación; fijar la sesión actual es distinto.

## Próximas evaluaciones que reducen más incertidumbre

Camino crítico aprobado, sin crear EV ni ampliar el roadmap:

**Identidad actual → observación calificada → localización de repetición y STOP → caracterización RF representativa → cobertura más amplia.**

Repetición y STOP pueden investigarse en ramas después de calificar observación. No requieren reconstruir todos los manifests históricos.

| Paso / prioridad | Pregunta y motivo | Setup, controles y evidencia requerida | Resultado discriminante / nivel | Dependencia |
|---|---|---|---|---|
|1. Endpoints/recovery actuales — P1|¿Qué unidad/interfaz/build participa y recupera? Evita ambigüedad futura|Endpoints etiquetados; USB/location/interfaces; despliegue/API actuales cuando disponibles; Shell explícito; mismo estado. Comandos/salidas/tiempos completos|Asociación/recovery correctos:C/B. Selector discrepante:host localizado. Recovery fallido requiere traza, no asumir pin|Sin dependencia; EV-00/04 existentes|
|2. Calificar OTA/conteo — P1|¿Requests, paquetes RF y eventos entregados se distinguen fiablemente?|IEEE25/potencia configurada documentada; marcador por ensayo; sinTX; requests RAW individuales; roles; serial crudo; receptor/captura capaz de discriminar paquetes|Acuerdo valida observador acotado. RF presente sin entrega apunta aRX/salida. Duplicados invalidan conteo directo. A+C/B|Paso1; EV-12/20|
|3. Reproducir/localizar repetición — P1|¿SiguienteTX se agenda, intenta, completa, emite o pierde en observación?|CasosBURST40/25000,5/250000; CONT0/1/250000. Controles pareados por modo, estado/roles/potencia fijados y host abierto; primero reproducción sinfix, luego instrumentación mínima now/next_due/retornos con coste registrado|Reloj/due sin avanzar apoya timebase; fallo backend apoya cancelación; RF repetido sin reportes apunta a observación/entrega; llamadas repetidas sinRF requieren backend. A+B+C|1–2; no asumir causa común|
|4. STOP extremo a extremo — P1|¿Dónde falta confirmaciónRX y cesaTX físicamente?|Wire bidireccionalID/SEQ/CRC, handler/estado RF/generación ACK/colas; tráfico cero/conocido; callbacks nuevos vsbuffer; estímuloRX externo que continúa; observaciónTX independiente; no INIT antes de medir|ACK en wire sin match:correlación. ACK generado sinwire:entrega. Request sintransición:lifecycle. handle RX/callback fresco evalúa estado; RF despuésTXSTOP evalúa cese. C/B paraRX; A paraTX|1–2; alta carga sólo con generador verificado|
|5. RF representativa, luego ampliar — P2|¿Parámetros documentados se producen físicamente?|Instrumento/decoder calificado; frontend/montaje/roles; IEEE probado y CW/PRBS con settings registrados; BLE/sub-G/prop representativos. Archivar settings, raw, criterio; no inventar tolerancias|Onda/paquete medido acredita dimensión ensayada. Frecuencia/modulación/patrón incorrectos localizan mismatch. Sin señal con medición no calificada:INCONCLUSIVE. A|Observación/lifecycle fiables; luego plan existente|

Analizador espectral no prueba automáticamente bitpattern PRBS ni conteo de paquetes: hace falta demodulación/correlación/captura apropiada. Observador RF externo no confirma por sí solo que un receptor dejó de recibir internamente; RX_STOP requiere estado/handle/callback bajo estímulo verificado. cese TX sí puede observarse por aire.

Conservar solicitud, raw bidireccional, eventos completos, identidad/estado/trial, ajustes instrumentales y criterio explícito. Validar → localizar → corregir → revalidar. No se ejecutó ningún paso durante esta corrección documental.

## Trazabilidad de los 35 KI del plan

El inventario KI original queda preservado en la guía; esta tabla evita convertir sus afirmaciones históricas/estáticas en reproducción actual.

| KI | Situación tras cruzar todos los EV | Cierre / próximo paso |
|---|---|---|
|01 OOK lock|Documentado, no reproducido actual; excluido EV-11|EV-24 con recovery|
|02 deadlock al cambiar PHY|Histórico;EV-14 mínima transición exitosa|EV-41 ampliado, no cerrado|
|03 cambio 433/868|Workaround documentado; reset en cadena parcial|EV-42 con manifest|
|04 433 marginal|Sólo control actual 6 filas|EV-23 RF|
|05 OOK4330/10|Histórico, sin reensayo|EV-23/24|
|06 unidad TX868 débil|Histórico; identidad de unidades actuales insuficiente|EV-22 roles/HWID|
|07 HighPA/potencia|Estático/rango inconsistente, sin medida|EV-13/53 instrumento|
|08 prop 2440 pending vs 10/10|Dos de control no resuelve RF|EV-27 OTA|
|09 BLE2M workaround|Canal 9 control sí, OTA no|EV-21|
|10 RF_runCmdFS|Riesgo estático, no causa actual probada|EV-41/46 logs|
|11 RF_close|Riesgo estático, no causa actual probada|EV-41/46 logs|
|12 firma synth/error|EV-14 no firma/error en ventanas, objetivo de error real pendiente|EV-14 estímulo/bytes|
|13 colas/output drops|Estático;RX_STOPsíntoma no atribuido a cola|EV-43/44 ywire STOP|
|14 ACK≠TX|Confirmada brecha control/OTA y conteo EV-12|Observabilidad/counter/cese|
|15 Bridge+2|Discrepancia local COM90/87 confirmada EV-04|Discovery por interfaz|
|16 spectrum|Incompleto, no scan end-to-end|EV-51 depende implementación|
|17 jam por control|Sin criterio físico actual|EV-50 prerequisitos|
|18 reactive/pattern|IDs reservados, no prueba|Definir/implementar, no FAIL actual|
|19 MIOTY396|Preset y 0/10 histórico, no capacidad TSUNB actual|EV-52 depende CPE|
|20 WMBus N169|Dos de control no OTA|EV-25 equipo 169|
|21 RSA/AIS/15.4g|No implementado documentado|EV-53 depende del alcance/implementación|
|22 nombre de protocolo vs PHY|Ningún EV valida stack superior|Interop separada 20/25/26|
|23 BLE stack retirado|Retirado, restos no API|EV-54 no aplica|
|24 protocolo omite test/crypto/0x07|Discrepancia estática de contrato|Reconciliar protocolo con build;EV-13/30|
|25 versiones|Tres componentes;INFO constante|Manifest 01|
|26 pytest/path/KillerBee skip|Entorno histórico 422/1;EV-14 sólo 3 mocks|No contar skip como HIL|
|27 evidencia histórica desactualizada|Faltan artefactos/hash;actual también incompleto|EV-40 paquete de procedencia|
|28 bin vs hex|Limitación reportada, no flasheo actual|Preservar contrato; no afirmar una nueva prueba|
|29 límites|125/239/31/240 sin fronteras actuales ensayadas|EV-15/31|
|30 curvas/autenticación crypto|9/9 histórico no HIL actual|EV-30/31 vectores|
|31 RX_START ACK, error posterior|EV-14 mocks PASS;errorfísico no inducido|EV-14 HIL|
|32 CTF fuera API|Dependencia estática, estado actual no medido|EV-20/22/27 frontend|
|33 IDs Catnip no persistentes|Mapas y roles cambian;EXE/imports distintos|EV-00/HWID manifest|
|34 sniff auto flash/V2TI sniffer|Riesgo documentado, no acción actual|Preflight/herramienta y firmware específico|
|35 V2 no target|Compatibilidad no establecida|No inferir soporte P1 del P7|

## Información no reconciliable sin artefactos o aclaración

Fechas exactas anteriores y de EV-13/14; hashes y commit de FW/API/bridge efectivamente usados; identidad USB y linaje de placa sustituida; nueve comandos/stdout 902/915; dos repeticiones faltantes de CONT1/transition; reset 868→169; canal/duración/comando del negativo preliminar; pines/revisión y overlay efectivos; SEQ real de RX; contrato de `get_info` enla versión usada; default/clamp de potencia; inventario y calibración del instrumento y medición CTF. Las referencias a `.txt`/PCAP/logs/repos externos no presentes en el Vault no se verificaron. Ninguna de estas lagunas se rellenó con supuestos.

## Inventario de documentos, procedencia y nombres

Inventario cerrado antes de reorganizar:25 notas primarias. Títulos/fechas de esta tabla corresponden al origen; fechas de fuentes y auditorías no se convierten en fecha experimental. Las referencias actuales son canónicas. Las filas siguientes incluyen evidencia, propósito, alcance, estatus y relaciones; los hashes previos y originales completos están en los registros normalizados.

| Archivo anterior → actual | Título original | Propósito | Fecha/corte documentado | Capacidad/ID | Papel actual | Evidencia contenida | Relaciones | Solapamiento/discrepancia |
|---|---|---|---|---|---|---|---|---|
| `Arquitectura FeralRF.md` → [[Arquitectura FeralRF]] | FeralRF — Qué parte de CatSniffer reemplaza | Definición estática | 28-09-2026; fuente 22-07 | Arquitectura/RF/bridge | Canónica estática, alcance experimental previo | Análisis y referencias, no logs HIL actuales | Wiki,API,capacidad,fuentes | Solapa definición Wiki |
| `Matriz de capacidades.md` → [[Matriz de capacidades]] | Capacidades FeralRF — Cómo leer afirmaciones con rigor | Taxonomía y claims | 28-09-2026 | Todas capacidades | Canónica definición; coberturaactual enlazada | Tabla de implementación/tests/historia | Arquitectura,API,matriz de pruebas | No sustituye matrizde campaña |
| `Protocolo y API Python.md` → [[Protocolo y API Python]] | FeralRF desde la PC — Protocolo y API Python | Contrato host/wire | 28-09-2026 | API,SEQ,TX,crypto | Canónica estática | Comandos/límites/análisis previo | Arquitectura,Wiki,guía | Discrepancias SEQ/get_info/potencia |
| `Pruebas y evidencia existente.md` → [[Pruebas y evidencia existente]] | Evidencia FeralRF — Qué demuestran realmente las pruebas | Historia y calidad evidencia | 28-09-2026; historial de abril-mayo | RF,crypto,tests | Canónica fuentes históricas | Resultados reportados, sin artefactos adjuntos | Fuentes,matriz actual | No mezcla histórico con actual |
| `# EV-06 — Validación de exclusión R.md` → [[EV-06 — Exclusión RX y TX y recuperación de estado]] | EV-06 — Validación de exclusión RX/TX y recuperación ante `ERR_INVALID_STATE` | EV-06 experimento | No explícita; existeal corte 05-10 | Estado RX/TX | Canónico normalizado | Errores de harness+dos DUT válidos,0x05 | EV-05/10,API | No cuatro pruebas físicas |
| `# EV-11 — Presets propietarios, sól(1)..md` → [[EV-11 — Control de presets propietarios]] | EV-11 — Presets propietarios, sólo control | EV-11 cierre de campaña | No explícita | 169/902/915/2440 | Canónico consolidado | 4 stdout+9 resúmenes, totals 27 | Complementos 433/868 | 18 stdout global,9 resúmenes |
| `# EV-11 — Presets propietarios, sól.md` → [[EV-11 — Evidencia de control en 868 MHz]] | EV-11 — Presets propietarios, sólo control | EV-11 bloque 868 y revisión | No explícita | Presets 868/WMBus | Complementario/revisado | 8 stdout únicos, uno repetido | EV-11 canónico/433 | 6/8→8/8, no9 únicos |
| `# EV-12 — Informe técnico y auditor.md` → [[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]] | EV-12 — Informe técnico y auditoría de TX RAW / FRAME / BURST / CONTINUOUS en FeralRF | EV-12 etapa OTA | 06-10-2026 explícita | RAW/FRAME/BURST/CONT | Canónico resultado actual | Bytes/conteos/negativo/intervalos | EV-12 preliminar/13 | Criterios evolucionan; causa abierta |
| `# EV-12 — Validación de TX rawframe..MD` → [[EV-12 — Control preliminar y preparación de EV-13]] | EV-12 — Validación de TX raw/frame/burst/continuous y STOP | EV-12 control y plan EV-13 | No explícita | 4 modos TX/STOP | Complementario histórico | 4/4ACK; segunda H1es plan | EV-12 OTA/13 | No prueba RF ni EV-13 realizada |
| `# EV-13 — Registro técnico consolid.md` → [[EV-13 — CW PRBS y TX_TEST_STOP por control]] | EV-13 — Registro técnico consolidado de CW, PRBS y `TX_TEST_STOP` | EV-13 control CW/PRBS | No explícita; después de 12 | CW/PRBS/STOP/BLE | Canónico parcial | ACK+BLE/timeouts, no instrumento | EV-12/14 | Disponibilidad instrumental≠medición |
| `# EV-14 — Error RF asíncrono y firm.md` → [[EV-14 — Eventos RF asíncronos y firma RX]] | EV-14 — Error RF asíncrono y firma sintética | EV-14 errores y firma | No explícita; después de 13 | RX bytes/error/estado | Canónico normalizado; objetivo NOT FULLY VALIDATED | IEEE bytes, transición,3 mocks | EV-13/41/46 | Errorfísico no inducido |
| `# Registro provisional de validació.md` → [[Registro de validación FeralRF]] | Registro provisional de validación experimental de FeralRF | Registro inicial y preflight | No explícita | EV-00/01/02 y RX preliminar | Canónico cronológico actualizado | CLI/mapa/outputs 48/100 | Guía,EV-03/05 | No tercer EV-05 formal |
| `## EV-11 — Presets propietarios, só.md` → [[EV-11 — Evidencia de control en 433 MHz]] | EV-11 — Presets propietarios, sólo control | EV-11 bloque 433 | No explícita | Presets 433 | Complementario | 6 stdout,reset y quoting/logs host | EV-11 canónico/868 | Mismo ID, no EV distinto |
| `Auditoría técnica de validación FeralRF - EV ejecutadas.md` → [[Auditoría técnica de validación FeralRF - EV ejecutadas]] | Auditoría técnica de las validaciones FeralRF realizadas | Juicio técnico global | 05-10-2026 histórico | EV hasta 11 yplan 12 | Canónico actualizado; original histórico | Análisis, no nueva ejecución | Wiki,guía,matrix,EV | Corte antiguo preservado |
| `EV-04 — Reset y reinicialización.md` → [[EV-04 — Reset y reinicialización]] | EV-04 — Reset y reinicialización | EV-04 experimento | No explícita | Reset Shell/selector | Canónico normalizado | Manual boot/exit,timeout,COM90 | RP2040,EV-03/10 | API blocked,GPIO discrepa |
| `EV-05 — Primera observación RF IEEE.md` → [[EV-05 — Primera observación RF IEEE]] | EV-05 — Primera observación RF IEEE 802.15.4 | EV-05 formal | No explícita | IEEE25RX | Canónico normalizado | 3×30s41/43/43,metadatos | Registro,EV-04/14 | Sin bytes formal/emisorcontrolado |
| `EV-10 — Matriz de PHY por control.md` → [[EV-10 — Matriz de PHY por control]] | EV-10 — Matriz de PHY por control | EV-10 experimento | No explícita | 8 PHY | Canónico normalizado | 8 stdout+resets | EV-06/11,guía | Control no OTA |
| `FeralRF - Guía de validación experimental.md` → [[FeralRF - Guía de validación experimental]] | FeralRF - Guía de validación experimental | Plan/procedimiento | Cortes y versiones en original | 38 EV/35 KI | Canónico con enmienda actual | Criterios/comandos/requisitos | Wiki,4 notas FeralRF,matriz | Tarjetas vs operativo, mapas previos |
| `FeralRF - Matriz de pruebas.md` → [[FeralRF - Matriz de pruebas]] | FeralRF - Matriz de pruebas | Cobertura/estado | Corte original pre 12 | 38 EV y capacidad | Canónico reconciliado | Estados previos no ejecución | Guía,registro,auditoría | 12/13/14 antes pendientes |
| `FeralRF - Wiki técnica integral.md` → [[FeralRF - Wiki técnica integral]] | FeralRF — Wiki técnica integral | Definición integral | 06-10-2026; evidencia al corte 05-10 | Arquitectura/API/claims | Canónico con enmienda | Análisis estático/histórico | 4 FeralRF,guía,fuentes | No TX actual quedó obsoleto |
| `Registro — EV-03 reconexión limpia.md` → [[EV-03 — Reconexión limpia entre procesos]] | Registro — EV-03: reconexión limpia | EV-03 experimento | No explícita | Reconexión serial | Canónico renombrado | 5 DeviceInfo iguales | EV-04/46/47 | Serial FERALRF1 no identifica persistentemente |
| `Fuentes FeralRF.md` → [[Fuentes FeralRF]] | Fuentes FeralRF | Procedencia de fuentes | 28-09-2026; commits fechados | FW/API/tests | Canónica recursos | URL s/commits/análisis previo | Arquitectura,evidencia | No checkout/build cargado verificado |
| `Fuentes firmware oficial.md` → [[Fuentes firmware oficial]] | Fuentes firmware oficial | Procedencia de bridge/stock | Fechas de commits en original | RP2040/firmware | Canónica recursos | URL s/archivos/contratos | Firmware RP2040,EV-04 | No binario instalado verificado |
| `Fuentes hardware CatSniffer v3.1.md` → [[Fuentes hardware CatSniffer v3.1]] | Fuentes hardware CatSniffer v3.1 | Procedencia HW | 25-09-2026; tag 2024/ECO2026 | PCB/CTF/PA | Canónica recursos | Referencias PCB/revisiones | Arquitectura HW/FeralRF | No medición de pines/rutaactual |
| `Fuentes herramientas PC.md` → [[Fuentes herramientas PC]] | Fuentes herramientas PC | Procedencia host | main bc80979/04-09; versión 3.3.2.1 | Catnip/tools | Canónica recursos histórica | Referencias/scripts/CLI | Catnip CLI,guía | Guía más nueva 126f13b/3.3.3.0 |

## Huérfanos, referencias e identificación de vacíos

Las notas EV tenían escasos enlaces activos y no figuraban uniformemente en matrices; los registros 12–14 ejecutados estaban ausentes del estado actualizado. Ahora los canónicos/complementos están enlazados desde registro/matriz/auditoría. No había una justificación para nuevos IDs; no se creó. Referencias a 00/01/02 corresponden a evidencia embebida; a 15/20–54 corresponden a plan, con ejecución parcial/no ejecutado/bloqueado/retirado explícita. Las referencias `.txt`, rutas externas y archivos de evidencia mencionados sin artefacto se conservan como no verificados, no como enlaces a un supuesto fichero existente. Las discrepancias de nombres/h1 y duplicaciones de IDs están explicadas por el mapa anterior y roles. No se reordenaron áreas no relacionadas.

## Verificación final y límites de la revisión

La revisión comprueba originales completos preservados, conteo 25, H1/filename,17 secciones por registro, enlaces Wiki y Markdown locales,38 ítems EV y 35 KI contabilizados, consenso entre matriz/EV/assessment, ydiff Git (incluido Wiki previamente sin seguimiento). Git no aporta resultados RF. Las conclusiones se acotan por configuración; hipótesis nunca se promueven a causa. Las cronologías inciertas y el plan pendiente permanecen explícitos. No se realizaron pruebas hardware ni cambios de firmware/software.

Resultado histórico de la comprobación del primer pase (no es el conteo de enlaces posterior a estas correcciones):25/25 textos de origen preservados; 25 notas primarias conservadas; 12 archivos EV con 17 secciones (nueve IDs canónicos y tres complementos); 38 ítems del plan y 35 KI contabilizados; 493 wikilinks activos y 14 anclas de encabezado resueltos en 44Markdown del Vault. No se encontraron enlaces Markdown locales activos en ese conjunto. H1 y nombres coinciden en las 25 notas. `git diff --check` del alcance pasa con la configuración de finales de línea Windows. La comparación de cada renombrado contra su transcripción previa incluye archivos nuevos aún sin seguimiento, que el diff ordinario de Git no muestra como destino. No se creó commit ni se preparó el índice Git. Las copias de comprobación son temporales; el inventario y las fuentes originales permanecen en estas notas.

Comprobación de las correcciones aprobadas del segundo pase: 21 notas actualizadas en interpretación; 25 fuentes originales y cuerpos históricos preservados; cuatro recursos sin cambios; 12 registros EV mantienen 17 secciones; 38 ítems del plan, 35 KI y 35 capacidades contabilizados. Se resolvieron 520 wikilinks activos y 41 anclas en 44 Markdown; H1/nombres consistentes y git diff --check sin errores. No hubo renombres, nuevos documentos del Vault, experimentos, investigación externa, cambios de firmware, commit ni staging.

## 17. Notas originales preservadas y material pendiente

Fuente: `Auditoría técnica de validación FeralRF - EV ejecutadas.md`. SHA-256 previo: `292C4EBF4FAFB2DB0371062D7E633A7AE807123E1A99486EA0D6FEC4A4E6A0E0`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
# Auditoría técnica de las validaciones FeralRF realizadas

Fecha de auditoría: 2026-10-05

## 1. Alcance y estado de los repositorios

Esta auditoría reconstruye la procedencia y el valor probatorio de las evaluaciones que tienen evidencia de preparación o ejecución en `CatSniffer-Understanding/05 - Evaluación`. No se ejecutó hardware, Catnip, FeralRF, pruebas unitarias ni scripts de validación durante la auditoría. Tampoco se flasheó, cambió de rama o modificó ningún repositorio fuente. Los informes existentes se trataron como registros sujetos a comprobación, no como autoridad.

### 1.1 Taxonomía de evidencia usada

| Etiqueta | Significado en este informe |
| --- | --- |
| **Declarado** | Afirmación de README, guía, matriz o comentario; no implica que el comportamiento ocurra. |
| **Implementado** | Existe una ruta ejecutable en el código actual. |
| **Probado unitariamente** | Un test automatizado comprueba una propiedad de software; no prueba el hardware. |
| **Histórico** | El repositorio declara una ejecución física anterior, pero faltan en el workspace actual sus logs crudos, hashes de binario o paquete completo de evidencia. |
| **Procedimiento registrado** | El informe conserva el comando o describe lo que el usuario hizo. |
| **Observación física** | La salida sólo se explica razonablemente por interacción con la placa o la radio. |
| **Inferencia** | Interpretación compatible con los datos, pero no observada de forma directa. |
| **Conclusión** | Veredicto limitado al alcance que permiten las categorías anteriores. |

La jerarquía seguida fue: código enlazado y formato de protocolo actual → scripts ejecutables actuales → tests de contrato → documentación técnica actual → historial Git → matrices históricas → guía local → informe de laboratorio. Una fuente posterior no convierte retroactivamente una observación incompleta en evidencia literal.

### 1.2 Repositorios y versiones inspeccionados

| Repositorio | Rama | Commit inspeccionado | Estado encontrado |
| --- | --- | --- | --- |
| FeralRF | `main` | `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`, 2026-07-22 | submódulo TI SDK modificado (`m firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01`) y `python/catnip` sin seguimiento |
| CatSniffer | `master` | `6da5050fefde59138fc11d9e2ecf7c0de36f3a22`, 2025-12-09 | limpio |
| CatSniffer-Firmware | `v3.x` | `c0cd5a45e019dbd14ed11d039aacb13f300e5731`, 2026-09-03 | limpio |
| CatSniffer-Tools | `fix/CLI_control` | `126f13bc0441ad3f37fe0b029160a3c526e4d309`, 2026-09-28 | `py` sin seguimiento; el checkout estaba tres commits detrás de `origin/fix/CLI_control` según el preflight anterior |
| CatSniffer-Tools-v3.3.2.1 | rama no mostrada por `branch --show-current` | `a06b7887a20693129ffb2ed567ed73284908d0a6`, 2026-07-09 | limpio |
| Sniffle | `master` | `3a53f5ab21df7599ad0c46475d322603c1a54bb7`, 2025-09-24 | limpio |

Los estados no limpios ya existían al iniciar esta auditoría y no se limpiaron ni alteraron. En particular, no puede atribuirse el contenido no seguido a esta auditoría.

### 1.3 Plataforma y cadena realmente examinada

El DUT registrado es CatSniffer V3, con RP2040 como terminador USB/puente y CC1352P7 como procesador que ejecuta FeralRF. La ruta de las pruebas FeralRF fue:

```text
PowerShell / CPython en Windows
  → scripts o feralrf.Radio
  → tramas COBS/CRC del protocolo FeralRF
  → COM88 / Cat-Bridge a 921600
  → RP2040 con passthrough UART
  → HostIFTask / CommandProcessor en CC1352P7
  → ControlTask / DataTask
  → RadioIF / TI RF driver
  → radio integrada y front-end de CatSniffer
```

Fuentes: `FeralRF/python/feralrf/radio.py`, `protocol.py`, `commands.py`; `firmware/cc1352/src/{host_if_task,command_processor,control_task,data_task,radio_if}.c`; `CatSniffer-Firmware/RP2040/catsniffer/src/main.c`; hardware CatSniffer. El SX1262/Cat-LoRa no intervino en las EV ejecutadas.

La identificación física inicial provino de un ejecutable instalado, `C:\Program Files\Catnip\catnip.exe`, que mostró `v3.3.3.0`. El checkout local de CatSniffer-Tools no pudo ejecutarse en aquel entorno por ausencia de `rich`. Por ello, la correspondencia exacta entre el binario Catnip que produjo el mapa físico y `CatSniffer-Tools@126f13b` **no está establecida**. Coinciden la versión visible y el formato esperado, pero no existe hash/build provenance del EXE.

### 1.4 Inventario de estado real

| EV | Estado que permite la evidencia | Nota principal |
| --- | --- | --- |
| EV-00 | **PARTIAL** | Se obtuvo mapa V3/COM con `catnip devices`, pero no se preservaron `devices --debug`, `identify`, `status` ni enumeración Windows completa exigidos por la guía revisada. |
| EV-01 | **PARTIAL**, evidencia de control favorable | Hay numerosos `INIT/GET_INFO` y varios `GET_STATS`, pero no una serie dedicada y temporizada de 3/3 `init+info+stats` conforme al criterio formal. |
| EV-02 | **PASS-control** | `SET_PHY/CHANNEL`, `RX_START`, lectura y `RX_STOP` se repitieron; EV-05 además aporta RF. |
| EV-03 | **PASS-control 5/5** | Cinco procesos independientes abrieron, inicializaron y cerraron COM88. |
| EV-04 | **BLOCKED** para `Radio.reset_device()`; **PASS-recovery manual 1/1** | KI-15 reproducido: COM90 calculado frente a COM87 real. La recuperación manual fue favorable, no 3/3 ni API pública. |
| EV-05 | **PASS-RF (recepción IEEE 802.15.4)** con limitación de atribución | Tres ventanas formales recibieron 41/43/43 paquetes; no se guardaron bytes de todos los frames ni captura independiente simultánea. |
| EV-06 | **PASS-control/state-rejection** | Error síncrono `0x05` y recuperación mediante STOP/STATS. No prueba RF simultánea. |
| EV-10 | **PASS-control 8/8** | Ocho PHY completaron el smoke; no hay evidencia RF individual. |
| EV-11 | **PASS-control reportado 27/27**, auditabilidad desigual | 18 presets tienen salida individual preservada; nueve de 902/915 MHz sólo tienen resumen agregado. Ninguno tiene PASS-RF actual. |
| EV-12 | **PREPARED / NOT TESTED** | Sólo existe definición e inspección de scripts; no hay comando ni salida física. |

La matriz y `00 - Proyecto/Estado y siguientes pasos.md` todavía dicen `NOT YET RUN` o “ninguno fue ejecutado físicamente”. Son documentos de planificación que quedaron obsoletos respecto de los logs; no invalidan éstos, pero no deben utilizarse como estado actual.

## 2. Cómo se derivó el plan de validación

### 2.1 Base histórica real

El plan local no apareció de una única especificación upstream. Se reconstruyó principalmente a partir de:

- `FeralRF/docs/VALIDATION_MATRIX.md`, originado en `7bd36a6` (2026-04-07, “Add RF validation matrix and baseline smoke tooling”) y actualizado por `bee7f67`, `f327cf6`, `b089f71`, `c45114b`, `10fc47a`, `6e34ef2`, `2595276` y `7ad0a1a`;
- `FeralRF/python/examples/run_validation_baseline.sh`, que ejecuta pasos de control/OTA y llama a reset entre pasos;
- scripts `smoke_phase2.py`, `smoke_phy4_ieee154.py`, `smoke_prop_phase1.py` y la familia `smoke_tx_*`;
- API Python y handlers actuales, que definen comandos, respuestas y errores;
- resultados históricos declarados en `VALIDATION_MATRIX.md`: baseline 18/18, OTA del 2026-04-08, 433 MHz marginal, OOK 433 0/10, Wi-SUN/Sidewalk 70/70 y MIOTY 0/10;
- riesgos actuales observables en código, por ejemplo ACK previo a trabajo RF diferido, reset por `Bridge+2`, lock OOK y errores RF asíncronos;
- metodología nueva añadida en la guía local: repeticiones 3/3 o 5/5, criterios `PASS-control/PASS-RF`, tráfico Zigbee independiente y reglas de parada.

Los datos históricos son procedencia válida para formular hipótesis y repetir ensayos, pero no son resultados físicos de esta campaña. `VALIDATION_MATRIX.md` no incluye los logs crudos, hash del HEX o identificación completa de las placas de 2026-04/05.

### 2.2 Mapa de procedencia por EV ejecutada o preparada

| EV | Objetivo, comando y parámetros: fuente fuerte | Expectativa/fallo/recuperación: fuente fuerte | Naturaleza del diseño |
| --- | --- | --- | --- |
| EV-00 | `Radio.list_devices/_get_shell_port` en `radio.py`; Catnip `usb_connection.py:find_devices/_group_ports_by_device/_map_roles` y `device/cli.py` en `CatSniffer-Tools@126f13b` | Riesgo `Bridge+2` visible en `radio.py:_get_shell_port`; identificación por Catnip/Windows | **Ensamblada.** No hay EV-00 upstream; identidad inequívoca, comparación cruzada y criterio STOP son metodología local. |
| EV-01 | `Radio.init/get_stats`; `command_processor.c` casos `CMD_RADIO_INIT/GET_INFO/GET_STATS`; baseline histórico | ACK/INFO y STATS se derivan del protocolo. Tres repeticiones y tiempo son criterio local | **Ensamblada**, aunque usa operaciones upstream directas. |
| EV-02 | Directamente `smoke_phy4_ieee154.py`; PHY 4, canal 25 y 5 s también aparecen en `run_validation_baseline.sh` | `DataTask_poll` difiere el `RadioIF_startRx`; posible `ERR_RF_INIT_FAILED`; STOP en API/handler | **Directamente derivada**, con clasificación control/RF añadida localmente. |
| EV-03 | `Radio.connect/disconnect/init` y sus reintentos | No existe script upstream dedicado ni resultado histórico específico | **Diseñada parcialmente por inferencia.** Cinco procesos/5 de 5 y demora creciente son metodología local. |
| EV-04 | `radio.py:reset_device/_get_shell_port`; `run_validation_baseline.sh:_reset_one`; comandos `boot/exit` del RP2040 | `CatSniffer-Firmware/.../shell_commands.c:cmd_boot/cmd_exit` y `main.c:change_mode/reset_cc1352`; OOK/switching histórico | **Ensamblada.** El bloqueo seguro por mismatch y 3/3 son criterios locales. |
| EV-05 | `smoke_phy4_ieee154.py`; `radio_if.c:RadioIF_processIeee154Packets`; canal 25 aportado por la fuente física conocida | CRC/RSSI/LQI/timestamp vienen de RadioIF/DataTask; fuente independiente y 3×30 s son metodología local | **Script directo + diseño experimental local.** La atribución al dispositivo Zigbee no estaba especificada upstream. |
| EV-06 | `command_processor.c:CMD_TX_RAW`; `control_task.c:ControlTask_canStartTx` y `ERR_INVALID_STATE`; `Radio.transmit`/`CommandError.error_code` | Rechazo antes de encolar TX y recuperación STOP/STATS se deducen del estado y API | **Ensamblada.** El harness PowerShell fue creado para la campaña. |
| EV-10 | `smoke_phase2.py`, enum `PHY`, baseline y matriz histórica | 8 PHY y ACK de control son upstream; reset entre pasos viene del baseline/histórico | **Directamente derivada**, salvo selección `(PHY 1, canal 9)` y criterio 8/8 local. El baseline actual usa canal 37 para PHY 1. |
| EV-11 | `PROP_PRESETS`, `smoke_prop_phase1.py`, `run_validation_baseline.sh`, `Radio.configure_prop`, `RadioIF_setPropConfig` | OOK lock en API/código; MIOTY pending en preset y commit `6e34ef2`; 433 marginal y switching en matriz histórica | **Script directo + barrido exhaustivo diseñado localmente.** Upstream no define exactamente el subconjunto “31−4=27” como una sola prueba. |
| EV-12 | Familia `smoke_tx_{phase1,frame_phase1,burst_phase1,continuous_phase1}.py`; handlers de TX | ACK antes de finalización en `control_task.c:ControlTask_processTxRaw`; observador requerido | **Directamente derivada**, pero sólo preparada. |

### 2.3 Criterios que no se trazan a una especificación upstream

No se encontró una fuente upstream única que imponga 3/3 para EV-01/04/05, 5/5 para EV-03, 8/8 para EV-10, ni el inventario exhaustivo de 27 presets para EV-11. Son criterios razonables de diseño experimental creados al construir la guía. Deben presentarse como metodología de esta campaña, no como requisitos oficiales de FeralRF.

La distinción `PASS-control` frente a `PASS-RF` también es una mejora metodológica local. Está apoyada por la arquitectura del código —los ACK se emiten antes o sin confirmación física—, pero no es una taxonomía formal upstream.

## 3. Auditoría por EV

## 3.1 EV-00 — Enumeración e identidad

**Intención.** Identificar placa y interfaces antes de abrir FeralRF o resetear otro puerto por error.

**Procedimiento registrado.** `# Registro provisional de validación experimental...txt` conserva `catnip devices` y su fila:

```text
CatSniffer #1 | v3 (RP2040 + CC1352P7) | Bridge COM88 | LoRa COM86 | Shell COM87
```

También conserva resolución del EXE, `catnip --help`, `pip show catnip`, `find_spec('catnip')` y el fallo del checkout local por `ModuleNotFoundError: rich`. No conserva la salida de `devices --debug`, `identify --device N`, `status --device N` ni `Get-CimInstance Win32_SerialPort`.

**Qué prueba.** La salida observada prueba que el Catnip instalado agrupó tres interfaces y clasificó la placa como V3. La desigualdad `COM87 != COM88+2` queda establecida. No prueba por sí sola que CatSniffer #1 sea una identidad persistente, ni el serial/HWID/location, ni que el EXE corresponda exactamente a `126f13b`.

**Esperado frente a observado.** El resultado satisface el núcleo del preflight original, pero no el procedimiento ampliado de la guía actual. El informe inicial dice correctamente que no se debía ejecutar `reset_device()` a ciegas.

**Veredicto: PARTIAL.** Mapeo funcional suficiente para continuar con COM88 y bloquear el reset inseguro; trazabilidad física/USB incompleta según la guía actual.

## 3.2 EV-01 — Init, GET_INFO y GET_STATS

**Intención.** Confirmar comunicación host→RP2040→CC1352P7 y obtener identidad/contadores sin atribuir funcionamiento RF.

**Ruta.** `Radio.init()` abre COM, envía `RADIO_INIT`, exige ACK, envía `GET_INFO` y decodifica 12 bytes; `get_stats()` exige `RSP_STATS` de al menos 16 bytes. En firmware, `CMD_RADIO_INIT` llama `ControlTask_onRadioInit`, que detiene RX y reinicia métricas; GET_INFO/GET_STATS serializan estado.

**Procedimiento real.** No hay informe EV-01 dedicado con tres ciclos `init+info+stats`. Sí existen muchas salidas literales de:

```text
DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ...)
```

EV-03 aporta cinco `init`; EV-04, EV-06, EV-10 y EV-11 aportan respuestas posteriores, varias con STATS.

**Qué prueba.** La ruta de control y parsing funciona repetidamente en la campaña. El serial hexadecimal decodifica bytes constantes definidos por firmware (`FERALRF1`), no un identificador único de la placa. La versión `1.0.0` es el payload de firmware actual, no prueba por sí sola el commit o hash del binario flasheado.

**Veredicto: PARTIAL respecto al EV formal; evidencia funcional fuerte.** No corresponde afirmar 3/3 dedicado con timings porque no está registrado. Tampoco corresponde dejarlo como “no ejecutado”: sus operaciones fueron observadas repetidamente.

## 3.3 EV-02 — RX IEEE controlado y parada

**Intención.** Configurar IEEE 802.15.4, solicitar RX y volver a idle.

**Procedimiento real.** Las cinco corridas RX preservadas (dos preliminares y tres formales EV-05) usaron `smoke_phy4_ieee154.py`, canal 25, ventanas de 30 o 60 s, y terminaron con `RX_STOP ACK`.

**Ruta y matiz temporal.** `RX_START ACK` se envía en `command_processor.c` después de marcar el evento, antes de que `DataTask_poll` ejecute `RadioIF_startRx`. Si éste falla, llega `RSP_ERROR/ERR_RF_INIT_FAILED` asíncrono. Por eso un ACK aislado sería sólo control. En estas corridas llegaron paquetes y STOP, de modo que la evidencia supera el ACK.

**Veredicto: PASS-control.** RX start/stop fue repetible y el estado fue recuperable. La evidencia RF se asigna separadamente a EV-05.

## 3.4 EV-03 — Reconexión limpia

**Procedimiento literal.** `Registro — EV-03 reconexión limpia.txt` conserva el pipeline PowerShell que lanza cinco procesos Python independientes. Cada uno crea `Radio(port='COM88')`, ejecuta `init()` y `disconnect()`.

**Observado.** Cinco `DeviceInfo` idénticos; sin timeout, excepción ni puerto ocupado.

**Qué prueba.** Apertura, vaciado de buffers, intercambio init/info, cierre y nueva apertura entre procesos 5/5. No prueba una sesión larga, reconexión en una misma instancia, reconexión tras USB loss ni ausencia de fuga interna en el RP2040/CC.

**Corrección procedimental.** Coincide exactamente con el comando que la guía terminó definiendo. Como ese comando fue diseñado localmente, la coincidencia demuestra disciplina respecto al plan, no reproducción de un test upstream.

**Veredicto: PASS-control 5/5.** El adverbio “provisionalmente” del informe es conservador; el escenario concreto quedó cerrado.

## 3.5 EV-04 — Reset y reinicialización / KI-15

**Intención.** Validar recuperación antes de modos frágiles sin abrir un COM ajeno.

**Código actual.** `Radio._get_shell_port()` extrae el sufijo numérico de COM88 y suma dos, por lo que retorna COM90. `reset_device()` cerraría Bridge, abriría ese puerto, escribiría `boot\r\n`, después `exit\r\n`, esperaría, reabriría Bridge y ejecutaría `init()`.

**Procedimiento real.** Se verificó `_get_shell_port()` y se obtuvo COM90. Correctamente no se invocó la API contra ese puerto. Sobre el Shell mapeado por Catnip, COM87, se ejecutó:

1. escritura exacta de `boot\r\n`;
2. intento `init/get_stats` en COM88, que agotó timeout;
3. escritura exacta de `exit\r\n`;
4. `init/get_stats` exitosos por COM88 sin reconectar USB.

**Qué prueban `OPEN/SENT/CLOSED`.** Sólo que pySerial abrió, escribió y cerró. No son respuestas del RP2040. El cambio posterior de disponibilidad en COM88 y la recuperación constituyen la observación física fuerte.

**Correspondencia con firmware oficial.** En `CatSniffer-Firmware@c0cd5a4`, `shell_commands.c:cmd_boot` llama `change_mode(BOOT)` y `cmd_exit` llama `change_mode(PASSTHROUGH)`. `main.c:change_mode` acciona boot/reset del CC, cambia el UART a 500000 en BOOT y a 921600 al salir. Por tanto, la secuencia manual es técnicamente coherente con el firmware RP2040 actual, aunque el log no leyó las respuestas textuales `BOOT`/`PASSTHROUGH`.

**Contradicción descubierta.** La guía describe la ruta como “GPIO15→RESET_N”, siguiendo `FeralRF/hardware/PINOUT.md`. Sin embargo, el firmware RP2040 ejecutable actual obtiene `pin-reset` del overlay `rpi_pico.overlay`, donde el alias apunta a `gpio0 3`; `pin-boot` apunta a GPIO2. El esquema contiene una red `RESET_CC`, pero la afirmación concreta GPIO15 no coincide con este target ejecutable. Para describir el procedimiento actual debe citarse el alias/overlay, no GPIO15, hasta reconciliar revisiones de hardware/documentación.

**Veredicto dual.**

- `Radio.reset_device()`: **BLOCKED**, KI-15 reproducido (`COM90` calculado frente a `COM87` real).
- Ruta manual por Shell explícito: **PASS-recovery 1/1**, no PASS formal 3/3 y no prueba “watchdog reset”.

Los informes posteriores que llaman “reset correcto” a sólo `OPEN/SENT/CLOSED` son demasiado fuertes si no incluyen verificación posterior. Cuando una prueba siguiente obtiene INFO/STATS, existe evidencia indirecta de recuperación, pero no de cada detalle eléctrico interno.

## 3.6 EV-05 — Primera recepción RF IEEE 802.15.4

**Intención.** Pasar de aceptación de comandos a recepción física con `PROTOCOL-DEVICE-ZIGBEE-CH25` como fuente independiente.

**Procedimiento formal preservado.** Tres ejecuciones literales de:

```powershell
python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
```

Resultados: 41, 43 y 43 paquetes. La primera trama mostrada en cada corrida tuvo `crc_ok=True`, canal 25, RSSI −85/−79/−83 dBm, LQI 52/57/56 y longitudes 52/5/52 bytes. Dos corridas preliminares añadieron 48 paquetes/30 s y 100/60 s. Existe además un control negativo descrito en otro canal, pero no conserva canal, comando ni stdout y sólo vale como observación auxiliar no reproducible.

**Ruta de datos.** `RadioIF_processIeee154Packets` extrae RSSI, correlación/LQI, bit de error CRC y timestamp del entry TI; `DataTask_emitRxPacket` los serializa; `Radio.read_packets` los convierte en `Packet`. Esto respalda que `crc_ok` no fue inventado por el script.

**Cumplimiento del plan.** Se cumplieron tres ventanas de 30 s y STOP. No se cumplió la instrucción de registrar los bytes raw y metadatos de **cada** paquete: el script sólo imprime conteo y primera trama, y ni siquiera imprime sus bytes. Tampoco hubo observador independiente simultáneo que correlacionara frames o confirmara la actividad de la fuente durante las mismas ventanas.

**Qué prueba.** Prueba recepción física reproducible de frames compatibles con IEEE 802.15.4 en canal 25 a través del CC1352P7 y la ruta host. La atribución a la fuente Zigbee conocida es plausible y fuerte por contexto, repetición y canal, pero sigue siendo inferencia sin correlación simultánea. No prueba asociación, parsing Zigbee, stack Zigbee, sensibilidad, PER ni exactitud RSSI.

**Veredicto: PASS-RF limitado a recepción IEEE 802.15.4.** Mantener `PASS-RF` es defendible si su etiqueta se lee con ese alcance. La afirmación más fuerte “procedente de la fuente concreta” debe llevar la limitación anterior. El registro sería académicamente más sólido con PCAP/bytes y observador concurrente.

## 3.7 EV-06 — Exclusión RX/TX

**Procedimiento.** El informe conserva dos errores de harness sin interacción útil (quoting de PowerShell e import `Phy` en vez de `PHY`), una primera ejecución válida que consultó el atributo incorrecto `code`, inspección local de `Radio.transmit`/`CommandError`, y una ejecución definitiva. Ésta inició RX en PHY 4/canal 25, llamó `transmit(b'\x01', power_dbm=-20)`, obtuvo `CommandError`, `error_code=5/0x05`, detuvo RX y obtuvo STATS.

**Ruta exacta.** `start_rx()` hace que `ControlTask_onRxStart` establezca `s_rx_enabled=true`. Al llegar `CMD_TX_RAW`, `ControlTask_onTxRaw` llama `ControlTask_canStartTx`, que exige `!s_rx_enabled`; retorna falso y `command_processor.c` envía `ERR_INVALID_STATE`. La solicitud se rechaza antes de copiar/encolar el payload para `RadioIF_transmitRaw`.

**Qué prueba.** Regla lógica de exclusión y recuperación del protocolo/estado. En esta ruta concreta, el código respalda que no se solicitó TX al backend RF. No prueba simultaneidad real de RX/TX ni que el receptor RF hubiera terminado de abrir: basta el flag lógico activado por RX_START.

**Corrección procedimental.** La corrección de `e.code` a `e.error_code` fue necesaria y está bien documentada. Los intentos inválidos no deben contarse como repeticiones del DUT.

**Veredicto: PASS-control/state-rejection.** El informe final `PASS` es válido si se conserva este alcance; sería mejor nombrarlo explícitamente `PASS-control`.

## 3.8 EV-10 — Matriz de PHY por control

**Procedimiento.** Ocho invocaciones literales de `smoke_phase2.py` para PHY 0–7, potencia 0 dBm y canales 37, 9, 37, 37, 25, 0, 0, 0. Todas muestran INFO, ACK de SET_PHY/CHANNEL/POWER, ACK de RX_START/STOP y `SMOKE TEST PASS`. Entre filas se registró `boot`/`exit` por COM87 y verificación INFO/STATS; hubo además reset final.

**Qué prueba.** La API, enum, protocolo y handlers aceptan las ocho configuraciones y la sesión responde después. No prueba frecuencia, potencia, modulación, apertura efectiva duradera, recepción ni transmisión RF. En particular, PHY 7 sin `configure_prop` no representa un preset concreto.

**Matiz de procedencia.** El script es upstream directo, pero la matriz exacta de la guía no es idéntica al baseline actual: `run_validation_baseline.sh` usa canal 37 para PHY 1, mientras esta campaña usó 9. Canal 9 es plausible para BLE 2M raw y aparece en el antecedente de Extended ADV, pero no debe describirse como copia exacta del baseline de control.

**Reset.** El reset entre cada step reproduce el workaround del baseline y `VALIDATION_MATRIX.md` §10, donde el cambio sin reset tuvo `RF_close deadlock`. No es una necesidad demostrada por EV-10 ni una propiedad universal del chip; es un workaround conservador derivado de fallos históricos. Los resets manuales fueron más numerosos que lo estrictamente necesario para demostrar ACK, pero coherentes con el plan de aislamiento.

**Veredicto: PASS-control 8/8.** El informe limita correctamente el alcance y no asigna PASS-RF.

## 3.9 EV-11 — Presets propietarios, sólo control

### Conteo independiente

`FeralRF/python/feralrf/presets.py@0178721` contiene 31 entradas. Las exclusiones fueron exactamente:

```text
ook_433_4k8
ook_433_2k4
ook_868_4k8
mioty_868_tsunb
```

Quedan 27 objetivos: 6 de 433 MHz, 8 de 868 MHz, 2 de 169 MHz, 9 de 902/915 MHz y 2 de 2.44 GHz. El conteo de la conclusión es correcto.

### Duplicados y evidencia preservada

`gfsk_868_50k` fue ejecutado dos veces y correctamente no se contó como un preset adicional. Los tres informes EV-11 preservan salida individual completa para:

- 6/6 de 433 MHz;
- 8/8 de 868 MHz, más el duplicado;
- 2/2 de 169 MHz;
- 2/2 de 2.44 GHz.

Para los nueve presets 902/915 MHz sólo se conserva una tabla y un patrón de salida agregado común. No aparecen los nueve comandos ni sus nueve stdout individuales. Por tanto, “se ejecutaron nueve y todos pasaron” es un **registro narrativo del procedimiento**, no evidencia terminal individual auditable desde el Vault. No hay base para afirmar que no se ejecutaron, pero tampoco puede reconstruirse cada corrida sólo desde el material conservado.

### Qué hace realmente el smoke

`smoke_prop_phase1.py` ejecuta INIT, PHY 7, `SET_PROP_CONFIG`, SET_POWER, RX_START, lee tres segundos con `min_packets=0`, RX_STOP, TX_RAW y GET_STATS.

Hay tres límites de observabilidad importantes:

1. `CMD_SET_PROP_CONFIG` llama a la función `void RadioIF_setPropConfig(...)` y envía ACK sin recibir un resultado de éxito/fallo del backend. “Backend aceptado” es una formulación demasiado fuerte; lo demostrado es que el handler aceptó el payload y llamó a la configuración sin error síncrono visible.
2. RX_START recibe ACK antes de que `DataTask_poll` ejecute `RadioIF_startRx`. El script excluye de la lista todo `RxStreamError`, de modo que un error RF asíncrono durante la ventana no queda impreso ni hace fallar por sí solo la prueba.
3. TX_RAW recibe ACK al encolarse. `ControlTask_processTxRaw` ejecuta después `RadioIF_transmitRaw`; si falla, emite un error asíncrono. El script no espera TX_DONE ni consume explícitamente ese evento después del ACK.

Por ello `PROP PRESET SMOKE PASS` prueba que la cadena de comandos síncronos terminó y que el dispositivo siguió contestando, pero no prueba que RX o TX RF se hayan completado físicamente.

### Valor de STATS

Los ceros `rx_crc_err/rx_drop/rx_overflow` son datos reales devueltos por GET_STATS, pero tienen valor probatorio bajo aquí: `RADIO_INIT` reinicia métricas, las ventanas duraron tres segundos, `min_packets=0` y se observaron cero paquetes. Muestran ausencia de contadores de error durante una ventana sin tráfico recibido; no caracterizan capacidad de cola, robustez ni calidad RF. La frase “no se registró overflow” es literalmente cierta; usarla como argumento fuerte de estabilidad sería engañoso.

### Reset entre bandas

Hay evidencia literal de envío `boot/exit` para 433→868, 169→902/915 y 902/915→2.4 GHz. El propio cierre reconoce que no aparece la salida de 868→169. Esto es **evidencia faltante**, no evidencia de que el reset no ocurrió.

Además, en algunos cambios sólo se imprimió `OPEN/SENT/CLOSED`; eso confirma escritura host, no por sí solo reset/recuperación. El éxito del primer preset posterior aporta evidencia indirecta de que el DUT estaba funcional. La transición 433→868 no conserva un INFO/STATS inmediatamente intermedio, aunque las pruebas 868 posteriores sí respondieron.

El reset entre bandas proviene de `VALIDATION_MATRIX.md` §10 y `PYTHON_API.md`, que documentan deadlock/inconsistencia histórica sin reset. Es un workaround conservador. No hay en `smoke_prop_phase1.py` una obligación intrínseca de reset entre todos los presets de la misma banda.

### 433 MHz, nombres de protocolo y exclusiones

`PASS-control 6/6` en 433 MHz no contradice el historial OTA: GFSK 6–10/10, FSK/MSK 1/10 y OOK 0/10. EV-23 sigue siendo la caracterización RF pendiente.

Los nombres Wireless M-Bus, Wi-SUN y Sidewalk seleccionan parámetros PHY. EV-11 no ejecutó framing, MAC, asociación, seguridad ni interoperabilidad de esos protocolos. “W-MBus N preset control validado” es correcto; “Wireless M-Bus N validado” sin calificador no lo sería.

Excluir OOK fue correcto: `Radio.configure_prop`, `presets.py` y `RadioIF_setPropConfig` advierten que cargar los patches OOK deja el radio bloqueado hasta reset/power-cycle. Debe revisitarse en **EV-24**. Excluir MIOTY también fue correcto: el preset se conserva como API pero está marcado pending; commit `6e34ef2` y la matriz histórica registran 0/10 por falta de soporte TS-UNB/CPE. Debe revisitarse en **EV-52**, no contarse como un cuarto fallo de EV-11.

### Veredicto

**PASS-control reportado 27/27, con nivel de evidencia desigual.** Hay evidencia literal sólida para 18 presets y una afirmación agregada para nueve. No hay PASS-RF. La conclusión defendible es: “los 27 objetivos fueron registrados como terminados por el script de control; el Vault permite auditar individualmente 18 y sólo resumidamente los nueve de 902/915”.

**Enmienda documental 2026-10-08:** [[Sub-GHz en CatSniffer V3 - CC1352P7, ruta RF y control de banda]] verificó contra SWRS251A que 169.45 MHz queda fuera de las bandas del CC1352P7 y contra el frente U4 publicado que éste cubre 862–928/2400–2500 MHz. Además, el `rf_prop_cmd.h` exacto del SDK 8.30.01.01 reserva en el setup examinado los `modType` que FeralRF numera 4/5/6. Esto no borra el resultado histórico PASS-control: lo restringe a aceptación/continuidad del flujo y evita tratar N169, MSK o 4-(G)FSK como cobertura backend/física demostrada.

## 3.10 EV-12 — Estado de preparación

La guía identifica y describe los cuatro scripts TX y sus riesgos. En los informes no existe comando ejecutado, stdout, observación RF ni resultado de EV-12. La inspección de `smoke_tx_phase1.py` o del código no constituye ejecución.

**Veredicto: PREPARED / NOT TESTED.** No existe PASS parcial. No se continuó EV-12 durante esta auditoría.

## 4. Hallazgos transversales

### 4.1 ACK no equivale a RF

La campaña respetó conceptualmente esta distinción en EV-10 y EV-11, pero algunos verbos de los informes (“backend aceptado”, “TX completado”) deben acotarse. En el firmware actual:

- SET_PROP_CONFIG ACK no recibe confirmación del backend;
- RX_START ACK precede a `RadioIF_startRx`;
- TX_RAW ACK precede a `RadioIF_transmitRaw` y no es TX_DONE;
- el fallo posterior puede viajar como error asíncrono.

Sólo EV-05 contiene evidencia RF propia actual. EV-11 incluye TX breve real solicitado, pero sin observador ni TX_DONE sigue siendo **RF no establecida**.

### 4.2 Preset no equivale a protocolo

Las conclusiones finales de EV-11 suelen conservar esta distinción. Debe mantenerse en cualquier síntesis: nombres Wi-SUN, Sidewalk, W-MBus o MIOTY describen parámetros/intención, no implementación de sus stacks. El repositorio incluso califica Sidewalk como sólo capa FSK y excluye Sidewalk LR del CC1352.

### 4.3 Reset y KI-15

La decisión de bloquear la API fue correcta y evitó abrir COM90. El procedimiento manual es coherente con el firmware oficial y produjo una transición observable. La evidencia no autoriza a decir que `Radio.reset_device()` funciona en esta enumeración; precisamente se demostró lo contrario respecto a selección de puerto.

Los resets posteriores deben registrarse en dos niveles: “bytes enviados al Shell” y “recuperación comprobada por INFO/STATS o siguiente prueba”. Mezclarlos oculta la diferencia.

### 4.4 Switching PHY/banda

La fuente histórica declara fallo sin reset y éxito con reset. El código actual contiene gestión de handles, frecuencia y configuraciones separadas, pero eso no demuestra que el fallo histórico persista en HEAD. En esta campaña los resets fueron un aislamiento conservador y coherente con el baseline, no una nueva reproducción de EV-41 ni una prueba de necesidad causal.

### 4.5 RF marginal histórica

No se midió 433 MHz por aire en la campaña. EV-11 no puede confirmar ni refutar marginalidad, link budget o limitación de antena. Esas afirmaciones permanecen históricas hasta EV-23/instrumentación.

### 4.6 Versiones e identidad

`GET_INFO 1.0.0`, paquete Python `0.3.0`, documentación con otros números y Catnip `3.3.3.0` pertenecen a componentes/capas diferentes. Ninguno identifica por sí solo el hash del HEX ejecutado. La campaña conoce el commit fuente bajo evaluación, pero **no está establecido a partir de la evidencia disponible que el binario físico sea reproduciblemente trazable a ese commit mediante hash/build manifest**.

## 5. Auditoría de los informes de evaluación

### 5.1 Conservar literalmente

- comandos realmente copiados de terminal y directorio de trabajo;
- mapa `COM88/COM86/COM87`, ruta y versión del Catnip instalado;
- stdout/stderr que discrimina fallo de harness de fallo del DUT;
- `DeviceInfo`, códigos de error y excepciones relevantes;
- conteos RX, canal, duración, RSSI, LQI, CRC, longitud y, en el futuro, bytes/PCAP;
- comandos `boot/exit`, puerto explícito y verificación posterior;
- una salida completa por clase de comportamiento y toda salida inesperada;
- lista exacta de presets, parámetros que cambian y duplicados;
- hashes de fuente/binario y estado Git cuando existan.

Estos elementos permiten reproducir condiciones y revisar interpretaciones sin confiar en la narrativa.

### 5.2 Conservar, pero resumir con referencia al log crudo

- ocho salidas idénticas de EV-10: tabla por PHY con comando, retorno y anomalía basta, conservando el log original;
- secuencias repetidas de EV-11: tabla por preset con frecuencia/modulación/rate, resultado y enlace al bloque crudo;
- `OPEN/SENT/CLOSED` repetidos: conservar una plantilla y una tabla de transiciones, salvo errores o tiempos distintos;
- largas explicaciones repetidas de la misma cadena PC→RP2040→CC1352;
- la reiteración de que `packets=0` es permitido y ACK≠RF;
- análisis paso a paso de cada línea que no cambia el veredicto.

El resumen no debe fingir que existe stdout individual cuando sólo existe una afirmación agregada, como ocurre en 902/915 MHz.

### 5.3 Omitir de un futuro informe consolidado

- recomendaciones conversacionales del “siguiente candidato” al final de cada log;
- teoría repetida que ya pertenece a la guía o a una sección metodológica única;
- conclusiones duplicadas en cinco formatos dentro del mismo informe;
- diagramas repetidos sin nueva información;
- especulación sobre el tipo de frame Zigbee sin bytes/decodificación;
- inferencias causales formuladas como hechos (“backend configurado correctamente”, “reset eléctrico demostrado”) cuando sólo hubo ACK o escritura serial;
- errores de quoting/import una vez resumidos como incidencias de harness, salvo que expliquen por qué una corrida no cuenta;
- listas reiteradas de lo que no se probó si pueden centralizarse en una tabla de alcance.

No se recomienda borrar los logs actuales: su verbosidad preserva contexto de ejecución. La recomendación aplica a un futuro reporte consolidado, que debe enlazar a esos originales.

### 5.4 Inexactitudes o sobreafirmaciones concretas

1. EV-05 dice que el procedimiento registró paquetes conforme a la guía; en realidad sólo se guardó conteo y primera metadata, no bytes/metadatos de todos.
2. EV-11 usa “backend aceptado”; el handler ACK no recibe retorno de `RadioIF_setPropConfig`.
3. EV-11 afirma ausencia de error asíncrono visible, pero el script filtra `RxStreamError` durante RX y no espera conclusión RF de TX.
4. Algunos resets posteriores se llaman “ejecutados correctamente” sólo por `OPEN/SENT/CLOSED`; eso prueba envío, no recuperación completa.
5. La ruta GPIO15 de reset en la guía/FeralRF PINOUT contradice el overlay ejecutable RP2040 actual (GPIO3 para reset, GPIO2 para boot).
6. EV-10 se presenta como la matriz prescrita por el baseline, pero PHY 1/canal 9 no coincide con `run_validation_baseline.sh` actual, que usa canal 37.
7. La matriz local y el estado del proyecto aún marcan las EV como no ejecutadas; son estados desactualizados, no evidencia negativa.

No se encontró un caso donde un comando reconstruido se presentara inequívocamente como copia literal contra evidencia contraria. Sí existen descripciones sin transcripción —control negativo EV-05 y nueve presets 902/915— que deben rotularse como tales, como en general ya hacen los informes.

## 6. Asuntos abiertos / pruebas aún requeridas

Sin rediseñar el roadmap existente:

- **EV-00:** completar identidad debug/status/HWID/location y vincular físicamente DUT con el ID temporal.
- **EV-01:** si se desea cierre formal, conservar la serie dedicada 3/3 con INFO, STATS y tiempos.
- **EV-04:** permanece bloqueada para API hasta resolver o parametrizar KI-15; la repetición manual no sustituye el criterio de la API.
- **EV-05/EV-20/EV-29:** captura independiente simultánea y bytes/PCAP reforzarían atribución, FCS y decodificación, sin convertir FeralRF en stack Zigbee.
- **EV-11/EV-22/EV-23/EV-25/EV-26/EV-27:** falta RF/OTA/instrumentación de presets; 433 sigue abierto.
- **EV-24:** OOK diferido, con recuperación segura disponible.
- **EV-41/EV-42:** determinar si switching sin reset sigue fallando en el binario actual; EV-10/11 no contestan eso.
- **EV-52:** MIOTY continúa como limitación histórica/pending.
- **EV-12:** sigue no ejecutada.
- **EV-40:** reproducir el baseline histórico contra HEAD con paquete de evidencia y hash de firmware.

## 7. Conclusión técnica general

Los procedimientos ejecutados fueron, en su mayoría, técnicamente sensatos y coherentes con la guía. Las mejores evidencias son EV-03, EV-05, EV-06 y EV-10: conservan comandos, salida y un criterio claro. EV-04 manejó correctamente el riesgo al no abrir COM90 y reprodujo KI-15. EV-11 aplicó bien las exclusiones y la separación control/RF, pero su veredicto debe leerse con una semántica más estrecha de la que sugieren algunas frases: finalización del smoke síncrono, no confirmación del backend físico.

Las conclusiones no deben agregarse en un único “FeralRF funciona”. Lo actualmente defendible es:

- la comunicación host y los comandos básicos responden repetidamente;
- la máquina de estados rechaza TX durante RX con `0x05` y se recupera;
- ocho IDs PHY y 27 presets fueron recorridos a nivel de control, con evidencia individual incompleta para nueve presets;
- el DUT recibió físicamente frames IEEE 802.15.4 en canal 25 de forma repetible;
- la API de reset no es segura con el mapa COM observado;
- no se han validado físicamente los TX, la exactitud de frecuencia/potencia/modulación, la mayoría de PHY, los stacks nombrados por presets ni el baseline OTA actual.

La procedencia del plan es mayoritariamente sólida, pero híbrida: combina scripts y código upstream con criterios experimentales diseñados localmente. Eso es válido, siempre que las repeticiones y umbrales locales no se presenten como requisitos originales del proyecto.

## 8. Síntesis para asesor académico

La campaña ha pasado de una revisión estática a evidencia física limitada pero significativa. El resultado más fuerte es la recepción IEEE 802.15.4: en tres ejecuciones formales de 30 segundos sobre canal 25, el CatSniffer V3 con FeralRF entregó 41, 43 y 43 paquetes, con al menos la primera trama de cada ventana marcada como CRC válida y con RSSI/LQI plausibles. Otras dos ventanas produjeron 48 y 100 paquetes. Esto demuestra que la cadena antena/radio CC1352P7→firmware→UART→RP2040→USB→Python produjo datos RF reales y repetibles. No demuestra un stack Zigbee: FeralRF recibió frames raw. La atribución al dispositivo Zigbee conocido es razonable, pero falta correlación simultánea con un segundo receptor y no se conservaron los bytes de todos los paquetes.

También hay evidencia robusta del plano de control. Cinco procesos independientes inicializaron y cerraron el dispositivo sin dejar el puerto ocupado. Ocho valores de PHY recorrieron configuración y start/stop. Una prueba negativa demostró que TX solicitado mientras el estado lógico RX está activo es rechazado por el firmware con `ERR_INVALID_STATE=0x05`, y que STOP/STATS siguen funcionando después. Este último resultado puede trazarse exactamente desde la excepción Python hasta `ControlTask_canStartTx`, que impide encolar la transmisión.

EV-11 amplió la cobertura de configuración propietaria. El código contiene 31 presets; se excluyeron correctamente tres OOK por su lock conocido y MIOTY por soporte nativo pendiente, dejando 27. Los informes registran los 27 como PASS-control y no les asignan PASS-RF. La aritmética y las exclusiones son correctas, y sólo hubo un duplicado (`gfsk_868_50k`), no contado dos veces. Sin embargo, la evidencia no tiene la misma calidad para todos: hay stdout individual para 18 presets, mientras los nueve de 902/915 MHz aparecen sólo en un resumen. Además, el script recibe ACK antes de que RX/TX RF necesariamente terminen y filtra ciertos errores asíncronos. Por eso EV-11 acredita que el flujo síncrono de control terminó y el equipo siguió respondiendo; no acredita ondas, frecuencia, modulación ni interoperabilidad Wi-SUN, Sidewalk o Wireless M-Bus.

El problema concreto ya reproducido es KI-15. Catnip agrupó la placa como V3 con Bridge COM88, LoRa COM86 y Shell COM87. FeralRF calcula el Shell como Bridge+2 y elegiría COM90. La campaña hizo lo correcto: no llamó la API pública contra el puerto equivocado. Al enviar manualmente `boot` a COM87, FeralRF dejó de responder por COM88; tras `exit`, INIT y STATS regresaron sin desconectar USB. Eso apoya el mecanismo subyacente de recuperación una vez, pero la API permanece bloqueada. También surgió una contradicción documental: FeralRF PINOUT/guía hablan de GPIO15, mientras el target RP2040 oficial actual usa el alias de reset en GPIO3 y boot en GPIO2.

Los problemas históricos de RF continúan abiertos. En particular, un PASS-control a 433 MHz no elimina el antecedente de OTA marginal ni el fallo OOK 433. Los resets entre bandas usados en EV-10/11 fueron un workaround prudente derivado del baseline histórico; no demuestran que HEAD aún necesite reset ni caracterizan el fallo sin reset. OOK debe tratarse en EV-24, 433 en EV-23 y MIOTY en EV-52. EV-12 no fue ejecutada: estudiar sus scripts no equivale a validar TX.

La confianza apropiada en el baseline actual es moderada para transporte, sesión, rechazo de estado y recepción IEEE 802.15.4 en la condición probada; baja o nula para transmisión física, exactitud espectral, otras PHY por aire y protocolos completos. La metodología es creíble porque conserva comandos y stdout, separa ACK de RF, usa una fuente independiente y documenta fallos del harness. Debe fortalecerse con hashes del binario, captura raw/PCAP, observadores independientes, logs individuales completos y consumo explícito de errores RF asíncronos. Hasta entonces, la formulación académicamente defendible es “baseline de control ampliamente recorrido y una ruta RX IEEE validada físicamente”, no “FeralRF validado en todas sus capacidades”.

## Questions / evidence gaps that should be resolved before continuing validation

1. ¿Qué hash exacto del `.hex` está flasheado en DUT-V3 y qué build reproducible lo vincula con `FeralRF@0178721`?
2. ¿Puede recuperarse la salida individual original de los nueve presets 902/915 MHz y la evidencia del reset 868→169, o debe conservarse explícitamente como pérdida de evidencia?
3. ¿Qué canal, duración, comando y stdout correspondieron al control negativo informal de EV-05?
4. ¿Puede una captura simultánea independiente correlacionar bytes/tiempos de canal 25 con el DUT para cerrar la atribución al `PROTOCOL-DEVICE-ZIGBEE-CH25`?
5. ¿Qué revisión/esquema explica la discrepancia GPIO15 frente al overlay RP2040 GPIO3 para `RESET_CC`?
6. ¿Existe hash o manifest del `C:\Program Files\Catnip\catnip.exe` que lo vincule con un commit concreto, además del encabezado `v3.3.3.0`?

````

## Addendum experimental posterior — 2026-10-08

Este addendum no reescribe la auditoría del 7 de octubre ni hace parecer que ésta conocía evidencia posterior. El corte de cuatro corridas, 40 solicitudes aceptadas y cero hits de [[Reporte OTA Sub-GHz - GFSK 868 y 915 MHz]] era correcto para su fecha; ahora es un subconjunto de [[Segunda campaña OTA proprietary FeralRF — Ampliación de presets y verificación del procedimiento]].

### Magnitudes verificadas

- 24 corridas acumuladas: cuatro iniciales y 20 posteriores.
- 240 retornos exitosos de `transmit()`: 40 + 200.
- 19 presets únicos; las repeticiones bidireccionales y concurrentes no se cuentan como presets nuevos.
- Cero paquetes/hits proprietary entregados y cero `RxStreamError` expuestos.

No se promueven los retornos a emisiones ni se calcula pérdida RF.

### Cambio metodológico relevante

El control inverso `gfsk_2440_50k` mantuvo `read_packets()` activo desde `12:34:13.916`; los ACK TX ocurrieron entre `12:34:17.980` y `12:34:19.241`, y RX terminó a `12:34:43.931`. Quedaron `24.690 s` de observación después del último ACK y el resultado fue `events=0, packets=0, hits=0, errors=0`.

Esto debilita la lectura tardía del helper como explicación única, pero no acredita TX físico ni buffering universalmente sin pérdidas. Los dos `DeviceStats` posteriores en cero solo cubren los contadores expuestos para el intervalo pertinente.

### Relectura por validez técnica

- Los GFSK/FSK ejecutados en 868/902.2/915/2440, incluidos W-MBus S/T/C, Sidewalk y Wi-SUN como PHY/raw, son observaciones OTA negativas interpretables: aceptación de control y ninguna entrega peer; emisión, recepción y causa siguen abiertas.
- `msk_868_50k`, `4fsk_868_50k` y `4gfsk_868_50k` no acreditan su modulación nominal: el comando TI auditado reserva `modType` 4/5/6. Solo acreditan preset existente, solicitudes aceptadas y cero paquetes observados.
- Los ocho presets 433 no se ejecutaron: el silicio cubre 433, U4 publicado no, y el asesor impuso una restricción operacional. W-MBus N169 queda fuera del silicio y U4.
- OOK se difirió por lifecycle/recuperación y MIOTY por implementación pending/CPE.

La campaña actual obtuvo `0/70` para los mismos cinco nombres Wi-SUN y dos Sidewalk del claim histórico `70/70`. **No reprodujo el claim bajo las condiciones actuales**, pero no demuestra regresión porque no se ha probado equivalencia de imágenes, revisiones, dispositivos y procedimiento.

### Conclusión del addendum

El plano de control funciona, el cache de banda cambia y el control concurrente confirma solapamiento RX/ACK TX. Ningún paquete proprietary fue entregado. El dominio de fallo sigue abarcando configuración, ejecución TX/RX, ruta externa y entrega host. Se justifica detener la expansión ciega y pasar a una observación física discriminante `gfsk_868_50k` sobre CTF/U2/J1, según [[Seguimiento documental OTA Sub-GHz - ruta RF y control de banda#Siguiente observación física discriminante]].
