# EV-14 — Eventos RF asíncronos y firma RX

Registro canónico. Fecha experimental no documentada; posterior a EV-13 por dependencia. Tests host registrados separados de observaciones del hardware.

## 1. Contexto de evaluación

El historial de errores RF y una firma sintética motivaron observar eventos y bytes sin descartarlos, incluyendo una transición BLE→IEEE sin reset.

## 2. Objetivo de validación

Buscar errores asíncronos y coincidencia exacta con firma `8e89be`, evaluar transición mínima y verificar con FakeSerial el contrato de errores asíncronos host.

## 3. Capacidad o requisito FeralRF evaluado

`rx_packets`, `RxStreamError` y correlación de ERROR con SEQ0/FF; estado multi-PHY. [[Protocolo y API Python]], [[Arquitectura FeralRF]].

## 4. Precondiciones y condiciones

DUT COM33. IEEE canal 25 durante 10 s. Transición BLE1M canal 37 RX2 s /STOP→IEEE25 RX10 s /STOP sin reset. Hashes/identidad binaria no documentados. Test host en el registro: Python 3.14.7, pytest 9.1.1, FakeSerial; no ejecutado de nuevo en esta auditoría.

## 5. Resultado esperado

El objetivo completo requiere provocar/observar un error RF físico y demostrar su propagación. Para la búsqueda negativa se esperaba registrar cualquier evento y comparar bytes con la firma exacta; ausencia no prueba imposibilidad. Mock: SEQ0 y FF deben emerger como error asíncrono, mientras ERROR con FF esperado como respuesta se devuelve normalmente.

## 6. Procedimiento y ejecución cronológica

1. Ejecutar IEEE10 s, imprimir todos los bytes y metadatos, contar eventos/firma exacta, detener.
2. Ejecutar transición mínima BLE2 s →IEEE10 s sin reset, declarada tres veces; una corrida completa disponible.
3. Registrar ejecución de `python -m pytest .\tests\test_async_error_surfacing.py -v` con tres casos de FakeSerial.
No se inventa un estímulo de ERR_RF_INIT_FAILED ni una firma realmente observada.

## 7. Resultado observado

IEEE inicial: ocho paquetes con bytes completos, RSSI−72, CRC válido, cero errores asíncronos y cero coincidencias exactas; STOP ACK. Transición corrida literal: BLE10 paquetes/eventos 0; IEEE10 con bytes y CRC válidos, RSSI mayormente−73 y uno−92, eventos 0/firma 0; ambos STOP ACK. Tres repeticiones declaradas, una salida completa. Tests host: 3 PASSED en 0,25 s.

## 8. Evidencia

Bytes, salidas y reporte pytest íntegros en §17. La comparación es `data == firma`, no búsqueda de subcadena ni exclusión de todas las firmas sintéticas posibles. Los mocks son evidencia host, no provocación HIL de fallo RF.

## 9. Comparación entre lo esperado y lo observado

Baseline y transición mínima no presentaron anomalía buscada. El contrato host ensayado se cumple en tres mocks. El objetivo de error RF físico propagado no se alcanzó porque no se desencadenó el error.

## 10. Interpretación técnica

La transición mínima contradice que BLE→IEEE falle inevitablemente; no demuestra ausencia universal de problemas de estado. El test host acredita manejo de tres casos concretos de SEQ, no el comportamiento de una cola RF real ni todos los paquetes.

## 11. Anomalías, desviaciones y limitaciones

Sin estímulo de error físico, sin reproducción de firma, sin campañas de 20 reinicializaciones ni canary de 300 s. Dos repeticiones sólo narradas. Los bytes actuales no corrigen capturas anteriores incompletas.

## 12. Resultado de la evaluación

NOT FULLY VALIDATED para error RF real/firma; PASS para tres mocks y baseline local; PARTIAL para cambio PHY sin reset. La observación negativa queda INCONCLUSIVE respecto de inexistencia del fallo.

## 13. Confianza

High para salida literal y tres mocks; Medium para transición global; Low para excluir un error que no se provocó.

## 14. Preguntas abiertas

¿Qué estímulo controlado provoca ERR_RF_INIT_FAILED? ¿La firma aparece en otra configuración/estado? ¿Qué secuencia de SEQ/error coexiste con una respuesta pendiente real?

