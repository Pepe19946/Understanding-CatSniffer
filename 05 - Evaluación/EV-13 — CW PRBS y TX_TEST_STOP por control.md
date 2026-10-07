# EV-13 — CW PRBS y TX_TEST_STOP por control

Registro canónico. Fecha experimental no documentada; dependencia posterior a EV-12 explícita. La instrumentación se declaró disponible pero deliberadamente diferida; modelo y calibración desconocidos.

## 1. Contexto de evaluación

Después de las anomalías de EV-12 se separaron los modos RF de prueba (CW/PRBS) de la repetición de paquetes y se evitó depender del helper upstream sin adaptación.

## 2. Objetivo de validación

Comprobar aceptación de CW, PRBS15/PRBS32 y TX_TEST_STOP, incluido stop repetido en idle; documentar lo que todavía falta medir físicamente.

## 3. Capacidad o requisito FeralRF evaluado

Comandos 0x55–0x57 de modos RF de prueba; no son RAW/BURST ni una demostración de PRBS9. [[Protocolo y API Python]], [[Matriz de capacidades]].

## 4. Precondiciones y condiciones

DUT COM33; observador COM88. CW BLE1M canal 37, 0 dBm, espera host 0,3 s; PRBS15/32 SUB_GHZ_868 canal 0, 0 dBm, espera 0,3 s. `finally` invoca stop. El helper F22 no se usó directamente: puertos derivados y potencia +5 dBm documentados incompatibles con la adaptación local. Instrumento disponible según nota, pero sin modelo/calibración/mediciones.

## 5. Resultado esperado

ACK de inicio y stop; stop repetido en idle aceptado. La validación completa requiere energía/frecuencia/patrón y cese físicos, no satisfechos por ACK. El criterio RX ambiental de más de 30 paquetes era exploratorio, no un test válido de interferencia CW completado.

## 6. Procedimiento y ejecución cronológica

1. Ejecutar harness adaptado: CW BLE1M/37/0, espera 0,3 s, stop en finally.
2. Ejecutar PRBS15 y PRBS32 sub 868/0/0, con la misma espera y stop.
3. Ejecutar dos stops consecutivos en idle.
4. Observar BLE ambiental con receptor COM88; registrar baseline y timeouts de RX_STOP. No consta una comparación CW on/off completa con instrumento.

## 7. Resultado observado

ACK para CW, ambas variantes PRBS y sus stops; dos stops idle con ACK. RX BLE inicial: dos paquetes y timeout RX_STOP con cinco inesperados/último 0x90. Repeticiones reportadas 10/11/11; una salida completa de 11 registra cuatro inesperados/último 0x90. No se superó el umbral exploratorio 30.

## 8. Evidencia

Comandos/harness, ACK, salidas BLE y recomendaciones íntegras en §17. Cantidades repetidas parcialmente narradas; no tres stdout completos. No hay espectro, potencia, frecuencia o patrón PRBS capturado.

## 9. Comparación entre lo esperado y lo observado

Objetivo estrecho de control logrado. Energía/patrón y cese quedan abiertos. Baseline BLE insuficiente para atribuir variación a CW; comparación de interferencia INCONCLUSIVE.

## 10. Interpretación técnica

Se demuestra idempotencia de ACK de stop idle bajo este caso. RX_STOP puede fallar incluso con pocas tramas, por lo que la carga alta de CONT0 no es una explicación suficiente por sí sola. No se demuestra que RX siguiera activo físicamente tras timeout ni que CW haya interferido.

## 11. Anomalías, desviaciones y limitaciones

ACK no acredita emisión; tiempo 0,3 s es espera host, no duración RF medida. Mismo firmware en observador. Bytes BLE completos no documentados. La disponibilidad declarada del instrumento no equivale a una medición realizada.

## 12. Resultado de la evaluación

PARTIAL: control PASS; CW/PRBS y cese físicos NOT FULLY VALIDATED; interferencia exploratoria INCONCLUSIVE.

## 13. Confianza

High para ACK y timeout literal; Medium para repetición narrada; Low para comportamiento físico y causa del timeout.

## 14. Preguntas abiertas

