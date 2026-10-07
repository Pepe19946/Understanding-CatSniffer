# Registro de validación FeralRF

Registro cronológico canónico y puerta de entrada a los resultados. Actualización documental: 7 de octubre de 2026. Fechas experimentales sólo cuando constan expresamente. No se usa orden de archivos ni Git como prueba de cronología o resultado.

## Jerarquía documental

[[FeralRF - Wiki técnica integral]] → [[Arquitectura FeralRF]] / [[Matriz de capacidades]] / [[Protocolo y API Python]] → [[FeralRF - Guía de validación experimental]] → registros EV y sus notas originales → [[Pruebas y evidencia existente]] / [[Fuentes FeralRF]] / demás recursos → [[FeralRF - Matriz de pruebas]] → [[Auditoría técnica de validación FeralRF - EV ejecutadas]]. La guía define intención; el EV demuestra ejecución; la matriz expresa cobertura; la auditoría juzga alcance y brechas.

Modelo canónico: [[FeralRF - Matriz de pruebas#Modelo de evidencia A–F]]. A=RF física independiente; B=comportamiento directo del dispositivo; C=control/API (mocks identificados); D=fuente/documentación; E=inferencia/hipótesis; F=dimensión no evaluada. Nivel, estado, confianza y procedencia se informan por separado. D no demuestra conducta del binario instalado; F no es FAIL.

## Épocas de montaje e identidad

| Época documentada | Roles | Evidencia / cautela |
|---|---|---|
|Etapa inicial, disponible al corte de auditoría 5-10-2026|DUT Bridge COM88/LoRa COM86/Shell COM87; guía proponía peer COM31|Preflight y EV-03/04/05/10/11; COM31 es plan, no prueba de uso|
|Etapa OTA explícita 6-10-2026, tras sustitución de placa|DUT/TX COM33/LoRa COM34/Shell COM35; observador COM88/LoRa COM86/Shell COM87|EV-12; el papel de COM88 cambió. EV-13/14 usan este mapa, pero su día exacto no consta|

`GET_INFO` retorna serial constante `FERALRF1`; no basta para identificar unidades. Faltan serial USB/HWID, binario/commit cargado, inventario de sustitución y continuidad física de placas. No se convierte la coincidencia COM88 en identidad persistente. La aritmética Bridge+2 falló para COM88, aunque COM33+2 coincide en la etapa posterior; debe descubrirse por interfaz y placa.

## Cadena de preguntas y resultados

| Orden respaldado / fecha | Pregunta y motivo | Prueba efectivamente realizada | Qué establece | Incertidumbre creada / siguiente paso |
|---|---|---|---|---|
|Preflight inicial, sin fecha|¿Cuál es el puente y qué CLI está disponible?|Catnip EXE3.3.3.0 encuentra V3#1COM88/86/87; pip no detecta instalación, import local requiere `rich`|EXE funcional y checkout/import distintos|Identidad/debug/status incompletos→EV-00/40 siguen parciales|
|Inicio EV-01/02, sin fecha|¿Responde FeralRF y puede abrir/cerrar RX?|INIT/info/stats y RX_START/STOP en original§17|Control inicial observado|No binario fechado ni protocolo completo→03/04/05|
|Observaciones preliminares, sin fecha|¿Hay paquetes en IEEE25?|30 s: 48 paquetes, primero timestamp 343358331/RSSI−77/LQI58/CRC true/52B; 60 s: 100 paquetes, primero timestamp 4859346/−74/60/CRC true/51B|Recepción preliminar textual|Canal negativo informal sin comando/duración/canal conocidos; no control firme|
|[[EV-03 — Reconexión limpia entre procesos]], sin fecha|¿El cierre libera el puerto?|5 procesos independientes|PASS+C:reuso entre cinco procesos|No garantiza reset, interrupción o reinit en la misma instancia→04/46/47|
|[[EV-04 — Reset y reinicialización]], después 03 por contexto|¿Se puede recuperar sin ciclo USB?|boot→timeout INIT→exit→INIT; luego selector Shell|Manual B/C un ciclo; selector FAIL+C; API no ejecutada/global NOT FULLY VALIDATED|Mapeo frágil y sólo un ciclo→reset debe fijarse antes de automatizar|
|[[EV-05 — Primera observación RF IEEE]], después 04|¿Hay RF recibida tras recuperar control?|3×30s canal25:41/43/43; primer paquete CRCtrue en las tres|PASS+B RX local|Bytes incompletos/atribución no correlacionada/rendimiento no medido; observador suplementario; sin stackZigbee→20/14|
|[[EV-06 — Exclusión RX y TX y recuperación de estado]], tras RX funcional|¿Cómo se rechaza TX durante RX?|Dos intentos de harness inválidos, dos DUT válidos; error 0x05 y stop|PASS+C rechazo/recovery; ausencia física de emisión no medida|RF del rechazo no medida; otras transiciones→10/41|
|[[EV-10 — Matriz de PHY por control]], secuencia relativa|¿Responden ocho PHY?|8/8 smoke con reset entre filas|PASS+C ocho secuencias aceptadas|OTA y cambios sin reset abiertos→11/20/21/41|
|[[EV-11 — Control de presets propietarios]], secuencia de bloques|¿El control cubre presets?|433 MHz (6)→868 MHz (6+2)→169 MHz (2)→902/915 MHz (9 resumidos)→2440 MHz (2)|PARTIAL:27 control reportados; 18 transcripciones+9 resúmenes; RF por preset F|RF no observada, reset 868→169 no documentado,9 logs faltan→22–28|
|[[EV-12 — Control preliminar y preparación de EV-13]], sin fecha|¿Se aceptan cuatro modos TX?|RAW/FRAME/BURST/CONT/STOP por ACK|PASS+C histórico4/4|ACK no RF; prepara EV-13 y observador→OTA12|
|Auditoría histórica 5-10-2026|¿Qué podía concluirse con fuentes hasta 11?|Revisión documental preservada|Corte histórico sin OTA12|No es estado actual; wiki 6-10 conserva corte de evidencia 5-10|
|[[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]],6-10-2026|¿Los ACK corresponden a emisión/repetición?|Segunda placa; negativos/marcadores; luego conteos y variación de intervalos|PARTIAL:RAW/FRAME A marcador, conteo exacto no establecido; BURST1 match/umbral FAIL, DUT INCONCLUSIVE; CONT0:99 registros coincidentes CRC-válidos, conteo físico no establecido; positivos ensayados1 match|Min_hits1 era débil; reloj/aborto/observador y RX_STOP abiertos→diagnóstico 12 y 13|
|[[EV-13 — CW PRBS y TX_TEST_STOP por control]], posterior 12, sin día|¿CW/PRBS y stop son accesibles sin helper problemático?|Harness 0 dBm/0,3 s; dos stop idle; RX BLE|PARTIAL:C modos/B BLE; F física; timeout de confirmación host RX_STOP con pocos reportes|Carga instantánea/backlog/colas no medidos, causa no localizada; instrumentación y correlación de stop pendientes→14 y diagnóstico|
|[[EV-14 — Eventos RF asíncronos y firma RX]], posterior 13, sin día|¿Aparece error RF/firma? ¿Sobrevive transición mínima?|IEEE bytes, BLE→IEEE sin reset,3 tests FakeSerial|Global NOT FULLY VALIDATED; B bytes/negativos; B/C transiciónPARTIAL; 3 mocksPASS+C host|Errorfísico no inducido; no prueba inexistencia ni ciclos completos→14 controlado/41/46|

El orden inicial se apoya en dependencias y referencias de los registros; las fechas exactas entre esas pruebas no están documentadas. La secuencia de bloques de EV-11 no acredita el reset no registrado ni fechas distintas. EV-13/14 son posteriores por relación explícita, pero no se les asigna automáticamente el 6 o 7 de octubre. La sucesión 13→14 no equivale a una prueba de RX_STOP: EV-14 atiende la pregunta de error RF/firma de su propio registro; su relación causal específica con los timeouts de EV-13 no está establecida.

RX_STOP sigue la cadena host request → serialization → transport → firmware handler → RF state transition → completion/error generation → response transport → host correlation → observed timeout. No se trazó el request fallido por las etapas internas; el diagnóstico host reaparece al final. Sólo último ID 0x90 está preservado: no identificar las demás respuestas como paquetes ni inferir recepción nueva. [[Auditoría técnica de validación FeralRF - EV ejecutadas#Cadena causal de RX_STOP]] muestra la evidencia por etapa.

## Roles de los registros y duplicaciones

EV-03/04/05/06/10/11/12/13/14 tienen un registro canónico por ID. Los dos complementos EV-11 son evidencia de 433 y 868; el canónico conserva 169/902-915/2440 y la cobertura total. Las tres notas no son tres evaluaciones diferentes. EV-12 preliminar contiene control y preparación de EV-13; el registro OTA es canónico para resultado actual. No se crean A/B ni se cuenta el plan EV-13 como otra ejecución.

Los IDs 00/01/02 permanecen embebidos en este registro. El conjunto completo de 38 ítems está contabilizado en [[FeralRF - Matriz de pruebas]]: existencia de un ítem planeado no implica que haya un fichero experimental faltante. No existe evidencia suficiente para inventar EV-07/08/09 ni continuar numeración automáticamente.

## Próxima cadena de validación

**Identidad actual → observación calificada → localización de repetición y STOP → caracterización RF representativa → cobertura más amplia.**

P1: endpoints/interfaz actuales y Shell, observador/conteo, repetición, confirmación RX_STOP, cese TX y semánticas. P2: recuperar historia/transcripciones, cobertura y contratos secundarios. P3:usabilidad/roadmap. Sin P0 incondicional; P1 no localiza causa. Repetición y STOP pueden investigarse en ramas tras observación calificada; recuperar todo el historial no es prerrequisito de la sesión actual. Preguntas/controles en [[Auditoría técnica de validación FeralRF - EV ejecutadas#Próximas evaluaciones que reducen más incertidumbre]]; propuestas, no ejecuciones.

## 17. Notas originales preservadas y material pendiente

Fuente: `# Registro provisional de validació.md`. SHA-256 previo: `03FD375E819CAF6E2ADD7F3085D253C5DCA33D66C64441E5F96EA5FEA25BBAC9`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
# Registro provisional de validación experimental de FeralRF

Este registro contiene únicamente las pruebas ejecutadas físicamente durante nuestra campaña actual. Los resultados históricos del repositorio se utilizan como referencia, pero no se consideran resultados propios.

---

## 1. Identificación de la CatSniffer y mapeo de interfaces

### Comando realizado explícitamente

```powershell
catnip devices
```

### Acción que realiza

Solicita a Catnip la enumeración de las CatSniffer conectadas y muestra la revisión de placa detectada junto con las interfaces USB asociadas a sus funciones principales:

* `Cat-Bridge (CC1352)`
* `Cat-LoRa (SX1262)`
* `Cat-Shell (Config)`

### Intención dentro del seguimiento de pruebas

Antes de comunicarnos con FeralRF era necesario determinar qué puerto COM correspondía realmente al `Cat-Bridge`, ya que ésa es la interfaz que transporta los comandos entre la PC y el firmware ejecutado por el CC1352P7.

Esta identificación también permite conservar el mapeo real de puertos antes de probar funciones que dependan de otras interfaces, especialmente `reset_device()`.

### Resultado obtenido a nivel de terminal

La información relevante mostrada fue:

```text
│ Device        │ Board                  │ Cat-Bridge (CC1352) │ Cat-LoRa (SX1262) │ Cat-Shell (Config) │
├───────────────┼────────────────────────┼─────────────────────┼───────────────────┼────────────────────┤
│ CatSniffer #1 │ v3 (RP2040 + CC1352P7) │ COM88               │ COM86             │ COM87              │
```

### Interpretación teórica

La placa identificada es una **CatSniffer V3**, compuesta en esta ruta por:

* RP2040 como host/bridge USB.
* CC1352P7 como MCU/radio principal.
* SX1262 accesible mediante la interfaz `Cat-LoRa`.

Para nuestras pruebas de FeralRF, el puerto relevante es:

```text
Cat-Bridge = COM88
```

También quedó registrado:

```text
Cat-Shell = COM87
```

El resultado concuerda con el hardware esperado para la plataforma sobre la que está construido el FeralRF evaluado.

### Breve análisis

La enumeración fue satisfactoria y permitió continuar con las pruebas sin adivinar el puerto de comunicación.

Además apareció una observación importante para pruebas posteriores:

```text
Bridge = COM88
Shell  = COM87
```

Por lo tanto, en esta enumeración:

```text
Shell ≠ Bridge + 2
```

Esto es relevante porque el código actual de `reset_device()` de FeralRF calcula internamente el puerto Shell a partir del número del Bridge. Todavía **no se ha ejecutado `reset_device()`**, por lo que no debe registrarse como un fallo reproducido; por ahora es una **incompatibilidad potencial entre el supuesto del código y la enumeración real de Windows**.

---

## 2. Identificación del Catnip utilizado en la PC

Esta comprobación se realizó para saber exactamente qué instalación de Catnip estaba respondiendo a nuestros comandos antes de utilizarla como herramienta auxiliar de la validación.

### Comando realizado explícitamente

```powershell
(Get-Command catnip -ErrorAction SilentlyContinue) | Format-List Source,Path,CommandType
```

### Acción que realiza

PowerShell resuelve el comando `catnip` y muestra qué ejecutable utilizará realmente.

### Intención dentro del seguimiento de pruebas

Evitar confundir el ejecutable instalado de Catnip con el código fuente local presente dentro del repositorio `CatSniffer-Tools`.

### Resultado obtenido a nivel de terminal

```text
Source      : C:\Program Files\Catnip\catnip.exe
Path        : C:\Program Files\Catnip\catnip.exe
CommandType : Application
```

### Interpretación y análisis

El comando `catnip` utilizado durante las pruebas corresponde al ejecutable instalado en:

```text
C:\Program Files\Catnip\catnip.exe
```

Esto quedó correctamente identificado.

---

### Comando realizado explícitamente

```powershell
python -m pip show catnip
```

### Acción que realiza

Pregunta al entorno Python activo si existe un paquete instalado mediante `pip` llamado `catnip`.

### Resultado obtenido a nivel de terminal

```text
WARNING: Package(s) not found: catnip
```

### Interpretación y análisis

Catnip **no está instalado como paquete pip dentro de ese entorno Python**.

Esto no representa un error de la herramienta instalada, ya que anteriormente se confirmó que Windows ejecuta `C:\Program Files\Catnip\catnip.exe`.

---

### Comando realizado explícitamente

```powershell
python -c "import importlib.util; s=importlib.util.find_spec('catnip'); print(s.origin if s else 'NOT FOUND')"
```

### Acción que realiza

Pregunta a Python qué módulo llamado `catnip` encontraría mediante su mecanismo de importación.

### Resultado obtenido a nivel de terminal

```text
C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\CatSniffer-Tools\catnip\catnip.py
```

### Interpretación y análisis

Python encuentra el archivo del **checkout local de `CatSniffer-Tools`**, debido al directorio desde el que se ejecutó el comando.

Esto no implica que dicho checkout esté instalado o listo para ejecutarse con todas sus dependencias.

---

### Comando realizado explícitamente

```powershell
catnip --help
```

### Acción que realiza

Ejecuta el Catnip resuelto por Windows y solicita su ayuda principal.

### Resultado obtenido a nivel de terminal

El comando se ejecutó correctamente y mostró, entre otra información:

```text
catnip
v3.3.3.0
```

### Interpretación y análisis

La aplicación instalada funciona y reporta **Catnip v3.3.3.0**.

Ésta es la implementación que efectivamente se utilizó durante la enumeración física de la CatSniffer.

---

### Comando realizado explícitamente

```powershell
python .\catnip.py --help
```

### Acción que realiza

Intenta ejecutar directamente el código fuente local de Catnip utilizando el entorno Python activo.

### Resultado obtenido a nivel de terminal

La ejecución falló con:

```text
ModuleNotFoundError: No module named 'rich'
```

### Interpretación y análisis

El entorno Python desde el que se ejecutó el checkout local no dispone de todas las dependencias necesarias, siendo `rich` la primera dependencia ausente detectada.

Esto **no bloquea las pruebas de FeralRF**, porque el ejecutable instalado de Catnip funciona correctamente.

También conviene conservar esta diferencia como información de reproducibilidad:

```text
Catnip utilizado físicamente:
C:\Program Files\Catnip\catnip.exe
v3.3.3.0

Checkout local:
CatSniffer-Tools\catnip\catnip.py
No ejecutable actualmente en el entorno Python probado por dependencia faltante.
```

No se ha demostrado que el ejecutable instalado haya sido construido exactamente a partir del mismo commit que el checkout local.

---

# 3. Primera recepción física IEEE 802.15.4 — 30 segundos

### Comando realizado explícitamente

Ejecutado desde:

```text
C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python
```

Comando:

```powershell
python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
```

### Acción que realiza

El script abre la comunicación con FeralRF a través de `COM88` y ejecuta una secuencia de prueba que incluye:

```text
RADIO_INIT / GET_INFO
SET_PHY IEEE_802_15_4
SET_CHANNEL 25
RX_START
recepción durante 30 s
RX_STOP
```

La ruta física bajo prueba es:

```text
PC
→ Python / API FeralRF
→ USB / COM88
→ RP2040
→ UART
→ CC1352P7 ejecutando FeralRF
→ radio IEEE 802.15.4
```

### Intención dentro del seguimiento de pruebas

La prueba se realizó para avanzar desde una simple validación de comunicación hacia una **recepción RF real**.

Existía una fuente Zigbee conocida transmitiendo tráfico en IEEE 802.15.4 canal 25, por lo que la prueba podía utilizarse para comprobar si FeralRF era capaz de recibir tráfico físico en ese canal.

### Resultado obtenido a nivel de terminal

```text
C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python>python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
FeralRF IEEE 802.15.4 Smoke Test
================================
port=COM88 baudrate=921600 phy=4 channel=25 duration=30.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY IEEE_802_15_4
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=48
[ OK ] first: ts=343358331us ch=25 rssi=-77 lqi=58 crc_ok=True len=52
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS
```

### Interpretación teórica

La comunicación de control funcionó correctamente:

```text
RADIO_INIT / GET_INFO → OK
SET_PHY               → ACK
SET_CHANNEL           → ACK
RX_START              → ACK
RX_STOP               → ACK
```

`GET_INFO` reportó:

```text
firmware=1.0.0
capabilities=0x07
serial=464552414c524631
```

Más importante aún, durante la ventana de recepción se obtuvieron:

```text
48 paquetes en 30 s
```

El primer paquete reportado fue:

```text
channel = 25
RSSI    = -77 dBm
LQI     = 58
CRC     = válido
length  = 52 bytes
```

Por tanto, la prueba no se limitó a recibir ACK del firmware: hubo tráfico recibido por la ruta RF.

El resultado concuerda con el esperado para una fuente IEEE 802.15.4 activa en canal 25.

### Breve análisis

La prueba fue satisfactoria.

Provisionalmente aporta evidencia para:

```text
EV-01 → comunicación/init/info correcta
EV-02 → ruta de control RX_START/RX_STOP correcta
EV-05 → recepción RF real observada
```

La presencia de `crc_ok=True` refuerza que se trata de una trama recibida correctamente por la capa IEEE 802.15.4.

Sin embargo, este resultado **no demuestra soporte del stack Zigbee**. Lo demostrado es recepción IEEE 802.15.4 raw en el canal utilizado por una fuente Zigbee.

Tampoco existe todavía una captura independiente que permita comparar paquete por paquete los 48 frames con otro receptor.

---

# 4. Segunda recepción IEEE 802.15.4 — 60 segundos

### Comando realizado explícitamente

```powershell
python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 60
```

### Acción que realiza

Ejecuta la misma ruta funcional anterior, manteniendo la recepción IEEE 802.15.4 en canal 25 durante una ventana mayor, de **60 segundos**.

### Intención dentro del seguimiento de pruebas

Comprobar si el resultado de la primera captura era **repetible** y observar si la recepción se mantenía estable durante una ventana más larga.

La guía experimental define EV-05 precisamente como una prueba que debe reproducirse varias veces contra una fuente conocida.

### Resultado obtenido a nivel de terminal

```text
C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python>python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 60
FeralRF IEEE 802.15.4 Smoke Test
================================
port=COM88 baudrate=921600 phy=4 channel=25 duration=60.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY IEEE_802_15_4
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=100
[ OK ] first: ts=4859346us ch=25 rssi=-74 lqi=60 crc_ok=True len=51
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS
```

### Interpretación teórica

La secuencia completa volvió a finalizar correctamente y FeralRF recibió:

```text
100 paquetes en 60 s
```

El primer paquete tuvo:

```text
channel = 25
RSSI    = -74 dBm
LQI     = 60
CRC     = válido
length  = 51 bytes
```

La tasa aproximada observada fue:

```text
Primera ejecución:
48 / 30 s ≈ 1.60 paquetes/s

Segunda ejecución:
100 / 60 s ≈ 1.67 paquetes/s
```

No esperamos que ambas tasas sean idénticas porque la fuente no está siendo controlada por FeralRF, pero los valores son cercanos.

El resultado concuerda con lo esperado.

### Breve análisis

La segunda prueba **refuerza considerablemente la repetibilidad de EV-05**.

No sólo volvió a aparecer tráfico válido en el canal correcto, sino que el orden de magnitud de la tasa recibida fue muy similar entre ambas ventanas.

También se observa una pequeña variación normal entre primeras tramas:

```text
Ejecución 1: RSSI -77 dBm, LQI 58, len 52
Ejecución 2: RSSI -74 dBm, LQI 60, len 51
```

No hay aquí ninguna anomalía evidente. La diferencia en RSSI, LQI y longitud puede corresponder simplemente a frames distintos y a las condiciones normales de recepción.

Hasta este punto no se observaron:

* timeouts;
* errores de inicialización;
* errores asíncronos reportados;
* bloqueo de RX;
* problemas al ejecutar `RX_STOP`;
* necesidad de reset o power-cycle.

El comportamiento observado es consistente con una ruta RX funcional.

---

# 5. Comprobación en un canal sin actividad esperada

### Comando realizado explícitamente

Se ejecutó nuevamente `smoke_phy4_ieee154.py` utilizando **otro canal IEEE 802.15.4**, escogido porque en observaciones anteriores realizadas con Cativity no se había detectado actividad.

**El número exacto del canal y la línea exacta del comando no quedaron registrados todavía en este documento.**

Por esa razón no deben reconstruirse ni inventarse.

### Acción que realiza

La prueba configura la misma ruta de recepción FeralRF, pero desplazándola a un canal donde previamente no se esperaba tráfico.

### Intención dentro del seguimiento de pruebas

Usar una especie de **control negativo experimental**:

* Canal 25 → se esperaba actividad y FeralRF recibió paquetes.
* Canal alternativo → previamente no se había observado actividad y FeralRF tampoco reportó paquetes.

Esto ayuda a comprobar que la recepción anterior no consiste simplemente en datos apareciendo independientemente del canal configurado.

### Resultado obtenido a nivel de terminal

El usuario observó que la ejecución **no recibió paquetes**.

No se dispone todavía de la transcripción literal completa de esa ejecución, por lo que no debe añadirse una salida fabricada.

### Interpretación teórica

El comportamiento es coherente con las observaciones anteriores realizadas con Cativity: un canal previamente identificado como inactivo tampoco presentó tráfico durante la escucha con FeralRF.

### Breve análisis

El resultado es **coherente y favorable**, pero por ahora debe tratarse como evidencia complementaria y no como una comparación instrumental estricta.

Cativity y FeralRF no necesariamente observaron el canal exactamente al mismo tiempo ni bajo idénticas condiciones.

Para convertir esta observación en un registro reproducible necesitamos conservar en la próxima ocasión:

```text
canal utilizado
duración
comando completo
stdout completo
estado simultáneo de la fuente
```

Aun con esta limitación, el comportamiento observado resulta útil porque no contradice lo esperado y aumenta la confianza de que la actividad detectada en CH25 está relacionada con tráfico real presente en ese canal.

---

# Estado provisional después de estas pruebas

La evidencia obtenida hasta ahora permite considerar:

**EV-01 — Init / Info:** resultado favorable, aunque la guía requiere varias ejecuciones específicas para cerrarlo formalmente.

**EV-02 — RX IEEE controlado:** comportamiento correcto observado durante ambas ejecuciones.

**EV-05 — Primera RX física:** evidencia RF real reproducida en dos ejecuciones independientes:

```text
30 s → 48 paquetes
60 s → 100 paquetes
```

ambas con frames CRC-válidos en canal 25.

Se añadió además un control negativo informal en otro canal sin actividad esperada, cuyo comportamiento coincidió con la expectativa, aunque falta conservar su comando y stdout completos para incorporarlo como evidencia reproducible.

No se han observado hasta ahora fallos, hangs, necesidad de reset ni incoherencias funcionales en esta ruta RX.

Sí permanecen dos observaciones para investigación posterior:

1. `GET_INFO` reporta `firmware=1.0.0`, mientras que otras capas documentales utilizan otras versiones de FeralRF. Esto debe conservarse como discrepancia de versionado, no como fallo funcional.

2. La enumeración real `COM88 Bridge / COM87 Shell` no satisface el supuesto `Shell = Bridge + 2` empleado por `reset_device()`. No se probará esa función a ciegas.

````