## 15. Acciones de seguimiento

Diseñar estímulo de error reproducible y no destructivo con captura serial/estado; ampliar ciclos conforme EV-41/46 después de estabilizar recuperación. Conservar eventos sin filtrarlos en el harness.

## 16. Trazabilidad

Guía EV-14/41/46; [[FeralRF - Guía de validación experimental]]; [[FeralRF - Matriz de pruebas]]; [[Matriz de capacidades]]; [[EV-13 — CW PRBS y TX_TEST_STOP por control]]; [[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]]; [[Pruebas y evidencia existente]]; [[Registro de validación FeralRF]].

Definición específica: [[FeralRF - Wiki técnica integral#7.4 ERROR síncrono y asíncrono]].



## 17. Notas originales preservadas y material pendiente

Fuente: `# EV-14 — Error RF asíncrono y firm.md`. SHA-256 previo: `97C1F00FDA60CC5FE3557DD16923316AA8D7581B2D6012410D1B761CE613ADCF`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
# EV-14 — Error RF asíncrono y firma sintética

## 1. Identificación de la prueba

**EV:** EV-14 — Error RF asíncrono y firma sintética
**Proyecto:** CatSniffer - FeralRF
**Firmware evaluado:** FeralRF
**Commit de referencia:** `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`
**Firmware reportado por el dispositivo:** `1.0.0`
**Hardware:** CatSniffer v3
**DUT:** `COM33`
**PHY utilizados:** BLE 1M e IEEE 802.15.4
**Canales utilizados:** BLE 37 e IEEE 802.15.4 canal 25
**Fuente de tráfico IEEE:** dispositivo/switch Zigbee operando en canal 25
**Instrumentación RF externa:** no requerida

La guía define EV-14 como una validación destinada a caracterizar el fallo tardío de inicialización RF sin confundirlo con tráfico válido. La ruta de interés es `RX_START ACK → inicio RF diferido → posible RSP_ERROR asíncrono`. También advierte que el fallo quizá no sea reproducible sin una condición RF real y que no debe forzarse daño al hardware.

---

# 2. Objetivo técnico

El objetivo central fue determinar si FeralRF distingue correctamente entre:

1. aceptación lógica del comando `RX_START`;
2. inicio real del backend RF;
3. fallo posterior del backend;
4. tráfico RF real;
5. posible tráfico sintético generado como fallback.

La situación que EV-14 intenta caracterizar es:

```text
PC
 ↓
RX_START
 ↓
ACK
 ↓
procesamiento diferido en firmware
 ↓
RadioIF_startRx()
 ↓
 ├─ éxito → recepción RF normal
 │
 └─ fallo → RSP_ERROR asíncrono
             ERR_RF_INIT_FAILED
```

Esto es relevante porque el ACK inicial no demuestra por sí solo que el RF Core haya iniciado correctamente. La Wiki identifica explícitamente esta limitación: `RX_START` puede fallar después del ACK y el host debe consumir `RxStreamError`.

---

# 3. Discrepancia histórica investigada

La EV también busca resolver una discrepancia entre documentación histórica y comportamiento esperado por el código actual.

La documentación histórica menciona una posible firma:

```text
8E 89 BE
```

asociada a un backend sintético usado cuando falla la inicialización RF.

La guía actual, en cambio, establece que ante un fallo de `RadioIF_startRx()` debe emitirse un error asíncrono:

```text
RSP_ERROR
ERR_RF_INIT_FAILED
```

Por tanto, EV-14 debe distinguir:

```text
tráfico RF real
≠
RxStreamError
≠
firma sintética 8e89be
```

La guía identifica esta discrepancia como parte de KI-12/KI-31.

---

# 4. Metodología de selección de herramientas

## 4.1 Inventario upstream

La Wiki técnica contiene una sección específica de inventario:

```text
13. Todos los ejemplos y scripts propios
```

y registra 27 scripts Python bajo `python/examples/`, además de un orquestador Bash. Para cada script se documentan propósito, entradas, API utilizada y alcance real de un PASS.

La metodología adoptada durante EV-14 fue:

> Identificar primero la instrumentación upstream existente en la Wiki, contrastar su comportamiento con el código del commit evaluado y ejecutar el script original completo cuando cubre directamente el objetivo de la EV. Cuando el script upstream no proporciona la observabilidad necesaria o presenta supuestos incompatibles con el banco, utilizar un harness mínimo derivado de la misma API pública, identificándolo explícitamente como instrumentación adaptada.

---

# 5. Herramientas upstream relacionadas

## 5.1 `python/examples/smoke_phy4_ieee154.py`

La Wiki lo define como:

```text
RX IEEE y packets
```

y señala que con paquetes observados supera una validación basada únicamente en ACK, aunque la fuente/interoperabilidad requieren control externo.

Ruta funcional tomada como referencia:

```text
Radio.init()
↓
set_phy(PHY.IEEE_802_15_4)
↓
start_rx()
↓
read_packets()
↓
stop_rx()
```

En EV-14.1 no se ejecutó literalmente el script completo porque éste no clasifica explícitamente:

```text
RxStreamError
8e89be
resultado de RX_STOP
```

Por ello se creó un harness específico de observación.

---

## 5.2 `python/examples/lab/smoke_f9_phy_matrix_ota.py`

La Wiki lo describe como:

```text
cambia PHY sin reset + markers
```

y lo clasifica como una prueba de regresión OTA entre dos endpoints FeralRF.

No se ejecutó completo porque:

* requiere dos puertos/placas;
* incluye múltiples transiciones PHY no necesarias para EV-14;
* contiene la suposición histórica `Shell = Bridge+2`, no portable a la enumeración Windows observada.

La Wiki documenta expresamente que este script se encuentra entre los afectados por dicha suposición.

Se reutilizó únicamente la idea experimental relevante:

```text
cambiar PHY sin reset
```

adaptándola a:

```text
BLE 1M ch37
↓
RX
↓
STOP
↓
IEEE 802.15.4 ch25
↓
RX
```

---

## 5.3 `python/tests/test_async_error_surfacing.py`

Este archivo sí se ejecutó **completo y sin modificaciones**.

La Wiki lo define específicamente como:

```text
errores seq 0/FF y buffering host
```

Esta prueba utiliza `FakeSerial`, por lo que valida exclusivamente la capa host/protocolo Python y no requiere hardware RF.

---

# 6. EV-14.1 — Baseline limpio IEEE 802.15.4

## Objetivo

Establecer cómo se comporta el sistema en una sesión RX sana antes de intentar cualquier cambio de PHY.

Se buscó confirmar:

```text
RX_START ACK
+
paquetes RF reales
+
0 errores asíncronos
+
0 firma 8e89be
+
RX_STOP funcional
```

## Condiciones

```text
DUT: COM33
PHY: IEEE_802_15_4
channel: 25
ventana: 10 s
switch Zigbee: activo
reset entre modos: no aplica
```

---

## Comando ejecutado

```powershell
@'
from feralrf import Radio, PHY, RxStreamError

r = Radio(port="COM33", baudrate=921600)

packets = []
errors = []
synth_count = 0

try:
    info = r.init()
    print("INFO:", info)

    r.set_phy(PHY.IEEE_802_15_4, channel=25)
    print("PHY=IEEE_802_15_4 channel=25")

    r.start_rx()
    print("RX_START ACK")

    print("READ_WINDOW=10s")

    for event in r.read_packets(timeout=10.0):
        if isinstance(event, RxStreamError):
            errors.append(event)
            print(
                "ASYNC_ERROR:",
                "error_code=0x%02X" % event.error_code,
                "context=0x%02X" % event.context,
            )
            continue

        packets.append(event)

        data_hex = event.data.hex()

        if event.data == bytes.fromhex("8e89be"):
            synth_count += 1
            print(
                "SYNTH_SIGNATURE:",
                data_hex,
                "rssi=",
                event.rssi_dbm,
                "crc_ok=",
                event.crc_ok,
            )
        else:
            print(
                "PACKET:",
                data_hex,
                "rssi=",
                event.rssi_dbm,
                "crc_ok=",
                event.crc_ok,
            )

    print("SUMMARY:")
    print("PACKETS_TOTAL=", len(packets))
    print("ASYNC_ERRORS=", len(errors))
    print("SYNTH_8E89BE=", synth_count)

    try:
        r.stop_rx()
        print("RX_STOP ACK")
    except Exception as exc:
        print("RX_STOP ERROR:", type(exc).__name__, exc)

finally:
    r.disconnect()
    print("Disconnected")
'@ | python -
```

---

# 7. Instrumentación añadida respecto del script upstream

La lógica de radio utilizada procede de la API oficial:

```text
Radio()
init()
set_phy()
start_rx()
read_packets()
stop_rx()
disconnect()
```

La instrumentación añadida específicamente para EV-14 fue:

```python
isinstance(event, RxStreamError)
```

para contabilizar errores asíncronos;

```python
event.data == bytes.fromhex("8e89be")
```

para detectar explícitamente la firma histórica;

y los contadores:

```text
PACKETS_TOTAL
ASYNC_ERRORS
SYNTH_8E89BE
```

También se capturó explícitamente una posible excepción de `RX_STOP` debido a los timeouts ya observados durante EV-12/EV-13.

---

# 8. Resultado de EV-14.1

Salida obtenida:

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
PHY=IEEE_802_15_4 channel=25
RX_START ACK
READ_WINDOW=10s
PACKET: 02001786d1 rssi= -72 crc_ok= True
PACKET: 6188d087a001005a52480201005a521e162881984101c7d1d70d01881700006dc85d020562a8a5ff02eab18a14c97ed34f7b7662c67a9e9ae853 rssi= -72 crc_ok= True
PACKET: 4188d187a0ffff5a520912fcff01001dd9e2ab7f05018817002882984101c7d1d70d01881700004adf1622e8fa1c161a72ae3a rssi= -72 crc_ok= True
PACKET: 4188d287a0ffff5a520912fcff01001dd9e2ab7f05018817002883984101c7d1d70d01881700005ea92ea2c29eb634c0f854b7 rssi= -72 crc_ok= True
PACKET: 4188d387a0ffff5a520912fcff01001dd9e2ab7f05018817002884984101c7d1d70d0188170000c81080d6db68624803a12d02 rssi= -72 crc_ok= True
PACKET: 02001c556f rssi= -72 crc_ok= True
PACKET: 6188d487a001005a52480201005a521e182885984101c7d1d70d0188170000482e33c94eaf848fa989eaae787354e586181430e459d070b33010 rssi= -72 crc_ok= True
PACKET: 4188d587a0ffff5a520912fcff5a520119c7d1d70d018817002886984101c7d1d70d0188170000599aa9742ebe57b8777ef1 rssi= -72 crc_ok= True
SUMMARY:
PACKETS_TOTAL= 8
ASYNC_ERRORS= 0
SYNTH_8E89BE= 0
RX_STOP ACK
Disconnected
```

---

# 9. Resultado esperado vs obtenido — EV-14.1

| Elemento        | Esperado         | Obtenido     |
| --------------- | ---------------- | ------------ |
| `RX_START`      | ACK              | ACK          |
| paquetes RF     | `>0` deseable    | 8            |
| CRC             | válidos          | 8/8 `True`   |
| `RxStreamError` | 0 en estado sano | 0            |
| `8e89be`        | 0                | 0            |
| `RX_STOP`       | ACK              | ACK          |
| recuperación    | no necesaria     | no necesaria |

## Veredicto

```text
EV-14.1
PASS baseline sano
```

El backend produjo tráfico IEEE 802.15.4 observable y no solamente un ACK de control.

---

# 10. EV-14.2 — Transición BLE → IEEE sin reset

## Objetivo

Intentar reproducir de forma no destructiva una condición histórica de cambio de PHY.

La secuencia fue:

```text
BLE 1M ch37
↓
RX
↓
STOP
↓
SIN RESET
↓
IEEE 802.15.4 ch25
↓
RX
```

La prueba se ejecutó **tres veces consecutivas**.

El operador reportó que no se produjeron errores en ninguna ejecución.

Sólo la tercera ejecución fue preservada íntegramente en el registro actual, por lo que debe distinguirse:

```text
3/3 runs reportados sin error
1/3 runs con stdout completo preservado
```

---

# 11. Comando EV-14.2

```powershell
@'
from feralrf import Radio, PHY, RxStreamError

r = Radio(port="COM33", baudrate=921600)

packets = []
errors = []
synth_count = 0

try:
    info = r.init()
    print("INFO:", info)

    print("STEP1: BLE_1M channel=37")
    r.set_phy(PHY.BLE_1M, channel=37)
    print("BLE SET_PHY ACK")

    r.start_rx()
    print("BLE RX_START ACK")

    ble_events = list(r.read_packets(timeout=2.0))
    ble_packets = [e for e in ble_events if not isinstance(e, RxStreamError)]
    ble_errors = [e for e in ble_events if isinstance(e, RxStreamError)]

    print("BLE PACKETS=", len(ble_packets))
    print("BLE ASYNC_ERRORS=", len(ble_errors))

    try:
        r.stop_rx()
        print("BLE RX_STOP ACK")
    except Exception as exc:
        print("BLE RX_STOP ERROR:", type(exc).__name__, exc)

    print("STEP2: IEEE_802_15_4 channel=25 WITHOUT RESET")
    r.set_phy(PHY.IEEE_802_15_4, channel=25)
    print("IEEE SET_PHY ACK")

    r.start_rx()
    print("IEEE RX_START ACK")

    print("IEEE READ_WINDOW=10s")

    for event in r.read_packets(timeout=10.0):
        if isinstance(event, RxStreamError):
            errors.append(event)
            print(
                "ASYNC_ERROR:",
                "error_code=0x%02X" % event.error_code,
                "context=0x%02X" % event.context,
            )
            continue

        packets.append(event)
        data_hex = event.data.hex()

        if event.data == bytes.fromhex("8e89be"):
            synth_count += 1
            print(
                "SYNTH_SIGNATURE:",
                data_hex,
                "rssi=",
                event.rssi_dbm,
                "crc_ok=",
                event.crc_ok,
            )
        else:
            print(
                "PACKET:",
                data_hex,
                "rssi=",
                event.rssi_dbm,
                "crc_ok=",
                event.crc_ok,
            )

    print("SUMMARY:")
    print("IEEE PACKETS_TOTAL=", len(packets))
    print("IEEE ASYNC_ERRORS=", len(errors))
    print("IEEE SYNTH_8E89BE=", synth_count)

    try:
        r.stop_rx()
        print("IEEE RX_STOP ACK")
    except Exception as exc:
        print("IEEE RX_STOP ERROR:", type(exc).__name__, exc)

finally:
    r.disconnect()
    print("Disconnected")
'@ | python -
```

---

# 12. Resultado preservado de EV-14.2

Run 3:

```text
INFO: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STEP1: BLE_1M channel=37
BLE SET_PHY ACK
BLE RX_START ACK
BLE PACKETS= 10
BLE ASYNC_ERRORS= 0
BLE RX_STOP ACK
STEP2: IEEE_802_15_4 channel=25 WITHOUT RESET
IEEE SET_PHY ACK
IEEE RX_START ACK
IEEE READ_WINDOW=10s
PACKET: 61887c87a05a52010048025a5201001e87289bc03902e2ab7f0501881700006009f981764eb50ba4f92f4f4305aea9f17abf81b0 rssi= -92 crc_ok= True
PACKET: 02007c530c rssi= -73 crc_ok= True
PACKET: 6188ea87a001005a52480201005a521e35289b9a4101c7d1d70d0188170000aa2aac6cb627bb8956d13effc3dc17c57b6553760108974a4a0ff8 rssi= -73 crc_ok= True
PACKET: 02007dda1d rssi= -73 crc_ok= True
PACKET: 6188eb87a001005a52480201005a521e37289c9a4101c7d1d70d01881700001bbbc59f54395afb0df1bacec2504d8cb160c4b81c7b719cb76a2f rssi= -73 crc_ok= True
PACKET: 4188ec87a0ffff5a520912fcff01001d8be2ab7f0501881700289d9a4101c7d1d70d01881700005ad7057fe504a18f539f0464 rssi= -73 crc_ok= True
PACKET: 4188ed87a0ffff5a520912fcff01001d8be2ab7f0501881700289e9a4101c7d1d70d01881700008c5b65ac388f5a3d979924cf rssi= -73 crc_ok= True
PACKET: 4188ee87a0ffff5a520912fcff01001d8be2ab7f0501881700289f9a4101c7d1d70d0188170000d2addbb3ef6abbc4f303ffae rssi= -73 crc_ok= True
PACKET: 020082a212 rssi= -73 crc_ok= True
PACKET: 6188ef87a001005a52480201005a521e3928a09a4101c7d1d70d01881700007698c8440a406f4c99c65fe107ec0f5cf2196719fb9b6e302e5d09 rssi= -73 crc_ok= True
SUMMARY:
IEEE PACKETS_TOTAL= 10
IEEE ASYNC_ERRORS= 0
IEEE SYNTH_8E89BE= 0
IEEE RX_STOP ACK
Disconnected
```

---

# 13. Resultado esperado vs obtenido — EV-14.2

## BLE

```text
SET_PHY ACK            esperado / obtenido
RX_START ACK           esperado / obtenido
PACKETS > 0            obtenido: 10
ASYNC_ERRORS = 0       obtenido: 0
RX_STOP ACK            obtenido
```

## Transición BLE → IEEE

```text
sin reset
```

No se realizó power-cycle ni reset del CC1352 entre ambos PHY.

## IEEE

```text
SET_PHY ACK            obtenido
RX_START ACK           obtenido
PACKETS_TOTAL          10
CRC true               10/10
ASYNC_ERRORS           0
SYNTH_8E89BE           0
RX_STOP ACK            obtenido
```

## Veredicto

```text
EV-14.2
PASS — transición BLE→IEEE sin reset
no reproduce el fallo histórico bajo estas condiciones
```

La prueba fue repetida tres veces y no se reportaron errores en ninguno de los tres intentos.

---

# 14. Conclusión específica sobre switching de PHY

La evidencia permite rechazar la afirmación fuerte:

> “BLE → IEEE sin reset provoca necesariamente fallo de inicialización RF.”

No ocurrió en el hardware evaluado.

La conclusión defendible es:

> En el firmware y hardware evaluados, la transición mínima BLE 1M canal 37 → IEEE 802.15.4 canal 25 sin reset fue estable en 3/3 ejecuciones. No se observó `RxStreamError`, firma `8E89BE`, timeout de RX_START ni necesidad de recuperación.

Esto no demuestra que no exista ninguna combinación interna de estado capaz de producir `RadioIF_startRx() == false`.

---

# 15. EV-14.3 — Validación upstream del manejo host

## Objetivo

Comprobar que, si el firmware emite un error RF asíncrono, la librería Python actual lo expone correctamente.

## Herramienta

```text
python/tests/test_async_error_surfacing.py
```

Tipo:

```text
test upstream completo
```

Adaptaciones:

```text
ninguna
```

Hardware:

```text
no requerido
```

La Wiki identifica este test específicamente como la prueba de errores `seq 0/FF` y buffering host.

---

# 16. Comando ejecutado

```powershell
python -m pytest .\tests\test_async_error_surfacing.py -v
```

---

# 17. Resultado EV-14.3

```text
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python> python -m pytest .\tests\test_async_error_surfacing.py -v
================================================= test session starts =================================================
platform win32 -- Python 3.14.7, pytest-9.1.1, pluggy-1.6.0 -- C:\Users\Support\AppData\Local\Programs\Python\Python314\python.exe
cachedir: .pytest_cache
rootdir: C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python
configfile: pyproject.toml
plugins: platformdirs-4.12.2, asyncio-1.4.0, cov-7.1.0
asyncio: mode=Mode.AUTO, debug=False, asyncio_default_fixture_loop_scope=None, asyncio_default_test_loop_scope=function
collected 3 items

tests/test_async_error_surfacing.py::test_read_packets_yields_rx_stream_error_seq_zero PASSED                    [ 33%]
tests/test_async_error_surfacing.py::test_read_packets_yields_rx_stream_error_seq_0xff_compat PASSED             [ 66%]
tests/test_async_error_surfacing.py::test_read_response_returns_seq_0xff_error_when_error_expected PASSED        [100%]

================================================== 3 passed in 0.25s ==================================================
```

---

# 18. Qué valida cada test

## `test_read_packets_yields_rx_stream_error_seq_zero`

Valida el contrato actual:

```text
RSP_ERROR
seq=0
↓
Radio.read_packets()
↓
RxStreamError
```

Este es el formato relevante para el comportamiento actual esperado del firmware.

---

## `test_read_packets_yields_rx_stream_error_seq_0xff_compat`

Valida compatibilidad histórica:

```text
RSP_ERROR
seq=0xFF
↓
RxStreamError
```

El host actual conserva compatibilidad con versiones anteriores.

---

## `test_read_response_returns_seq_0xff_error_when_error_expected`

Comprueba que `_read_response()` no descarte un error asíncrono `seq=0xFF` por una comparación estricta con el sequence ID del comando síncrono.

---

# 19. Resultado esperado vs obtenido — EV-14.3

```text
tests esperados: 3
tests pasados:   3
tests fallidos:  0
```

## Veredicto

```text
PASS-software
```

La capa host/protocolo maneja correctamente los errores asíncronos cubiertos por el test upstream.

---

# 20. Separación estricta de evidencias

EV-14 produjo tres clases diferentes de evidencia.

## Evidencia física

Obtenida con CatSniffer real:

```text
EV-14.1 IEEE limpio
EV-14.2 BLE→IEEE sin reset
```

Resultados:

```text
RX real observado
CRC válido
sin error async
sin 8e89be
```

---

## Evidencia software

Obtenida con:

```text
test_async_error_surfacing.py
FakeSerial
```

Resultado:

```text
RSP_ERROR seq0 / seqFF
↓
RxStreamError
```

validado 3/3.

---

## Ruta física de error

No fue reproducida:

```text
RX_START ACK
↓
RadioIF_startRx() == false
↓
ERR_RF_INIT_FAILED real
```

Por tanto:

```text
NO VALIDADA FÍSICAMENTE
NO DECLARADA AUSENTE
```

---

# 21. Resultado consolidado

| Elemento                    | Resultado               |
| --------------------------- | ----------------------- |
| IEEE RX limpio              | **PASS-HW**             |
| paquetes IEEE reales        | **observados**          |
| CRC IEEE                    | **8/8 true en EV-14.1** |
| BLE RX antes de switching   | **PASS-HW**             |
| BLE→IEEE sin reset          | **PASS 3/3**            |
| IEEE posterior al switching | **PASS-HW**             |
| CRC run preservado EV-14.2  | **10/10 true**          |
| `RxStreamError` físico      | **no observado**        |
| `ERR_RF_INIT_FAILED` físico | **no reproducido**      |
| `8E89BE` físico             | **no observado**        |
| `RX_STOP` EV-14.1           | **ACK**                 |
| `RX_STOP` EV-14.2           | **ACK**                 |
| host async seq=0            | **PASS-SW**             |
| host async seq=0xFF         | **PASS-SW**             |
| `_read_response()` seq=0xFF | **PASS-SW**             |

---

# 22. Veredicto general de EV-14

EV-14 se clasifica como:

> **PASS para la ruta normal de recepción en hardware y PASS para el manejo host de errores RF asíncronos; la condición física que produce `ERR_RF_INIT_FAILED` no fue reproducida y la firma sintética histórica `8E 89 BE` no fue observada.**

En formato resumido:

```text
Normal RX path               VALIDADO EN HW
BLE→IEEE switching           VALIDADO EN HW 3/3
Async error parser seq=0     VALIDADO EN SW
Async error parser seq=FF    VALIDADO EN SW
Physical RF init failure     NO REPRODUCIDO
Synthetic 8E89BE             NO OBSERVADO
```

---

# 23. Qué NO demuestra EV-14

No puede afirmarse:

> “`ERR_RF_INIT_FAILED` nunca ocurre.”

Tampoco puede afirmarse:

> “el backend sintético ya no puede ejecutarse.”

Lo demostrado es únicamente:

> Las condiciones ensayadas no alcanzaron esas rutas.

Es posible que existan condiciones internas más específicas asociadas a:

```text
RF handle lifecycle
modo RF previo
estado de comandos activos
cola RF
cambio entre otras bandas
estado residual de sesiones TX/RX
workarounds del TI RF Driver
```

que puedan provocar un fallo diferente.

Esto debe investigarse posteriormente mediante auditoría estática cruzada con los resultados físicos de esta campaña.

---

# 24. Implicación para una auditoría posterior con Codex

Los resultados obtenidos permiten plantear una auditoría posterior con una restricción experimental clara.

Codex no deberá asumir:

```text
BLE→IEEE = causa del fallo
```

porque la transición pasó 3/3.

La auditoría deberá determinar qué rutas concretas pueden producir:

```text
RadioIF_startRx() == false
```

y contrastarlas con:

```text
EV-14.1 PASS
EV-14.2 PASS 3/3
EV-14.3 PASS 3/3 software
```

Las áreas prioritarias a analizar serán:

```text
RadioIF_startRx()
RadioIF_switchRfMode()
RadioIF_startBleRfBackend()
RadioIF_stopRx()
RadioIF_createRfDataQueue()
RadioIF_resetRfDataQueue()
```

y estado persistente como:

```text
s_rf_handle
s_non433_handle
s_rf_mode
s_backend
s_rf_rx_cmd
s_rx_running
s_rf_data_queue
s_rf_read_entry
s_selected_phy
```

La meta será identificar condiciones plausibles que no contradigan los resultados físicos ya obtenidos.

---

# 25. Relación con otros hallazgos de la campaña

EV-14 debe mantenerse separada del problema observado durante EV-12/EV-13:

```text
RX_STOP timeout
while RSP_RX_PACKET 0x90 remains active
```

Ese problema ocurrió con una sesión RX ya funcional.

EV-14 estudia:

```text
fallo durante inicio del backend
```

Por tanto son fenómenos distintos hasta que exista evidencia de una causa común.

Durante EV-14, de hecho:

```text
RX_STOP EV-14.1 = ACK
RX_STOP BLE EV-14.2 = ACK
RX_STOP IEEE EV-14.2 = ACK
```

lo cual demuestra que el timeout anterior no ocurre en todas las sesiones RX.

---

# 26. Recursos utilizados

## Hardware

```text
1 × CatSniffer v3 FeralRF
COM33
```

## Fuente de tráfico

```text
Switch/dispositivo Zigbee
IEEE 802.15.4 channel 25
```

## Segunda CatSniffer

```text
No requerida
```

## Instrumentación externa

```text
No requerida
```

## Software upstream relacionado

```text
python/examples/smoke_phy4_ieee154.py
python/examples/lab/smoke_f9_phy_matrix_ota.py
python/tests/test_async_error_surfacing.py
```

## Instrumentación personalizada

```text
EV-14.1 diagnostic harness
EV-14.2 PHY-switch diagnostic harness
```

Los harnesses personalizados utilizaron exclusivamente la API pública de FeralRF y añadieron observabilidad específica.

---

# 27. Condiciones clave de reproducibilidad

```text
Firmware DeviceInfo:
1.0.0

Capabilities:
7

Reported serial:
464552414c524631

DUT:
COM33

Baudrate:
921600

IEEE PHY:
IEEE_802_15_4

IEEE channel:
25

BLE PHY:
BLE_1M

BLE channel:
37

EV-14.1 window:
10 s

EV-14.2 BLE window:
2 s

EV-14.2 IEEE window:
10 s

PHY reset between BLE/IEEE:
NO

Zigbee source:
active

External RF instrumentation:
NO
```

---

# 28. Conclusión final

EV-14 logró establecer un baseline reproducible de recepción RF sana y comprobar que el switching mínimo BLE→IEEE no reproduce el fallo histórico en el firmware evaluado.

La evidencia física muestra:

```text
IEEE RX sano
BLE RX sano
BLE→IEEE sano 3/3
0 RxStreamError
0 8e89be
```

La evidencia software upstream muestra:

```text
RSP_ERROR seq=0
RSP_ERROR seq=0xFF
↓
correctamente expuestos como RxStreamError
```

Por tanto, el sistema actual dispone de una ruta host adecuada para reportar errores RF asíncronos, pero la condición física necesaria para activar `ERR_RF_INIT_FAILED` no apareció durante la campaña.

La discrepancia histórica sobre el backend sintético permanece como una ruta no reproducida y debe conservarse como cuestión abierta, no como defecto demostrado ni como comportamiento descartado.

**Resultado global:**

> **EV-14 — PASS-HW para recepción/switching normal + PASS-SW para manejo de errores asíncronos. `ERR_RF_INIT_FAILED` físico no reproducido. Firma sintética `8E 89 BE` no observada.**

````