¿Hubo portadora/patrón correcto? ¿STOP cesa inmediatamente por aire? ¿Por qué RX_STOP no recibe/correlaciona ACK con cargas bajas? ¿El helper F22 conserva potencia +5 tras el wrapper de reset?

## 15. Acciones de seguimiento

Registrar instrumento y medir CW/PRBS/cese con parámetros justificados; auditar helper de potencia; trazar serial de stop y estados del firmware antes de atribuir el timeout a saturación.

## 16. Trazabilidad

Guía EV-13 y EV-43/44; [[FeralRF - Guía de validación experimental]]; [[FeralRF - Matriz de pruebas]]; [[Arquitectura FeralRF]]; [[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]]; [[EV-14 — Eventos RF asíncronos y firma RX]]; [[Registro de validación FeralRF]]; [[Fuentes herramientas PC]].

Definición específica: [[FeralRF - Wiki técnica integral#9.6 CW y PRBS]].



## 17. Notas originales preservadas y material pendiente

Fuente: `# EV-13 — Registro técnico consolid.md`. SHA-256 previo: `FDB7D55707C88F12B14A1202E65747947502C9FD45A0307ACA8CE54668B748CE`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
# EV-13 — Registro técnico consolidado de CW, PRBS y `TX_TEST_STOP`

## 1. Identificación

**EV:** EV-13 — CW / PRBS / STOP
**Proyecto:** CatSniffer - FeralRF
**Firmware evaluado:** FeralRF
**HEAD de referencia:** `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`
**Firmware reportado por GET_INFO:** `1.0.0`
**DUT:** CatSniffer v3 — Cat-Bridge `COM33`
**Observer disponible:** CatSniffer v3 — Cat-Bridge `COM88`
**Potencia solicitada durante las pruebas:** `0 dBm`

La guía define EV-13 como una validación de modos de prueba RF `CW`, `PRBS` y `STOP`, con intención final de verificar frecuencia, potencia relativa y comportamiento temporal mediante instrumentación. También especifica que una segunda CatSniffer sólo aporta evidencia parcial y no reemplaza un analizador/power meter.

La matriz clasifica EV-13 como una validación L1/L2 P2 y distingue explícitamente la parte de control de la validación RF instrumentada.

---

# 2. Objetivo real de la campaña ejecutada

La EV completa contempla:

```text
CW
PRBS-15
PRBS-32
TX_TEST_STOP
frecuencia
potencia
espectro/patrón
cese de energía
```

Sin embargo, se decidió **posponer la instrumentación RF** para avanzar primero en la evaluación funcional del firmware.

Por tanto, la campaña ejecutada tuvo como objetivo inmediato:

1. validar que FeralRF acepta y procesa `CW`;
2. validar `PRBS-15`;
3. validar `PRBS-32`;
4. validar `TX_TEST_STOP`;
5. comprobar que `TX_TEST_STOP` sea idempotente;
6. explorar si la segunda CatSniffer podía aportar evidencia OTA indirecta sin instrumentación;
7. registrar anomalías del host/RX observadas durante el proceso.

La caracterización física de:

```text
frecuencia real
potencia real
pureza espectral
patrón PRBS real
armónicos
```

queda deliberadamente pendiente.

---

# 3. Diferencia respecto de EV-12

EV-12 probó transmisión de paquetes:

```text
TX_RAW
TX_FRAME
TX_BURST
TX_CONTINUOUS
```

EV-13 utiliza otra ruta:

```text
TX_CW
TX_PRBS
TX_TEST_STOP
```

Los IDs de protocolo correspondientes son:

```text
0x55 CMD_TX_CW
0x56 CMD_TX_PRBS
0x57 CMD_TX_TEST_STOP
```

Estos comandos están presentes en la superficie actual del firmware.

---

# 4. Qué representan CW y PRBS

## 4.1 CW

`CW` significa **Continuous Wave**.

Es una portadora RF continua, sin datos útiles modulados, que permanece activa hasta recibir `TX_TEST_STOP`.

La Wiki técnica describe CW como una portadora sin modulación de datos mantenida hasta la orden de parada.

Conceptualmente:

```text
CW:

RF ───────────────────────────
       portadora continua
```

No es equivalente a:

```text
TX_CONTINUOUS
```

de EV-12.

`TX_CONTINUOUS` repite paquetes; `CW` mantiene una señal de prueba RF.

---

## 4.2 PRBS

`PRBS` significa **Pseudo-Random Binary Sequence**.

FeralRF expone actualmente:

```text
PRBS-15
PRBS-32
```

como patrones de prueba.

La documentación consolidada identifica una discrepancia histórica: existen referencias a PRBS-9, pero la implementación y API actuales de FeralRF utilizan PRBS-15 y PRBS-32.

PRBS no representa tráfico de aplicación.

Su propósito es ejercitar el transmisor con un patrón pseudoaleatorio apropiado para evaluación RF.

---

# 5. Ruta de firmware

La ruta conceptual bajo prueba es:

```text
PC
 ↓
feralrf.Radio
 ↓
CMD_TX_CW / CMD_TX_PRBS
 ↓
USB CDC Cat-Bridge
 ↓
RP2040 passthrough
 ↓
UART
 ↓
CC1352P7
 ↓
CommandProcessor
 ↓
RadioIF_runTxTest()
 ↓
TI RF Driver
 ↓
CMD_TX_TEST
 ↓
RF Core
```

El firmware utiliza `CMD_TX_TEST`, lo configura para ejecución continua y conserva el handle necesario para detenerlo posteriormente.

Esto significa que EV-13 recorre una ruta distinta de la lógica de scheduling temporal de BURST/CONTINUOUS identificada como problemática en EV-12.

---

# 6. Decisión metodológica importante: no ejecutar `smoke_f22_tx_test.py` directamente

El repositorio contiene:

```text
python/examples/lab/smoke_f22_tx_test.py
```

pero se decidió **no utilizarlo directamente**.

## Razón 1 — KI-15

El script deriva el puerto Shell mediante:

```text
Bridge + 2
```

Esto no es portable al inventario actual.

Para el observer:

```text
Bridge = COM88
Shell real = COM87
```

pero el script inferiría:

```text
COM90
```

La Wiki identifica explícitamente `smoke_f22_tx_test.py` entre los scripts afectados por esta suposición.

---

## Razón 2 — potencia hardcodeada

El script histórico utiliza:

```python
power_dbm=5
```

mientras que para esta campaña se decidió trabajar inicialmente con:

```text
0 dBm
```

para mantener una condición conservadora y alineada con la guía experimental.

---

# 7. EV-13.1 — CW control-path

## Objetivo

Determinar si FeralRF puede:

```text
configurar PHY
activar CW
mantenerlo brevemente
detenerlo mediante TX_TEST_STOP
preservar comunicación
```

sin evaluar todavía la señal físicamente.

## Configuración

```text
DUT: COM33
PHY: BLE_1M
channel: 37
power: 0 dBm
duration: ~300 ms
```

## Comando utilizado

```powershell
@'
import time
from feralrf import Radio, PHY

r = Radio(port="COM33", baudrate=921600)

try:
    info = r.init()
    print("INFO:", info)

    r.set_phy(PHY.BLE_1M, channel=37)
    print("PHY=BLE_1M channel=37")

    print("TX_CW start power=0 dBm")
    r.tx_cw(power_dbm=0)
    print("TX_CW ACK")

    time.sleep(0.3)

finally:
    print("TX_TEST_STOP")
    try:
        r.tx_test_stop()
        print("TX_TEST_STOP ACK")
    finally:
        r.disconnect()
        print("Disconnected")
'@ | python -
```

## Resultado obtenido

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
PHY=BLE_1M channel=37
TX_CW start power=0 dBm
TX_CW ACK
TX_TEST_STOP
TX_TEST_STOP ACK
Disconnected
```

## Resultado esperado

```text
INIT correcto
SET_PHY aceptado
TX_CW ACK
TX_TEST_STOP ACK
disconnect limpio
```

## Resultado obtenido vs esperado

Coincidente.

## Veredicto

```text
CW: PASS-control
TX_TEST_STOP: PASS-control
```

## Límite

No demuestra:

```text
CW físicamente presente
frecuencia correcta
0 dBm reales
pureza espectral
```

---

# 8. Protección de STOP incluida en el harness

`TX_TEST_STOP` no se ejecutó manualmente por separado.

Se incluyó deliberadamente dentro del bloque:

```python
finally:
    r.tx_test_stop()
```

Esto fue una medida de seguridad.

CW/PRBS permanecen activos hasta una orden explícita de parada, por lo que el harness fue diseñado para intentar detener la señal incluso si se genera una excepción durante la prueba.

Por tanto:

```text
TX_TEST_STOP
```

fue enviado automáticamente por el script de validación diseñado para la campaña.

No fue una acción automática del hardware ni del firmware.

---

# 9. Intento de validación OTA indirecta mediante BLE

El script histórico intenta demostrar CW indirectamente utilizando tráfico BLE ambiental.

La metodología original es:

```text
medir BLE ch37 sin CW
 ↓
activar CW ch37
 ↓
medir BLE ch37 con CW
 ↓
comparar caída de recepción
```

La lógica es que una señal CW suficientemente fuerte en la misma zona espectral degrade la capacidad del receptor para decodificar paquetes BLE.

---

# 10. Por qué se utilizó BLE

`BLE` significa:

```text
Bluetooth Low Energy
```

y pertenece a la familia Bluetooth.

Para esta prueba se utilizó:

```text
PHY.BLE_1M
channel 37
```

Esto no significa que EV-13 esté evaluando una pila Bluetooth completa.

La Wiki establece que FeralRF conserva soporte BLE a bajo nivel/PHY, pero no implementa actualmente una pila completa de alto nivel como GATT/conexiones BLE públicas.

En EV-13, BLE se utilizó únicamente como **PHY conveniente para ejercitar CW en 2.4 GHz**.

---

# 11. Switch Zigbee: no participa en esta parte de EV-13

El switch Zigbee utilizado en pruebas anteriores opera mediante:

```text
IEEE 802.15.4
canal 25
```

EV-13 utilizó:

```text
BLE 1M
canal 37
```

Por tanto:

```text
IEEE 802.15.4 ch25 ≠ BLE ch37
```

El switch Zigbee **no es la fuente de los paquetes BLE observados**.

No debe utilizarse para incrementar artificialmente el tráfico de esta subprueba.

---

# 12. Primera medición BLE ambiental

## Comando inicial

```powershell
python -c "import time; from feralrf import Radio,PHY,RxStreamError; r=Radio(port='COM88',baudrate=921600); info=r.init(); r.set_phy(PHY.BLE_1M,channel=37); r.start_rx(); time.sleep(2); pkts=[p for p in r.read_packets(timeout=0.5) if not isinstance(p,RxStreamError)]; print('INFO:',info); print('BLE37_IDLE_PACKETS=',len(pkts)); r.stop_rx(); r.disconnect()"
```

## Resultado

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
BLE37_IDLE_PACKETS= 2
```

seguido de:

```text
feralrf.exceptions.TimeoutError:
Response timeout (ignored 5 unexpected response(s), last=0x90)
```

## Interpretación

Sólo se observaron:

```text
2 paquetes BLE
```

durante la ventana.

Esto resultó insuficiente para aplicar de forma robusta el criterio histórico del script.

---

# 13. Repeticiones del baseline BLE

Para determinar si el valor `2` era circunstancial, se repitió la medición tres veces.

Resultados reportados:

```text
run 1 → 10
run 2 → 11
run 3 → 11
```

Una ejecución preservada literalmente produjo:

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
PHY=BLE_1M channel=37
RX_START ACK
BLE37_IDLE_PACKETS= 11
RX_STOP ERROR: TimeoutError Response timeout (ignored 4 unexpected response(s), last=0x90)
Disconnected
```

Promedio aproximado:

```text
10.7 paquetes / ventana
```

## Resultado esperado según script histórico

El script original requiere:

```text
n_idle > 30
```

antes de considerar válida la comparación de interferencia.

## Resultado real

```text
2
10
11
11
```

Ninguna ejecución alcanzó ese umbral.

---

# 14. Evaluación de la posibilidad de ampliar la ventana

Se discutió la posibilidad de aumentar el tiempo de observación.

Esto es técnicamente posible, pero requiere mantener comparabilidad:

```text
idle → T segundos
CW   → T segundos
```

o comparar:

```text
packets/second
```

No sería correcto comparar conteos absolutos obtenidos con ventanas diferentes.

Dado que la intención actual es avanzar en el plan sin mantener CW durante periodos más largos innecesariamente, se decidió no forzar esta metodología.

El criterio `n_idle > 30` pertenece al entorno experimental del script histórico; no constituye una especificación física de FeralRF.

---

# 15. Veredicto del smoke OTA CW

La subprueba:

```text
CW OTA indirecto mediante caída de BLE
```

queda clasificada como:

```text
INCONCLUSO
```

Motivo:

> El entorno presenta tráfico BLE real pero insuficiente, dentro de las ventanas empleadas, para reproducir de forma robusta el criterio histórico de caída de paquetes.

Esto **no constituye un FAIL de CW**.

---

# 16. Anomalía reproducida — RX_STOP frente a RSP_RX_PACKET

Durante la primera medición apareció:

```text
Response timeout
(ignored 5 unexpected response(s), last=0x90)
```

Durante una repetición:

```text
RX_STOP ERROR:
TimeoutError
Response timeout
(ignored 4 unexpected response(s), last=0x90)
```

`0x90` corresponde a:

```text
RSP_RX_PACKET
```

## Importancia

El mismo fenómeno había aparecido previamente en EV-12 bajo una tasa RX muy alta.

En EV-13 reapareció con:

```text
11 paquetes
```

por lo que ya no parece depender exclusivamente de saturación extrema.

## Hipótesis actual

Existe una interacción problemática entre:

```text
Radio.stop_rx()
Radio._read_response()
RSP_RX_PACKET asíncronos
ACK de RX_STOP
```

La API puede continuar recibiendo respuestas `0x90` mientras espera el ACK de `RX_STOP`, agotando el timeout.

## Lo que NO está demostrado

No se puede afirmar todavía que:

```text
el firmware no ejecuta RX_STOP
```

La anomalía puede estar en:

```text
orden de frames
backlog serial
correlación de seq
manejo de eventos asíncronos
host-side response handling
```

Debe investigarse posteriormente como hallazgo transversal EV-12/EV-13.

---

# 17. EV-13.3 — PRBS-15

## Objetivo

Validar la ruta de control de:

```text
PRBS-15
```

sin medir todavía el patrón RF.

## Configuración

```text
DUT: COM33
PHY: SUB_1GHZ_868
channel: 0
power: 0 dBm
duration: ~300 ms
```

## Comando

```powershell
@'
import time
from feralrf import Radio, PHY

r = Radio(port="COM33", baudrate=921600)

try:
    info = r.init()
    print("INFO:", info)

    r.set_phy(PHY.SUB_1GHZ_868, channel=0)
    print("PHY=SUB_1GHZ_868 channel=0")

    print("TX_PRBS15 start power=0 dBm")
    r.tx_prbs(power_dbm=0, pattern="prbs15")
    print("TX_PRBS15 ACK")

    time.sleep(0.3)

finally:
    print("TX_TEST_STOP")
    try:
        r.tx_test_stop()
        print("TX_TEST_STOP ACK")
    finally:
        r.disconnect()
        print("Disconnected")
'@ | python -
```

## Resultado

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
PHY=SUB_1GHZ_868 channel=0
TX_PRBS15 start power=0 dBm
TX_PRBS15 ACK
TX_TEST_STOP
TX_TEST_STOP ACK
Disconnected
```

## Veredicto

```text
PRBS-15: PASS-control
TX_TEST_STOP: PASS-control
```

## No validado

```text
patrón PRBS-15 físico
espectro
frecuencia
potencia
```

---

# 18. EV-13.4 — PRBS-32

## Comando

```powershell
@'
import time
from feralrf import Radio, PHY

r = Radio(port="COM33", baudrate=921600)

try:
    info = r.init()
    print("INFO:", info)

    r.set_phy(PHY.SUB_1GHZ_868, channel=0)
    print("PHY=SUB_1GHZ_868 channel=0")

    print("TX_PRBS32 start power=0 dBm")
    r.tx_prbs(power_dbm=0, pattern="prbs32")
    print("TX_PRBS32 ACK")

    time.sleep(0.3)

finally:
    print("TX_TEST_STOP")
    try:
        r.tx_test_stop()
        print("TX_TEST_STOP ACK")
    finally:
        r.disconnect()
        print("Disconnected")
'@ | python -
```

## Resultado

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
PHY=SUB_1GHZ_868 channel=0
TX_PRBS32 start power=0 dBm
TX_PRBS32 ACK
TX_TEST_STOP
TX_TEST_STOP ACK
Disconnected
```

## Veredicto

```text
PRBS-32: PASS-control
TX_TEST_STOP: PASS-control
```

---

# 19. EV-13.5 — Idempotencia de TX_TEST_STOP

## Objetivo

Comprobar que:

```text
TX_TEST_STOP
```

sea seguro incluso cuando no existe CW/PRBS activo.

## Comando

```powershell
@'
from feralrf import Radio

r = Radio(port="COM33", baudrate=921600)

try:
    info = r.init()
    print("INFO:", info)

    print("TX_TEST_STOP #1")
    r.tx_test_stop()
    print("TX_TEST_STOP #1 ACK")

    print("TX_TEST_STOP #2")
    r.tx_test_stop()
    print("TX_TEST_STOP #2 ACK")

finally:
    r.disconnect()
    print("Disconnected")
'@ | python -
```

## Resultado

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
TX_TEST_STOP #1
TX_TEST_STOP #1 ACK
TX_TEST_STOP #2
TX_TEST_STOP #2 ACK
Disconnected
```

## Veredicto

```text
TX_TEST_STOP: PASS-control
idempotencia: PASS-control
```

No fue necesario:

```text
reset
power-cycle
reconnect manual
```

---

# 20. Resumen consolidado

| Función              | Configuración         | Esperado              | Obtenido   | Estado           |
| -------------------- | --------------------- | --------------------- | ---------- | ---------------- |
| CW                   | BLE 1M ch37, 0 dBm    | ACK + STOP            | ACK + STOP | **PASS-control** |
| PRBS-15              | Sub-1 GHz 868, 0 dBm  | ACK + STOP            | ACK + STOP | **PASS-control** |
| PRBS-32              | Sub-1 GHz 868, 0 dBm  | ACK + STOP            | ACK + STOP | **PASS-control** |
| `TX_TEST_STOP`       | después de CW/PRBS    | ACK                   | ACK        | **PASS-control** |
| STOP idempotente     | dos STOP consecutivos | ACK/ACK               | ACK/ACK    | **PASS-control** |
| CW OTA vía caída BLE | BLE ch37              | baseline >30 + caída  | 2/10/11/11 | **INCONCLUSO**   |
| CW frecuencia        | instrumento           | valor esperado        | no medido  | **PENDIENTE**    |
| CW potencia          | instrumento           | ~potencia configurada | no medido  | **PENDIENTE**    |
| PRBS espectral       | instrumento           | patrón esperado       | no medido  | **PENDIENTE**    |

---

# 21. Estado técnico de EV-13

El veredicto apropiado es:

> **EV-13: PASS-control / RF instrumentado pendiente.**

Se validaron correctamente:

```text
CW
PRBS-15
PRBS-32
TX_TEST_STOP
STOP idempotente
```

a nivel de API, protocolo y control del firmware.

No debe declararse:

```text
PASS-RF completo
```

porque no existe todavía evidencia instrumental de:

```text
frecuencia
potencia
espectro
PRBS real
cese físico de energía
```

---

# 22. Relación con EV-12

EV-12 detectó un fallo asociado a:

```text
TX_BURST
TX_CONTINUOUS
interval_us > 0
```

EV-13 utiliza una ruta distinta basada en:

```text
CMD_TX_TEST
```

y no presentó fallos de control.

Por tanto, la evidencia acumulada mantiene por ahora acotado el problema de EV-12 a:

```text
scheduling temporal de transmisión repetitiva
```

y no permite generalizarlo a todos los modos TX del CC1352P7.

---

# 23. Hallazgos para correcciones posteriores

## EV13-F01 — Falta de validación RF real de CW/PRBS

**Estado:** pendiente.

### Acción futura

Instrumentar:

```text
CW:
- frecuencia
- potencia
- estabilidad
- armónicos
- STOP

PRBS:
- ancho espectral
- patrón
- potencia
- STOP
```

---

## EV13-F02 — `RX_STOP` timeout frente a stream asíncrono

**Estado:** reproducido nuevamente.

### Áreas de código a revisar

```text
python/feralrf/radio.py

Radio.stop_rx()
Radio._read_response()
_pending_async
sequence matching
```

y firmware:

```text
DataTask
OutputIF
RSP_RX_PACKET generation
RX_STOP ordering
```

---

## EV13-F03 — Script histórico F22 no portable

`smoke_f22_tx_test.py` presenta:

```text
Shell = Bridge + 2
```

y usa:

```text
power_dbm=5
```

### Mejora futura

* eliminar inferencia numérica de puertos;
* utilizar descubrimiento Catnip/USB;
* parametrizar potencia;
* separar control vs RF;
* no asumir tráfico BLE ambiental elevado.

---

## EV13-F04 — Método OTA dependiente del entorno

La caída de paquetes BLE depende de:

```text
tráfico ambiental suficiente
```

y por tanto no es una validación reproducible universal.

### Mejora futura

Usar:

```text
instrumento RF
o
fuente BLE controlada
```

en lugar de depender del ambiente.

---

# 24. Consideración sobre Bluetooth/BLE dentro del alcance del proyecto

BLE significa:

```text
Bluetooth Low Energy
```

y forma parte de Bluetooth.

Sin embargo, FeralRF utiliza BLE principalmente a nivel PHY/raw y no constituye actualmente una pila Bluetooth completa.

La decisión de proyecto puede ser:

> Mantener las capacidades BLE de bajo nivel útiles para experimentación RF, pero no priorizar el desarrollo de una pila Bluetooth completa si el firmware oficial de CatSniffer ya cubre suficientemente las funciones Bluetooth requeridas.

Esto debe tratarse como una decisión de alcance del proyecto, no como afirmación de que FeralRF “no soporta Bluetooth”.

---

# 25. Condiciones de entorno relevantes

Durante EV-13 deben quedar registradas las siguientes condiciones:

```text
DUT              COM33
Observer         COM88
hardware         CatSniffer v3
potencia prueba  0 dBm
CW BLE           ch37
PRBS             Sub-1 GHz 868
ventana test     ~300 ms
BLE idle counts  2 / 10 / 11 / 11
switch Zigbee    NO utilizado en EV-13
instrumentación  disponible pero deliberadamente diferida
```

Estas condiciones son esenciales para poder reproducir o reinterpretar posteriormente los resultados.

---

# 26. Conclusión de auditoría

EV-13 consiguió demostrar que el firmware actual puede recorrer de manera consistente la superficie de control de los modos de test:

```text
TX_CW
TX_PRBS15
TX_PRBS32
TX_TEST_STOP
```

sin pérdida de conectividad ni necesidad de recuperación.

`TX_TEST_STOP` también fue validado como idempotente a nivel de API/control.

El intento de obtener una prueba OTA indirecta de CW mediante degradación del tráfico BLE no produjo evidencia concluyente porque el ambiente presentó una tasa baja de paquetes BLE respecto del criterio histórico empleado por el script upstream.

Adicionalmente, EV-13 volvió a reproducir el timeout de `RX_STOP` mientras continúan llegando respuestas `RSP_RX_PACKET (0x90)`, reforzando un hallazgo transversal ya observado durante EV-12.

Por tanto:

> **EV-13 demuestra funcionalidad de control de CW/PRBS/STOP, pero la caracterización física RF permanece pendiente. La principal anomalía adicional observada no pertenece directamente a CW/PRBS, sino al manejo host de RX_STOP frente a eventos RX asíncronos.**

````
