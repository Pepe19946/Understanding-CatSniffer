# EV-06 — Exclusión RX y TX y recuperación de estado

Registro canónico de EV-06. Los errores de PowerShell y de inspección de excepciones se conservan como fallos del procedimiento, separados de respuestas del dispositivo.

## 1. Contexto de evaluación

La recepción funcional de EV-05 permitió plantear qué sucede al pedir TX mientras RX está activo y si el estado sigue siendo recuperable.

## 2. Objetivo de validación

Provocar una transición TX no permitida durante RX y comprobar la respuesta `ERR_INVALID_STATE` y la recuperación mediante RX_STOP/estadísticas.

## 3. Capacidad o requisito FeralRF evaluado

Exclusión de estados RF, errores de protocolo y `CommandError` de Python. [[Protocolo y API Python]].

## 4. Precondiciones y condiciones

CatSniffer V3 (RP2040 + CC1352P7), Cat-Bridge COM88 a921600; PHY IEEE 802.15.4, canal 25, tras RX_START aceptado; TX de `01` a −20 dBm en la prueba válida. Firmware FeralRF en CC1352P7; hash/binario instalado y tiempos exactos no documentados. No hay medición RF de TX durante el rechazo.

## 5. Resultado esperado

Rechazo explícito por estado inválido, sin convertirlo en éxito; posibilidad de detener RX y continuar usando el control.

## 6. Procedimiento y ejecución cronológica

1. Primer intento: fallo de quoting de PowerShell, sin prueba válida del DUT.
2. Segundo: importación `Phy` incorrecta frente a `PHY`, sin prueba válida del DUT.
3. Primera ejecución válida: RX_START, TX, excepción; el helper consultó `e.code` y mostró `None`; RX_STOP/estadísticas siguieron funcionando.
4. Inspeccionar la API/atributo real de `CommandError` según la nota.
5. Repetir consultando `error_code`: valor 5/0x05; detener RX y leer estadísticas en cero.
Dos ejecuciones válidas, no cuatro pruebas de hardware.

## 7. Resultado observado

Error explícito 0x05 en la ejecución correctamente instrumentada; control recuperado mediante stop. `None` fue una lectura del atributo incorrecto del harness, no ausencia probada de error del firmware.

## 8. Evidencia

Tracebacks, comandos, salida y examen de `Radio.transmit`/`CommandError` en §17. La captura de `error_code` conserva la interpretación de `payload[0]`.

**Nivel de evidencia:** C: rechazo 0x05 y recuperación del control. F: ausencia física de emisión durante el rechazo. El error del atributo del harness no es un resultado RF.

