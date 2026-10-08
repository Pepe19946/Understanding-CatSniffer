# FeralRF - Matriz de pruebas

Matriz canónica de ejecución y cobertura. Reconciliación documental: 7 de octubre de 2026. No constituye una nueva ejecución ni modifica criterios retrospectivamente. Las fuentes históricas se conservan íntegras al final. Para requisitos, [[Matriz de capacidades]] y [[FeralRF - Wiki técnica integral]]; procedimientos, [[FeralRF - Guía de validación experimental]]; razonamiento temporal, [[Registro de validación FeralRF]]; juicio técnico y próximos experimentos, [[Auditoría técnica de validación FeralRF - EV ejecutadas]].

## Criterios de lectura

Procedimientos canónicos: [[FeralRF - Guía de validación experimental#Procedimiento operativo completo por EV|procedimientos por EV]] y [[FeralRF - Guía de validación experimental#Procedimiento operativo de Catnip CLI|preflight y roles Catnip]]. La enmienda inicial de la guía actualiza mapa y precedencia.

PASS acredita sólo el objetivo/observable acotado y no se asigna sólo por un comando exitoso fuera del objetivo. Un criterio receptor FAIL puede coexistir con un requisito DUT INCONCLUSIVE si el experimento no localiza emisión frente a observación/entrega. PARTIAL indica cobertura incompleta; FAIL se refiere a un criterio efectivamente observado como incumplido; INCONCLUSIVE no discrimina la pregunta; NOT FULLY VALIDATED conserva una pregunta central sin evidencia suficiente. “No ejecutado” describe ejecución, no un fallo del producto. “Bloqueado” identifica dependencia pendiente; “retirado” no es FAIL. High/Medium/Low califican la inferencia acotada, no todo el sistema. Los resultados oficiales históricos y los mocks no son validación HIL de esta campaña. No hay logs/PCAP/binarios adjuntos; evidencia es texto incrustado en las notas.

## Modelo de evidencia A–F

| Nivel | Tipo de evidencia | Alcance |
|---|---|---|
|A|Evidencia física RF directa|Receptor o instrumento independiente, incluida otra radio que observa separadamente al transmisor|
|B|Evidencia directa de comportamiento del dispositivo/firmware|Comportamiento de ejecución observado, sin caracterización RF independiente|
|C|Evidencia de control/API|Aceptación, ACK, respuesta, configuración o estado reportado; indicar si usa hardware real o mocks|
|D|Evidencia estática de implementación/documentación|Contrato o análisis de la fuente de referencia; no acredita el binario instalado|
|E|Evidencia indirecta/hipótesis|Inferencia no localizada directamente|
|F|No evaluado|Sin evaluación de la dimensión indicada|

El nivel es independiente del estado PASS/PARTIAL/FAIL/INCONCLUSIVE/NOT FULLY VALIDATED, de la confianza High/Medium/Low y de la procedencia. F no tiene confianza experimental asignable; Low de una fila no ejecutada califica el respaldo de la afirmación, no una medición inexistente. La confianza en D se refiere al análisis documentado, no al binario cargado. Una misma capacidad puede tener C para control y F para RF, o A para presencia del marcador y E para su causa/conteo físico. F no significa FAIL. La evidencia narrada debe distinguirse de una transcripción individual; una fecha o hash ausente limita atribución/reproducibilidad sin borrar automáticamente la observación.

Otra placa FeralRF cuenta como A al observar RF del transmisor: compartir implementación limita independencia de fallos y metrología, pero no elimina la evidencia OTA. Los paquetes/metadatos entregados por el DUT que recibe tráfico ambiental cuentan como B. FakeSerial aporta C exclusivamente host/mock; no demuestra B del dispositivo ni A de RF. Los hallazgos de fuente son D para el análisis referenciado; sin vínculo de despliegue no son comportamiento confirmado del firmware cargado.

## Reconciliación de los 38 ítems de la guía

Todos estaban planificados; los huecos numéricos son agrupaciones del plan, no EV desaparecidos. EV-00/01/02 tienen evidencia dentro del registro, no archivos individuales. Ningún EV fue renumerado. Cambios de criterio se registran en la enmienda de la guía.

| Ítem / intención original | Ejecución real / evolución | Resultado y confianza | Nivel A–F | Evidencia | Brecha para cierre |
|---|---|---|---|---|---|
|EV-00 identidad de puertos|Parcial; discovery inicial disponible|PARTIAL / Medium|C; F asociación completa|[[Registro de validación FeralRF]]|Debug/HWID, identify/status/LED y manifest físico por época|
|EV-01 INIT/GET_INFO/STATS|Parcial; varias lecturas, sin repetición formal completa|PARTIAL / Medium|C; F serie formal completa|Registro y EV-03/04|3/3 con condiciones/tiempos y binario fijados; no confundir constante serial con identidad|
|EV-02 RX control/stop|Ejecutado en registro temprano|PASS control / High|C|[[Registro de validación FeralRF]] original|No extrapolar a todos los estados; timeouts posteriores requieren investigación|
|EV-03 reconnect limpio|5 procesos independientes|PASS acotado / High|C|[[EV-03 — Reconexión limpia entre procesos]]|No unplug ni misma instancia ni interrupción|
|EV-04 reset/reinit|Manual un ciclo; selección API equivocada|NOT FULLY VALIDATED / Medium|C selector; B/C manual; F ciclo API|[[EV-04 — Reset y reinicialización]]|Mapeo por placa, repetir criterio de guía; API no ensayada como ciclo completo|
|EV-05 RX IEEE inicial|3×30 s, 41/43/43 paquetes|PASS RX local + B / High entrega; Medium atribución|B; C control|[[EV-05 — Primera observación RF IEEE]]|Bytes y captura concurrente independiente para verificar el emisor declarado; canal negativo controlado|
|EV-06 exclusión RX/TX|Dos ejecuciones válidas; error 0x05 y recuperación|PASS control / High|C; F ausencia de emisión física|[[EV-06 — Exclusión RX y TX y recuperación de estado]]|RF del rechazo no medida; otros modos/estados|
|EV-10 matriz PHY control|8/8 con reset entre filas|PASS control / High|C; F RF conjunto|[[EV-10 — Matriz de PHY por control]]|OTA y cambios sin reset|
|EV-11 presets control|27 pruebas de control reportadas; 18 stdout individuales, nueve resumidas|PARTIAL / High 18 transcripciones; menor confianza nueve resúmenes|C individual/resumido; F OTA|[[EV-11 — Control de presets propietarios]] y dos complementos|Nueve logs 902/915, parámetros efectivos y RF de 27; OOK/MIOTY fuera de alcance|
|EV-12 RAW/FRAME/BURST/CONT/STOP|Control 4/4→OTA 6-10-2026→criterios de conteo más fuertes|PARTIAL global / Medium; criterio receptor FAIL, semántica DUT INCONCLUSIVE|A/C; E semántica/conteo|[[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]] y preliminar|Scheduler/aborto/observador, TX efectivo, cese físico, roles|
|EV-13 CW/PRBS instrumentados|Adaptado a control 0 dBm; medición física diferida|PARTIAL / Medium|C/B; F onda/cese|[[EV-13 — CW PRBS y TX_TEST_STOP por control]]|Energía/frecuencia/patrón/cese; instrumento identificado|
|EV-14 error asíncrono/firma|Baselines, transición mínima, 3 mocks; error físico no inducido|NOT FULLY VALIDATED / Medium|B; C host/mock; F error inducido|[[EV-14 — Eventos RF asíncronos y firma RX]]|Estímulo de fallo real y correlación de error; ausencia no prueba eliminación|
|EV-15 límites/parámetros inválidos|No ejecutado dedicado|NOT FULLY VALIDATED / Low|D/F|Guía y [[Protocolo y API Python]]|Fronteras 125/239/31/240, errores sin corrupción, formatos/COBS/CRC|
|EV-20 IEEE OTA dos placas|Parcial indirecto por RAW/FRAME de EV-12|PARTIAL / Medium|A/C parcial EV-12|EV-12 OTA|Serie bidireccional 10/10, observador independiente, roles y bytes|
|EV-21 BLE raw 1M/2M/Coded|1M RX ambiental parcial en EV-13/14; otros sólo control EV-10|PARTIAL / Low|B BLE1M; C demás; F OTA completo|EV-10/13/14|TX/OTA cuatro PHY y repetición; reconciliar criterios iniciales/operativos|
|EV-22 sub 868/915/4FSK|Campaña actual: GFSK 868/902/915 con cero paquetes entregados; MSK/4-(G)FSK solicitados usan `modType` reservados|FAIL del criterio receptor; TX/RX físicos y causa INCONCLUSIVE / Medium|C control; A negativa de entrega peer; D para límite `modType`|[[Segunda campaña OTA proprietary FeralRF — Ampliación de presets y verificación del procedimiento]]|Una observación física `gfsk_868_50k` en CTF/U2/J1; no tratar los nombres MSK/4FSK como modulación demostrada|
|EV-23 caracterización 433|No ejecutado en campaña actual: U4 publicado no cubre 433 y el asesor indicó detener/no probar|BLOCKED operacional/hardware publicado / Medium|D hardware; F RF actual|Segunda campaña; fuentes primarias|El silicio sí cubre 433; no inferir causa eléctrica exacta sin revisión ensamblada|
|EV-24 OOK/lock/recovery|Diferido: lifecycle OOK puede exigir reset/power cycle; API Shell insegura; OOK433 además bloqueado por U4|DEFERRED / Medium|D implementación; F actual|Guía KI-01/05; segunda campaña|Procedimiento de recuperación específico antes de ejecutar|
|EV-25 W-MBus S/T/C/N|S/T/C: 30 solicitudes aceptadas, cero paquetes entregados; N169 no ejecutado y fuera de silicio/U4|FAIL del criterio receptor S/T/C; causa INCONCLUSIVE; N169 BLOCKED / Medium|C control; A negativa de entrega; D límite 169; F interop|Segunda campaña; [[Sub-GHz en CatSniffer V3 - CC1352P7, ruta RF y control de banda]]|Diagnóstico físico representativo, no expansión; PHY≠stack; retirar N169 de campaña P7|
|EV-26 Wi-SUN/Sidewalk|Cinco Wi-SUN + dos Sidewalk: 70 solicitudes aceptadas, cero paquetes entregados|FAIL del criterio receptor; causa/interop INCONCLUSIVE / Medium|C control; A negativa de entrega; F interop|Segunda campaña|No reprodujo `70/70` histórico; no llamar regresión sin equivalencia de binario, placa y procedimiento|
|EV-27 propietario 2,4GHz|GFSK 50k/250k y controles concurrentes: cero eventos/paquetes; RX solapó ACK TX|FAIL del criterio receptor; TX/RX físicos y causa INCONCLUSIVE / Medium|C control; A negativa de entrega; B/C timing y stats|Segunda campaña|Observar RF en J1; el control concurrente debilita la explicación de lectura tardía, no valida RF|
|EV-28 emulación PHY-level|No ejecutado dedicado|NOT FULLY VALIDATED / Low|D/F helper|Guía/API|Helpers con captura RF; no MAC/stack superior|
|EV-29 KillerBee real|Bloqueado por dependencia/baseline; mocks históricos|NOT FULLY VALIDATED / Low|D/F real|Guía y fuentes|Paquete/parche externo, sniff/inject real con artefactos|
|EV-30 crypto hardware/vectores|No HIL actual; 9/9 histórico 30-4-2026|NOT FULLY VALIDATED actual / Low|D historia/F actual|[[Pruebas y evidencia existente]]|Vectores independientes, binario actual, salidas completas|
|EV-31 crypto límites/repetición|No ejecutado actual|NOT FULLY VALIDATED / Low|D/F actual|Guía/API|Bounds/auth/curvas/modos y stress, no contar skip como éxito|
|EV-40 baseline completo actual|No ejecutado completo; reset/catnip/KillerBee pendientes|NOT FULLY VALIDATED / Low|C parcial/F completo|Guía/EV-04|Manifest dos placas, descubrimiento por interfaz, paquete de evidencia|
|EV-41 cambios sin reset|Transición BLE1M→IEEE mínima EV-14|PARTIAL / Medium|B/C mínimo|EV-14|Ciclos/matriz completa; criterio original 10 vs operativo 3 explícito|
|EV-42 reset/entre bandas|Resets EV-10/11 parciales; no campaña dedicada|PARTIAL / Low|C parcial; F serie|EV-10/11|Repeticiones y prueba interbandas; recuperar reset 868→169|
|EV-43 RX soak/contadores|Ventanas cortas, no soak 60/300 s de guía|NOT FULLY VALIDATED / Low|B corto; C stats; F soak|EV-05/12/13|Carga/tasa controladas, serial completo y todas las colas|
|EV-44 presión cola/burst|No stress válido; generador BURST anomalía EV-12|NOT FULLY VALIDATED / Low|A anomalía12; F capacidad|EV-12|Generador fiable y conteo independiente antes de medir pérdidas/capacidad|
|EV-45 wrap SEQ >253|No 300 comandos reales documentados|NOT FULLY VALIDATED / Low|D/F wire actual|Guía; tests host históricos|Campaña wire con correlación antes/después del wrap|
|EV-46 reinit/ciclo vida|Procesos EV-03 no equivalen a 20 init misma instancia|NOT FULLY VALIDATED / Low|C procesos; F lifecycle completo|EV-03/14|Criterio explícito init vs init+RF; estado/timebase y eventos|
|EV-47 interrupción/error|No Ctrl+C/recuperación dedicada|NOT FULLY VALIDATED / Low|D/F|Guía|Interrupciones controladas y reconexión con evidencia|
|EV-50 jamming continuo físico|No ejecutado; experimental|NOT FULLY VALIDATED / Low|D/F|Guía/API|Montaje y criterio físico, recuperación; IDs reactive/pattern no implementados|
|EV-51 spectrum/RSSI scan|Bloqueado por ruta incompleta|NOT FULLY VALIDATED / Low|D incompleto/F|[[FeralRF - Wiki técnica integral]], [[Protocolo y API Python]]|Contrato/handler/data path; RSSI por paquete no es spectrum scan|
|EV-52 MIOTY TS-UNB|Diferido; implementación pending, 0/10 histórico y probable CPE custom|DEFERRED / Medium documental|D historia/implementación; F actual|Guía/API; segunda campaña|Implementación y contrato antes de nueva RF; preset no demuestra TS-UNB|
|EV-53 RSA/AIS/802.15.4g/HighPA|Bloqueado/incompleto; no ejecución actual|NOT FULLY VALIDATED / Low|D incompleto/F|Wiki/guía|Requisito y soporte efectivo; DIO29/rango de potencia por medir|
|EV-54 stack BLE retirado|No ejecutado; retirado deliberadamente|No aplica al alcance actual / High documental|D retirada/no aplica|[[Arquitectura FeralRF]], [[FeralRF - Wiki técnica integral]]|No crear FAIL por ausencia de GATT/conexión/active scan|

## Cobertura consolidada por capacidad

La fuente de definición es el contrato documentado, no una verificación nueva del código. Las cifras históricas permanecen en el original y en fuentes; no se suman a los ensayos actuales. Las filas de capacidad integran varios EV sin duplicar experimentos.

| Capacidad / intención | Definición | Ítem guía | Evaluaciones | Ensayo real | Resultado / confianza | Nivel A–F por dimensión | Evidencia | Límite conocido | Validación restante |
|---|---|---|---|---|---|---|---|---|---|
|CC1352P7 vía RP2040 triple CDC; SX1262 separado|[[Arquitectura FeralRF]], [[Firmware RP2040]]|00/01/40|Registro,03/04|Bridge INIT y Shell manual|PARTIAL / Medium|D arquitectura; C comunicación; B recovery|Textos seriales|Binario y unidad no trazados; no target V2 probado|Manifest USB, versiones y rutas|
|COBS+CRC16, SEQ, errores, sin fragmentación|[[Protocolo y API Python]]|01/14/15/45|06/14|Error 0x05 real y SEQ0/FF en mocks|PARTIAL / High acotado|C real/error; C host/mock; D framing|Excepción y 3 mocks|No malformed/wrap wire; Wiki discrepa SEQRX|Wire y fronteras; correlación exhaustiva|
|API Python init/info/stats y ciclo serial|[[Protocolo y API Python]], [[FeralRF - Wiki técnica integral]]|01/03/46/47|03 y registro|5 procesos init/cierre|PASS acotado / High|C|5 DeviceInfo|No lifecycle misma instancia/interrupción|20 init, Ctrl+C, reconexión USB|
|Reset/recovery y selección Shell|[[Arquitectura FeralRF]], [[Protocolo y API Python]]|04/40/42|04/10/11|Manual boot/exit; selector COM90≠87|NOT FULLY VALIDATED / Medium|C selector; B/C manual; F ciclo API|Traceback/puertos|Aritmética COM no universal, GPIO revisiones|Identidad por interfaz y ciclos|
|Estados exclusión RX/TX|[[Arquitectura FeralRF]], [[Protocolo y API Python]]|06/41|06|TX01−20 durante RX→0x05, stop|PASS control / High|C rechazo; F ausencia de emisión física|Salida literal|Ausencia RF no medida; otros estados|Matriz de estados y eventos tardíos|
|8 PHY seleccionables|[[Matriz de capacidades]]|10/21/22/41|10|8 filas con reset, BLE2M canal 9|PASS control / High|C selección/ACK; F RF conjunto|8 stdout|Reset oculta transición; no OTA completo|RX/TX por PHY sin confundir control|
|IEEE 802.15.4 RX y metadatos|[[Arquitectura FeralRF]], [[Protocolo y API Python]]|05/20|05/12/14|3×30 s y marcadores/bytes posteriores|PASS local / High entrega; Medium atribución|B RX local; A observador EV-12; C control|Conteos/bytes|CRC true del primer paquete en tres corridas; atribución no correlacionada independientemente; bytes incompletos; PER/sensibilidad no medidos|Baseline controlado, roles, calibración|
|RAW/FRAME IEEE TX con marcador|[[Protocolo y API Python]]|12/20|12|Marcadores y control sin TX|PASS entrega OTA acotada / High|A marcador OTA; C ACK; F conteo exacto/completitud|DEADBEEF/A1B2C3D4|Receptor independiente físicamente, mismo firmware; conteo exacto de una emisión no establecido; sin 10/10 roles|Serie bidireccional y medición independiente|
|BURST conteo/intervalo|API/Arquitectura|12/44|12|40/25000 y 5/250000 reciben 1|INCONCLUSIVE conteo DUT; FAIL criterio receptor / High observación|A registros; C solicitud; E causa/conteo físico|Tres condiciones OTA|No conteo TX efectivo; causa no confirmada|Instrumentar reloj/retornos y observador|
|CONTINUOUS repetición de paquetes|[[Protocolo y API Python]]|12|12|CONT0:99 registros coincidentes CRC-válidos; positivos ensayados:1 match|PARTIAL; positivos ensayados INCONCLUSIVE DUT / High registros|A registros; C control; E conteo exacto; F cadencia/cese|102 total/99 hits y salidas|Sin conteo físico exacto ni cese; no generalizar a todo intervalo/PHY|Tiempo/aborto y roles; medir STOP|
|STOP y eventos TX|API/Arquitectura|12/13|12/13|ACK STOP; idle dos ACK|PARTIAL / High control|C ACK; D contrato; F cese físico|Salidas|Sin TX_DONE; cese no observado|Emisión/fin trazables y cese físico|
|RX_STOP y drenaje/correlación|[[Protocolo y API Python]]|02/43/44|Registro/12/13/14|ACK en algunas corridas; timeout host en sesiones con distintos totales de reportes|PARTIAL con anomalía / High|C ACK/timeouts; E causa|9/5/4 respuestas inesperadas; sólo último ID 0x90 preservado|Timeout no demuestra fallo físico de stop|Captura wire+estado+ACK y carga controlada|
|BLE1M RX|[[Matriz de capacidades]], [[Protocolo y API Python]]|21|13/14|Recepción ambiental 2/10/11; transición|PARTIAL / Medium|B entrega local; C control|Conteos|Sin bytes BLE completo/peer controlado|TX y RX marcados/interoperables|
|BLE2M/Coded S8/S2; passive advertising/hop|[[Matriz de capacidades]], [[FeralRF - Wiki técnica integral]]|21/41|10|Sólo selección/control PHY|NOT FULLY VALIDATED / Low RF|C selección; D opciones; F OTA/adv/hop|Smoke|Workarounds 2M; adv/hop no ensayo dedicado|Cuatro PHY OTA, adv/hop y repetición|
|27 presets propietarios|[[Matriz de capacidades]], [[FeralRF - Wiki técnica integral]]|11/22–28|11|Control 18 stdout+9 resúmenes|PARTIAL / Medium|C 18 individuales+9 resúmenes; D inventario; F OTA por preset|3 notas complementarias|ACK config no verifica RF; OOK/MIOTY excluidos|Parámetros efectivos y OTA por preset|
|Sub868/915 y 4FSK|[[Matriz de capacidades]], [[Protocolo y API Python]]|22|10/11 + segunda campaña|GFSK 868/902/915: cero entregas; MSK/4-(G)FSK no interpretables por `modType` reservado|FAIL receptor / INCONCLUSIVE RF / Medium|C; A negativa; D modType|Reportes OTA|ACK no prueba TX; selector no medido|CTF/U2/J1 con `gfsk_868_50k`|
|433 GFSK/FSK/MSK y variantes|[[Matriz de capacidades]], [[Protocolo y API Python]]|23|11 complemento 433|6 control|NOT FULLY VALIDATED RF / Low|C; F OTA actual|6 stdout|Marginal histórico no revalidado|Caracterización/antena/ruta CTF|
|OOK y lock/recovery|[[Matriz de capacidades]], [[Protocolo y API Python]]|24|Ninguno actual|Historial reportado, no nueva ejecución|NOT FULLY VALIDATED / Low|D historia; F ejecución actual|Guía KI01/05|Lock documentado; reset fiable prerequisite|Medir y recuperar, último bloque|
|WMBus S/T/C/N169 PHY|[[Matriz de capacidades]], [[Protocolo y API Python]]|25|11 + segunda campaña|S/T/C: `0/30`; N169 no ejecutado|FAIL receptor S/T/C / INCONCLUSIVE físico; N169 no soportado|C; A negativa; D límite 169; F interop|Segunda campaña; fuentes primarias|PHY/raw no es stack; TX no confirmado|Diagnóstico representativo; excluir N169|
|Wi-SUN/Sidewalk PHY|[[Matriz de capacidades]], [[Protocolo y API Python]]|26|11 + segunda campaña|Actual `0/70` para 5+2 nombres; histórico `70/70`|No reproducido; no regresión demostrada / Medium|C; A negativa; F interop|Segunda campaña e historia|Condiciones/binarios históricos no equivalentes|Instrumentación del caso representativo antes de interop|
|Propietario 2440|[[Matriz de capacidades]], [[Protocolo y API Python]]|27|11 + segunda campaña|50k/250k, ambas direcciones y RX concurrente: cero entregas|FAIL receptor / INCONCLUSIVE físico / Medium|C; A negativa; B/C timing/stats|Segunda campaña|Historial contradictorio; ACK no es emisión|RF en J1 y parámetros si hay energía|
|Emulación PHY-level|[[FeralRF - Wiki técnica integral]], [[Protocolo y API Python]]|28|Indirecto 11|Sólo configuración asociada|NOT FULLY VALIDATED / Low|D; C configuración asociada; F helper dedicado|Sin helper dedicado|No upperstack ni señales capturadas|Captura y comparación externa|
|CW/PRBS15/32|[[Protocolo y API Python]]|13|13|Inicio/stop 0 dBm 0,3 s host|PARTIAL / High control|C; D backend descrito; F onda/patrón/cese|ACK|Sin energía/patrón/frecuencia/cese|Instrumento identificado y medición|
|Errores RF asíncronos/firma|API/Arquitectura|14|14|Cero eventos/firma y 3 mocks|NOT FULLY VALIDATED HIL / Medium|B ventanas negativas; C host/mock; F error RF inducido|Bytes IEEE y pytest|Error real no inducido|Estímulo reproducible/captura wire|
|Límites TX125/RX239/BLE31/SHA-TRNG240|[[Protocolo y API Python]]|15/31|Ninguno dedicado|Definición y tests históricos|NOT FULLY VALIDATED actual / Low|D; F fronteras actuales|Documentación|Sin fragmentación; límites de componentes distintos|Fronteras y errores sin truncado/corrupción|
|TRNG, AES, SHA256, ECDH, ECDSA|[[Protocolo y API Python]], [[FeralRF - Wiki técnica integral]]|30/31|Ninguno HIL actual|9/9 histórico y mocks|NOT FULLY VALIDATED actual / Low|D historia/tests; F HIL actual|Historia 30-4-2026|RSA ausente; Curve25519 ECDH no ECDSA|Vectores/autenticación/bounds/curvas reales|
|KillerBee sniff/inject/jam adapter|[[Matriz de capacidades]], [[FeralRF - Wiki técnica integral]]|29|Ninguno real actual|Mocks/dependencia ausente|NOT FULLY VALIDATED / Low|D mocks/dependencia; F RF vivo|Baseline 422 pass/1 skip reportado|Parche externo, sin integración RF|Instalar/fijar dependencia y capturar RF|
|Colas, contadores, soak, wrap SEQ|[[Arquitectura FeralRF]], [[Protocolo y API Python]]|43/44/45|Indirectos 05/12/13|Ventanas cortas, contadores cero|NOT FULLY VALIDATED / Low|C contadores; B entrega corta; D colas; F soak/wrap/capacidad|Notas|Output depth 32 y drops no todos reflejados en STATS|Carga controlada, wire, estados y wrap|
|Cambios PHY/banda y reinit|[[Arquitectura FeralRF]], [[Protocolo y API Python]]|41/42/46|10/11/14|Reset entre casos; transición mínima sin reset|PARTIAL / Medium|C reset-separado; B/C transición mínima|Salidas/relaciones|Deadlocks históricos; no campañas completas|Secuencias/ciclos/binarios y criterio claro|
|Jamming continuo|[[Matriz de capacidades]], [[Protocolo y API Python]]|50|Ninguno actual|Control histórico sólo|NOT FULLY VALIDATED / Low|D; F actual|Guía|Experimental, posible excepción RF_runCmd|Efecto físico controlado y recovery|
|Reactive/pattern, spectrum scan|[[FeralRF - Wiki técnica integral]], [[Protocolo y API Python]]|50/51|Ninguno|IDs reservados/builders sin ruta completa|Bloqueado / High documental|D incompletitud; F experimento|Análisis estático reportado|RSSI paquete no es scan|Definir/implementar contrato antes de HIL|
|MIOTY TS-UNB|[[Matriz de capacidades]], [[Protocolo y API Python]]|52|Ninguno actual|0/10 histórico|NOT FULLY VALIDATED actual / Low|D historia/pending; F actual|Historia|Preset 396≠TSUNB/CPE patch|Implementación y contrato antes de RF|
|RSA/AIS/802.15.4g/HighPA|[[FeralRF - Wiki técnica integral]], [[Arquitectura FeralRF]]|53|Ninguno|Ausencia/incompleto documentado|Bloqueado / High documental|D ausencia/incompleto; F actual|Guía|DIO29 no enrutado; +5/+14 sin medida|Requisito/implementación y potencia física|
|BLE GAP/GATT/conexión/active scan|[[Arquitectura FeralRF]], [[FeralRF - Wiki técnica integral]]|54|No aplica|Retirado 20-7-2026|Retirado / High documental|D retirada; fuera del alcance actual|Historia|Restos internos no API alcanzable|Limpiar promesas, no contar como fallo|
|V2/P1 y flasheo.hex/.bin|[[Fuentes firmware oficial]], Wiki|00/40, transversal|Ninguno actual|Target P7 documentado|NOT FULLY VALIDATED / Low|D target; F compatibilidad/flasheo actual|Fuentes|No target V2 FeralRF ni provisioning TI sniffer V2 probado|No asumir compatibilidad por familia MCU|

## Distinciones necesarias para PHY y presets

EV-10 demuestra exposición/selección y aceptación de ocho secuencias de control; no TX, recepción efectiva, frecuencia, modulación, forma de onda o interoperabilidad de los ocho PHY. ACK de RX_START/RX_STOP no verifica el estado interno RF. IEEE y BLE1M posteriores aportan evidencia de esas rutas específicas, sin promover las ocho filas a RF PASS.

Los 27 presets son el subconjunto planificado de 31 declarados, excluyendo tres OOK y MIOTY. Existencia D; selección/aceptación C (18 transcripciones y nueve resúmenes); estado/backend efectivos no verificados por preset; emisión, parámetros correctos e interoperabilidad F en la campaña actual. Los nueve resúmenes no prueban no ejecución, pero no tienen la auditabilidad individual de los otros 18.

**Actualización experimental 2026-10-08:** la campaña OTA posterior recorrió 19 presets únicos en 24 corridas y registró 240 retornos `transmit()` exitosos, cero paquetes entregados y cero `RxStreamError` expuestos. Es un conjunto distinto del barrido EV-11 de control. Ocho presets 433 y dos N169 quedaron bloqueados; OOK y MIOTY, diferidos. Los casos `mod_type=4/5/6` solo acreditan aceptación, no MSK/4FSK/4GFSK efectiva.

## Cobertura que cambió de interpretación

EV-12 demuestra por primera vez aquí TX IEEE por aire, de forma acotada; no convierte automáticamente EV-20 en una serie bidireccional completa. El criterio receptor fallido y el conteo DUT no resuelto tampoco constituyen una medición de capacidad RX de EV-44. EV-13/14 amplían BLE RX y cambio sin reset, sin cerrar EV-21/41. Los fallos históricos de RF y las definiciones estáticas siguen siendo antecedentes; sólo se consideran actuales si existe nueva evidencia bajo condiciones trazadas.

## 17. Notas originales preservadas y material pendiente

Fuente: `FeralRF - Matriz de pruebas.md`. SHA-256 previo: `A27AED0030FA25A984F05BF7F8E0182DC56E9BBDF028E49FE2B7A3C3F2C7DC70`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
# FeralRF - Matriz de pruebas

> La columna **Nuestra validación** conserva el estado auditado de la campaña. Los resultados de `Historia repo` son sólo antecedentes documentales. Las instrucciones ejecutables vigentes están en [[FeralRF - Guía de validación experimental#Procedimiento operativo completo por EV|Procedimiento operativo completo por EV]].

| ID | Cat. | Capacidad | Origen / KI | Estado README/docs | ¿Código? | Cobertura unitaria | Script HW | Historia repo | Nuestra validación | Hardware / externo | Evidencia esperada | Resultado / notas |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| EV-00 | L0 P0 | Puertos/autodetect | baseline; KI-15 | estable implícito | sí | mocks parciales | no | no separada | **PARTIAL** | 2 V3/Windows | `devices --debug`, status, identify, HWID/location | mapa COM existe; falta suplemento |
| EV-01 | L0 P0 | init/info/stats | estable; KI-25 | Stable | sí | strict responses | smoke varios | control 18/18 | **PARTIAL**, control favorable | DUT #1 | stdout/timing 3/3 | completar serie formal |
| EV-02 | L0 P0 | RX start/stop IEEE | estable; KI-31 | Stable | sí | host mocks | `smoke_phy4` | PASS control/OTA | **PASS-control** | DUT #1 | ACK/error/STOP | no repetir salvo regresión |
| EV-03 | L0 P0 | reconnect | stress | no estado | sí | parcial | no | sin dato | **PASS-control 5/5** | DUT #1 | 5/5 ciclos | cerrado para escenario probado |
| EV-04 | L0 P0 | reset/recovery | workaround; KI-15 | requerido | sí host/RP | mock adapter | baseline reset | PASS histórico | **BLOCKED API; PASS-recovery manual 1/1** | Shell explícitos COM87/COM30 | recovery explícito | nunca derivar Shell |
| EV-05 | L0 P1 | primera RX física | Stable; KI-22 | IEEE Stable | sí | no RF | `smoke_phy4` | 10/10 OTA | **PASS-RF RX IEEE**, atribución limitada | DUT + Zigbee CH25; peer opcional | trama/RSSI/LQI/CRC; PCAP pendiente | suplemento simultáneo opcional |
| EV-06 | L0 P1 | exclusión RX/TX | negativo; KI-31 | regla de estado | sí | parcial | no | sin dato | **PASS-control/state-rejection** | DUT #1 | ERROR 0x05+recovery | no repetir |
| EV-10 | L1 P1 | PHY 0–7 control | baseline; KI-02 | Stable/Experimental | sí | enum/contract | `smoke_phase2` | PASS control | **PASS-control 8/8** | DUT #1 | salida por PHY | no acredita RF |
| EV-11 | L1 P1/2 | presets control | baseline; KI-03/04/20 | mixto | sí | schema/roundtrip | `smoke_prop` | mixto | **PASS-control reportado 27/27** | DUT #1, antenas | 18 logs individuales; 9 resumidos | repetir sólo 9 si faltan logs |
| EV-12 | L1 P1 | raw/frame/burst/continuous | Stable; KI-14/29 | Stable | sí | builders | `smoke_tx_*` | PASS ambiguo | NOT YET RUN | 1 + observer ideal | ACK+captura | TX RF |
| EV-13 | L1/2 P2 | CW/PRBS/stop | Stable; KI-07 | Stable | sí | mocks | `smoke_f22` | PASS wire 2026-04-29 | NOT YET RUN | analizador/2 placas | frecuencia/potencia/stop | TX RF |
| EV-14 | L1 P0 | error RF asíncrono | KI-12/31 | documentado | sí | async errors | indirecto | troubleshooting | NOT YET RUN | 1 | timeline/error/bytes | — |
| EV-15 | L1 P2 | boundaries/invalid | KI-29/30 | límites docs | sí | amplia host | no | sin dato | NOT YET RUN | 1 | rechazo+recovery | — |
| EV-20 | L2/3 P1 | IEEE OTA | Stable; KI-22/32 | Stable raw | sí | no RF | `smoke_ota_txrx` | 10/10 2026-04-08 | NOT YET RUN | 2 placas | markers 10/10 | — |
| EV-21 | L3 P1 | BLE raw 4 PHY | Stable raw; KI-09/23 | raw, sin stack | sí | host | OTA | 8–10/10 | NOT YET RUN | 2/observer BLE | markers+2º TX | — |
| EV-22 | L3 P1 | Sub-G 868/915/4FSK | Stable; KI-06 | GFSK/FSK sí; `modType` 4/5/6 reservado | sí | presets | OTA | 10/10 | RUN: cero entregas; causa abierta | 2 + antenas | CTF/U2/J1 | detener sweep |
| EV-23 | L3 P2 | 433 caracterización | KI-04/05 | marginal/experimental | sí | presets | F9/baseline | 1–10/10 | NOT YET RUN | 2/antena 433 | éxitos/100 | — |
| EV-24 | L3 P1/2 | OOK+recovery | KI-01/05 | Stable 868, fail 433 | sí | preset | OTA+auto-reset | 10/10;0/10 | NOT YET RUN | 2 | OTA+firma lock+recovery | TX RF |
| EV-25 | L4 P1/2 | W-MBus S/T/C/N | KI-20/22 | S/T/C plausible; N169 no soportado | preset raw | schema | OTA | S/T/C 10/10; N no | RUN S/T/C: `0/30`; N blocked | 2 + instrumento | energía/marker | PHY≠stack |
| EV-26 | L4 P2 | Wi-SUN/Sidewalk FSK | KI-22 | Experimental | preset raw | schema | F29/demos | 70/70 2026-05-03 | RUN: `0/70`, no reproducción | 2/SDR | energía/espectro | no regresión probada |
| EV-27 | L4 P1/2 | prop 2.4 GFSK | KI-08/32 | README Experimental | sí | preset | OTA | 10/10 contradictorio | RUN: cero entregas, concurrente incluido | 2/SDR | energía+2440 MHz | causa abierta |
| EV-28 | L4 P2 | emulación helpers | KI-22 | PHY-level | sí host | payloads | F17/demos | 7/7 wire | NOT YET RUN | 2/tercero | bytes observados | no stack |
| EV-29 | L4 P2 | KillerBee | KI-15/22/26 | adapter | sí | mocks; 1 skip local | sniff/runbook | RF pendiente | NOT YET RUN | Linux+2/fuente | PCAP/FCS/inject | — |
| EV-30 | L5 P1 | crypto vectors HW | KI-30 | Stable | sí | host vectors/mocks | F25 | 9/9 2026-04-30 | NOT YET RUN | 1 + cryptography | bytes vs oráculo | — |
| EV-31 | L5 P2 | crypto stress/bounds | KI-29/30 | límites | sí | host validation | F25 parcial | sin soak | NOT YET RUN | 1 | 20/20+latencia | — |
| EV-40 | L6 P1 | baseline completo | KI-27 | recomendado | sí | n/a | Bash baseline | full 2026-04-08 | NOT YET RUN | 1/2 + Git Bash | log completo | OOK último |
| EV-41 | L6 P1 | switch sin reset | KI-02/10/11 | FAIL histórico | sí | no HW | F9 parcial | deadlock 2º ciclo | NOT YET RUN | 1 | ciclo/último ACK | — |
| EV-42 | L6 P1 | switch con reset | KI-03/15 | workaround | sí | mocks reset | baseline | PASS histórico | NOT YET RUN | 1 | 10 ciclos | — |
| EV-43 | L6 P2 | RX soak/stats | KI-13 | no claim | sí | stats parser | canary | sin log | NOT YET RUN | 1 + fuente | stats monotónicas | — |
| EV-44 | L6 P2 | queue pressure | KI-13 | limitación | sí | no HIL | burst/canary | sin dato | NOT YET RUN | 2/generador | tx/rx/drop/ovf | TX RF |
| EV-45 | L6 P2 | SEQ wrap | bug histórico | corregido host | sí | `test_radio_seq` | no | timeout TX 253 pre-fix | NOT YET RUN | 1 | 300/300 stats | — |
| EV-46 | L6 P1/2 | re-init/lifecycle | KI-11 | no claim | sí | init mocks | no | sin dato | NOT YET RUN | 1 | 20/20 | — |
| EV-47 | L6 P2 | interrupted recovery | recovery | recomendado | sí | parcial | smokes | sin dato | NOT YET RUN | 1 | baseline posterior | — |
| EV-50 | L7 P3 | jamming continuo | KI-17/18 | Experimental | parcial | adapter mocks | jam smoke | ACK sólo | NOT YET RUN | recinto+observer | PER/espectro | LAB ONLY |
| EV-51 | L7 P3 | spectrum scan | KI-16 | Pending | no E2E | no | no | ninguno | BLOCKED | futura ruta | API+handler+datos | no ejecutar |
| EV-52 | L7 P3 | MIOTY | KI-19 | Planned | preset insuficiente | schema | demo/OTA genérico | FAIL 0/10 | NOT YET RUN | 2/MIOTY | OTA/espectro | — |
| EV-53 | L7 P3 | RSA/AIS/15.4g/High-PA | KI-07/21 | Planned/pending | no o incompleto | no | no | no validado | BLOCKED | futuro equipo | data path+medición | no ejecutar |
| EV-54 | L7 P3 | BLE stack retirado | KI-23 | Removed | no ruta pública | n/a | Sniffle externo | retirado 2026-07-20 | NOT APPLICABLE | Sniffle si se desea | reachability | no es bug |

## Resumen de recursos

- **Disponible ahora:** `DUT-V3-FERAL` #1 (`COM88/COM86/COM87`) y `PEER-V3-FERAL` #2 (`COM31/COM32/COM30`) permiten los controles de una placa y OTA simétrica EV-12, EV-20–28 y EV-44 mediante los procedimientos seguros de la guía. `PROTOCOL-DEVICE-ZIGBEE-CH25` aporta recepción RF real a EV-05/43. Los V2 permanecen stock.
- **Dos V3 con FeralRF:** habilitan evidencia física entre placas, pero siguen siendo la misma implementación. EV-40 permanece bloqueado porque el baseline oficial deriva Shell como `Bridge+2`; no debe ejecutarse sin corregir upstream.
- **AUX-V3-STOCK:** fortalece IEEE/sniffing y tooling oficial; no se presume que cubra OOK, 4FSK o propietario 2.4 GHz.
- **Equipo RF independiente:** EV-13, 23–27 y 50; SDR sirve para energía/frecuencia/decodificación compatible, analizador/power meter para potencia/espectro.
- **Dispositivo/software tercero:** EV-25 (W-MBus), EV-26 (Wi-SUN/Sidewalk), EV-29 (KillerBee/Wireshark), EV-30 (`cryptography`). Un PC necesita radio compatible para producir RF.

## Matriz compacta de roles y preparación

Leyenda: `D`=`DUT-V3-FERAL` #1; `AF`=`PEER-V3-FERAL`/`AUX-V3-FERAL` #2, disponible ahora; `AS`=`AUX-V3-STOCK`; `V2`=`OBS-V2-A/B-STOCK`; `Z25`=`PROTOCOL-DEVICE-ZIGBEE-CH25`; `I`=`RF-OBSERVER`; `P`=`PROTOCOL-DEVICE`; `H`=`HOST-TOOL`. `Sí*` significa sólo la parte indicada. `DISC`, `ZSN` y `FSK` remiten a [[FeralRF - Guía de validación experimental#Procedimiento operativo de Catnip CLI|Procedimiento operativo de Catnip CLI]] §§3–9; allí están los comandos exactos y sus efectos. Para los comandos concretos de cada EV, usar [[FeralRF - Guía de validación experimental#Procedimiento operativo completo por EV|Procedimiento operativo completo por EV]]. `—`=sin acción Catnip adicional después del preflight.

| EV | Mínimo hardware/roles | Ahora D+V2 | Con AF | Con AS | ¿Independiente preferible? | Preparación Catnip | Firmware peer requerido |
|---|---|---|---|---|---|---|---|
| EV-00 | D+H | Sí | Sí | Sí | No | DISC | ninguno |
| EV-01 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-02 | D | Sí control | igual | igual | No | DISC | ninguno |
| EV-03 | D | Sí | igual | igual | No | DISC tras reconexión | ninguno |
| EV-04 | D, Shell verificado | Sí | por placa | por placa | No | DISC; comparar Shell con +2 | ninguno |
| EV-05 | D+Z25 | Sí, RF real | comparación simétrica | observación fuerte | Sí | DISC; ZSN sólo AS/V2 ya compatible | AS `ti_sniffer`; V2 sólo preinstalado |
| EV-06 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-10 | D | Sí control | igual | igual | Para RF | DISC | ninguno |
| EV-11 | D; observer para RF | Sí control; V2 FSK/GFSK parcial | OTA presets | parcial según firmware | Sí | DISC; FSK opcional | AF FeralRF; stock según PHY |
| EV-12 | D; receptor/I para PASS-RF | Sí control; V2 FSK parcial | Sí completo simétrico | IEEE/BLE/FSK parcial | Sí | DISC; ZSN/FSK según PHY | AF FeralRF o receptor compatible |
| EV-13 | D+I | No instrumentado | smoke parcial | no sustituye I | Sí, obligatorio | DISC | ninguno; instrumento |
| EV-14 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-15 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-20 | D+AF para script; D+Z25 para RX | Sí* RX independiente | Sí baseline | Sí observer IEEE | Sí para compliance | DISC; ZSN AS | AF FeralRF / AS TI sniffer |
| EV-21 | D+AF o sniffer BLE | No con V2 preservado | Sí baseline | parcial con Sniffle | Sí | DISC; BLE sniff puede auto-flash | AF FeralRF / stock Sniffle |
| EV-22 | D+AF | V2 FSK/GFSK parcial | Sí | FSK/GFSK parcial SX1262 | Sí | DISC; FSK | AF FeralRF |
| EV-23 | D+AF+antena 433 | V2 FSK/GFSK parcial | Sí | FSK/GFSK parcial | Sí, SDR/analizador | DISC; FSK | AF FeralRF |
| EV-24 | D+AF; I preferido | No OTA OOK | Sí | No workflow OOK confirmado | Sí | DISC | AF FeralRF |
| EV-25 | D+AF para marker; P para interop | V2 PHY FSK parcial | Sí PHY | PHY FSK parcial | Sí, P W-MBus | DISC; FSK | AF FeralRF / dispositivo W-MBus |
| EV-26 | D+AF para marker; P/I para interop | V2 PHY FSK parcial | Sí PHY | PHY FSK parcial | Sí | DISC; FSK | AF FeralRF / nodo Wi-SUN/Sidewalk |
| EV-27 | D+AF; I 2.4 | No RF independiente | Sí | no confirmado | Sí | DISC | AF FeralRF |
| EV-28 | D+AF o receptor conforme | No completo | Sí firmas | subconjunto según PHY | Sí para interop | DISC; ZSN/FSK condicional | AF FeralRF / receptor conforme |
| EV-29 | D+H+Z25; peer para inject | Sí* sniff si KillerBee instalado | Sí inject/simétrico | Sí captura independiente | Sí | DISC; ZSN para AS | AS TI sniffer; V2 sólo preinstalado |
| EV-30 | D+H (`cryptography`) | Sí | igual | igual | Oráculo host | DISC | ninguno |
| EV-31 | D+H | Sí | igual | igual | Oráculo host | DISC | ninguno |
| EV-40 | D; D+AF para full OTA | Sí* control | Sí completo | sólo observación separada | Sí para claims RF | DISC ambas; verificar Shell | AF FeralRF |
| EV-41 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-42 | D | Sí | igual | igual | No | DISC/EV-04 | ninguno |
| EV-43 | D+Z25 | Sí | también controlable | observación paralela | Útil, no obligatorio | DISC; ZSN condicional | ninguno |
| EV-44 | D+generador controlado | No; Z25 no controlado | Sí | no generador confirmado | Sí, opcional | DISC | AF FeralRF o generador RF |
| EV-45 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-46 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-47 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-50 | D+víctima+I, recinto | No | víctima simétrica | víctima independiente | Sí, obligatorio | DISC; no `verify` | víctima compatible |
| EV-51 | ruta futura | BLOCKED | BLOCKED | BLOCKED | Sí cuando exista | — | no implementado |
| EV-52 | D+AF/P MIOTY+I | No | limitación reproducible | no confirmado | Sí | DISC | AF FeralRF / dispositivo MIOTY |
| EV-53 | implementación/equipo futuro | BLOCKED | BLOCKED | BLOCKED | Sí | — | no implementado |
| EV-54 | análisis; Sniffle externo opcional | N/A | N/A | caso externo | Sí para BLE stack | DISC; BLE puede auto-flash | stock Sniffle, no FeralRF |

**Anterior:** [[FeralRF - Guía de validación experimental]]

````