Modelo: [[FeralRF - Matriz de pruebas#Modelo de evidencia A–F]]. Nivel, resultado, confianza y procedencia son dimensiones separadas.

## 9. Comparación entre lo esperado y lo observado

El rechazo y la recuperación cumplen la expectativa de control. No se observó de forma independiente ausencia de emisión RF durante el intento.

## 10. Interpretación técnica

Hallazgo confirmado: rechazo por estado y continuidad del control bajo la secuencia registrada. No permite afirmar ausencia absoluta de RF ni robustez de todas las transiciones.

## 11. Anomalías, desviaciones y limitaciones

Dos intentos iniciales no llegaron a medir el DUT. Sin trazas de RF ni estrés/repetición amplia. Estadísticas en cero no equivalen a ausencia de errores en todas las colas internas.

## 12. Resultado de la evaluación

PASS + C para rechazo de la solicitud TX con 0x05 tras RX_START y recuperación local del control. No demuestra de forma independiente ausencia física de RF ni exclusión para todos los modos.

## 13. Confianza

High para código 0x05 y recuperación registrada; limitado a esta transición.

## 14. Preguntas abiertas

¿Se rechazan del mismo modo CW/PRBS/BURST durante RX? ¿Se produce algún evento tardío después del rechazo?

## 15. Acciones de seguimiento

Extender transiciones sólo con control de estado, captura de eventos y evidencia física; mantener `error_code` como contrato del harness.

## 16. Trazabilidad

Guía EV-06; [[FeralRF - Guía de validación experimental]]; [[FeralRF - Matriz de pruebas]]; [[Matriz de capacidades]]; [[EV-05 — Primera observación RF IEEE]]; [[EV-10 — Matriz de PHY por control]]; [[Registro de validación FeralRF]].

Definición específica: [[FeralRF - Wiki técnica integral#6.3 Métodos RF públicos y semántica real]].



## 17. Notas originales preservadas y material pendiente

Fuente: `# EV-06 — Validación de exclusión R.md`. SHA-256 previo: `69798410C4E0B4495A0120180BDFF01A83768D7B9F6E99B36ADE9746818D8E52`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
# EV-06 — Validación de exclusión RX/TX y recuperación ante `ERR_INVALID_STATE`

**Estado final:** `PASS`

## 1. Propósito de la prueba

EV-06 fue ejecutada para comprobar experimentalmente un comportamiento documentado de FeralRF: cuando el dispositivo se encuentra con recepción RF activa, una solicitud de transmisión no debe iniciarse, sino ser rechazada mediante un error de estado inválido.

La documentación de validación define para EV-06 una prueba de exclusión entre RX y TX. El comportamiento esperado es que, con RX activo, una llamada de transmisión produzca `ERR_INVALID_STATE`, correspondiente al código `0x05`, y que posteriormente el dispositivo continúe respondiendo normalmente. La matriz resume el resultado esperado como `ERROR 0x05 + recovery`.
Por tanto, esta prueba no intenta demostrar recepción física como EV-05 ni caracterizar potencia de transmisión. Su objetivo específico es reproducir la siguiente condición documentada:

```text
RX activo
    ↓
Solicitud de TX
    ↓
TX debe ser rechazado
    ↓
ERR_INVALID_STATE / 0x05
    ↓
El dispositivo debe continuar operativo
```

La validación debía comprobar dos puntos independientes:

1. que `transmit()` fuera rechazado mientras RX permanecía activo;
2. que, después de ese rechazo, fuera posible ejecutar `RX_STOP` y `GET_STATS` sin reset, power-cycle ni reconexión USB.

---

## 2. Plataforma utilizada

La prueba se realizó sobre la misma plataforma utilizada en EV-00 a EV-05.

```text
Hardware:
CatSniffer V3

Procesadores relevantes:
RP2040
CC1352P7

Procesador donde se ejecuta FeralRF:
CC1352P7

Interfaz host utilizada:
Cat-Bridge

Puerto asignado:
COM88

Baudrate:
921600

PHY seleccionado:
IEEE 802.15.4

Canal:
25

Payload solicitado para transmisión:
b"\x01"

Potencia solicitada:
-20 dBm
```

El flujo funcional que se intentó reproducir fue:

```text
PC / Python
    ↓
feralrf.Radio
    ↓
COM88 / Cat-Bridge
    ↓
RP2040
    ↓ UART
CC1352P7 ejecutando FeralRF
    ↓
Estado RX activo
    ↓
Solicitud TX_RAW
```

---

# 3. Primer intento de ejecución

## 3.1 Comando ejecutado

El primer intento utilizó un único comando `python -c` desde PowerShell:

```powershell
python -c "from feralrf import Radio,Phy; from feralrf.exceptions import CommandError; r=Radio(port='COM88'); print('INIT:',r.init()); r.set_phy(Phy.IEEE_802_15_4); r.set_channel(25); r.start_rx(); print('RX: STARTED'); exec(\"try:\n r.transmit(b'\\x01', power_dbm=-20)\n print('TX: UNEXPECTEDLY ACCEPTED')\nexcept CommandError as e:\n print('TX REJECTED:',repr(e))\"); r.stop_rx(); print('RX: STOPPED'); print('STATS:',r.get_stats()); r.disconnect()"
```

## 3.2 Intención

Este comando pretendía realizar en una sola ejecución:

```text
Radio(port="COM88")
    ↓
r.init()
    ↓
SET_PHY IEEE_802_15_4
    ↓
SET_CHANNEL 25
    ↓
RX_START
    ↓
transmit(b"\x01", power_dbm=-20)
    ↓
capturar CommandError
    ↓
RX_STOP
    ↓
GET_STATS
```

El propósito era comprobar si la transmisión era rechazada durante RX activo y si posteriormente el dispositivo continuaba respondiendo.

## 3.3 Resultado obtenido exactamente

PowerShell devolvió:

```text
En línea: 1 Carácter: 254
+ ... x(); print('RX: STARTED'); exec(\"try:\n r.transmit(b'\\x01', power_d ...
+                                                                 ~
Falta un argumento en la lista de parámetros.
En línea: 1 Carácter: 358
+ ... ACCEPTED')\nexcept CommandError as e:\n print('TX REJECTED:',repr(e)) ...
+                                                                  ~
Falta una expresión después de ','.
En línea: 1 Carácter: 358
+ ... PTED')\nexcept CommandError as e:\n print('TX REJECTED:',repr(e))\"); ...
+                                                              ~~~~
Token 'repr' inesperado en la expresión o la instrucción.
En línea: 1 Carácter: 358
+ ... ACCEPTED')\nexcept CommandError as e:\n print('TX REJECTED:',repr(e)) ...
+                                                                  ~
Falta el paréntesis de cierre ')' en la expresión.
En línea: 1 Carácter: 365
+ ... ')\nexcept CommandError as e:\n print('TX REJECTED:',repr(e))\"); r.s ...
+                                                                 ~
Token ')' inesperado en la expresión o la instrucción.
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : MissingArgument
```

## 3.4 Análisis

Este resultado no provino de FeralRF ni del CatSniffer.

PowerShell no pudo interpretar correctamente la combinación de comillas, caracteres escapados y el `exec()` contenido dentro de `python -c`.

La ejecución falló en el parser de PowerShell antes de iniciar correctamente el código Python que debía comunicarse con COM88.

Por tanto:

```text
Interacción con COM88: NO
Interacción con RP2040: NO
Interacción con CC1352P7: NO
RX iniciado: NO
TX solicitado: NO
Valor para PASS/FAIL de EV-06: NINGUNO
```

Este intento se conserva únicamente como evidencia de una incidencia en el procedimiento de validación.

La corrección aplicada fue dejar de utilizar un `python -c` con quoting complejo y enviar el programa completo mediante un here-string de PowerShell hacia `python -`.

---

# 4. Segundo intento de ejecución

## 4.1 Código utilizado

El siguiente intento comenzó con:

```python
from feralrf import Radio, Phy
from feralrf.exceptions import CommandError
```

## 4.2 Resultado obtenido exactamente

Python devolvió:

```text
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: cannot import name 'Phy' from 'feralrf' (C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\__init__.py). Did you mean: 'PHY'?
```

## 4.3 Análisis

La API instalada/evaluada no exporta un símbolo llamado:

```python
Phy
```

sino:

```python
PHY
```

La excepción ocurrió durante el `import`, antes de ejecutar:

```python
r = Radio(port="COM88")
```

Por tanto, tampoco hubo interacción con el dispositivo.

```text
Interacción con COM88: NO
Radio creado: NO
RX iniciado: NO
TX solicitado: NO
Valor para PASS/FAIL de EV-06: NINGUNO
```

Este segundo intento se registra como una corrección necesaria del procedimiento de prueba, no como defecto del DUT.

---

# 5. Primera ejecución válida sobre el dispositivo

Una vez corregido `Phy` por `PHY`, se ejecutó una prueba completa contra COM88.

## 5.1 Comando ejecutado exactamente

```powershell
@'
from feralrf import Radio, PHY
from feralrf.exceptions import CommandError

r = Radio(port="COM88")

try:
    print("INIT:", r.init())

    r.set_phy(PHY.IEEE_802_15_4)
    r.set_channel(25)
    r.start_rx()
    print("RX: STARTED")

    try:
        r.transmit(b"\x01", power_dbm=-20)
        print("TX: UNEXPECTEDLY ACCEPTED")
    except CommandError as e:
        print("TX REJECTED:", repr(e))
        print("ERROR CODE:", getattr(e, "code", None))

    r.stop_rx()
    print("RX: STOPPED")
    print("STATS:", r.get_stats())

finally:
    r.disconnect()
'@ | python -
```

## 5.2 Acción realizada por el código

El programa realiza, en este orden:

```python
r = Radio(port="COM88")
```

Crea la interfaz host asociada al Cat-Bridge conocido.

Después:

```python
r.init()
```

inicializa la comunicación con FeralRF.

Posteriormente:

```python
r.set_phy(PHY.IEEE_802_15_4)
```

selecciona el PHY IEEE 802.15.4.

Después:

```python
r.set_channel(25)
```

selecciona el canal 25.

Luego:

```python
r.start_rx()
```

solicita al firmware entrar en estado RX.

Con RX activo se ejecuta:

```python
r.transmit(b"\x01", power_dbm=-20)
```

La intención es provocar específicamente la condición documentada de conflicto RX/TX.

La transmisión se encuentra dentro de:

```python
try:
    ...
except CommandError as e:
```

por lo que un rechazo generado por FeralRF puede observarse sin terminar el programa.

Después se ejecutan:

```python
r.stop_rx()
```

y:

```python
r.get_stats()
```

para comprobar que el dispositivo continúa respondiendo tras el error esperado.

Finalmente:

```python
r.disconnect()
```

cierra la conexión host.

---

## 5.3 Resultado obtenido exactamente

La salida completa fue:

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
RX: STARTED
TX REJECTED: CommandError('Transmit failed')
ERROR CODE: None
RX: STOPPED
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

---

# 6. Análisis de la primera ejecución válida

La primera línea:

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
```

demuestra que el programa llegó al DUT y que `r.init()` completó satisfactoriamente.

La respuesta identifica:

```text
firmware_version='1.0.0'
capabilities=7
serial='464552414c524631'
```

por lo que esta ejecución ya no corresponde a un error local de PowerShell o Python.

Posteriormente se obtuvo:

```text
RX: STARTED
```

lo que demuestra que el programa ejecutó sin excepción:

```python
r.set_phy(PHY.IEEE_802_15_4)
r.set_channel(25)
r.start_rx()
```

y alcanzó el punto de prueba requerido: RX estaba activo cuando se solicitó TX.

La siguiente salida fue:

```text
TX REJECTED: CommandError('Transmit failed')
```

Este resultado es importante porque demuestra que:

```python
r.transmit(b"\x01", power_dbm=-20)
```

no retornó normalmente.

Si hubiera sido aceptado, el programa habría producido explícitamente:

```text
TX: UNEXPECTEDLY ACCEPTED
```

Esa línea no apareció.

En su lugar, `Radio.transmit()` generó:

```text
CommandError('Transmit failed')
```

Por tanto, la primera parte del comportamiento documentado quedó reproducida:

```text
RX activo
    ↓
solicitud TX_RAW
    ↓
TX rechazado
```

Sin embargo, la línea:

```text
ERROR CODE: None
```

todavía no permitía afirmar que el rechazo correspondiera específicamente a `ERR_INVALID_STATE = 0x05`.

Después del rechazo, el programa continuó y produjo:

```text
RX: STOPPED
```

por lo que:

```python
r.stop_rx()
```

se ejecutó correctamente después del `CommandError`.

Finalmente se obtuvo exactamente:

```text
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

Esto demuestra que `r.get_stats()` recibió una respuesta válida después del error.

Los campos observados fueron:

```text
rx_ok=0
rx_crc_err=0
rx_drop=0
rx_overflow=0
ll_kind_unknown=0
ll_kind_adv=0
ll_kind_scan=0
ll_kind_connect=0
ll_kind_data=0
```

EV-06 no exige tráfico RX durante esta prueba, por lo que estos contadores en cero no representan un fallo.

El dato importante en esta etapa es que `GET_STATS` respondió correctamente después del rechazo de TX.

Por tanto, la primera ejecución válida reproducía:

```text
INIT correcto
    ↓
RX activo
    ↓
TX rechazado
    ↓
RX_STOP correcto
    ↓
GET_STATS correcto
```

pero todavía faltaba comprobar que el código del rechazo fuera exactamente `0x05`.

---

# 7. Inspección estática de la implementación Python

Para determinar por qué la prueba mostraba:

```text
ERROR CODE: None
```

se decidió no repetir inmediatamente la operación contra el DUT.

Primero se inspeccionó la implementación real de `Radio.transmit()` y `CommandError`.

## 7.1 Comando ejecutado exactamente

```powershell
python -c "import inspect; from feralrf import Radio; from feralrf.exceptions import CommandError; print('=== Radio.transmit ==='); print(inspect.getsource(Radio.transmit)); print('=== CommandError ==='); print(inspect.getsource(CommandError))"
```

## 7.2 Propósito

El objetivo era responder a estas preguntas:

```text
¿Radio.transmit() recibe Response.ERROR?
¿Extrae un código del payload?
¿Ese código se conserva?
¿CommandError tiene un atributo específico para acceder a él?
```

Este comando únicamente inspecciona código Python local.

No abre COM88 y no modifica el estado del DUT.

---

## 7.3 Resultado obtenido exactamente

```text
=== Radio.transmit ===
    def transmit(self, packet: bytes, power_dbm: int = -128, timeout: float = 5.0) -> None:
        """Transmit a packet"""
        self._send_command(Command.TX_RAW, CommandBuilder.tx_raw(packet, power_dbm))
        cmd_id, seq, payload = self._read_response(
            timeout=timeout, expected={Response.ACK, Response.ERROR}
        )

        if cmd_id == Response.ERROR:
            raise CommandError("Transmit failed", payload[0] if payload else 0)
        if cmd_id != Response.ACK:
            raise ProtocolError(f"Unexpected response to TX_RAW: 0x{cmd_id:02X}")

=== CommandError ===
class CommandError(FeralRFError):
    """Command execution error"""

    def __init__(self, message: str, error_code: int = 0):
        super().__init__(message)
        self.error_code = error_code
```

---

# 8. Interpretación del código inspeccionado

La implementación real de `transmit()` muestra primero:

```python
self._send_command(Command.TX_RAW, CommandBuilder.tx_raw(packet, power_dbm))
```

Por tanto, la API construye y envía el comando `TX_RAW`.

Después espera explícitamente uno de dos tipos de respuesta:

```python
expected={Response.ACK, Response.ERROR}
```

Esto confirma que el camino normal de la función contempla tanto aceptación como rechazo por parte del dispositivo.

Cuando:

```python
cmd_id == Response.ERROR
```

el código ejecuta:

```python
raise CommandError("Transmit failed", payload[0] if payload else 0)
```

Esto demuestra que el primer byte del payload de la respuesta `ERROR` se transmite a `CommandError`.

La implementación de `CommandError` almacena ese valor mediante:

```python
self.error_code = error_code
```

Por tanto, el recorrido de información implementado es:

```text
Respuesta del dispositivo
Response.ERROR
      ↓
payload[0]
      ↓
Radio.transmit()
      ↓
CommandError(..., error_code)
      ↓
e.error_code
```

La salida anterior:

```text
ERROR CODE: None
```

no fue causada por pérdida del código dentro de FeralRF.

El problema estaba en el código de prueba:

```python
getattr(e, "code", None)
```

Se consultó un atributo denominado:

```text
code
```

pero la clase implementa:

```text
error_code
```

Por ello `getattr(..., None)` devolvió `None`.

Esta observación es importante para el registro porque evita clasificar incorrectamente el resultado como defecto de la API.

La evidencia del código demuestra que `Radio.transmit()` sí conserva el código de error proporcionado en la respuesta.

---

# 9. Ejecución definitiva para validar `ERR_INVALID_STATE`

Una vez identificado el atributo correcto, se repitió EV-06 utilizando:

```python
e.error_code
```

## 9.1 Comando ejecutado exactamente

```powershell
@'
from feralrf import Radio, PHY
from feralrf.exceptions import CommandError

r = Radio(port="COM88")

try:
    print("INIT:", r.init())

    r.set_phy(PHY.IEEE_802_15_4)
    r.set_channel(25)
    r.start_rx()
    print("RX: STARTED")

    try:
        r.transmit(b"\x01", power_dbm=-20)
        print("TX: UNEXPECTEDLY ACCEPTED")
    except CommandError as e:
        print("TX REJECTED:", repr(e))
        print("ERROR CODE DEC:", e.error_code)
        print("ERROR CODE HEX: 0x%02X" % e.error_code)

    r.stop_rx()
    print("RX: STOPPED")
    print("STATS:", r.get_stats())

finally:
    r.disconnect()
'@ | python -
```

---

## 9.2 Intención de esta ejecución

Esta segunda ejecución válida no fue realizada como una repetición arbitraria.

Su objetivo específico fue completar el criterio documental que permanecía sin comprobar después de la primera ejecución:

```text
¿El CommandError obtenido durante RX activo corresponde realmente a 0x05?
```

La secuencia fue nuevamente:

```text
INIT
    ↓
SET_PHY IEEE802.15.4
    ↓
SET_CHANNEL 25
    ↓
RX_START
    ↓
TX_RAW durante RX
    ↓
capturar CommandError
    ↓
leer error_code
    ↓
RX_STOP
    ↓
GET_STATS
```

---

# 10. Resultado definitivo obtenido exactamente

La salida completa fue:

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
RX: STARTED
TX REJECTED: CommandError('Transmit failed')
ERROR CODE DEC: 5
ERROR CODE HEX: 0x05
RX: STOPPED
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

---

# 11. Análisis detallado de la ejecución definitiva

## 11.1 Inicialización

Se obtuvo:

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
```

Por tanto, `r.init()` completó correctamente y el dispositivo respondió por COM88.

Los valores observados fueron:

```text
firmware_version='1.0.0'
capabilities=7
serial='464552414c524631'
```

Esto confirma que el DUT estaba accesible antes de provocar la condición de error.

---

## 11.2 Entrada en RX

La salida:

```text
RX: STARTED
```

se imprime únicamente después de ejecutar:

```python
r.set_phy(PHY.IEEE_802_15_4)
r.set_channel(25)
r.start_rx()
```

sin excepción.

Por tanto, la condición previa necesaria para EV-06 quedó establecida:

```text
PHY = IEEE 802.15.4
canal = 25
estado = RX activo
```

---

## 11.3 Solicitud de transmisión durante RX

Con RX activo se ejecutó específicamente:

```python
r.transmit(b"\x01", power_dbm=-20)
```

Los parámetros utilizados fueron:

```text
payload = b"\x01"
power_dbm = -20
```

El código contenía una condición explícita para detectar una aceptación incorrecta:

```python
print("TX: UNEXPECTEDLY ACCEPTED")
```

Esa línea no apareció.

En su lugar se obtuvo:

```text
TX REJECTED: CommandError('Transmit failed')
```

Por tanto, `Radio.transmit()` recibió una condición de error y generó `CommandError`.

---

## 11.4 Código exacto de rechazo

La prueba imprimió el atributo correcto:

```text
ERROR CODE DEC: 5
ERROR CODE HEX: 0x05
```

Estos dos valores representan el mismo byte:

```text
decimal:     5
hexadecimal: 0x05
```

Por tanto, el error devuelto durante la solicitud de TX con RX activo fue exactamente el código esperado por EV-06.

La documentación de validación establece `ERR_INVALID_STATE / 0x05` como comportamiento esperado para esta condición.

La matriz resume igualmente el resultado esperado como:

```text
ERROR 0x05 + recovery
```

La parte `ERROR 0x05` quedó reproducida directamente en la salida del programa.

---

# 12. Validación de recuperación después de `0x05`

Después de recibir:

```text
ERROR CODE HEX: 0x05
```

el mismo proceso Python continuó sin reiniciar el DUT.

Se obtuvo:

```text
RX: STOPPED
```

Esto confirma que:

```python
r.stop_rx()
```

respondió correctamente después del error.

Posteriormente se ejecutó:

```python
r.get_stats()
```

y la respuesta completa fue:

```text
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

Los valores recibidos fueron:

```text
rx_ok=0
rx_crc_err=0
rx_drop=0
rx_overflow=0
ll_kind_unknown=0
ll_kind_adv=0
ll_kind_scan=0
ll_kind_connect=0
ll_kind_data=0
```

La importancia de esta respuesta para EV-06 no está en que los contadores sean distintos de cero.

La prueba no tenía como propósito recibir tráfico IEEE 802.15.4.

Lo relevante es que `GET_STATS` pudo completar y devolver una estructura válida después de:

```text
RX activo
    ↓
TX rechazado con 0x05
    ↓
RX_STOP
```

No fue necesario realizar:

```text
power-cycle
USB disconnect/reconnect
boot/exit
reset_device()
reset físico
```

Por tanto, la segunda parte del criterio, `recovery`, también quedó reproducida.

---

# 13. Comparación directa entre documentación y observación experimental

## Comportamiento documentado

EV-06 espera:

```text
1. RX activo
2. intento de transmitir
3. ERR_INVALID_STATE
4. código 0x05
5. dispositivo todavía operativo
6. GET_STATS debe responder
```

## Comportamiento observado

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')

RX: STARTED

TX REJECTED: CommandError('Transmit failed')

ERROR CODE DEC: 5
ERROR CODE HEX: 0x05

RX: STOPPED

STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

Correspondencia:

```text
Documentado: RX activo
Observado:   RX: STARTED
Resultado:   coincide
```

```text
Documentado: TX debe rechazarse
Observado:   TX REJECTED: CommandError('Transmit failed')
Resultado:   coincide
```

```text
Documentado: ERR_INVALID_STATE = 0x05
Observado:   ERROR CODE DEC: 5
             ERROR CODE HEX: 0x05
Resultado:   coincide
```

```text
Documentado: el dispositivo debe recuperarse
Observado:   RX: STOPPED
Resultado:   coincide
```

```text
Documentado: GET_STATS debe responder
Observado:   STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0,
             rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0,
             ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
Resultado:   coincide
```

---

# 14. Resultado funcional validado

La prueba permite afirmar, con evidencia experimental directa, que en el DUT evaluado:

```text
CatSniffer V3
CC1352P7 ejecutando FeralRF
Cat-Bridge COM88
PHY IEEE 802.15.4
canal 25
```

al entrar en RX mediante:

```python
r.start_rx()
```

y solicitar posteriormente:

```python
r.transmit(b"\x01", power_dbm=-20)
```

la transmisión es rechazada mediante:

```text
CommandError('Transmit failed')
```

con:

```text
error_code = 5
error_code = 0x05
```

Después del rechazo:

```python
r.stop_rx()
```

funciona correctamente y:

```python
r.get_stats()
```

devuelve:

```text
DeviceStats(
    rx_ok=0,
    rx_crc_err=0,
    rx_drop=0,
    rx_overflow=0,
    ll_kind_unknown=0,
    ll_kind_adv=0,
    ll_kind_scan=0,
    ll_kind_connect=0,
    ll_kind_data=0
)
```

sin requerir recuperación externa del hardware.

---

# 15. Alcance exacto de la validación

EV-06 valida específicamente:

```text
RX activo
    +
TX_RAW solicitado
    →
rechazo por error 0x05
    →
recuperación del camino de control
```

La prueba demuestra que la política de exclusión RX/TX documentada se reproduce en el hardware evaluado.

También demuestra que la API Python utilizada conserva el código de error mediante:

```python
CommandError.error_code
```

y que `Radio.transmit()` obtiene dicho código del primer byte del payload de `Response.ERROR`.

---

# 16. Límites de la prueba

EV-06 no debe interpretarse como validación de aspectos que no fueron medidos.

Esta prueba no demuestra:

```text
TX IEEE 802.15.4 exitoso
potencia RF real de -20 dBm
precisión de potencia
calidad de modulación
Packet Error Rate
recepción IEEE 802.15.4
otros PHY
todos los conflictos posibles entre estados RF
comportamiento multihilo
comportamiento prolongado
```

Tampoco hubo un segundo receptor RF o analizador de espectro observando el canal durante el intento rechazado.

Por ello, el experimento demuestra que:

```text
la solicitud TX_RAW fue rechazada por el firmware
```

pero no constituye por sí solo una medición física que demuestre ausencia absoluta de cualquier emisión RF.

La afirmación que sí está sustentada es:

> Con RX activo, FeralRF rechazó la solicitud `TX_RAW` mediante el código `0x05` y permaneció operativo después del rechazo.

---

# 17. Incidencias encontradas durante la reproducción

Durante la construcción de la prueba aparecieron tres observaciones metodológicas que deben conservarse separadas del resultado del DUT.

## 17.1 PowerShell `ParserError`

El primer intento falló por quoting complejo dentro de `python -c`.

No hubo interacción con el DUT.

## 17.2 `Phy` vs `PHY`

El segundo intento utilizó un nombre incorrecto para la enumeración exportada.

La API real utiliza:

```python
PHY
```

No hubo interacción con el DUT.

## 17.3 `code` vs `error_code`

La primera prueba válida consultó:

```python
getattr(e, "code", None)
```

y obtuvo:

```text
ERROR CODE: None
```

La inspección del código mostró que el atributo correcto es:

```python
e.error_code
```

La repetición con dicho atributo devolvió:

```text
ERROR CODE DEC: 5
ERROR CODE HEX: 0x05
```

Por tanto, esta discrepancia correspondió al código de validación, no al firmware ni a la API.

---

# 18. Estado final

## EV-06 — `PASS`

La prueba reprodujo el comportamiento documentado.

Se observó experimentalmente la siguiente secuencia completa:

```text
INIT correcto
    ↓
PHY IEEE 802.15.4
    ↓
canal 25
    ↓
RX activo
    ↓
TX_RAW solicitado:
b"\x01" @ -20 dBm
    ↓
TX rechazado
    ↓
CommandError('Transmit failed')
    ↓
error_code = 5
    ↓
error_code = 0x05
    ↓
RX_STOP correcto
    ↓
GET_STATS correcto
```

La respuesta final de `GET_STATS` fue exactamente:

```text
DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

No se requirió power-cycle, reset, reconexión USB ni recuperación manual.

La conclusión consolidada para el registro experimental es:

> **En la CatSniffer V3 evaluada, con FeralRF ejecutándose sobre el CC1352P7 y utilizando COM88 como Cat-Bridge, se reprodujo satisfactoriamente el comportamiento documentado de exclusión RX/TX. Después de configurar IEEE 802.15.4 en canal 25 e iniciar RX, una solicitud `TX_RAW` realizada mediante `r.transmit(b"\x01", power_dbm=-20)` fue rechazada con `CommandError('Transmit failed')` y `error_code=5`, equivalente a `0x05`, cumpliendo el `ERR_INVALID_STATE` esperado. Después del rechazo, `r.stop_rx()` respondió correctamente y `r.get_stats()` devolvió `DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)`, sin reset ni reconexión. EV-06 queda por tanto validada como `PASS` bajo el criterio `ERROR 0x05 + recovery` definido en la documentación actual.**

````
