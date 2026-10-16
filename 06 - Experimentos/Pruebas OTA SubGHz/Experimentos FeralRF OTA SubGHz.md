Claro. Yo te indicaré **exactamente qué comandos ejecutar, en qué orden y qué resultado esperamos**. No necesitas interpretar la guía por tu cuenta.

Trabajaremos **un paso a la vez**. Tú ejecutarás los comandos en tu PC y me enviarás la salida completa. Yo la revisaré antes de indicarte el siguiente paso.

## Paso 1 — Verificar el entorno de trabajo

**Objetivo:** confirmar que estamos usando la versión correcta de FeralRF antes de comunicarnos con los CatSniffer.

### 1. Abre PowerShell en Windows

Copia y pega **todo este bloque**:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF'

$env:PYTHONPATH = (Resolve-Path '.\python').Path

git branch --show-current

git rev-parse HEAD

git status --short

python --version
```

### 2. ¿Qué esperamos encontrar?

| Comprobación      | Resultado esperado                                           |
| ----------------- | ------------------------------------------------------------ |
| Rama de Git       | `feature/hardware-validation`                                |
| Commit            | `ba27fbeb8c5683a31cca4339982cadeaaa406444`                   |
| Versión de Python | `Python 3.14.7`                                              |
| Estado de Git     | Puede mostrar archivos modificados o nuevos; los revisaremos |

**Importante:** este paso no transmite señales de radio, no reinicia los CatSniffer y no modifica su firmware.

### 3. ¿Qué necesito de ti?

Respóndeme con:

1. **La salida completa de PowerShell**, incluyendo cualquier error.
2. Confírmame si tienes **los dos CatSniffer conectados por USB y con sus antenas instaladas**.
3. Confírmame si estás autorizado a realizar las pruebas de transmisión en **902.2 MHz** en tu entorno de laboratorio.

**No ejecutes todavía comandos `boot`, `exit`, `band1`, `band2` ni comandos de transmisión.**

Cuando me envíes estos resultados, te daré el **Paso 2 con los comandos exactos**, sin saltarnos ninguna verificación.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF'
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF>
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> $env:PYTHONPATH = (Resolve-Path '.\python').Path
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF>
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> git branch --show-current
feature/hardware-validation
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF>
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> git rev-parse HEAD
ba27fbeb8c5683a31cca4339982cadeaaa406444
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF>
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> git status --short
?? MANUAL_FERALRF_SUBGHZ.md
?? SUBGHZ_PROVENANCE_AUDIT.md
?? SUBGHZ_RECOVERY_AUDIT.md
?? SUBGHZ_RECOVERY_RUN.md
?? experimental_artifacts/
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF>
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python --version
Python 3.14.7

---

## Paso 1 — Completado correctamente ✅

Revisé la salida de PowerShell. **Las tres comprobaciones principales coinciden con el entorno esperado.**

| Comprobación             | Resultado observado                        | Estado                  |
| ------------------------ | ------------------------------------------ | ----------------------- |
| Rama Git                 | `feature/hardware-validation`              | ✅ Correcto              |
| Commit                   | `ba27fbeb8c5683a31cca4339982cadeaaa406444` | ✅ Correcto              |
| Python                   | `3.14.7`                                   | ✅ Correcto              |
| Archivos sin seguimiento | 5 entradas                                 | ℹ️ No impiden continuar |

Los archivos marcados con `??` no representan por sí mismos un error. **No los borres ni ejecutes `git clean`.**

**Conclusión:** podemos continuar con la preparación de las pruebas, pero todavía no hemos comprobado la comunicación con los CatSniffer ni realizado ninguna transmisión RF.

---

## Paso 2 — Identificar los puertos COM

Antes de enviar comandos a los dispositivos, necesitamos confirmar qué puertos COM tienen asignados actualmente.

### Ejecuta este comando en la misma ventana de PowerShell

```powershell
Get-CimInstance Win32_PnPEntity |
    Where-Object { $_.Name -match '\(COM\d+\)' } |
    Select-Object Name, DeviceID |
    Format-List
```

### ¿Qué esperamos?

En la validación anterior se utilizaron estos puertos:

| Dispositivo        | Bridge | Shell |
| ------------------ | ------ | ----- |
| CatSniffer #2 — TX | COM33  | COM35 |
| CatSniffer #3 — RX | COM88  | COM87 |

**Estos valores son históricos.** Debemos comprobar que Windows sigue asignando los mismos puertos.

El comando únicamente consulta los dispositivos conectados. No reinicia hardware, no modifica configuraciones y no transmite RF.

### Envíame dos cosas

1. **La salida completa del comando.**
2. Confirma:

   * ¿Están conectados los dos CatSniffer por USB?
   * ¿Ambos tienen sus antenas instaladas y adecuadas para Sub-GHz?
   * ¿Tienes autorización para realizar las pruebas de transmisión a 902.2 MHz en tu laboratorio?

Con esa información identificaremos los puertos y pasaremos a comprobar la comunicación con cada dispositivo.

**Estado actual: Paso 1 PASS — Paso 2 pendiente.**

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> Get-CimInstance Win32_PnPEntity |
>>     Where-Object { $_.Name -match '\(COM\d+\)' } |
>>     Select-Object Name, DeviceID |
>>     Format-List


Name     : USB Serial Device (COM34)
DeviceID : USB\VID_1209&PID_BABB&MI_02\6&1E70E7AF&0&0002

Name     : USB Serial Device (COM38)
DeviceID : USB\VID_1209&PID_BABB&MI_00\6&290E3DD9&0&0000

Name     : USB Serial Device (COM88)
DeviceID : USB\VID_1209&PID_BABB&MI_00\6&106D987F&0&0000

Name     : Communications Port (COM1)
DeviceID : ACPI\PNP0501\0

Name     : USB Serial Device (COM35)
DeviceID : USB\VID_1209&PID_BABB&MI_04\6&1E70E7AF&0&0004

Name     : USB Serial Device (COM36)
DeviceID : USB\VID_1209&PID_BABB&MI_02\6&290E3DD9&0&0002

Name     : USB Serial Device (COM86)
DeviceID : USB\VID_1209&PID_BABB&MI_02\6&106D987F&0&0002

Name     : USB Serial Device (COM33)
DeviceID : USB\VID_1209&PID_BABB&MI_00\6&1E70E7AF&0&0000

Name     : USB Serial Device (COM37)
DeviceID : USB\VID_1209&PID_BABB&MI_04\6&290E3DD9&0&0004

Name     : USB Serial Device (COM87)
DeviceID : USB\VID_1209&PID_BABB&MI_04\6&106D987F&0&0004

---

## Paso 2 — Completado correctamente ✅

Windows reconoce los puertos que utilizamos en la validación anterior.

| Dispositivo        | Bridge  | Shell   | Estado      |
| ------------------ | ------- | ------- | ----------- |
| CatSniffer #2 — TX | `COM33` | `COM35` | ✅ Detectado |
| CatSniffer #3 — RX | `COM88` | `COM87` | ✅ Detectado |

También aparecen `COM34`, `COM36`, `COM37`, `COM38` y `COM86`, correspondientes a otras interfaces o dispositivos USB.

**Importante:** la correspondencia entre los puertos y CatSniffer #2/#3 coincide con el registro histórico, pero todavía no hemos verificado la identidad física de cada dispositivo en esta sesión.

---

## Paso 3 — Consultar el estado de los dos CatSniffer

**Objetivo:** comprobar que podemos comunicarnos con las interfaces Shell de ambos dispositivos.

Esta prueba es de **solo lectura**: no reinicia los CC1352, no cambia la banda y no transmite RF.

### Ejecuta este bloque completo en PowerShell

```powershell
@'
import serial
import time

devices = [
    ("CatSniffer #2 - TX", "COM35"),
    ("CatSniffer #3 - RX", "COM87"),
]

for name, port in devices:
    print(f"\n===== {name} ({port}) =====")

    try:
        with serial.Serial(port, 115200, timeout=1) as ser:
            time.sleep(0.3)
            ser.reset_input_buffer()
            ser.write(b"\r\n")
            ser.flush()
            time.sleep(0.7)

            response = ser.read_all().decode(
                "utf-8", errors="replace"
            )

            print(response if response.strip() else "(Sin respuesta)")

    except Exception as exc:
        print(f"ERROR: {type(exc).__name__}: {exc}")
'@ | python -
```

### ¿Qué esperamos observar?

Idealmente, cada dispositivo debe responder con información de su firmware o presentar el prompt de su Shell.

En el registro anterior, ambos reportaron:

* Firmware RP2040: `v3.1.0.1`.
* Firmware CC1352: `feralrf_cc1352 (custom)`.
* Banda reportada: `Band: 0`.

Esos son **resultados históricos**, no resultados confirmados de la sesión actual.

### Envíame la salida completa

Si aparece `Access is denied`, `Sin respuesta` o cualquier otro error, **no intentes solucionarlo todavía**. Revisaremos el resultado antes de continuar.

También sigue pendiente tu confirmación de que ambos dispositivos tienen antenas Sub-GHz adecuadas y que puedes transmitir legalmente a 902.2 MHz.

**Estado actual:** Paso 1 ✅ · Paso 2 ✅ · Paso 3 pendiente.

Todavía no hemos realizado ninguna transmisión RF.

---

===== CatSniffer #2 - TX (COM35) =====
(Sin respuesta)

===== CatSniffer #3 - RX (COM87) =====
(Sin respuesta)

---

## Paso 3 — Resultado inconcluso ⚠️

Los dos puertos Shell se abrieron sin errores, pero **ninguno devolvió texto**:

| Dispositivo        | Puerto  | Resultado     |
| ------------------ | ------- | ------------- |
| CatSniffer #2 — TX | `COM35` | Sin respuesta |
| CatSniffer #3 — RX | `COM87` | Sin respuesta |

Esto **no demuestra que exista un problema con los CatSniffer**.

El comando anterior únicamente envió un salto de línea (`Enter`). Es posible que el firmware no responda a esa entrada y que necesitemos enviar un comando explícito.

**No vamos a reiniciar ni cambiar la configuración todavía.**

## Paso 3A — Consultar el estado con `status`

Vamos a enviar el comando `status` a cada interfaz Shell.

### Copia y ejecuta este bloque completo en PowerShell

```powershell
@'
import serial
import time

devices = [
    ("CatSniffer #2 - TX", "COM35"),
    ("CatSniffer #3 - RX", "COM87"),
]

for name, port in devices:
    print(f"\n===== {name} ({port}) =====")

    try:
        with serial.Serial(port, 115200, timeout=1) as ser:
            time.sleep(0.5)
            ser.reset_input_buffer()

            ser.write(b"status\r\n")
            ser.flush()

            time.sleep(1.0)

            response = ser.read_all().decode(
                "utf-8", errors="replace"
            )

            print(response if response.strip() else "(Sin respuesta)")

    except Exception as exc:
        print(f"ERROR: {type(exc).__name__}: {exc}")
'@ | python -
```

### ¿Qué queremos comprobar?

Que ambos dispositivos respondan al comando `status` y podamos identificar su firmware y estado actual.

Este comando es de consulta: **no debería modificar la configuración ni transmitir por radio**.

### Qué necesito ahora

Envíame **la salida completa** de este nuevo bloque.

Si vuelve a aparecer `(Sin respuesta)`, detendremos este paso y revisaremos cómo funciona la Shell antes de enviar otros comandos.

**Estado de la campaña:** Paso 1 ✅ · Paso 2 ✅ · Paso 3 ⚠️ En diagnóstico.

---

===== CatSniffer #2 - TX (COM35) =====
Mode: 0, Band: 0, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes



===== CatSniffer #3 - RX (COM87) =====
Mode: 0, Band: 0, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes

---

## Paso 3 — Completado correctamente ✅

¡Bien! **Ambos CatSniffer respondieron al comando `status`.** Ya confirmamos que podemos comunicarnos con sus interfaces Shell.

### Resultados observados

| Parámetro                 | CatSniffer #2 — TX        | CatSniffer #3 — RX        |
| ------------------------- | ------------------------- | ------------------------- |
| Puerto Shell              | `COM35`                   | `COM87`                   |
| Firmware RP2040           | `v3.1.0.1`                | `v3.1.0.1`                |
| Firmware CC1352 reportado | `feralrf_cc1352 (custom)` | `feralrf_cc1352 (custom)` |
| Mode                      | `0`                       | `0`                       |
| Band                      | `0`                       | `0`                       |
| Radio                     | `LoRa`                    | `LoRa`                    |
| LoRa                      | `initialized`             | `initialized`             |
| UART overrun              | `0`                       | `0`                       |
| Ring dropped              | `0 bytes`                 | `0 bytes`                 |

**Interpretación:** la comunicación PC → RP2040 Shell funciona en ambos dispositivos. Sin embargo, `Band: 0` no demuestra qué ruta RF está seleccionada físicamente, y el nombre del firmware CC1352 no confirma por sí solo que su comunicación esté funcionando correctamente.

Todavía **no hemos validado la comunicación entre los dos CatSniffer por radio**.

---

## Paso 4 — Confirmar las condiciones físicas antes de preparar Sub-GHz

Antes de ejecutar comandos que reinicien el CC1352 o cambien la banda, necesito confirmar tres condiciones.

**Respóndeme estas tres preguntas:**

1. **Antenas:** ¿Los dos CatSniffer tienen instalada una antena adecuada para 902–928 MHz?
2. **Autorización:** ¿Puedes realizar transmisiones de prueba a **902.2 MHz** en tu entorno de laboratorio, respetando las restricciones locales aplicables?
3. **Identificación física:** ¿Confirmas que el CatSniffer conectado a `COM33/COM35` es el **#2 (TX)** y el conectado a `COM88/COM87` es el **#3 (RX)**?

Por ahora, **no ejecutes ningún comando adicional**.

Una vez confirmadas estas condiciones, te proporcionaré el siguiente bloque de PowerShell para preparar los dispositivos, siguiendo la secuencia de recuperación documentada.

**Estado de la campaña:** Paso 1 ✅ · Paso 2 ✅ · Paso 3 ✅ · Paso 4 pendiente.

---

Condiciones confirmadas

---

## Paso 4 — Preparar los CC1352 mediante reinicio controlado

Perfecto. Quedan confirmadas las condiciones físicas, la identificación de los dispositivos y la autorización para las pruebas.

Ahora vamos a ejecutar la secuencia de reinicio documentada en `SUBGHZ_RECOVERY_RUN.md`, sección **Checkpoint 1 — Reset verification**.

**Objetivo:** colocar ambos CC1352 en un estado inicial controlado antes de seleccionar la banda Sub-GHz.

Este procedimiento utiliza la interfaz **Shell del RP2040**, que controla el reinicio del CC1352.

| Dispositivo        | Puerto Shell |
| ------------------ | ------------ |
| CatSniffer #2 — TX | `COM35`      |
| CatSniffer #3 — RX | `COM87`      |

### Ejecuta este bloque completo en PowerShell

No necesitas modificar nada.

```powershell
@'
import serial
import time

devices = [
    ("CatSniffer #2 - TX", "COM35"),
    ("CatSniffer #3 - RX", "COM87"),
]

for name, port in devices:
    print(f"\n===== {name} ({port}) =====")

    try:
        with serial.Serial(port, 115200, timeout=1) as ser:
            time.sleep(0.3)
            ser.reset_input_buffer()

            print("Enviando boot...")
            ser.write(b"boot\r\n")
            ser.flush()
            time.sleep(0.5)

            response_boot = ser.read_all().decode(
                "utf-8", errors="replace"
            )

            print("Enviando exit...")
            ser.write(b"exit\r\n")
            ser.flush()
            time.sleep(0.3)

            response_exit = ser.read_all().decode(
                "utf-8", errors="replace"
            )

            print("Esperando estabilizacion del CC1352...")
            time.sleep(3.5)

            print("Consultando status...")
            ser.write(b"status\r\n")
            ser.flush()
            time.sleep(1.0)

            response_status = ser.read_all().decode(
                "utf-8", errors="replace"
            )

            print("\n--- Respuesta boot ---")
            print(response_boot or "(Sin respuesta inmediata)")

            print("--- Respuesta exit ---")
            print(response_exit or "(Sin respuesta inmediata)")

            print("--- Estado final ---")
            print(response_status or "(Sin respuesta)")

    except Exception as exc:
        print(f"ERROR: {type(exc).__name__}: {exc}")
        print("Proceso detenido.")
        break
'@ | python -
```

### ¿Qué esperamos observar?

Para **cada CatSniffer**, esperamos encontrar estas respuestas, aunque pueden aparecer juntas después de `exit`:

```text
boot
BOOT

exit
PASSTHROUGH
```

Y después:

```text
Mode: 0, Band: 0, ...
FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
```

### Criterios para aprobar este paso

* Ambos dispositivos responden `BOOT` y `PASSTHROUGH`.
* Ambos responden al `status` posterior al reinicio.
* Ambos terminan con `Mode: 0`.
* No aparecen errores de acceso a los puertos.

**Límite de esta comprobación:** estas respuestas verifican la ruta de control del reinicio desde el RP2040, pero no constituyen una medición eléctrica del pin RESET del CC1352.

### ¿Qué sigue?

**Envíame la salida completa del bloque.**

No ejecutes todavía `band2` ni comandos de transmisión. Primero comprobaré que ambos reinicios fueron correctos.

**Estado:** entorno ✅ · puertos ✅ · Shell ✅ · reinicio CC1352 ⏳.

---

===== CatSniffer #2 - TX (COM35) =====
Enviando boot...
Enviando exit...
Esperando estabilizacion del CC1352...
Consultando status...

--- Respuesta boot ---
(Sin respuesta inmediata)
--- Respuesta exit ---
BOOT

PASSTHROUGH


--- Estado final ---
Mode: 0, Band: 0, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes



===== CatSniffer #3 - RX (COM87) =====
Enviando boot...
Enviando exit...
Esperando estabilizacion del CC1352...
Consultando status...

--- Respuesta boot ---
(Sin respuesta inmediata)
--- Respuesta exit ---
BOOT

PASSTHROUGH


--- Estado final ---
Mode: 0, Band: 0, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes

---

## Paso 4 — Reinicio controlado completado ✅

Los dos CatSniffer respondieron correctamente a la secuencia `boot` → `exit` → `status`.

| Comprobación            | CatSniffer #2 (TX) | CatSniffer #3 (RX) |
| ----------------------- | ------------------ | ------------------ |
| Respuesta `BOOT`        | ✅                  | ✅                  |
| Respuesta `PASSTHROUGH` | ✅                  | ✅                  |
| Respuesta a `status`    | ✅                  | ✅                  |
| `Mode: 0`               | ✅                  | ✅                  |
| `Band: 0`               | ✅                  | ✅                  |
| Errores UART reportados | 0                  | 0                  |

**Clasificación: PASS — control de reinicio.**

Esto confirma que la secuencia de control responde como esperamos. No demuestra por sí solo que hayamos medido eléctricamente el reinicio del CC1352.

---

## Paso 5 — Seleccionar la banda Sub-GHz

Ahora indicaremos al RP2040 de cada CatSniffer que seleccione la ruta Sub-GHz.

**Objetivo:** ejecutar `band2` en ambos dispositivos y comprobar que el estado reportado cambia de `Band: 0` a `Band: 1`.

Seguiremos el procedimiento registrado en `SUBGHZ_RECOVERY_RUN.md`, sección *Checkpoint 2 — RF switch / band control*.

### Ejecuta este bloque completo en PowerShell

```powershell
@'
import serial
import time

devices = [
    ("CatSniffer #2 - TX", "COM35"),
    ("CatSniffer #3 - RX", "COM87"),
]

for name, port in devices:
    print(f"\n===== {name} ({port}) =====")

    try:
        with serial.Serial(port, 115200, timeout=1) as ser:
            time.sleep(0.3)
            ser.reset_input_buffer()

            print("Seleccionando Sub-GHz...")
            ser.write(b"band2\r\n")
            ser.flush()
            time.sleep(0.8)

            response_band = ser.read_all().decode(
                "utf-8", errors="replace"
            )

            print("\n--- Respuesta band2 ---")
            print(response_band or "(Sin respuesta)")

            print("Consultando status...")
            ser.write(b"status\r\n")
            ser.flush()
            time.sleep(1.0)

            response_status = ser.read_all().decode(
                "utf-8", errors="replace"
            )

            print("\n--- Estado final ---")
            print(response_status or "(Sin respuesta)")

            if "SUB-GHz Band" not in response_band:
                print("ADVERTENCIA: band2 no fue confirmado.")
                print("Proceso detenido.")
                break

            if "Band: 1" not in response_status:
                print("ADVERTENCIA: el estado Sub-GHz no fue confirmado.")
                print("Proceso detenido.")
                break

    except Exception as exc:
        print(f"ERROR: {type(exc).__name__}: {exc}")
        print("Proceso detenido.")
        break
'@ | python -
```

### Resultado esperado

Para cada dispositivo esperamos:

**Respuesta de `band2`:**

```text
band2
SUB-GHz Band
```

**Respuesta de `status`:**

```text
Mode: 0, Band: 1, Radio: LoRa, ...
FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
```

### ¿Qué demostraría este resultado?

Si ambos dispositivos responden correctamente, habremos confirmado:

* Que el RP2040 acepta la solicitud de selección Sub-GHz.
* Que ambos reportan el estado lógico `Band: 1`.

**Lo que todavía no demuestra:** que el RF switch RFSW8006 haya cambiado físicamente de posición. Para demostrarlo directamente necesitaríamos mediciones eléctricas o una comprobación independiente de la ruta RF.

Este paso **no transmite señales de radio**.

---

### Qué necesito ahora

Envíame **la salida completa del bloque**.

Después evaluaremos si podemos avanzar a la prueba OTA conocida a **902.2 MHz**, en la que el CatSniffer #2 transmitirá y el CatSniffer #3 intentará recibir los paquetes.

**Estado:** entorno ✅ · puertos ✅ · Shell ✅ · reinicio ✅ · selección Sub-GHz ⏳.

---

===== CatSniffer #2 - TX (COM35) =====
Seleccionando Sub-GHz...

--- Respuesta band2 ---
SUB-GHz Band


Consultando status...

--- Estado final ---
Mode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes



===== CatSniffer #3 - RX (COM87) =====
Seleccionando Sub-GHz...

--- Respuesta band2 ---
SUB-GHz Band


Consultando status...

--- Estado final ---
Mode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes

---

## Paso 5 — Selección Sub-GHz completada ✅

Los dos CatSniffer respondieron correctamente al comando `band2`.

| Comprobación               | CatSniffer #2 — TX | CatSniffer #3 — RX |
| -------------------------- | ------------------ | ------------------ |
| Respuesta `SUB-GHz Band`   | ✅                  | ✅                  |
| Estado `Band: 1`           | ✅                  | ✅                  |
| Firmware RP2040 `v3.1.0.1` | ✅                  | ✅                  |
| Errores UART reportados    | 0                  | 0                  |

**Clasificación: PASS — control lógico de banda.**

Esto confirma que ambos RP2040 aceptaron la selección Sub-GHz. Sin embargo, todavía no demuestra eléctricamente que el RF switch `RFSW8006` haya seleccionado la ruta correcta.

### Antes de transmitir a 902.2 MHz

El procedimiento de `SUBGHZ_RECOVERY_RUN.md`, sección *Checkpoint 2*, contempla medir las señales `CTF1`, `CTF2` y `CTF3` para verificar el control físico del RF switch.

La combinación esperada después de `band2` es:

| Señal  | Valor esperado               |
| ------ | ---------------------------- |
| `CTF1` | LOW (aproximadamente 0 V)    |
| `CTF2` | LOW (aproximadamente 0 V)    |
| `CTF3` | HIGH (aproximadamente 3.3 V) |

Estas mediciones requieren acceder a puntos pequeños de la PCB. **No recomiendo intentar medirlos sin herramientas adecuadas**, porque existe riesgo de provocar un cortocircuito.

Podemos realizar una prueba OTA funcional sin esas mediciones, pero debemos dejar claro que comprobaríamos la recepción de paquetes, no directamente el estado eléctrico del RF switch.

### Necesito que elijas cómo continuar

**Opción A — Continuar con la prueba OTA (recomendada si no tienes equipo de medición adecuado).**

Omitimos la medición eléctrica, registramos esa limitación y ejecutamos el control conocido de **902.2 MHz**:

* TX: CatSniffer #2 (`COM33`).
* RX: CatSniffer #3 (`COM88`).
* Preset: `wisun_915_fsk_50k`.
* Potencia solicitada: `0 dBm`.
* Diez paquetes con marcador `DE AD BE EF`.
* Criterio de éxito: recepción de los diez paquetes con CRC válido.

El nombre del preset no implica que estemos validando interoperabilidad Wi-SUN; únicamente utilizaremos sus parámetros PHY.

**Opción B — Verificar primero las señales eléctricas.**

Te guiaré para localizar los puntos de medición y registrar los voltajes, sin transmitir.

**¿Continuamos con la opción A o B?**

No ejecutes todavía más comandos. Una vez que elijas, te proporcionaré el siguiente bloque de PowerShell.

**Estado:** entorno ✅ · puertos ✅ · Shell ✅ · reinicio ✅ · selección lógica Sub-GHz ✅ · validación RF pendiente.

---

Perfecto. Continuamos con la **opción A: prueba OTA a 902.2 MHz**, sin mediciones eléctricas del RF switch.

## Paso 6 — Comprobar que tenemos el programa de prueba

Ya completamos las condiciones previas:

| Comprobación              | Resultado            |
| ------------------------- | -------------------- |
| Entorno de Python y Git   | ✅                    |
| Puertos COM identificados | ✅                    |
| Comunicación Shell        | ✅                    |
| Secuencia `boot` → `exit` | ✅                    |
| Selección lógica `band2`  | ✅                    |
| Antenas y autorización    | ✅ Confirmadas por ti |

Ahora utilizaremos el procedimiento del **Checkpoint 3A** documentado en `SUBGHZ_RECOVERY_RUN.md`.

En esa ejecución anterior se utilizó un programa llamado `checkpoint_3a_single_run.py`, diseñado para una sola prueba controlada.

Antes de ejecutarlo, quiero verificar que está disponible en tu computadora. **No vamos a transmitir todavía.**

### Ejecuta este único comando en PowerShell

```powershell
Test-Path '.\experimental_artifacts\checkpoint_3a\checkpoint_3a_single_run.py'
```

### Resultado esperado

```text
True
```

Si aparece `True`, significa que el archivo existe y podremos revisar cómo invocarlo.

Si aparece `False`, buscaremos la alternativa documentada. No necesitas crear ni modificar archivos.

### ¿Qué hará posteriormente la prueba OTA?

Cuando hayamos verificado el programa, prepararemos una única ejecución con estos parámetros:

| Parámetro              | Valor                   |
| ---------------------- | ----------------------- |
| Transmisor             | CatSniffer #2 — `COM33` |
| Receptor               | CatSniffer #3 — `COM88` |
| PHY                    | `PROPRIETARY_GFSK`      |
| Preset                 | `wisun_915_fsk_50k`     |
| Frecuencia             | `902200000 Hz`          |
| Potencia TX solicitada | `0 dBm`                 |
| Datos transmitidos     | `DE AD BE EF`           |
| Cantidad               | 10 paquetes             |

**Criterio de éxito:** recibir en el CatSniffer #3 los diez paquetes con el marcador correcto y `crc_ok = true`.

No consideraremos suficiente que el transmisor responda `ACK`: necesitamos evidencia de recepción en el otro dispositivo.

**Envíame únicamente el resultado (`True` o `False`) del comando.** Con eso te daré la instrucción exacta para continuar, sin repetir reinicios ni cambiar bandas innecesariamente.

**Estado actual:** preparación completada ✅ · prueba OTA pendiente.

---

True

---

Perfecto. ✅ El resultado `True` confirma que **el archivo de prueba existe** en tu computadora.

Antes de transmitir, vamos a comprobar que el programa utiliza los puertos y parámetros correctos. Así evitamos ejecutar accidentalmente una configuración distinta.

## Paso 6A — Revisar el programa de prueba

**Objetivo:** verificar qué hará `checkpoint_3a_single_run.py` antes de ejecutarlo.

### Ejecuta este comando en PowerShell

```powershell
Get-Content '.\experimental_artifacts\checkpoint_3a\checkpoint_3a_single_run.py'
```

Este comando **solo muestra el contenido del archivo**. No ejecuta el programa, no cambia la configuración de los CatSniffer y no transmite RF.

### ¿Qué vamos a revisar?

Según `SUBGHZ_RECOVERY_RUN.md`, el programa utilizado anteriormente realiza una sola prueba con:

| Parámetro           | Valor esperado      |
| ------------------- | ------------------- |
| TX                  | `COM33`             |
| RX                  | `COM88`             |
| Frecuencia          | 902.2 MHz           |
| Preset              | `wisun_915_fsk_50k` |
| Potencia solicitada | 0 dBm               |
| Paquetes            | 10                  |
| Marcador            | `DE AD BE EF`       |

También comprobaremos que no incluya reinicios automáticos, cambios de banda inesperados ni repeticiones de la prueba.

### ¿Qué necesito ahora?

**Copia aquí toda la salida de `Get-Content`.**

La revisaré y, si coincide con las condiciones acordadas, te daré el comando exacto para ejecutar la prueba OTA.

**Estado:** preparación Sub-GHz completada ✅ · programa localizado ✅ · revisión previa a transmisión pendiente ⏳.

---

"""Checkpoint 3A only: one controlled wisun_915_fsk_50k OTA execution.

Experimental helper. Fixed explicit Cat-Bridge ports, one preset, one direction,
ten TX attempts, and no automatic test retry. Shell reset and band control are
performed and recorded separately before this helper is invoked.
"""

from __future__ import annotations

import sys
import time
import traceback
from datetime import datetime, timezone
from pathlib import Path


REPO_ROOT = Path(__file__).resolve().parents[2]
sys.path.insert(0, str(REPO_ROOT / "python"))

from feralrf import PHY, PROP_PRESETS, Radio, RxStreamError  # noqa: E402


TX_PORT = "COM33"
RX_PORT = "COM88"
PRESET_NAME = "wisun_915_fsk_50k"
MARKER = bytes.fromhex("DE AD BE EF")
TX_POWER_DBM = 0
TX_COUNT = 10


def utc_now() -> str:
    return datetime.now(timezone.utc).isoformat().replace("+00:00", "Z")


def emit(event: str, **fields: object) -> None:
    rendered = " ".join(f"{key}={value!r}" for key, value in fields.items())
    print(f"{utc_now()} {event}" + (f" {rendered}" if rendered else ""), flush=True)


def main() -> int:
    preset = PROP_PRESETS[PRESET_NAME].copy()
    emit(
        "TEST_BEGIN",
        checkpoint="3A",
        tx_port=TX_PORT,
        rx_port=RX_PORT,
        preset_name=PRESET_NAME,
        preset=preset,
        marker=MARKER.hex(" ").upper(),
        tx_power_dbm=TX_POWER_DBM,
        tx_count=TX_COUNT,
    )

    tx = Radio(port=TX_PORT, baudrate=921600)
    rx = Radio(port=RX_PORT, baudrate=921600)
    rx_started = False
    tx_accepted = 0
    received = []
    rx_async_errors = []
    tx_async_errors = []
    fatal: BaseException | None = None

    try:
        emit("TX_INIT_BEGIN", port=TX_PORT)
        tx_info = tx.init()
        emit("TX_INIT_OK", info=tx_info)

        emit("RX_INIT_BEGIN", port=RX_PORT)
        rx_info = rx.init()
        emit("RX_INIT_OK", info=rx_info)

        emit("TX_SET_PHY_BEGIN", phy="PROPRIETARY_GFSK", channel=0)
        tx.set_phy(PHY.PROPRIETARY_GFSK, channel=0)
        emit("TX_SET_PHY_ACK")

        emit("RX_SET_PHY_BEGIN", phy="PROPRIETARY_GFSK", channel=0)
        rx.set_phy(PHY.PROPRIETARY_GFSK, channel=0)
        emit("RX_SET_PHY_ACK")

        emit("TX_CONFIGURE_PROP_BEGIN", preset=preset)
        tx.configure_prop(**preset)
        emit("TX_CONFIGURE_PROP_ACK")

        emit("RX_CONFIGURE_PROP_BEGIN", preset=preset)
        rx.configure_prop(**preset)
        emit("RX_CONFIGURE_PROP_ACK")

        emit("TX_SET_POWER_BEGIN", power_dbm=TX_POWER_DBM)
        tx.set_power(TX_POWER_DBM)
        emit("TX_SET_POWER_ACK")

        emit("TX_STATS_BEFORE_BEGIN")
        tx_stats_before = tx.get_stats()
        emit("TX_STATS_BEFORE", stats=tx_stats_before)

        emit("RX_STATS_BEFORE_BEGIN")
        rx_stats_before = rx.get_stats()
        emit("RX_STATS_BEFORE", stats=rx_stats_before)

        emit("RX_START_BEGIN")
        rx.start_rx()
        rx_started = True
        emit("RX_START_ACK")

        delay_start = time.monotonic()
        emit("PRE_TX_WAIT_BEGIN", requested_seconds=0.3)
        time.sleep(0.3)
        emit("PRE_TX_WAIT_COMPLETE", elapsed_seconds=time.monotonic() - delay_start)

        for index in range(1, TX_COUNT + 1):
            emit("TX_ATTEMPT_BEGIN", index=index, payload=MARKER.hex(" ").upper())
            try:
                result = tx.transmit(MARKER, power_dbm=TX_POWER_DBM)
            except Exception as exc:
                emit(
                    "TX_ATTEMPT_EXCEPTION",
                    index=index,
                    exception_type=type(exc).__name__,
                    exception=repr(exc),
                )
            else:
                tx_accepted += 1
                emit("TX_ATTEMPT_ACK", index=index, return_value=result)

            spacing_start = time.monotonic()
            time.sleep(0.1)
            emit(
                "TX_SPACING_COMPLETE",
                index=index,
                elapsed_seconds=time.monotonic() - spacing_start,
            )

        post_tx_start = time.monotonic()
        emit("POST_TX_WAIT_BEGIN", requested_seconds=1.0)
        time.sleep(1.0)
        emit("POST_TX_WAIT_COMPLETE", elapsed_seconds=time.monotonic() - post_tx_start)

        rx_window_start = time.monotonic()
        emit("RX_OBSERVATION_BEGIN", requested_seconds=2.0)
        for item in rx.read_packets(timeout=2.0):
            if isinstance(item, RxStreamError):
                rx_async_errors.append(item)
                emit(
                    "RX_ASYNC_ERROR",
                    error_code=item.error_code,
                    context=item.context,
                )
                continue

            received.append(item)
            emit(
                "RX_PACKET",
                ordinal=len(received),
                device_timestamp_us=item.timestamp_us,
                channel=item.channel,
                rssi_dbm=item.rssi_dbm,
                lqi=item.lqi,
                crc_ok=item.crc_ok,
                payload_hex=item.data.hex(" ").upper(),
                marker_present=MARKER in item.data,
            )
        emit(
            "RX_OBSERVATION_COMPLETE",
            elapsed_seconds=time.monotonic() - rx_window_start,
            packets=len(received),
            async_errors=len(rx_async_errors),
        )

        tx_error_window_start = time.monotonic()
        emit("TX_ASYNC_OBSERVATION_BEGIN", requested_seconds=0.5)
        for item in tx.read_packets(timeout=0.5):
            if isinstance(item, RxStreamError):
                tx_async_errors.append(item)
                emit(
                    "TX_ASYNC_ERROR",
                    error_code=item.error_code,
                    context=item.context,
                )
            else:
                emit(
                    "TX_UNEXPECTED_RX_PACKET",
                    device_timestamp_us=item.timestamp_us,
                    channel=item.channel,
                    rssi_dbm=item.rssi_dbm,
                    lqi=item.lqi,
                    crc_ok=item.crc_ok,
                    payload_hex=item.data.hex(" ").upper(),
                )
        emit(
            "TX_ASYNC_OBSERVATION_COMPLETE",
            elapsed_seconds=time.monotonic() - tx_error_window_start,
            async_errors=len(tx_async_errors),
        )

        emit("RX_STOP_BEGIN")
        rx.stop_rx(retries=1, timeout=1.0)
        rx_started = False
        emit("RX_STOP_ACK")

        emit("TX_STATS_AFTER_BEGIN")
        tx_stats_after = tx.get_stats()
        emit("TX_STATS_AFTER", stats=tx_stats_after)

        emit("RX_STATS_AFTER_BEGIN")
        rx_stats_after = rx.get_stats()
        emit("RX_STATS_AFTER", stats=rx_stats_after)

    except BaseException as exc:
        fatal = exc
        emit("TEST_FATAL_EXCEPTION", exception_type=type(exc).__name__, exception=repr(exc))
        traceback.print_exc()
    finally:
        if rx_started:
            emit("RX_STOP_CLEANUP_BEGIN")
            try:
                rx.stop_rx(retries=1, timeout=1.0)
            except Exception as exc:
                emit(
                    "RX_STOP_CLEANUP_EXCEPTION",
                    exception_type=type(exc).__name__,
                    exception=repr(exc),
                )
            else:
                emit("RX_STOP_CLEANUP_ACK")

        emit("TX_DISCONNECT_BEGIN", port=TX_PORT)
        try:
            tx.disconnect()
        except Exception as exc:
            emit(
                "TX_DISCONNECT_EXCEPTION",
                exception_type=type(exc).__name__,
                exception=repr(exc),
            )
        else:
            emit("TX_DISCONNECT_COMPLETE", port=TX_PORT)

        emit("RX_DISCONNECT_BEGIN", port=RX_PORT)
        try:
            rx.disconnect()
        except Exception as exc:
            emit(
                "RX_DISCONNECT_EXCEPTION",
                exception_type=type(exc).__name__,
                exception=repr(exc),
            )
        else:
            emit("RX_DISCONNECT_COMPLETE", port=RX_PORT)

    crc_good_marker = sum(1 for pkt in received if pkt.crc_ok and MARKER in pkt.data)
    emit(
        "TEST_SUMMARY",
        tx_accepted=tx_accepted,
        tx_requested=TX_COUNT,
        total_peer_packets=len(received),
        peer_crc_good_marker=crc_good_marker,
        peer_expected=TX_COUNT,
        rx_async_errors=len(rx_async_errors),
        tx_async_errors=len(tx_async_errors),
        fatal_exception=None if fatal is None else repr(fatal),
    )
    emit("TEST_END", exit_code=0 if fatal is None else 1)
    return 0 if fatal is None else 1


if __name__ == "__main__":
    raise SystemExit(main())

---

## Paso 6A — Revisión del programa completada ✅

He revisado el contenido de `checkpoint_3a_single_run.py` que compartiste. **Coincide con la prueba OTA que queremos realizar.**

### Configuración verificada en el código

| Parámetro                           | Valor                   |
| ----------------------------------- | ----------------------- |
| Transmisor                          | CatSniffer #2 — `COM33` |
| Receptor                            | CatSniffer #3 — `COM88` |
| PHY                                 | `PROPRIETARY_GFSK`      |
| Preset                              | `wisun_915_fsk_50k`     |
| Frecuencia nominal del preset       | 902.2 MHz               |
| Potencia TX solicitada              | 0 dBm                   |
| Cantidad de transmisiones           | 10                      |
| Payload                             | `DE AD BE EF`           |
| Reintentos automáticos de la prueba | Ninguno                 |

El script configura ambos radios, inicia primero la recepción, realiza diez intentos de transmisión y después recopila los paquetes recibidos.

También intenta detener la recepción y desconectar los dispositivos al finalizar.

**Una precisión:** el script no contiene directamente el valor `902200000`. Lo obtiene de `PROP_PRESETS["wisun_915_fsk_50k"]`. El evento `TEST_BEGIN` imprimirá los parámetros efectivos del preset, lo que nos permitirá comprobar la frecuencia solicitada.

### Limitación importante

El programa transmite los diez paquetes antes de consultar los recibidos mediante `read_packets()`. Por tanto, el resultado depende de que la implementación conserve los eventos de recepción durante ese intervalo.

Además, que el script termine con `exit_code=0` **no significa necesariamente que la prueba RF haya sido exitosa**. Evaluaremos los paquetes recibidos y su CRC.

---

## Paso 7 — Ejecutar el control OTA a 902.2 MHz

**Objetivo:** comprobar si el CatSniffer #3 recibe correctamente los diez paquetes transmitidos por el CatSniffer #2.

**Condiciones iniciales confirmadas:**

* Ambos dispositivos conectados y con antenas adecuadas.
* TX identificado en `COM33`.
* RX identificado en `COM88`.
* Secuencia `boot` → `exit` completada.
* Ambos dispositivos reportan `Band: 1`.
* Autorización de transmisión confirmada.

### Ejecuta este único comando en PowerShell

```powershell
python -u '.\experimental_artifacts\checkpoint_3a\checkpoint_3a_single_run.py'
```

**No ejecutes el comando dos veces**, aunque el resultado sea inesperado.

### ¿Qué esperamos encontrar?

Durante la ejecución aparecerán eventos como:

```text
TEST_BEGIN
TX_INIT_OK
RX_INIT_OK
TX_SET_PHY_ACK
RX_SET_PHY_ACK
TX_CONFIGURE_PROP_ACK
RX_CONFIGURE_PROP_ACK
RX_START_ACK
TX_ATTEMPT_ACK
RX_PACKET
TEST_SUMMARY
TEST_END
```

El resultado que buscamos es:

```text
tx_accepted=10
tx_requested=10
total_peer_packets=10
peer_crc_good_marker=10
peer_expected=10
rx_async_errors=0
tx_async_errors=0
fatal_exception=None
```

**Criterio `PASS-RF`:** diez paquetes recibidos por el otro CatSniffer, con CRC válido y marcador `DE AD BE EF`.

Si aparecen diez `TX_ATTEMPT_ACK`, pero ningún `RX_PACKET`, **no consideraremos exitosa la transmisión RF**. Los ACK solamente confirman que las solicitudes fueron aceptadas por la interfaz correspondiente.

### Qué necesito de ti

Ejecuta el comando **una sola vez** y envíame la salida completa, desde `TEST_BEGIN` hasta `TEST_END`, incluyendo cualquier error.

Con esos datos clasificaremos el resultado como `PASS-RF`, `PARTIAL`, `FAIL`, `BLOCKED` o `INCONCLUSIVE`, y decidiremos si hace falta investigar algo.

**Estado actual:** preparación completada ✅ · control OTA 902.2 MHz listo para ejecutar ⏳.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\checkpoint_3a\checkpoint_3a_single_run.py'
2026-10-15T21:08:08.159021Z TEST_BEGIN checkpoint='3A' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_50k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='DE AD BE EF' tx_power_dbm=0 tx_count=10
2026-10-15T21:08:08.159163Z TX_INIT_BEGIN port='COM33'
2026-10-15T21:08:09.275321Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T21:08:09.275422Z RX_INIT_BEGIN port='COM88'
2026-10-15T21:08:10.421500Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T21:08:10.421707Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T21:08:10.454071Z TX_SET_PHY_ACK
2026-10-15T21:08:10.454166Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T21:08:10.485020Z RX_SET_PHY_ACK
2026-10-15T21:08:10.485126Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T21:08:10.518569Z TX_CONFIGURE_PROP_ACK
2026-10-15T21:08:10.518676Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T21:08:10.554109Z RX_CONFIGURE_PROP_ACK
2026-10-15T21:08:10.554203Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T21:08:10.586285Z TX_SET_POWER_ACK
2026-10-15T21:08:10.586394Z TX_STATS_BEFORE_BEGIN
2026-10-15T21:08:10.747251Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T21:08:10.747350Z RX_STATS_BEFORE_BEGIN
2026-10-15T21:08:10.911452Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T21:08:10.911549Z RX_START_BEGIN
2026-10-15T21:08:10.942729Z RX_START_ACK
2026-10-15T21:08:10.943051Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T21:08:11.243292Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30045109999991837
2026-10-15T21:08:11.243416Z TX_ATTEMPT_BEGIN index=1 payload='DE AD BE EF'
2026-10-15T21:08:11.277051Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T21:08:11.377445Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10022720000051777
2026-10-15T21:08:11.378007Z TX_ATTEMPT_BEGIN index=2 payload='DE AD BE EF'
2026-10-15T21:08:11.421607Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T21:08:11.522230Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10014990000127
2026-10-15T21:08:11.522362Z TX_ATTEMPT_BEGIN index=3 payload='DE AD BE EF'
2026-10-15T21:08:11.551832Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T21:08:11.652506Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10049989999970421
2026-10-15T21:08:11.653022Z TX_ATTEMPT_BEGIN index=4 payload='DE AD BE EF'
2026-10-15T21:08:11.695839Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T21:08:11.796565Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10024500000145053
2026-10-15T21:08:11.797081Z TX_ATTEMPT_BEGIN index=5 payload='DE AD BE EF'
2026-10-15T21:08:11.832192Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T21:08:11.932690Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10034610000002431
2026-10-15T21:08:11.933136Z TX_ATTEMPT_BEGIN index=6 payload='DE AD BE EF'
2026-10-15T21:08:11.971397Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T21:08:12.071971Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10044059999927413
2026-10-15T21:08:12.072321Z TX_ATTEMPT_BEGIN index=7 payload='DE AD BE EF'
2026-10-15T21:08:12.104687Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T21:08:12.205000Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.100185600000259
2026-10-15T21:08:12.205298Z TX_ATTEMPT_BEGIN index=8 payload='DE AD BE EF'
2026-10-15T21:08:12.236110Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T21:08:12.336471Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10021340000093915
2026-10-15T21:08:12.336895Z TX_ATTEMPT_BEGIN index=9 payload='DE AD BE EF'
2026-10-15T21:08:12.379827Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T21:08:12.480499Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10020640000220737
2026-10-15T21:08:12.481016Z TX_ATTEMPT_BEGIN index=10 payload='DE AD BE EF'
2026-10-15T21:08:12.516039Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T21:08:12.616425Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10022459999890998
2026-10-15T21:08:12.616883Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T21:08:13.617657Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0007561999991594
2026-10-15T21:08:13.618181Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T21:08:13.768547Z RX_PACKET ordinal=1 device_timestamp_us=915313 channel=37 rssi_dbm=-73 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:13.888360Z RX_PACKET ordinal=2 device_timestamp_us=915840 channel=37 rssi_dbm=14 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.006973Z RX_PACKET ordinal=3 device_timestamp_us=916401 channel=37 rssi_dbm=-83 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.123329Z RX_PACKET ordinal=4 device_timestamp_us=916914 channel=37 rssi_dbm=-56 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.238251Z RX_PACKET ordinal=5 device_timestamp_us=917476 channel=37 rssi_dbm=76 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.349436Z RX_PACKET ordinal=6 device_timestamp_us=918008 channel=37 rssi_dbm=-125 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.463389Z RX_PACKET ordinal=7 device_timestamp_us=918551 channel=37 rssi_dbm=26 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.575831Z RX_PACKET ordinal=8 device_timestamp_us=919070 channel=37 rssi_dbm=109 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.693774Z RX_PACKET ordinal=9 device_timestamp_us=919584 channel=37 rssi_dbm=88 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:14.812369Z RX_PACKET ordinal=10 device_timestamp_us=920148 channel=37 rssi_dbm=-121 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T21:08:15.622661Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.004462799999601 packets=10 async_errors=0
2026-10-15T21:08:15.623139Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T21:08:16.138875Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5157290000024659 async_errors=0
2026-10-15T21:08:16.139273Z RX_STOP_BEGIN
2026-10-15T21:08:16.174900Z RX_STOP_ACK
2026-10-15T21:08:16.174998Z TX_STATS_AFTER_BEGIN
2026-10-15T21:08:16.341787Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T21:08:16.341887Z RX_STATS_AFTER_BEGIN
2026-10-15T21:08:16.503325Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T21:08:16.503442Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T21:08:16.504296Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T21:08:16.504362Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T21:08:16.505067Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T21:08:16.505140Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T21:08:16.505192Z TEST_END exit_code=0

---

# Paso 7 — Control OTA a 902.2 MHz: PASS-RF ✅

**La prueba fue exitosa.** El CatSniffer #2 realizó diez solicitudes de transmisión y el CatSniffer #3 recibió diez paquetes con CRC válido y el marcador esperado `DE AD BE EF`.

Esta vez tenemos evidencia de recepción en el dispositivo remoto, no solamente respuestas `ACK` del transmisor.

## 1. Resultados observados

**Prueba:** Checkpoint 3A
**Fecha:** 15 de octubre de 2026, 21:08 UTC
**Script:** `experimental_artifacts/checkpoint_3a/checkpoint_3a_single_run.py`

| Parámetro                                             | Resultado               |
| ----------------------------------------------------- | ----------------------- |
| TX                                                    | CatSniffer #2 — `COM33` |
| RX                                                    | CatSniffer #3 — `COM88` |
| PHY solicitado                                        | `PROPRIETARY_GFSK`      |
| Preset                                                | `wisun_915_fsk_50k`     |
| Frecuencia configurada                                | **902.2 MHz**           |
| Potencia TX solicitada                                | 0 dBm                   |
| Transmisiones solicitadas                             | 10                      |
| Solicitudes TX aceptadas                              | **10/10**               |
| Paquetes recibidos por RX                             | **10/10**               |
| Paquetes con CRC válido y marcador correcto           | **10/10**               |
| Errores asíncronos TX/RX                              | 0                       |
| Errores CRC contabilizados en RX                      | 0                       |
| Paquetes descartados o perdidos según estadísticas RX | 0                       |
| Excepción fatal                                       | Ninguna                 |

### Evidencia principal

El receptor registró diez eventos `RX_PACKET`, todos con:

```text
crc_ok=True
payload_hex='04 DE AD BE EF 33 C8'
marker_present=True
```

Además, sus estadísticas cambiaron de:

```text
RX_STATS_BEFORE
rx_ok=0
```

a:

```text
RX_STATS_AFTER
rx_ok=10
rx_crc_err=0
rx_drop=0
rx_overflow=0
```

Esto proporciona dos evidencias concordantes: los eventos de recepción individuales y el contador de paquetes recibidos correctamente.

**Resultado: `PASS-RF` — 10/10 paquetes recibidos correctamente en esta ejecución.**

---

## 2. ¿Qué hemos demostrado?

Podemos afirmar que, bajo las condiciones de esta prueba:

1. La aplicación Python logró comunicarse con ambos CC1352 a través de sus interfaces Bridge.
2. Ambos dispositivos aceptaron la configuración `PROPRIETARY_GFSK` con el preset `wisun_915_fsk_50k`.
3. El receptor entregó diez paquetes con CRC válido que contenían el marcador transmitido.
4. La recepción funcionó después de ejecutar la secuencia de reinicio y seleccionar `band2` en ambos CatSniffer.

**Conclusión técnica:** existe evidencia funcional de comunicación OTA entre ambos CatSniffer usando la configuración Sub-GHz solicitada de 902.2 MHz.

Esto es especialmente relevante porque permite establecer un **control conocido funcional** para futuras pruebas y comparaciones.

---

## 3. Observaciones que debemos conservar

### A. Los valores RSSI no son confiables todavía

Los diez paquetes incluyen valores como:

```text
rssi_dbm=-73
rssi_dbm=14
rssi_dbm=76
rssi_dbm=-125
rssi_dbm=109
```

La presencia de valores positivos muy elevados y variaciones extremas durante una prueba corta es sospechosa.

**No debemos interpretar esos valores como mediciones físicas válidas de potencia recibida.**

Esto no invalida el resultado de recepción con CRC correcto, pero sí limita cualquier conclusión sobre sensibilidad, alcance o calidad del enlace.

### B. El campo `channel=37` requiere investigación

Los paquetes reportan:

```text
channel=37
```

Sin embargo, configuramos `PROPRIETARY_GFSK` con `channel=0` y frecuencia de 902.2 MHz.

Por ahora, no podemos asegurar qué representa ese campo dentro de los metadatos entregados por el firmware.

No debemos interpretarlo automáticamente como un canal físico de RF.

### C. La frecuencia y potencia son valores solicitados

El registro demuestra que ambos dispositivos aceptaron la configuración de `902200000 Hz` y que el transmisor aceptó `0 dBm`.

No hemos medido la frecuencia de la portadora ni la potencia de salida con instrumentación RF.

Asimismo, esta prueba valida el funcionamiento de una configuración PHY; **no demuestra interoperabilidad con Wi-SUN**.

---

## 4. Estado de la validación

| Etapa                            | Resultado             |
| -------------------------------- | --------------------- |
| Verificación de entorno          | PASS                  |
| Identificación de puertos        | PASS                  |
| Comunicación Shell               | PASS                  |
| Secuencia de reinicio            | PASS — control        |
| Selección `band2`                | PASS — control lógico |
| Configuración GFSK               | PASS — ACK/API        |
| **Control OTA 902.2 MHz**        | **PASS-RF — 10/10**   |
| Medición eléctrica del RF switch | No realizada          |
| Validación de RSSI/LQI           | Pendiente             |

**La prueba de control ha concluido correctamente. No necesitamos repetirla ahora.**

## 5. ¿Qué hacemos a continuación?

Por el momento, **no ejecutes más comandos**.

Hemos completado el objetivo inicial de esta sesión: reconstruir el estado de los dispositivos y obtener un control OTA funcional a 902.2 MHz.

Mi recomendación es conservar esta ejecución como **referencia conocida funcional**, incluyendo su salida completa y las limitaciones identificadas.

El siguiente paso de la campaña será seleccionar una prueba adicional del manual Sub-GHz y compararla contra este control, pero primero debemos decidir qué comportamiento queremos validar.

**Resultado final del Checkpoint 3A: `PASS-RF (10/10)`**.

---

Gracias por compartir el manual completo. Ahora sí podemos trabajar con **`MANUAL_FERALRF_SUBGHZ.md` como documento principal de la campaña**, utilizando las auditorías y registros históricos únicamente como evidencia complementaria.

Lo he revisado y puedo confirmar algo importante:

**El manual sí contiene los procedimientos y parámetros necesarios para ejecutar los 14 casos de la campaña Sub-GHz, incluido el control positivo.** No necesitamos inventar comandos ni diseñar una metodología nueva.

Sin embargo, encontré algunos detalles que conviene corregir o tener presentes antes de continuar.

## 1. Resultado de la revisión

| Elemento                                           | Evaluación    | Sección      |
| -------------------------------------------------- | ------------- | ------------ |
| Identificación de los dos CatSniffer y puertos COM | ✅ Completo    | Introducción |
| Preparación de PowerShell y Python                 | ✅ Completo    | §2           |
| Consulta del estado inicial                        | ✅ Completo    | §3           |
| Reinicio y selección de banda                      | ✅ Completo    | §4           |
| Limpieza y recuperación                            | ✅ Completo    | §5           |
| Prueba OTA de un paquete                           | ✅ Completo    | §6           |
| **Prueba formal de diez paquetes**                 | ✅ Completo    | §7           |
| **Definición de los 14 presets**                   | ✅ Completo    | §8           |
| Presets excluidos                                  | ✅ Completo    | §9           |
| Comparación histórica                              | ✅ Documentada | §10          |
| **Comando reutilizable para cada preset**          | ✅ Completo    | §11          |
| Inserción de controles positivos                   | ✅ Completo    | §12          |
| Criterios de aceptación                            | ✅ Completo    | §13          |
| Registro de resultados                             | ✅ Completo    | §14          |
| Checklist final                                    | ✅ Completo    | §15          |

### ¿Cómo se ejecutan los distintos presets?

La clave está en las secciones **7 y 11**.

La sección 7 define una función de PowerShell:

`Invoke-FeralManualPreset`

Esta función ejecuta la misma secuencia de validación OTA que acabamos de utilizar: configura ambos CC1352, inicia RX, realiza diez transmisiones, recupera los paquetes y registra los resultados.

Para cambiar de prueba solo necesitamos modificar:

* `Preset`: configuración PHY que queremos evaluar.
* `MarkerHex`: identificador único de los paquetes.
* `Label`: identificador de la ejecución.

Por ejemplo, para el siguiente caso, el propio manual establece:

```powershell
Invoke-FeralManualPreset `
    -Preset 'gfsk_902_50k' `
    -MarkerHex 'A5010001' `
    -Label 'campaign-gfsk-902-50k-run-01'
```

**Importante:** este comando funciona únicamente después de definir `Invoke-FeralManualPreset` en la sesión actual de PowerShell. No es una función instalada permanentemente.

---

## 2. Corrección necesaria en el manual

Encontré una discrepancia concreta en la sección 2.

El documento dice que el valor esperado del `sync_word` es:

```python
'sync_word': 2466996702
```

Pero inmediatamente afirma que ese número corresponde a `0x930B51DE`.

Eso es incorrecto.

| Representación | Valor          |
| -------------- | -------------- |
| `0x930B51DE`   | **2466992606** |
| `0x930B61DE`   | 2466996702     |

En la ejecución que acabamos de completar, Python reportó:

```text
'sync_word': 2466992606
```

Por tanto, **el valor observado coincide con `0x930B51DE` y con las tablas de presets de la sección 8**.

La sección 2 debe corregirse para que no genere confusión durante las comprobaciones.

También hay una cuestión menor en la sección 7: el código clasifica automáticamente como `PASS-RF` una ejecución con diez coincidencias, pero debemos revisar igualmente si hubo errores asíncronos, problemas de limpieza o anomalías que obliguen a clasificar el resultado como inconcluso.

Finalmente, el texto que pegaste contiene caracteres escapados como `\*\*` y `\_\_`. Si esos caracteres aparecen literalmente en el archivo original, podrían impedir ejecutar algunos bloques de Python. Si son únicamente resultado del formato al pegarlo en el chat, no hay problema.

---

## 3. Nuestra campaña seguirá exactamente este orden

| Prueba           | Preset                  | Frecuencia | Estado              |
| ---------------- | ----------------------- | ---------- | ------------------- |
| Control positivo | `wisun_915_fsk_50k`     | 902.2 MHz  | ✅ **PASS-RF 10/10** |
| 1                | `gfsk_902_50k`          | 902.2 MHz  | ⏳ Siguiente         |
| 2                | `wisun_915_fsk_100k`    | 902.2 MHz  | Pendiente           |
| 3                | `wisun_915_fsk_150k`    | 902.2 MHz  | Pendiente           |
| 4                | `wisun_915_fsk_200k`    | 902.2 MHz  | Pendiente           |
| 5                | `wisun_915_fsk_300k`    | 902.2 MHz  | Pendiente           |
| 6                | `gfsk_915_50k`          | 915 MHz    | Pendiente           |
| 7                | `sidewalk_915_fsk_50k`  | 915 MHz    | Pendiente           |
| 8                | `sidewalk_915_fsk_250k` | 915 MHz    | Pendiente           |
| 9                | `gfsk_868_50k`          | 868 MHz    | Pendiente           |
| 10               | `gfsk_868_100k`         | 868 MHz    | Pendiente           |
| 11               | `wireless_mbus_s_868`   | 868.3 MHz  | Pendiente           |
| 12               | `wireless_mbus_t_868`   | 868.95 MHz | Pendiente           |
| 13               | `wireless_mbus_c_868`   | 868.95 MHz | Pendiente           |

Las pruebas se ejecutarán **individualmente**, nunca como un barrido automático.

Según la sección 12, repetiremos el control positivo después de tres presets nuevos que hayan pasado y cuando se produzca un fallo o una anomalía relevante.

---

# 4. Continuemos: prueba 1 — `gfsk_902_50k`

Esta prueba es especialmente útil porque mantiene la frecuencia y el symbol rate del control positivo, pero cambia el parámetro de modulación de FSK (`mod_type=0`) a GFSK (`mod_type=1`).

Esto nos permitirá comparar dos configuraciones solicitadas sobre la misma frecuencia.

**Estado de preparación:** seguimos con el **Modo D**, ya que ejecutamos `boot` → `exit` y posteriormente `band2` en ambos CatSniffer.

No repetiremos estos comandos sin una razón experimental.

### Paso 8 — Comprobar el preset antes de transmitir

Vamos a seguir la sección 11 del manual.

Ejecuta este bloque en la misma PowerShell:

```powershell
$PRESET = 'gfsk_902_50k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

### Resultado esperado

```text
gfsk_902_50k {
    'frequency_hz': 902200000,
    'mod_type': 1,
    'symbol_rate': 50000,
    'deviation': 100,
    'rx_bw': 82,
    'sync_word': 2466992606
}
```

La presentación del diccionario puede aparecer en una sola línea.

**Este comando no transmite ni modifica la configuración de los CatSniffer.**

Cuando me envíes el resultado, prepararemos la función `Invoke-FeralManualPreset` de la sección 7 y después ejecutaremos únicamente `gfsk_902_50k`.

No tendrás que interpretar el manual ni decidir qué parámetros cambiar. Yo te proporcionaré las instrucciones exactas.

**Conclusión:** el manual es adecuado como base operativa de la campaña, con las correcciones señaladas. Ya completamos el control positivo y estamos listos para comenzar las trece configuraciones restantes.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
gfsk_902_50k {'frequency_hz': 902200000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}

---

## Paso 8 — Verificación de `gfsk_902_50k` completada ✅

La salida coincide exactamente con la configuración esperada en la **sección 8 de `MANUAL_FERALRF_SUBGHZ.md`**.

| Parámetro      |           Valor observado | Estado |
| -------------- | ------------------------: | ------ |
| `frequency_hz` |                 902200000 | ✅      |
| `mod_type`     |                  1 (GFSK) | ✅      |
| `symbol_rate`  |                     50000 | ✅      |
| `deviation`    |                       100 | ✅      |
| `rx_bw`        |               82 (`0x52`) | ✅      |
| `sync_word`    | 2466992606 (`0x930B51DE`) | ✅      |

**Resultado: PASS — verificación del preset en Python.**

Esto demuestra que el paquete Python contiene el diccionario esperado. Todavía no demuestra que el CC1352 haya aplicado la configuración ni que exista comunicación OTA con GFSK.

---

## Paso 9 — Preparar la función para las pruebas restantes

El manual, en su **sección 7**, define `Invoke-FeralManualPreset`, una función reutilizable que realiza diez transmisiones y registra los resultados.

Como el bloque original es extenso, vamos a evitar que tengas que copiarlo manualmente. Primero comprobaremos si la función ya está definida en tu PowerShell.

### Ejecuta este único comando

```powershell
Get-Command Invoke-FeralManualPreset -ErrorAction SilentlyContinue |
    Select-Object Name, CommandType
```

### Posibles resultados

**Caso A — Aparece:**

```text
Name                     CommandType
----                     -----------
Invoke-FeralManualPreset Function
```

La función ya está disponible y podremos utilizarla.

**Caso B — No aparece nada:**

Es normal. Significa que todavía no hemos cargado la función de la sección 7. En ese caso, te proporcionaré el bloque necesario para definirla.

**Este paso no transmite RF, no reinicia dispositivos y no modifica su configuración.**

### Estado de la campaña

* Control positivo `wisun_915_fsk_50k`: **PASS-RF (10/10)**.
* Preset `gfsk_902_50k`: **verificado en Python**.
* Ejecución OTA de `gfsk_902_50k`: **pendiente**.

Envíame la salida de este comando y continuaremos con la preparación de la prueba.

---

Perfecto. Que no aparezca ninguna salida significa que **`Invoke-FeralManualPreset` todavía no está definida en tu sesión de PowerShell**. No es un error.

Vamos a prepararla siguiendo la sección 7 de `MANUAL_FERALRF_SUBGHZ.md`.

### Paso 10 — Crear el script reutilizable de validación

Para evitar copiar una función extensa en PowerShell, utilizaremos el script que ya ejecutamos exitosamente en el Checkpoint 3A como base. Primero crearemos una copia independiente para la campaña.

Ejecuta **únicamente este comando**:

```powershell
Copy-Item `
    '.\experimental_artifacts\checkpoint_3a\checkpoint_3a_single_run.py' `
    '.\experimental_artifacts\manual_preset_runner.py'
```

**¿Qué hace?**

* Copia el script que produjo el resultado `PASS-RF 10/10`.
* Conserva intacto el script original.
* No ejecuta Python.
* No transmite RF ni modifica el estado de los CatSniffer.

**Resultado esperado:** PowerShell no mostrará ninguna salida si la copia se realiza correctamente.

Después ejecuta:

```powershell
Test-Path '.\experimental_artifacts\manual_preset_runner.py'
```

El resultado esperado es:

```text
True
```

Envíame ese resultado.

**Importante:** todavía no ejecutaremos `gfsk_902_50k`. Antes debemos adaptar y revisar la copia para que acepte los parámetros `Preset`, `MarkerHex` y `Label`, manteniendo la metodología de la sección 7 y evitando modificar el control positivo original.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> Copy-Item `
>>     '.\experimental_artifacts\checkpoint_3a\checkpoint_3a_single_run.py' `
>>     '.\experimental_artifacts\manual_preset_runner.py'
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> Test-Path '.\experimental_artifacts\manual_preset_runner.py'
True

---

## Paso 10 — Completado ✅

La salida `True` confirma que existe:

```text
.\experimental_artifacts\manual_preset_runner.py
```

El script original del Checkpoint 3A permanece separado de esta copia.

**Estado actual:**

* Control positivo `wisun_915_fsk_50k`: **PASS-RF (10/10)**.
* Configuración de `gfsk_902_50k`: **verificada**.
* Copia del script para la campaña: **creada**.
* Prueba OTA de `gfsk_902_50k`: **pendiente**.

### Paso 11 — Inspeccionar el script antes de adaptarlo

Antes de modificar la copia, necesitamos comprobar cómo están definidos sus parámetros y su ejecución.

Ejecuta únicamente:

```powershell
Get-Content '.\experimental_artifacts\manual_preset_runner.py' -Raw
```

Envíame la salida completa.

Con ese contenido prepararé una modificación concreta para que el script acepte `Preset`, `MarkerHex` y `Label` como argumentos, siguiendo la sección 7 del manual y conservando la secuencia OTA que ya funcionó.

**Todavía no ejecutes el script.** Este paso solo lee el archivo; no transmite RF ni cambia el estado de los CatSniffer.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> Get-Content '.\experimental_artifacts\manual_preset_runner.py' -Raw
"""Checkpoint 3A only: one controlled wisun_915_fsk_50k OTA execution.

Experimental helper. Fixed explicit Cat-Bridge ports, one preset, one direction,
ten TX attempts, and no automatic test retry. Shell reset and band control are
performed and recorded separately before this helper is invoked.
"""

from __future__ import annotations

import sys
import time
import traceback
from datetime import datetime, timezone
from pathlib import Path


REPO_ROOT = Path(__file__).resolve().parents[2]
sys.path.insert(0, str(REPO_ROOT / "python"))

from feralrf import PHY, PROP_PRESETS, Radio, RxStreamError  # noqa: E402


TX_PORT = "COM33"
RX_PORT = "COM88"
PRESET_NAME = "wisun_915_fsk_50k"
MARKER = bytes.fromhex("DE AD BE EF")
TX_POWER_DBM = 0
TX_COUNT = 10


def utc_now() -> str:
    return datetime.now(timezone.utc).isoformat().replace("+00:00", "Z")


def emit(event: str, **fields: object) -> None:
    rendered = " ".join(f"{key}={value!r}" for key, value in fields.items())
    print(f"{utc_now()} {event}" + (f" {rendered}" if rendered else ""), flush=True)


def main() -> int:
    preset = PROP_PRESETS[PRESET_NAME].copy()
    emit(
        "TEST_BEGIN",
        checkpoint="3A",
        tx_port=TX_PORT,
        rx_port=RX_PORT,
        preset_name=PRESET_NAME,
        preset=preset,
        marker=MARKER.hex(" ").upper(),
        tx_power_dbm=TX_POWER_DBM,
        tx_count=TX_COUNT,
    )

    tx = Radio(port=TX_PORT, baudrate=921600)
    rx = Radio(port=RX_PORT, baudrate=921600)
    rx_started = False
    tx_accepted = 0
    received = []
    rx_async_errors = []
    tx_async_errors = []
    fatal: BaseException | None = None

    try:
        emit("TX_INIT_BEGIN", port=TX_PORT)
        tx_info = tx.init()
        emit("TX_INIT_OK", info=tx_info)

        emit("RX_INIT_BEGIN", port=RX_PORT)
        rx_info = rx.init()
        emit("RX_INIT_OK", info=rx_info)

        emit("TX_SET_PHY_BEGIN", phy="PROPRIETARY_GFSK", channel=0)
        tx.set_phy(PHY.PROPRIETARY_GFSK, channel=0)
        emit("TX_SET_PHY_ACK")

        emit("RX_SET_PHY_BEGIN", phy="PROPRIETARY_GFSK", channel=0)
        rx.set_phy(PHY.PROPRIETARY_GFSK, channel=0)
        emit("RX_SET_PHY_ACK")

        emit("TX_CONFIGURE_PROP_BEGIN", preset=preset)
        tx.configure_prop(**preset)
        emit("TX_CONFIGURE_PROP_ACK")

        emit("RX_CONFIGURE_PROP_BEGIN", preset=preset)
        rx.configure_prop(**preset)
        emit("RX_CONFIGURE_PROP_ACK")

        emit("TX_SET_POWER_BEGIN", power_dbm=TX_POWER_DBM)
        tx.set_power(TX_POWER_DBM)
        emit("TX_SET_POWER_ACK")

        emit("TX_STATS_BEFORE_BEGIN")
        tx_stats_before = tx.get_stats()
        emit("TX_STATS_BEFORE", stats=tx_stats_before)

        emit("RX_STATS_BEFORE_BEGIN")
        rx_stats_before = rx.get_stats()
        emit("RX_STATS_BEFORE", stats=rx_stats_before)

        emit("RX_START_BEGIN")
        rx.start_rx()
        rx_started = True
        emit("RX_START_ACK")

        delay_start = time.monotonic()
        emit("PRE_TX_WAIT_BEGIN", requested_seconds=0.3)
        time.sleep(0.3)
        emit("PRE_TX_WAIT_COMPLETE", elapsed_seconds=time.monotonic() - delay_start)

        for index in range(1, TX_COUNT + 1):
            emit("TX_ATTEMPT_BEGIN", index=index, payload=MARKER.hex(" ").upper())
            try:
                result = tx.transmit(MARKER, power_dbm=TX_POWER_DBM)
            except Exception as exc:
                emit(
                    "TX_ATTEMPT_EXCEPTION",
                    index=index,
                    exception_type=type(exc).__name__,
                    exception=repr(exc),
                )
            else:
                tx_accepted += 1
                emit("TX_ATTEMPT_ACK", index=index, return_value=result)

            spacing_start = time.monotonic()
            time.sleep(0.1)
            emit(
                "TX_SPACING_COMPLETE",
                index=index,
                elapsed_seconds=time.monotonic() - spacing_start,
            )

        post_tx_start = time.monotonic()
        emit("POST_TX_WAIT_BEGIN", requested_seconds=1.0)
        time.sleep(1.0)
        emit("POST_TX_WAIT_COMPLETE", elapsed_seconds=time.monotonic() - post_tx_start)

        rx_window_start = time.monotonic()
        emit("RX_OBSERVATION_BEGIN", requested_seconds=2.0)
        for item in rx.read_packets(timeout=2.0):
            if isinstance(item, RxStreamError):
                rx_async_errors.append(item)
                emit(
                    "RX_ASYNC_ERROR",
                    error_code=item.error_code,
                    context=item.context,
                )
                continue

            received.append(item)
            emit(
                "RX_PACKET",
                ordinal=len(received),
                device_timestamp_us=item.timestamp_us,
                channel=item.channel,
                rssi_dbm=item.rssi_dbm,
                lqi=item.lqi,
                crc_ok=item.crc_ok,
                payload_hex=item.data.hex(" ").upper(),
                marker_present=MARKER in item.data,
            )
        emit(
            "RX_OBSERVATION_COMPLETE",
            elapsed_seconds=time.monotonic() - rx_window_start,
            packets=len(received),
            async_errors=len(rx_async_errors),
        )

        tx_error_window_start = time.monotonic()
        emit("TX_ASYNC_OBSERVATION_BEGIN", requested_seconds=0.5)
        for item in tx.read_packets(timeout=0.5):
            if isinstance(item, RxStreamError):
                tx_async_errors.append(item)
                emit(
                    "TX_ASYNC_ERROR",
                    error_code=item.error_code,
                    context=item.context,
                )
            else:
                emit(
                    "TX_UNEXPECTED_RX_PACKET",
                    device_timestamp_us=item.timestamp_us,
                    channel=item.channel,
                    rssi_dbm=item.rssi_dbm,
                    lqi=item.lqi,
                    crc_ok=item.crc_ok,
                    payload_hex=item.data.hex(" ").upper(),
                )
        emit(
            "TX_ASYNC_OBSERVATION_COMPLETE",
            elapsed_seconds=time.monotonic() - tx_error_window_start,
            async_errors=len(tx_async_errors),
        )

        emit("RX_STOP_BEGIN")
        rx.stop_rx(retries=1, timeout=1.0)
        rx_started = False
        emit("RX_STOP_ACK")

        emit("TX_STATS_AFTER_BEGIN")
        tx_stats_after = tx.get_stats()
        emit("TX_STATS_AFTER", stats=tx_stats_after)

        emit("RX_STATS_AFTER_BEGIN")
        rx_stats_after = rx.get_stats()
        emit("RX_STATS_AFTER", stats=rx_stats_after)

    except BaseException as exc:
        fatal = exc
        emit("TEST_FATAL_EXCEPTION", exception_type=type(exc).__name__, exception=repr(exc))
        traceback.print_exc()
    finally:
        if rx_started:
            emit("RX_STOP_CLEANUP_BEGIN")
            try:
                rx.stop_rx(retries=1, timeout=1.0)
            except Exception as exc:
                emit(
                    "RX_STOP_CLEANUP_EXCEPTION",
                    exception_type=type(exc).__name__,
                    exception=repr(exc),
                )
            else:
                emit("RX_STOP_CLEANUP_ACK")

        emit("TX_DISCONNECT_BEGIN", port=TX_PORT)
        try:
            tx.disconnect()
        except Exception as exc:
            emit(
                "TX_DISCONNECT_EXCEPTION",
                exception_type=type(exc).__name__,
                exception=repr(exc),
            )
        else:
            emit("TX_DISCONNECT_COMPLETE", port=TX_PORT)

        emit("RX_DISCONNECT_BEGIN", port=RX_PORT)
        try:
            rx.disconnect()
        except Exception as exc:
            emit(
                "RX_DISCONNECT_EXCEPTION",
                exception_type=type(exc).__name__,
                exception=repr(exc),
            )
        else:
            emit("RX_DISCONNECT_COMPLETE", port=RX_PORT)

    crc_good_marker = sum(1 for pkt in received if pkt.crc_ok and MARKER in pkt.data)
    emit(
        "TEST_SUMMARY",
        tx_accepted=tx_accepted,
        tx_requested=TX_COUNT,
        total_peer_packets=len(received),
        peer_crc_good_marker=crc_good_marker,
        peer_expected=TX_COUNT,
        rx_async_errors=len(rx_async_errors),
        tx_async_errors=len(tx_async_errors),
        fatal_exception=None if fatal is None else repr(fatal),
    )
    emit("TEST_END", exit_code=0 if fatal is None else 1)
    return 0 if fatal is None else 1


if __name__ == "__main__":
    raise SystemExit(main())

---

## Paso 11 — Inspección completada ✅

Ya revisé el contenido de `manual_preset_runner.py`.

**La copia conserva la secuencia OTA del Checkpoint 3A**, que anteriormente produjo `PASS-RF (10/10)`.

Sin embargo, encontramos dos aspectos importantes:

1. El script todavía tiene fijados `wisun_915_fsk_50k` y `DE AD BE EF`. Debemos convertirlos en argumentos.
2. **El código devuelve `exit_code=0` siempre que no haya una excepción fatal**, incluso si recibe 0/10 paquetes. Esto no cumple completamente los criterios de aceptación de la sección 13 del manual.

Vamos a corregir ambos puntos sin modificar la secuencia de transmisión y recepción.

### Paso 12 — Adaptar el script para la campaña

Ejecuta el siguiente bloque completo en PowerShell:

```powershell
@'
from pathlib import Path

path = Path("experimental_artifacts/manual_preset_runner.py")
source = path.read_text(encoding="utf-8")

old = '''PRESET_NAME = "wisun_915_fsk_50k"
MARKER = bytes.fromhex("DE AD BE EF")'''

new = '''if len(sys.argv) != 4:
    raise SystemExit(
        "Usage: python manual_preset_runner.py <preset> <marker_hex> <label>"
    )

PRESET_NAME = sys.argv[1]
MARKER = bytes.fromhex(sys.argv[2])
LABEL = sys.argv[3]

if PRESET_NAME not in PROP_PRESETS:
    raise SystemExit(f"Unknown preset: {PRESET_NAME}")

if len(MARKER) != 4:
    raise SystemExit("Marker must contain exactly 4 bytes")'''

assert old in source, "Expected preset configuration not found"
source = source.replace(old, new, 1)

old = '''        checkpoint="3A",
        tx_port=TX_PORT,'''

new = '''        checkpoint="manual-campaign",
        label=LABEL,
        tx_port=TX_PORT,'''

assert old in source, "Expected TEST_BEGIN block not found"
source = source.replace(old, new, 1)

old = '''    emit("TEST_END", exit_code=0 if fatal is None else 1)
    return 0 if fatal is None else 1'''

new = '''    if fatal is not None:
        classification = "BLOCKED"
    elif (
        crc_good_marker == TX_COUNT
        and tx_accepted == TX_COUNT
        and not rx_async_errors
        and not tx_async_errors
    ):
        classification = "PASS-RF"
    elif 1 <= crc_good_marker < TX_COUNT:
        classification = "PARTIAL"
    elif crc_good_marker == 0:
        classification = "FAIL"
    else:
        classification = "INCONCLUSIVE"

    emit(
        "TEST_CLASSIFICATION",
        label=LABEL,
        preset=PRESET_NAME,
        classification=classification,
    )

    exit_code = 0 if classification == "PASS-RF" else 2
    emit("TEST_END", exit_code=exit_code)
    return exit_code'''

assert old in source, "Expected TEST_END block not found"
source = source.replace(old, new, 1)

path.write_text(source, encoding="utf-8")

print("MANUAL RUNNER UPDATED")
'@ | python -
```

### ¿Qué modifica?

Únicamente la copia:

`experimental_artifacts/manual_preset_runner.py`

Ahora aceptará:

```text
<preset> <marker_hex> <label>
```

También imprimirá una clasificación explícita y devolverá un código distinto de cero cuando la prueba no cumpla los criterios de `PASS-RF`.

**No cambia:**

* Los puertos `COM33` y `COM88`.
* El PHY y el canal.
* La potencia solicitada de 0 dBm.
* La secuencia RX → TX.
* Las diez transmisiones.
* Los intervalos de espera.
* La lectura de paquetes y estadísticas.

### Resultado esperado

```text
MANUAL RUNNER UPDATED
```

**Este paso solamente modifica un archivo Python local. No ejecuta la prueba ni transmite RF.**

Envíame el resultado. Después verificaremos la sintaxis del script modificado antes de autorizar la primera ejecución de `gfsk_902_50k`.

**Estado:** `gfsk_902_50k` sigue pendiente de validación OTA.

---

**No. La adaptación del script se realiza una sola vez.** Después podremos utilizarlo para los 14 casos del manual sin volver a modificar el código.

La idea es separar dos cosas:

1. **El procedimiento de prueba:** permanece fijo. Incluye inicializar ambos CatSniffer, configurar el CC1352, iniciar RX, transmitir diez paquetes, recoger resultados y limpiar las conexiones.
2. **La configuración del experimento:** cambia según el preset seleccionado. Incluye frecuencia, modulación, symbol rate, deviation, RX bandwidth y sync word.

### ¿Qué haremos en cada prueba?

Una vez adaptado `manual_preset_runner.py`, bastará con ejecutar un comando como este:

**Prueba 1 — GFSK a 902.2 MHz**

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' gfsk_902_50k A5010001 campaign-gfsk-902-50k-run-01
```

**Prueba 2 — FSK a 902.2 MHz, 100 kbps**

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' wisun_915_fsk_100k A5020001 campaign-wisun-100k-run-01
```

El script consultará automáticamente `PROP_PRESETS` y aplicará los parámetros correspondientes tanto al TX como al RX.

### ¿Y cuando cambiemos de frecuencia?

Tampoco necesitaremos modificar el script. Sin embargo, hay una diferencia importante:

* **Cambiar la modulación:** el script configura ambos CC1352 con el nuevo preset.
* **Cambiar la frecuencia:** el script también configura la frecuencia del CC1352, pero eso **no garantiza que la ruta RF externa del CatSniffer sea la adecuada**.

Por eso, al pasar de 902.2 a 915 MHz, o posteriormente a 868 MHz, revisaremos el estado de banda, las antenas y las condiciones de prueba. Las transmisiones fuera de la frecuencia ya autorizada requerirán confirmar previamente que podemos realizarlas.

### Nuestra metodología

Seguiremos este ciclo:

**Verificar preset → ejecutar prueba → analizar resultado → registrar evidencia → siguiente preset.**

Además, repetiremos el control positivo cuando lo indique la sección 12 del manual.

**En resumen:** hacemos la adaptación una vez, mantenemos fijo el procedimiento y cambiamos únicamente los argumentos de cada ejecución. Así obtenemos resultados comparables y evitamos introducir modificaciones de código entre experimentos.

Por ahora, el script sigue siendo una copia sin adaptar. El siguiente paso es aplicar y verificar esa modificación una sola vez, antes de transmitir con `gfsk_902_50k`.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> @'
>> from pathlib import Path
>>
>> path = Path("experimental_artifacts/manual_preset_runner.py")
>> source = path.read_text(encoding="utf-8")
>>
>> old = '''PRESET_NAME = "wisun_915_fsk_50k"
>> MARKER = bytes.fromhex("DE AD BE EF")'''
>>
>> new = '''if len(sys.argv) != 4:
>>     raise SystemExit(
>>         "Usage: python manual_preset_runner.py <preset> <marker_hex> <label>"
>>     )
>>
>> PRESET_NAME = sys.argv[1]
>> MARKER = bytes.fromhex(sys.argv[2])
>> LABEL = sys.argv[3]
>>
>> if PRESET_NAME not in PROP_PRESETS:
>>     raise SystemExit(f"Unknown preset: {PRESET_NAME}")
>>
>> if len(MARKER) != 4:
>>     raise SystemExit("Marker must contain exactly 4 bytes")'''
>>
>> assert old in source, "Expected preset configuration not found"
>> source = source.replace(old, new, 1)
>>
>> old = '''        checkpoint="3A",
>>         tx_port=TX_PORT,'''
>>
>> new = '''        checkpoint="manual-campaign",
>>         label=LABEL,
>>         tx_port=TX_PORT,'''
>>
>> assert old in source, "Expected TEST_BEGIN block not found"
>> source = source.replace(old, new, 1)
>>
>> old = '''    emit("TEST_END", exit_code=0 if fatal is None else 1)
>>     return 0 if fatal is None else 1'''
>>
>> new = '''    if fatal is not None:
>>         classification = "BLOCKED"
>>     elif (
>>         crc_good_marker == TX_COUNT
>>         and tx_accepted == TX_COUNT
>>         and not rx_async_errors
>>         and not tx_async_errors
>>     ):
>>         classification = "PASS-RF"
>>     elif 1 <= crc_good_marker < TX_COUNT:
>>         classification = "PARTIAL"
>>     elif crc_good_marker == 0:
>>         classification = "FAIL"
>>     else:
>>         classification = "INCONCLUSIVE"
>>
>>     emit(
>>         "TEST_CLASSIFICATION",
>>         label=LABEL,
>>         preset=PRESET_NAME,
>>         classification=classification,
>>     )
>>
>>     exit_code = 0 if classification == "PASS-RF" else 2
>>     emit("TEST_END", exit_code=exit_code)
>>     return exit_code'''
>>
>> assert old in source, "Expected TEST_END block not found"
>> source = source.replace(old, new, 1)
>>
>> path.write_text(source, encoding="utf-8")
>>
>> print("MANUAL RUNNER UPDATED")
>> '@ | python -
MANUAL RUNNER UPDATED

---

## Paso 12 — Adaptación del script completada ✅

La salida:

```text
MANUAL RUNNER UPDATED
```

confirma que el script de modificación encontró los tres bloques esperados y escribió la versión adaptada en:

`experimental_artifacts/manual_preset_runner.py`

**Esto todavía no demuestra que el script modificado sea ejecutable.** Vamos a comprobarlo antes de transmitir.

### Paso 13 — Verificar la sintaxis de Python

Ejecuta únicamente este comando:

```powershell
python -c "import ast; from pathlib import Path; p=Path('experimental_artifacts/manual_preset_runner.py'); ast.parse(p.read_text(encoding='utf-8')); print('SYNTAX OK')"
```

**Resultado esperado:**

```text
SYNTAX OK
```

Esta comprobación:

* Lee y analiza la sintaxis del archivo.
* No abre los puertos COM.
* No inicializa los CatSniffer.
* No transmite RF.
* No modifica el script.

### Una observación para la validación

La adaptación conserva la secuencia OTA del Checkpoint 3A, pero la clasificación automática todavía tiene una limitación: **no considera explícitamente los errores de limpieza** (`stop_rx()` y `disconnect()`) ni todas las situaciones que el manual clasifica como `INCONCLUSIVE`.

Por tanto, revisaremos la salida completa de cada ejecución antes de asignar el resultado definitivo. No tomaremos `PASS-RF` del script como prueba suficiente por sí solo.

**Estado actual:**

* Control positivo: **PASS-RF 10/10**.
* `gfsk_902_50k`: configuración verificada.
* Script reutilizable: adaptado.
* Validación de sintaxis: pendiente.
* Transmisión OTA de `gfsk_902_50k`: todavía no ejecutada.

Envíame el resultado de la comprobación y continuaremos con la primera prueba.

---

SYNTAX OK

---

## Paso 13 — Verificación de sintaxis completada ✅

El resultado `SYNTAX OK` confirma que `manual_preset_runner.py` tiene una sintaxis Python válida.

**Estado de preparación:**

* Control positivo `wisun_915_fsk_50k`: **PASS-RF (10/10)**.
* Preset `gfsk_902_50k`: **parámetros verificados**.
* Script reutilizable: **adaptado**.
* Sintaxis Python: **correcta**.
* Validación OTA de `gfsk_902_50k`: **pendiente**.

### Paso 14 — Ejecutar `gfsk_902_50k`

Ahora utilizaremos el procedimiento del Checkpoint 3A, conservando los mismos dispositivos, frecuencia y tiempos. El cambio principal es `mod_type=1` (GFSK), frente a `mod_type=0` (FSK) del control positivo.

**Condiciones de la prueba:**

| Parámetro             | Valor                                |
| --------------------- | ------------------------------------ |
| TX                    | CatSniffer #2 — `COM33`              |
| RX                    | CatSniffer #3 — `COM88`              |
| Preset                | `gfsk_902_50k`                       |
| Frecuencia            | 902.2 MHz                            |
| Modulación solicitada | GFSK                                 |
| Symbol rate           | 50 000 símbolos/s                    |
| Potencia solicitada   | 0 dBm                                |
| Transmisiones         | 10                                   |
| Marker                | `A5 01 00 01`                        |
| Preparación           | Modo D, sin repetir reset ni `band2` |

**Antes de ejecutar:** confirma que ambos CatSniffer siguen conectados, que los puertos COM no han cambiado y que ninguna otra aplicación los está utilizando.

Ejecuta **una sola vez**:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    gfsk_902_50k `
    A5010001 `
    campaign-gfsk-902-50k-run-01
```

El script debe inicializar ambos CC1352, configurar el preset, iniciar RX, realizar diez transmisiones y recopilar los paquetes recibidos.

### Qué necesito que me compartas

Envíame **la salida completa**, desde `TEST_BEGIN` hasta `TEST_END`, incluyendo cualquier excepción.

Revisaremos especialmente:

* `TX_ATTEMPT_ACK`: cuántas transmisiones completaron la API.
* `RX_PACKET`: cuántos paquetes llegaron al receptor.
* `crc_ok` y `marker_present`: cuántos cumplen el criterio OTA.
* `RX_ASYNC_ERROR` y `TX_ASYNC_ERROR`: posibles errores.
* `RX_STATS_BEFORE` y `RX_STATS_AFTER`: evolución de los contadores.
* `RX_STOP_ACK` y desconexiones: limpieza de la sesión.

**No ejecutes todavía otro preset ni repitas esta prueba**, incluso si el resultado es negativo. Primero analizaremos la evidencia y determinaremos si corresponde `PASS-RF`, `PARTIAL`, `FAIL`, `BLOCKED` o `INCONCLUSIVE`.

Así mantendremos la campaña reproducible y podremos comparar directamente FSK y GFSK a 902.2 MHz.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     gfsk_902_50k `
>>     A5010001 `
>>     campaign-gfsk-902-50k-run-01
2026-10-15T22:11:27.285265Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-gfsk-902-50k-run-01' tx_port='COM33' rx_port='COM88' preset_name='gfsk_902_50k' preset={'frequency_hz': 902200000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='A5 01 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:11:27.285401Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:11:28.397810Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:11:28.397941Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:11:29.512943Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:11:29.513059Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:11:29.549531Z TX_SET_PHY_ACK
2026-10-15T22:11:29.549879Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:11:29.580984Z RX_SET_PHY_ACK
2026-10-15T22:11:29.581085Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:11:29.609918Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:11:29.610019Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:11:29.641538Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:11:29.641642Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:11:29.671379Z TX_SET_POWER_ACK
2026-10-15T22:11:29.671476Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:11:29.838857Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:11:29.838965Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:11:30.010097Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:11:30.010207Z RX_START_BEGIN
2026-10-15T22:11:30.038253Z RX_START_ACK
2026-10-15T22:11:30.038359Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:11:30.338696Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30029929999727756
2026-10-15T22:11:30.339204Z TX_ATTEMPT_BEGIN index=1 payload='A5 01 00 01'
2026-10-15T22:11:30.382522Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:11:30.483635Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10067200000048615
2026-10-15T22:11:30.484136Z TX_ATTEMPT_BEGIN index=2 payload='A5 01 00 01'
2026-10-15T22:11:30.525852Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:11:30.626466Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10034710000036284
2026-10-15T22:11:30.627034Z TX_ATTEMPT_BEGIN index=3 payload='A5 01 00 01'
2026-10-15T22:11:30.668959Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:11:30.769784Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10065670000039972
2026-10-15T22:11:30.770333Z TX_ATTEMPT_BEGIN index=4 payload='A5 01 00 01'
2026-10-15T22:11:30.814120Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:11:30.915295Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10065219999887631
2026-10-15T22:11:30.915799Z TX_ATTEMPT_BEGIN index=5 payload='A5 01 00 01'
2026-10-15T22:11:30.958055Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:11:31.058547Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.1002230000012787
2026-10-15T22:11:31.059006Z TX_ATTEMPT_BEGIN index=6 payload='A5 01 00 01'
2026-10-15T22:11:31.092536Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:11:31.192920Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10025519999908283
2026-10-15T22:11:31.193186Z TX_ATTEMPT_BEGIN index=7 payload='A5 01 00 01'
2026-10-15T22:11:31.226147Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:11:31.326718Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.1004480000010517
2026-10-15T22:11:31.326846Z TX_ATTEMPT_BEGIN index=8 payload='A5 01 00 01'
2026-10-15T22:11:31.361428Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:11:31.461748Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10020739999890793
2026-10-15T22:11:31.461886Z TX_ATTEMPT_BEGIN index=9 payload='A5 01 00 01'
2026-10-15T22:11:31.495097Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:11:31.595602Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10038910000002943
2026-10-15T22:11:31.595719Z TX_ATTEMPT_BEGIN index=10 payload='A5 01 00 01'
2026-10-15T22:11:31.625149Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:11:31.725802Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10047950000080164
2026-10-15T22:11:31.725965Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:11:32.726244Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0002653000010469
2026-10-15T22:11:32.726390Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:11:32.839300Z RX_PACKET ordinal=1 device_timestamp_us=3172670 channel=37 rssi_dbm=115 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:32.956438Z RX_PACKET ordinal=2 device_timestamp_us=3173236 channel=37 rssi_dbm=30 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.069501Z RX_PACKET ordinal=3 device_timestamp_us=3173793 channel=37 rssi_dbm=-18 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.185161Z RX_PACKET ordinal=4 device_timestamp_us=3174353 channel=37 rssi_dbm=69 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.299298Z RX_PACKET ordinal=5 device_timestamp_us=3174922 channel=37 rssi_dbm=-40 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.417030Z RX_PACKET ordinal=6 device_timestamp_us=3175481 channel=37 rssi_dbm=-9 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.533881Z RX_PACKET ordinal=7 device_timestamp_us=3176004 channel=37 rssi_dbm=52 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.650014Z RX_PACKET ordinal=8 device_timestamp_us=3176526 channel=37 rssi_dbm=-113 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.760664Z RX_PACKET ordinal=9 device_timestamp_us=3177053 channel=37 rssi_dbm=22 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:33.868621Z RX_PACKET ordinal=10 device_timestamp_us=3177577 channel=37 rssi_dbm=-87 lqi=0 crc_ok=True payload_hex='04 A5 01 00 01 E1 C2' marker_present=True
2026-10-15T22:11:34.738429Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0120225999999093 packets=10 async_errors=0
2026-10-15T22:11:34.738582Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:11:35.246027Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5074333000011393 async_errors=0
2026-10-15T22:11:35.246150Z RX_STOP_BEGIN
2026-10-15T22:11:35.280154Z RX_STOP_ACK
2026-10-15T22:11:35.280284Z TX_STATS_AFTER_BEGIN
2026-10-15T22:11:35.452852Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:11:35.452950Z RX_STATS_AFTER_BEGIN
2026-10-15T22:11:35.620850Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:11:35.621055Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:11:35.621929Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:11:35.621993Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:11:35.622665Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:11:35.622737Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:11:35.622791Z TEST_CLASSIFICATION label='campaign-gfsk-902-50k-run-01' preset='gfsk_902_50k' classification='PASS-RF'
2026-10-15T22:11:35.622868Z TEST_END exit_code=0
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF>

---

## Prueba 1 — `gfsk_902_50k`: PASS-RF (10/10) ✅

La ejecución terminó correctamente. Los registros muestran **10 transmisiones aceptadas por la API y 10 paquetes recibidos por CatSniffer #3 con CRC válido y el marcador esperado**.

### 1. Resultado de la validación

| Parámetro                      | Resultado observado                           |
| ------------------------------ | --------------------------------------------- |
| Fecha y hora                   | 2026-10-15, 22:11:27 UTC                      |
| Identificador                  | `campaign-gfsk-902-50k-run-01`                |
| TX                             | CatSniffer #2 — `COM33`                       |
| RX                             | CatSniffer #3 — `COM88`                       |
| PHY                            | `PROPRIETARY_GFSK`                            |
| Preset                         | `gfsk_902_50k`                                |
| Frecuencia configurada         | 902.2 MHz                                     |
| Modulación solicitada          | GFSK (`mod_type=1`)                           |
| Symbol rate                    | 50 000 símbolos/s                             |
| Potencia solicitada            | 0 dBm                                         |
| Marcador                       | `A5 01 00 01`                                 |
| Transmisiones solicitadas      | 10                                            |
| Transmisiones aceptadas        | **10/10**                                     |
| Paquetes recibidos             | **10**                                        |
| Paquetes CRC-good con marcador | **10/10**                                     |
| Errores asíncronos RX/TX       | **0/0**                                       |
| Excepciones fatales            | **0**                                         |
| Limpieza                       | `RX_STOP_ACK` y ambas desconexiones correctas |
| **Clasificación**              | **PASS-RF**                                   |

### 2. Evidencia de recepción

Los diez paquetes contienen exactamente el mismo payload reportado:

```text
04 A5 01 00 01 E1 C2
```

En todos los casos:

```text
crc_ok=True
marker_present=True
```

Los contadores del receptor también respaldan el resultado:

| Contador RX   | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

Esto constituye evidencia de recepción OTA entre los dos CatSniffer bajo la configuración solicitada.

**No obstante, el resultado no demuestra por sí solo las características espectrales de la señal GFSK.** Para confirmar físicamente la modulación y la desviación necesitaríamos mediciones RF independientes.

### 3. Anomalías que debemos conservar en el registro

Se repiten las anomalías observadas durante el control positivo:

* `channel=37`, aunque configuramos `channel=0`.
* `lqi=0` en todos los paquetes.
* Valores RSSI desde −113 hasta +115 dBm, incluyendo valores positivos poco plausibles.

Estas anomalías indican que **todavía no debemos utilizar esos metadatos como mediciones RF confiables**.

No invalidan automáticamente los diez paquetes CRC-good recibidos, pero sí limitan lo que podemos concluir sobre la calidad del enlace.

---

## 4. Comparación con el control positivo

| Característica | Control positivo    | Prueba 1       |
| -------------- | ------------------- | -------------- |
| Preset         | `wisun_915_fsk_50k` | `gfsk_902_50k` |
| Frecuencia     | 902.2 MHz           | 902.2 MHz      |
| `mod_type`     | 0 (FSK)             | 1 (GFSK)       |
| Symbol rate    | 50 000              | 50 000         |
| `deviation`    | 100                 | 100            |
| `rx_bw`        | 82                  | 82             |
| `sync_word`    | `0x930B51DE`        | `0x930B51DE`   |
| Resultado OTA  | **10/10**           | **10/10**      |

**Conclusión:** ambas configuraciones solicitadas permiten entregar diez paquetes CRC-good al receptor en las condiciones ensayadas.

Es una comparación especialmente útil porque el parámetro principal que cambia es `mod_type`.

---

## 5. Estado actualizado de la campaña

| Prueba  | Preset               | Resultado       |
| ------- | -------------------- | --------------- |
| Control | `wisun_915_fsk_50k`  | ✅ PASS-RF 10/10 |
| 1       | `gfsk_902_50k`       | ✅ PASS-RF 10/10 |
| 2       | `wisun_915_fsk_100k` | ⏳ Siguiente     |
| 3       | `wisun_915_fsk_150k` | Pendiente       |
| 4–13    | Presets restantes    | Pendientes      |

Según la sección 12 del manual, repetiremos el control positivo después de tres presets nuevos que pasen. **Por ahora llevamos uno**, así que no necesitamos repetirlo todavía.

---

# Paso 15 — Preparar `wisun_915_fsk_100k`

El siguiente preset mantiene 902.2 MHz, pero aumenta el symbol rate a 100 000 símbolos/s.

Antes de transmitir, verificaremos su configuración, igual que hicimos con GFSK.

Ejecuta únicamente:

```powershell
$PRESET = 'wisun_915_fsk_100k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

### Resultado esperado

```text
wisun_915_fsk_100k {'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
```

Este comando solo consulta el diccionario de Python; **no transmite RF**.

Cuando compartas el resultado, continuaremos con la segunda prueba utilizando el mismo script reutilizable, sin modificarlo.

**Avance: 2/14 configuraciones con PASS-RF**, contando el control positivo.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wisun_915_fsk_100k {'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}

---

## Paso 15 — Configuración de `wisun_915_fsk_100k` verificada ✅

La salida coincide exactamente con los parámetros esperados para el segundo preset de la campaña.

| Parámetro              | Valor confirmado     |
| ---------------------- | -------------------- |
| Preset                 | `wisun_915_fsk_100k` |
| Frecuencia configurada | 902.2 MHz            |
| Modulación solicitada  | FSK (`mod_type=0`)   |
| Symbol rate            | 100 000 símbolos/s   |
| `deviation`            | 150                  |
| `rx_bw`                | 86                   |
| `sync_word`            | `0x930B51DE`         |

**Importante:** aunque el nombre contiene `915`, la frecuencia configurada es **902.2 MHz**. No estamos cambiando de banda respecto a las pruebas anteriores.

La configuración está verificada en `PROP_PRESETS`, pero su funcionamiento OTA todavía no se ha comprobado.

---

## Paso 16 — Ejecutar la segunda prueba OTA

**Identificador:** `campaign-wisun-100k-run-01`

Mantendremos las mismas condiciones:

* **TX:** CatSniffer #2, `COM33`.
* **RX:** CatSniffer #3, `COM88`.
* **Potencia solicitada:** 0 dBm.
* **Número de transmisiones:** 10.
* **Marcador:** `A5 02 00 01`.
* **Preparación:** Modo D ya realizado; no repetir `boot`, `exit` ni `band2` si los dispositivos permanecen en el mismo estado.

Confirma que ambos CatSniffer siguen conectados y que sus puertos COM están disponibles.

Ejecuta **una sola vez**:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_100k `
    A5020001 `
    campaign-wisun-100k-run-01
```

Este comando **sí realizará diez transmisiones RF** a la frecuencia configurada de 902.2 MHz.

### Qué verificaremos

Al recibir la salida, comprobaremos las transmisiones aceptadas, los paquetes recibidos con `crc_ok=True`, la presencia del marcador, los contadores RX, los errores asíncronos y la limpieza de la sesión.

**No repitas la prueba ni avances al siguiente preset** hasta revisar el resultado.

**Estado de la campaña:** control positivo y `gfsk_902_50k` con `PASS-RF 10/10`; `wisun_915_fsk_100k` preparado para validación OTA.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wisun_915_fsk_100k {'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_100k `
>>     A5020001 `
>>     campaign-wisun-100k-run-01
2026-10-15T22:14:21.261279Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wisun-100k-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_100k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606} marker='A5 02 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:14:21.261414Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:14:22.376511Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:14:22.376628Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:14:23.493457Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:14:23.493566Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:14:23.526109Z TX_SET_PHY_ACK
2026-10-15T22:14:23.526209Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:14:23.560569Z RX_SET_PHY_ACK
2026-10-15T22:14:23.560671Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
2026-10-15T22:14:23.591813Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:14:23.591909Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
2026-10-15T22:14:23.622225Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:14:23.622343Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:14:23.654056Z TX_SET_POWER_ACK
2026-10-15T22:14:23.654146Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:14:23.820832Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:14:23.820927Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:14:23.988538Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:14:23.988638Z RX_START_BEGIN
2026-10-15T22:14:24.019939Z RX_START_ACK
2026-10-15T22:14:24.020041Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:14:24.320693Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.3006131000001915
2026-10-15T22:14:24.321158Z TX_ATTEMPT_BEGIN index=1 payload='A5 02 00 01'
2026-10-15T22:14:24.359119Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:14:24.459820Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10044940000079805
2026-10-15T22:14:24.460377Z TX_ATTEMPT_BEGIN index=2 payload='A5 02 00 01'
2026-10-15T22:14:24.499347Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:14:24.599795Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10033549999934621
2026-10-15T22:14:24.599923Z TX_ATTEMPT_BEGIN index=3 payload='A5 02 00 01'
2026-10-15T22:14:24.635202Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:14:24.735796Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.1004579999971611
2026-10-15T22:14:24.735916Z TX_ATTEMPT_BEGIN index=4 payload='A5 02 00 01'
2026-10-15T22:14:24.770975Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:14:24.871478Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.1003898000017216
2026-10-15T22:14:24.871601Z TX_ATTEMPT_BEGIN index=5 payload='A5 02 00 01'
2026-10-15T22:14:24.901336Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:14:25.002024Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10052569999970729
2026-10-15T22:14:25.002146Z TX_ATTEMPT_BEGIN index=6 payload='A5 02 00 01'
2026-10-15T22:14:25.035475Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:14:25.135829Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10022260000187089
2026-10-15T22:14:25.135974Z TX_ATTEMPT_BEGIN index=7 payload='A5 02 00 01'
2026-10-15T22:14:25.169820Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:14:25.270415Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.1004337000013038
2026-10-15T22:14:25.270531Z TX_ATTEMPT_BEGIN index=8 payload='A5 02 00 01'
2026-10-15T22:14:25.303097Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:14:25.403337Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10009610000270186
2026-10-15T22:14:25.403477Z TX_ATTEMPT_BEGIN index=9 payload='A5 02 00 01'
2026-10-15T22:14:25.438364Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:14:25.538912Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10039139999935287
2026-10-15T22:14:25.539027Z TX_ATTEMPT_BEGIN index=10 payload='A5 02 00 01'
2026-10-15T22:14:25.572185Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:14:25.672438Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10013249999974505
2026-10-15T22:14:25.672585Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:14:26.673019Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0003933000007237
2026-10-15T22:14:26.673515Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:14:26.809541Z RX_PACKET ordinal=1 device_timestamp_us=3852289 channel=37 rssi_dbm=43 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:26.925216Z RX_PACKET ordinal=2 device_timestamp_us=3852832 channel=37 rssi_dbm=102 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.049014Z RX_PACKET ordinal=3 device_timestamp_us=3853377 channel=37 rssi_dbm=49 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.165594Z RX_PACKET ordinal=4 device_timestamp_us=3853908 channel=37 rssi_dbm=74 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.282356Z RX_PACKET ordinal=5 device_timestamp_us=3854438 channel=37 rssi_dbm=115 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.399764Z RX_PACKET ordinal=6 device_timestamp_us=3854948 channel=37 rssi_dbm=-56 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.518390Z RX_PACKET ordinal=7 device_timestamp_us=3855471 channel=37 rssi_dbm=39 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.632098Z RX_PACKET ordinal=8 device_timestamp_us=3855996 channel=37 rssi_dbm=13 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.748111Z RX_PACKET ordinal=9 device_timestamp_us=3856515 channel=37 rssi_dbm=-128 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:27.868354Z RX_PACKET ordinal=10 device_timestamp_us=3857045 channel=37 rssi_dbm=-35 lqi=0 crc_ok=True payload_hex='04 A5 02 00 01 E1 FE' marker_present=True
2026-10-15T22:14:28.684743Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0112417000018468 packets=10 async_errors=0
2026-10-15T22:14:28.684886Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:14:29.198537Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5136349999993399 async_errors=0
2026-10-15T22:14:29.198741Z RX_STOP_BEGIN
2026-10-15T22:14:29.231177Z RX_STOP_ACK
2026-10-15T22:14:29.231307Z TX_STATS_AFTER_BEGIN
2026-10-15T22:14:29.400561Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:14:29.400657Z RX_STATS_AFTER_BEGIN
2026-10-15T22:14:29.559827Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:14:29.559956Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:14:29.560720Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:14:29.560805Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:14:29.561381Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:14:29.561514Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:14:29.561579Z TEST_CLASSIFICATION label='campaign-wisun-100k-run-01' preset='wisun_915_fsk_100k' classification='PASS-RF'
2026-10-15T22:14:29.561636Z TEST_END exit_code=0

---

## Prueba 2 — `wisun_915_fsk_100k`: PASS-RF (10/10) ✅

La segunda prueba de la campaña terminó correctamente. **CatSniffer #3 recibió diez paquetes con CRC válido y el marcador esperado**, mientras que CatSniffer #2 obtuvo diez respuestas satisfactorias de la API de transmisión.

### 1. Resultado observado

| Parámetro                      | Resultado                    |
| ------------------------------ | ---------------------------- |
| Identificador                  | `campaign-wisun-100k-run-01` |
| Fecha                          | 2026-10-15, 22:14:21 UTC     |
| TX                             | CatSniffer #2 — `COM33`      |
| RX                             | CatSniffer #3 — `COM88`      |
| Preset                         | `wisun_915_fsk_100k`         |
| Frecuencia configurada         | 902.2 MHz                    |
| Modulación solicitada          | FSK (`mod_type=0`)           |
| Symbol rate                    | 100 000 símbolos/s           |
| `deviation`                    | 150                          |
| `rx_bw`                        | 86                           |
| Potencia solicitada            | 0 dBm                        |
| Marcador                       | `A5 02 00 01`                |
| Transmisiones aceptadas        | **10/10**                    |
| Paquetes recibidos             | **10/10**                    |
| Paquetes CRC-good con marcador | **10/10**                    |
| Errores asíncronos             | **0**                        |
| Excepciones fatales            | **0**                        |
| Limpieza                       | Correcta                     |
| **Clasificación**              | **PASS-RF**                  |

### 2. Evidencia de recepción

Los diez paquetes presentan:

```text
payload_hex='04 A5 02 00 01 E1 FE'
crc_ok=True
marker_present=True
```

Además, los contadores RX evolucionaron de la siguiente manera:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

No se registraron errores asíncronos ni excepciones, y ambas conexiones se cerraron correctamente.

**Conclusión experimental:** la configuración solicitada `wisun_915_fsk_100k` permitió recibir diez paquetes CRC-good entre ambos CatSniffer bajo las condiciones ensayadas.

Esto no equivale a demostrar interoperabilidad con dispositivos Wi-SUN externos ni conformidad con el estándar Wi-SUN.

### 3. Anomalías persistentes

Se mantienen tres observaciones que debemos investigar posteriormente:

* `channel=37`, pese a solicitar `channel=0`.
* `lqi=0` en todos los paquetes.
* RSSI entre −128 y +115 dBm, con valores positivos físicamente poco plausibles.

Estas anomalías **no impiden clasificar la entrega de paquetes como PASS-RF**, pero sí impiden considerar los metadatos como mediciones fiables de calidad del enlace.

---

## 4. Estado de la campaña

| Prueba  | Preset               | Resultado       |
| ------- | -------------------- | --------------- |
| Control | `wisun_915_fsk_50k`  | ✅ PASS-RF 10/10 |
| 1       | `gfsk_902_50k`       | ✅ PASS-RF 10/10 |
| 2       | `wisun_915_fsk_100k` | ✅ PASS-RF 10/10 |
| 3       | `wisun_915_fsk_150k` | Pendiente       |

**Llevamos dos presets nuevos con PASS-RF.** Según el manual, después del tercero repetiremos el control positivo antes de continuar con la campaña.

---

## Paso 17 — Verificar `wisun_915_fsk_150k`

Antes de transmitir, verificaremos los parámetros definidos en el repositorio.

Ejecuta únicamente:

```powershell
$PRESET = 'wisun_915_fsk_150k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

Este comando **no transmite RF**.

Envíame la salida para confirmar la frecuencia, modulación, symbol rate, deviation, RX bandwidth y sync word.

Después realizaremos la tercera prueba OTA. Si obtiene `PASS-RF`, corresponderá repetir el control positivo `wisun_915_fsk_50k` antes de avanzar al cuarto preset.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wisun_915_fsk_150k {'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 150000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}

---

## Paso 17 — Configuración de `wisun_915_fsk_150k` verificada ✅

La salida confirma que el preset está definido en `PROP_PRESETS` con los siguientes parámetros:

| Parámetro              | Valor                |
| ---------------------- | -------------------- |
| Preset                 | `wisun_915_fsk_150k` |
| Frecuencia configurada | **902.2 MHz**        |
| Modulación solicitada  | FSK (`mod_type=0`)   |
| Symbol rate            | 150 000 símbolos/s   |
| `deviation`            | 200                  |
| `rx_bw`                | 90                   |
| `sync_word`            | `0x930B51DE`         |

**Observación:** aunque el nombre contiene `915`, la frecuencia configurada continúa siendo 902.2 MHz.

La definición del preset está verificada. Su funcionamiento OTA todavía está pendiente.

---

## Paso 18 — Ejecutar la tercera prueba OTA

Mantendremos las mismas condiciones experimentales:

| Condición           | Valor                        |
| ------------------- | ---------------------------- |
| TX                  | CatSniffer #2 — `COM33`      |
| RX                  | CatSniffer #3 — `COM88`      |
| Frecuencia          | 902.2 MHz                    |
| Potencia solicitada | 0 dBm                        |
| Transmisiones       | 10                           |
| Marcador            | `A5 03 00 01`                |
| Identificador       | `campaign-wisun-150k-run-01` |

No es necesario repetir `boot`, `exit` ni `band2` si ambos dispositivos permanecen conectados y conservan el estado de preparación anterior.

### Comando

Confirma que los puertos COM siguen disponibles y ejecuta **una sola vez**:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_150k `
    A5030001 `
    campaign-wisun-150k-run-01
```

**Este comando realizará diez transmisiones RF a la frecuencia configurada de 902.2 MHz.**

### Qué comprobaremos

Revisaremos las diez respuestas de transmisión, los paquetes recibidos con CRC válido y marcador correcto, los contadores RX, los errores asíncronos y la limpieza de ambas conexiones.

**Importante:** si obtenemos `PASS-RF`, habremos completado tres presets nuevos exitosos. Según la sección 12 del manual, el siguiente paso será repetir el control positivo `wisun_915_fsk_50k` antes de continuar con `wisun_915_fsk_200k`.

Envíame la salida completa, desde `TEST_BEGIN` hasta `TEST_END`. No repitas la prueba ni ejecutes otro preset antes de revisar el resultado.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wisun_915_fsk_150k {'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 150000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_150k `
>>     A5030001 `
>>     campaign-wisun-150k-run-01
2026-10-15T22:16:36.036716Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wisun-150k-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_150k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 150000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606} marker='A5 03 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:16:36.036862Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:16:37.169568Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:16:37.169664Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:16:38.284074Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:16:38.284174Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:16:38.317089Z TX_SET_PHY_ACK
2026-10-15T22:16:38.317190Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:16:38.348326Z RX_SET_PHY_ACK
2026-10-15T22:16:38.348431Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 150000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}
2026-10-15T22:16:38.379038Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:16:38.379135Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 150000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}
2026-10-15T22:16:38.412526Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:16:38.412623Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:16:38.442209Z TX_SET_POWER_ACK
2026-10-15T22:16:38.442466Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:16:38.606954Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:16:38.607152Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:16:38.779380Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:16:38.779480Z RX_START_BEGIN
2026-10-15T22:16:38.810164Z RX_START_ACK
2026-10-15T22:16:38.810288Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:16:39.110758Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.3004661000013584
2026-10-15T22:16:39.110886Z TX_ATTEMPT_BEGIN index=1 payload='A5 03 00 01'
2026-10-15T22:16:39.144355Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:16:39.244947Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10044030000062776
2026-10-15T22:16:39.245060Z TX_ATTEMPT_BEGIN index=2 payload='A5 03 00 01'
2026-10-15T22:16:39.279376Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:16:39.379990Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10039989999859245
2026-10-15T22:16:39.380134Z TX_ATTEMPT_BEGIN index=3 payload='A5 03 00 01'
2026-10-15T22:16:39.413044Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:16:39.513693Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10052220000216039
2026-10-15T22:16:39.513898Z TX_ATTEMPT_BEGIN index=4 payload='A5 03 00 01'
2026-10-15T22:16:39.549903Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:16:39.650324Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.1001168000002508
2026-10-15T22:16:39.650490Z TX_ATTEMPT_BEGIN index=5 payload='A5 03 00 01'
2026-10-15T22:16:39.682169Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:16:39.782664Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10037850000298931
2026-10-15T22:16:39.782789Z TX_ATTEMPT_BEGIN index=6 payload='A5 03 00 01'
2026-10-15T22:16:39.818268Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:16:39.918650Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.1001566000013554
2026-10-15T22:16:39.918759Z TX_ATTEMPT_BEGIN index=7 payload='A5 03 00 01'
2026-10-15T22:16:39.951582Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:16:40.052279Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10053329999936977
2026-10-15T22:16:40.052844Z TX_ATTEMPT_BEGIN index=8 payload='A5 03 00 01'
2026-10-15T22:16:40.088384Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:16:40.188982Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10046410000359174
2026-10-15T22:16:40.189106Z TX_ATTEMPT_BEGIN index=9 payload='A5 03 00 01'
2026-10-15T22:16:40.218204Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:16:40.318579Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10022270000263234
2026-10-15T22:16:40.319070Z TX_ATTEMPT_BEGIN index=10 payload='A5 03 00 01'
2026-10-15T22:16:40.362042Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:16:40.462926Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10039150000011432
2026-10-15T22:16:40.463396Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:16:41.464101Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0006897000021127
2026-10-15T22:16:41.464608Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:16:41.584490Z RX_PACKET ordinal=1 device_timestamp_us=184508 channel=37 rssi_dbm=-42 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:41.699004Z RX_PACKET ordinal=2 device_timestamp_us=185032 channel=37 rssi_dbm=74 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:41.814774Z RX_PACKET ordinal=3 device_timestamp_us=185560 channel=37 rssi_dbm=23 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:41.926070Z RX_PACKET ordinal=4 device_timestamp_us=186081 channel=37 rssi_dbm=-128 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:42.049058Z RX_PACKET ordinal=5 device_timestamp_us=186615 channel=37 rssi_dbm=105 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:42.166321Z RX_PACKET ordinal=6 device_timestamp_us=187132 channel=37 rssi_dbm=-83 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:42.285469Z RX_PACKET ordinal=7 device_timestamp_us=187664 channel=37 rssi_dbm=44 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:42.406809Z RX_PACKET ordinal=8 device_timestamp_us=188188 channel=37 rssi_dbm=92 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:42.527264Z RX_PACKET ordinal=9 device_timestamp_us=188719 channel=37 rssi_dbm=-45 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:42.642087Z RX_PACKET ordinal=10 device_timestamp_us=189228 channel=37 rssi_dbm=111 lqi=0 crc_ok=True payload_hex='04 A5 03 00 01 61 E9' marker_present=True
2026-10-15T22:16:43.473499Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.008871800000634 packets=10 async_errors=0
2026-10-15T22:16:43.474022Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:16:43.987177Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.513140499999281 async_errors=0
2026-10-15T22:16:43.987677Z RX_STOP_BEGIN
2026-10-15T22:16:44.031220Z RX_STOP_ACK
2026-10-15T22:16:44.031695Z TX_STATS_AFTER_BEGIN
2026-10-15T22:16:44.205529Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:16:44.205636Z RX_STATS_AFTER_BEGIN
2026-10-15T22:16:44.375046Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:16:44.375143Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:16:44.375845Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:16:44.375905Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:16:44.376584Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:16:44.376655Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:16:44.376709Z TEST_CLASSIFICATION label='campaign-wisun-150k-run-01' preset='wisun_915_fsk_150k' classification='PASS-RF'
2026-10-15T22:16:44.376778Z TEST_END exit_code=0

---

## Prueba 3 — `wisun_915_fsk_150k`: PASS-RF (10/10) ✅

La tercera prueba de la campaña terminó correctamente. **CatSniffer #3 recibió diez paquetes con CRC válido y el marcador esperado**, y CatSniffer #2 obtuvo diez respuestas satisfactorias de la API de transmisión.

Con este resultado, hemos completado **tres presets nuevos consecutivos con PASS-RF**. Según la sección 12 del manual, corresponde repetir el control positivo antes de continuar.

### 1. Resultado experimental

| Parámetro                      | Resultado observado          |
| ------------------------------ | ---------------------------- |
| Identificador                  | `campaign-wisun-150k-run-01` |
| Inicio                         | 2026-10-15, 22:16:36 UTC     |
| TX                             | CatSniffer #2 — `COM33`      |
| RX                             | CatSniffer #3 — `COM88`      |
| Preset                         | `wisun_915_fsk_150k`         |
| Frecuencia configurada         | 902.2 MHz                    |
| Modulación solicitada          | FSK (`mod_type=0`)           |
| Symbol rate                    | 150 000 símbolos/s           |
| `deviation`                    | 200                          |
| `rx_bw`                        | 90                           |
| Potencia solicitada            | 0 dBm                        |
| Marcador                       | `A5 03 00 01`                |
| Transmisiones aceptadas        | **10/10**                    |
| Paquetes CRC-good con marcador | **10/10**                    |
| Errores asíncronos RX/TX       | **0/0**                      |
| Excepciones fatales            | **0**                        |
| Limpieza                       | Correcta                     |
| **Clasificación**              | **PASS-RF**                  |

### 2. Evidencia de recepción

Los diez paquetes contienen:

```text
04 A5 03 00 01 61 E9
```

Todos presentan `crc_ok=True` y `marker_present=True`.

Los contadores del receptor también son consistentes con la recepción:

| Contador RX   | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

Se observó `RX_STOP_ACK` y ambas desconexiones terminaron correctamente.

**Conclusión experimental:** bajo las condiciones ensayadas, la configuración solicitada de FSK a 150 000 símbolos/s permitió recibir diez paquetes CRC-good entre ambos CatSniffer.

Esto no demuestra por sí solo conformidad con Wi-SUN ni confirma mediante instrumentación independiente los parámetros físicos de la señal.

### 3. Anomalías persistentes

Se repiten las observaciones de las pruebas anteriores:

* `channel=37`, aunque se solicitó `channel=0`.
* `lqi=0` en todos los paquetes.
* RSSI entre −128 y +111 dBm, con valores positivos poco plausibles.

Por ahora, estos campos deben considerarse **metadatos no validados**. No los utilizaremos para comparar la calidad del enlace entre presets.

---

## 4. Estado de la campaña

| Prueba             | Preset               | Resultado       |
| ------------------ | -------------------- | --------------- |
| Control inicial    | `wisun_915_fsk_50k`  | ✅ PASS-RF 10/10 |
| 1                  | `gfsk_902_50k`       | ✅ PASS-RF 10/10 |
| 2                  | `wisun_915_fsk_100k` | ✅ PASS-RF 10/10 |
| 3                  | `wisun_915_fsk_150k` | ✅ PASS-RF 10/10 |
| Control intermedio | `wisun_915_fsk_50k`  | ⏳ Pendiente     |
| 4                  | `wisun_915_fsk_200k` | Pendiente       |

**Balance:** 3/3 presets nuevos con PASS-RF; 30/30 paquetes recibidos con CRC válido y marcador correcto, distribuidos en tres ejecuciones.

---

## Paso 19 — Repetir el control positivo

Ahora debemos comprobar que el enlace continúa funcionando con la configuración de referencia.

Esto es importante porque, si una prueba posterior falla, necesitaremos distinguir entre:

1. Un problema específico del nuevo preset.
2. Una degradación o cambio del estado experimental de los CatSniffer.

### Condiciones del control

| Parámetro              | Valor                            |
| ---------------------- | -------------------------------- |
| Preset                 | `wisun_915_fsk_50k`              |
| Frecuencia configurada | 902.2 MHz                        |
| Modulación solicitada  | FSK                              |
| Symbol rate            | 50 000 símbolos/s                |
| Potencia solicitada    | 0 dBm                            |
| Marcador               | `DE AD BE EF`                    |
| Transmisiones          | 10                               |
| Identificador          | `control-after-3-presets-run-01` |

No necesitamos modificar el script ni repetir `boot`, `exit` o `band2` mientras los dispositivos conserven su estado de preparación.

### Ejecuta una sola vez

Tras confirmar que ambos dispositivos siguen conectados y que los puertos COM están disponibles:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_50k `
    DEADBEEF `
    control-after-3-presets-run-01
```

**Este comando realizará diez transmisiones RF a la frecuencia configurada de 902.2 MHz.**

### Resultado esperado

Para considerar satisfactorio el control, buscaremos:

* Diez transmisiones aceptadas.
* Diez paquetes recibidos con CRC válido y marcador `DE AD BE EF`.
* Cero errores asíncronos.
* Contadores RX consistentes.
* Limpieza correcta de ambas conexiones.

**No avances todavía a `wisun_915_fsk_200k`.** Compararemos primero este control con el inicial.

Envíame la salida completa, desde `TEST_BEGIN` hasta `TEST_END`, para cerrar el primer bloque de tres presets de la campaña.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_50k `
>>     DEADBEEF `
>>     control-after-3-presets-run-01
2026-10-15T22:21:16.361793Z TEST_BEGIN checkpoint='manual-campaign' label='control-after-3-presets-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_50k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='DE AD BE EF' tx_power_dbm=0 tx_count=10
2026-10-15T22:21:16.361958Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:21:17.482401Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:21:17.482528Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:21:18.597394Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:21:18.597537Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:21:18.627502Z TX_SET_PHY_ACK
2026-10-15T22:21:18.627604Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:21:18.660146Z RX_SET_PHY_ACK
2026-10-15T22:21:18.660263Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:21:18.692507Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:21:18.692607Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:21:18.726474Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:21:18.726570Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:21:18.757520Z TX_SET_POWER_ACK
2026-10-15T22:21:18.757703Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:21:18.924780Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:21:18.924881Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:21:19.086312Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:21:19.086410Z RX_START_BEGIN
2026-10-15T22:21:19.117522Z RX_START_ACK
2026-10-15T22:21:19.117626Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:21:19.418392Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30072780000045896
2026-10-15T22:21:19.418827Z TX_ATTEMPT_BEGIN index=1 payload='DE AD BE EF'
2026-10-15T22:21:19.462007Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:21:19.562780Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10026189999916824
2026-10-15T22:21:19.563285Z TX_ATTEMPT_BEGIN index=2 payload='DE AD BE EF'
2026-10-15T22:21:19.606939Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:21:19.707584Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10016140000152518
2026-10-15T22:21:19.707753Z TX_ATTEMPT_BEGIN index=3 payload='DE AD BE EF'
2026-10-15T22:21:19.740038Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:21:19.840844Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10048689999894123
2026-10-15T22:21:19.841261Z TX_ATTEMPT_BEGIN index=4 payload='DE AD BE EF'
2026-10-15T22:21:19.885541Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:21:19.985873Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10009809999974095
2026-10-15T22:21:19.986094Z TX_ATTEMPT_BEGIN index=5 payload='DE AD BE EF'
2026-10-15T22:21:20.022505Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:21:20.123092Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.1004222999981721
2026-10-15T22:21:20.123278Z TX_ATTEMPT_BEGIN index=6 payload='DE AD BE EF'
2026-10-15T22:21:20.158319Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:21:20.258617Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10016729999915697
2026-10-15T22:21:20.258773Z TX_ATTEMPT_BEGIN index=7 payload='DE AD BE EF'
2026-10-15T22:21:20.291845Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:21:20.392512Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.1005077000008896
2026-10-15T22:21:20.392943Z TX_ATTEMPT_BEGIN index=8 payload='DE AD BE EF'
2026-10-15T22:21:20.427051Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:21:20.527652Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10041300000011688
2026-10-15T22:21:20.527801Z TX_ATTEMPT_BEGIN index=9 payload='DE AD BE EF'
2026-10-15T22:21:20.558010Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:21:20.658791Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10062250000191852
2026-10-15T22:21:20.658970Z TX_ATTEMPT_BEGIN index=10 payload='DE AD BE EF'
2026-10-15T22:21:20.694365Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:21:20.794976Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10048610000012559
2026-10-15T22:21:20.795083Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:21:21.795855Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0007274000017787
2026-10-15T22:21:21.796349Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:21:21.918656Z RX_PACKET ordinal=1 device_timestamp_us=1279465 channel=37 rssi_dbm=100 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.036602Z RX_PACKET ordinal=2 device_timestamp_us=1280029 channel=37 rssi_dbm=72 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.152620Z RX_PACKET ordinal=3 device_timestamp_us=1280592 channel=37 rssi_dbm=-13 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.273912Z RX_PACKET ordinal=4 device_timestamp_us=1281115 channel=37 rssi_dbm=-122 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.388684Z RX_PACKET ordinal=5 device_timestamp_us=1281681 channel=37 rssi_dbm=90 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.508499Z RX_PACKET ordinal=6 device_timestamp_us=1282215 channel=37 rssi_dbm=-7 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.618321Z RX_PACKET ordinal=7 device_timestamp_us=1282744 channel=37 rssi_dbm=-63 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.735842Z RX_PACKET ordinal=8 device_timestamp_us=1283269 channel=37 rssi_dbm=-52 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.850997Z RX_PACKET ordinal=9 device_timestamp_us=1283795 channel=37 rssi_dbm=27 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:22.970027Z RX_PACKET ordinal=10 device_timestamp_us=1284308 channel=37 rssi_dbm=-6 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:21:23.810442Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.014108999999735 packets=10 async_errors=0
2026-10-15T22:21:23.810578Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:21:24.319411Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5088225999970746 async_errors=0
2026-10-15T22:21:24.319548Z RX_STOP_BEGIN
2026-10-15T22:21:24.353501Z RX_STOP_ACK
2026-10-15T22:21:24.353666Z TX_STATS_AFTER_BEGIN
2026-10-15T22:21:24.535421Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:21:24.535522Z RX_STATS_AFTER_BEGIN
2026-10-15T22:21:24.694564Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:21:24.694666Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:21:24.695365Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:21:24.695431Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:21:24.695947Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:21:24.696020Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:21:24.696084Z TEST_CLASSIFICATION label='control-after-3-presets-run-01' preset='wisun_915_fsk_50k' classification='PASS-RF'
2026-10-15T22:21:24.696139Z TEST_END exit_code=0

---

## Paso 19 — Control positivo intermedio: PASS-RF (10/10) ✅

El control `control-after-3-presets-run-01` terminó correctamente.

**Resultado:** después de ejecutar tres presets diferentes, la configuración de referencia `wisun_915_fsk_50k` volvió a recibir diez paquetes con CRC válido y el marcador esperado.

Esto permite **cerrar satisfactoriamente el primer bloque de la campaña OTA**.

### 1. Evidencia del control

| Parámetro                      | Resultado observado      |
| ------------------------------ | ------------------------ |
| Fecha                          | 2026-10-15, 22:21:16 UTC |
| Preset                         | `wisun_915_fsk_50k`      |
| Frecuencia configurada         | 902.2 MHz                |
| Modulación solicitada          | FSK                      |
| Symbol rate                    | 50 000 símbolos/s        |
| TX / RX                        | `COM33` / `COM88`        |
| Marcador                       | `DE AD BE EF`            |
| Transmisiones aceptadas        | **10/10**                |
| Paquetes recibidos             | **10/10**                |
| CRC válido y marcador correcto | **10/10**                |
| Errores asíncronos             | **0**                    |
| Excepciones                    | **0**                    |
| Limpieza                       | Correcta                 |
| **Clasificación**              | **PASS-RF**              |

Los diez paquetes recibidos contienen:

```text
04 DE AD BE EF 33 C8
```

Los contadores RX confirman:

```text
rx_ok:       0 → 10
rx_crc_err:  0 → 0
rx_drop:     0 → 0
rx_overflow: 0 → 0
```

Además, se observaron `RX_STOP_ACK`, `TX_DISCONNECT_COMPLETE` y `RX_DISCONNECT_COMPLETE`.

### 2. Comparación con el control inicial

| Indicador                | Control inicial | Control intermedio |
| ------------------------ | --------------: | -----------------: |
| TX aceptados             |              10 |                 10 |
| RX CRC-good con marcador |              10 |                 10 |
| Errores asíncronos       |               0 |                  0 |
| `rx_crc_err`             |               0 |                  0 |
| `rx_drop`                |               0 |                  0 |
| Resultado                |         PASS-RF |            PASS-RF |

**Conclusión:** no observamos una regresión en la entrega de paquetes del control positivo después de los tres primeros presets.

Esto respalda la continuidad operativa del enlace, aunque no demuestra que todos los parámetros físicos de RF hayan permanecido constantes.

### 3. Anomalías que permanecen abiertas

También se repiten:

* `channel=37` cuando solicitamos `channel=0`.
* `lqi=0` en todos los paquetes.
* RSSI con valores positivos poco plausibles, hasta +100 dBm en esta ejecución.

Estas observaciones siguen pendientes de investigación. No utilizaremos RSSI, LQI ni el canal reportado para sacar conclusiones sobre rendimiento RF.

---

## 4. Estado actualizado de la campaña

| Ejecución          | Preset               | Resultado   |
| ------------------ | -------------------- | ----------- |
| Control inicial    | `wisun_915_fsk_50k`  | ✅ 10/10     |
| Prueba 1           | `gfsk_902_50k`       | ✅ 10/10     |
| Prueba 2           | `wisun_915_fsk_100k` | ✅ 10/10     |
| Prueba 3           | `wisun_915_fsk_150k` | ✅ 10/10     |
| Control intermedio | `wisun_915_fsk_50k`  | ✅ 10/10     |
| Prueba 4           | `wisun_915_fsk_200k` | ⏳ Pendiente |

**Balance acumulado observado:**

* **3/3 presets nuevos:** PASS-RF.
* **2/2 controles positivos:** PASS-RF.
* **50/50 paquetes:** CRC-good con marcador correcto, distribuidos en cinco ejecuciones.

No debemos interpretar 50/50 como una tasa de éxito general del sistema: corresponde exclusivamente a estas cinco ejecuciones controladas.

---

## Paso 20 — Verificar `wisun_915_fsk_200k`

Continuamos con el cuarto preset, todavía dentro de la frecuencia de 902.2 MHz.

Antes de transmitir, comprobaremos sus parámetros directamente en `PROP_PRESETS`.

Ejecuta únicamente:

```powershell
$PRESET = 'wisun_915_fsk_200k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

**Este comando no transmite RF.**

Envíame el resultado. Verificaremos que la frecuencia y los demás parámetros correspondan al preset previsto antes de ejecutar la cuarta prueba OTA.

**Siguiente objetivo:** `wisun_915_fsk_200k`, marcador `A5 04 00 01`, diez transmisiones y sin modificar el script de campaña.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wisun_915_fsk_200k {'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 200000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}

---

## Paso 20 — Configuración de `wisun_915_fsk_200k` verificada ✅

La salida confirma que el preset está definido en `PROP_PRESETS` con los siguientes parámetros:

| Parámetro              | Valor                |
| ---------------------- | -------------------- |
| Preset                 | `wisun_915_fsk_200k` |
| Frecuencia configurada | **902.2 MHz**        |
| Modulación solicitada  | FSK (`mod_type=0`)   |
| Symbol rate            | 200 000 símbolos/s   |
| `deviation`            | 200                  |
| `rx_bw`                | 90                   |
| `sync_word`            | `0x930B51DE`         |

**Observación:** aunque el nombre del preset contiene `915`, su frecuencia configurada sigue siendo **902.2 MHz**.

La definición del preset está verificada en Python. Su funcionamiento OTA todavía no se ha comprobado.

---

## Paso 21 — Ejecutar la cuarta prueba OTA

Continuaremos con el mismo procedimiento utilizado en las pruebas anteriores, sin modificar `manual_preset_runner.py`.

### Condiciones experimentales

| Parámetro           | Valor                        |
| ------------------- | ---------------------------- |
| TX                  | CatSniffer #2 — `COM33`      |
| RX                  | CatSniffer #3 — `COM88`      |
| Preset              | `wisun_915_fsk_200k`         |
| Frecuencia          | 902.2 MHz                    |
| Potencia solicitada | 0 dBm                        |
| Transmisiones       | 10                           |
| Marcador            | `A5 04 00 01`                |
| Identificador       | `campaign-wisun-200k-run-01` |

El control positivo intermedio acaba de obtener **PASS-RF 10/10**, por lo que tenemos una referencia reciente del funcionamiento del enlace.

No necesitamos repetir `boot`, `exit` ni `band2` mientras ambos dispositivos mantengan su estado de preparación.

### Comando

Confirma que los CatSniffer continúan conectados y que los puertos COM están disponibles.

Ejecuta **una sola vez**:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_200k `
    A5040001 `
    campaign-wisun-200k-run-01
```

**Este comando realizará diez transmisiones RF a la frecuencia configurada de 902.2 MHz.**

### Qué verificaremos

Revisaremos las transmisiones aceptadas, los paquetes recibidos con CRC válido y marcador correcto, los contadores RX, los errores asíncronos y la limpieza de ambas conexiones.

**Estado de la campaña:**

* 3/3 presets nuevos: **PASS-RF**.
* 2/2 controles positivos: **PASS-RF**.
* Cuarta prueba: **pendiente de ejecución**.

Envíame la salida completa desde `TEST_BEGIN` hasta `TEST_END`. No repitas la prueba ni avances al siguiente preset hasta que revisemos el resultado.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_200k `
>>     A5040001 `
>>     campaign-wisun-200k-run-01
2026-10-15T22:23:13.879777Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wisun-200k-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_200k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 200000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606} marker='A5 04 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:23:13.879905Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:23:14.993653Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:23:14.993752Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:23:16.113982Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:23:16.114148Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:23:16.147025Z TX_SET_PHY_ACK
2026-10-15T22:23:16.147123Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:23:16.177502Z RX_SET_PHY_ACK
2026-10-15T22:23:16.177653Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 200000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}
2026-10-15T22:23:16.211247Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:23:16.211400Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 200000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}
2026-10-15T22:23:16.244496Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:23:16.244600Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:23:16.276437Z TX_SET_POWER_ACK
2026-10-15T22:23:16.276531Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:23:16.449163Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:23:16.449275Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:23:16.618135Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:23:16.618237Z RX_START_BEGIN
2026-10-15T22:23:16.648497Z RX_START_ACK
2026-10-15T22:23:16.648604Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:23:16.949046Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.3004338999999163
2026-10-15T22:23:16.949201Z TX_ATTEMPT_BEGIN index=1 payload='A5 04 00 01'
2026-10-15T22:23:16.980937Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:23:17.081277Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10022279999975581
2026-10-15T22:23:17.081427Z TX_ATTEMPT_BEGIN index=2 payload='A5 04 00 01'
2026-10-15T22:23:17.113815Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:23:17.214562Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10061580000183312
2026-10-15T22:23:17.214824Z TX_ATTEMPT_BEGIN index=3 payload='A5 04 00 01'
2026-10-15T22:23:17.248101Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:23:17.348325Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.1000845999988087
2026-10-15T22:23:17.348568Z TX_ATTEMPT_BEGIN index=4 payload='A5 04 00 01'
2026-10-15T22:23:17.377137Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:23:17.477638Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10038879999774508
2026-10-15T22:23:17.477744Z TX_ATTEMPT_BEGIN index=5 payload='A5 04 00 01'
2026-10-15T22:23:17.511662Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:23:17.612001Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10015760000169394
2026-10-15T22:23:17.612227Z TX_ATTEMPT_BEGIN index=6 payload='A5 04 00 01'
2026-10-15T22:23:17.649209Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:23:17.750030Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10067759999947157
2026-10-15T22:23:17.750537Z TX_ATTEMPT_BEGIN index=7 payload='A5 04 00 01'
2026-10-15T22:23:17.793649Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:23:17.894244Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.1001085999996576
2026-10-15T22:23:17.894389Z TX_ATTEMPT_BEGIN index=8 payload='A5 04 00 01'
2026-10-15T22:23:17.927709Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:23:18.028172Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10031240000171238
2026-10-15T22:23:18.028673Z TX_ATTEMPT_BEGIN index=9 payload='A5 04 00 01'
2026-10-15T22:23:18.069883Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:23:18.170647Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10058890000073006
2026-10-15T22:23:18.171067Z TX_ATTEMPT_BEGIN index=10 payload='A5 04 00 01'
2026-10-15T22:23:18.211019Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:23:18.311539Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.1002251999998407
2026-10-15T22:23:18.312048Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:23:19.312931Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0008951999989222
2026-10-15T22:23:19.313069Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:23:19.431982Z RX_PACKET ordinal=1 device_timestamp_us=1738568 channel=37 rssi_dbm=-25 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:19.549920Z RX_PACKET ordinal=2 device_timestamp_us=1739085 channel=37 rssi_dbm=87 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:19.665193Z RX_PACKET ordinal=3 device_timestamp_us=1739606 channel=37 rssi_dbm=125 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:19.784233Z RX_PACKET ordinal=4 device_timestamp_us=1740129 channel=37 rssi_dbm=107 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:19.905850Z RX_PACKET ordinal=5 device_timestamp_us=1740633 channel=37 rssi_dbm=53 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:20.024752Z RX_PACKET ordinal=6 device_timestamp_us=1741158 channel=37 rssi_dbm=44 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:20.141020Z RX_PACKET ordinal=7 device_timestamp_us=1741700 channel=37 rssi_dbm=0 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:20.260780Z RX_PACKET ordinal=8 device_timestamp_us=1742260 channel=37 rssi_dbm=55 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:20.375265Z RX_PACKET ordinal=9 device_timestamp_us=1742786 channel=37 rssi_dbm=-27 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:20.496221Z RX_PACKET ordinal=10 device_timestamp_us=1743342 channel=37 rssi_dbm=-120 lqi=0 crc_ok=True payload_hex='04 A5 04 00 01 E1 86' marker_present=True
2026-10-15T22:23:21.324148Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.011038100001315 packets=10 async_errors=0
2026-10-15T22:23:21.324611Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:23:21.839939Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5153032999987772 async_errors=0
2026-10-15T22:23:21.840446Z RX_STOP_BEGIN
2026-10-15T22:23:21.881428Z RX_STOP_ACK
2026-10-15T22:23:21.881696Z TX_STATS_AFTER_BEGIN
2026-10-15T22:23:22.049959Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:23:22.050056Z RX_STATS_AFTER_BEGIN
2026-10-15T22:23:22.220852Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:23:22.220957Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:23:22.221743Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:23:22.221826Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:23:22.222402Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:23:22.222512Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:23:22.222588Z TEST_CLASSIFICATION label='campaign-wisun-200k-run-01' preset='wisun_915_fsk_200k' classification='PASS-RF'
2026-10-15T22:23:22.222654Z TEST_END exit_code=0

---

## Prueba 4 — `wisun_915_fsk_200k`: PASS-RF (10/10) ✅

La cuarta prueba OTA terminó correctamente. **CatSniffer #3 recibió diez paquetes con CRC válido y el marcador esperado**, mientras que CatSniffer #2 obtuvo diez respuestas satisfactorias de la API de transmisión.

### 1. Resultado experimental

| Parámetro                      | Resultado observado          |
| ------------------------------ | ---------------------------- |
| Identificador                  | `campaign-wisun-200k-run-01` |
| Inicio                         | 2026-10-15, 22:23:13 UTC     |
| TX                             | CatSniffer #2 — `COM33`      |
| RX                             | CatSniffer #3 — `COM88`      |
| Preset                         | `wisun_915_fsk_200k`         |
| Frecuencia configurada         | 902.2 MHz                    |
| Modulación solicitada          | FSK (`mod_type=0`)           |
| Symbol rate                    | 200 000 símbolos/s           |
| `deviation`                    | 200                          |
| `rx_bw`                        | 90                           |
| Potencia solicitada            | 0 dBm                        |
| Marcador                       | `A5 04 00 01`                |
| Transmisiones aceptadas        | **10/10**                    |
| Paquetes CRC-good con marcador | **10/10**                    |
| Errores asíncronos RX/TX       | **0/0**                      |
| Excepciones fatales            | **0**                        |
| Limpieza                       | Correcta                     |
| **Clasificación**              | **PASS-RF**                  |

### 2. Evidencia de recepción

Los diez paquetes presentan:

```text
payload_hex='04 A5 04 00 01 E1 86'
crc_ok=True
marker_present=True
```

Sus `device_timestamp_us` son distintos y crecientes.

Los contadores del receptor también respaldan el resultado:

| Contador RX   | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

Además, se observaron `RX_STOP_ACK` y ambas desconexiones terminaron correctamente.

**Conclusión experimental:** bajo las condiciones ensayadas, ambos CatSniffer intercambiaron correctamente diez paquetes utilizando la configuración solicitada de FSK a 200 000 símbolos/s.

Esto demuestra recepción OTA entre los dispositivos, pero no verifica de manera independiente la modulación ni la velocidad de símbolos emitida.

### 3. Anomalías persistentes

Se repiten las anomalías observadas anteriormente:

* `channel=37`, aunque solicitamos `channel=0`.
* `lqi=0` en todos los paquetes.
* RSSI entre −120 y +125 dBm, incluyendo valores positivos físicamente poco plausibles.

Por tanto, mantenemos estos metadatos como **no validados**. No deben utilizarse para comparar el rendimiento RF de los presets.

---

## 4. Estado actualizado de la campaña

| Ejecución          | Preset               | Resultado   |
| ------------------ | -------------------- | ----------- |
| Control inicial    | `wisun_915_fsk_50k`  | ✅ 10/10     |
| Prueba 1           | `gfsk_902_50k`       | ✅ 10/10     |
| Prueba 2           | `wisun_915_fsk_100k` | ✅ 10/10     |
| Prueba 3           | `wisun_915_fsk_150k` | ✅ 10/10     |
| Control intermedio | `wisun_915_fsk_50k`  | ✅ 10/10     |
| Prueba 4           | `wisun_915_fsk_200k` | ✅ 10/10     |
| Prueba 5           | `wisun_915_fsk_300k` | ⏳ Pendiente |

**Balance acumulado observado:**

* **4/4 presets nuevos:** PASS-RF.
* **2/2 controles positivos:** PASS-RF.
* **60/60 paquetes:** CRC-good con marcador correcto, distribuidos en seis ejecuciones.

El último control positivo se realizó inmediatamente antes de esta prueba, por lo que no corresponde repetirlo todavía.

---

## Paso 22 — Verificar `wisun_915_fsk_300k`

Continuamos con el quinto preset de la campaña.

Antes de transmitir, verificaremos sus parámetros directamente en `PROP_PRESETS`.

Ejecuta únicamente:

```powershell
$PRESET = 'wisun_915_fsk_300k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

**Este comando no transmite RF.**

Envíame la salida para verificar la frecuencia, modulación, symbol rate, deviation, RX bandwidth y sync word.

**Siguiente objetivo:** `wisun_915_fsk_300k`, marcador `A5 05 00 01`, diez transmisiones, manteniendo el mismo procedimiento experimental.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wisun_915_fsk_300k {'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 300000, 'deviation': 200, 'rx_bw': 94, 'sync_word': 2466992606}

---

## Paso 22 — Configuración de `wisun_915_fsk_300k` verificada ✅

La salida confirma que el preset está definido en `PROP_PRESETS` con los siguientes parámetros:

| Parámetro              | Valor                  |
| ---------------------- | ---------------------- |
| Preset                 | `wisun_915_fsk_300k`   |
| Frecuencia configurada | **902.2 MHz**          |
| Modulación solicitada  | FSK (`mod_type=0`)     |
| Symbol rate            | **300 000 símbolos/s** |
| `deviation`            | 200                    |
| `rx_bw`                | 94                     |
| `sync_word`            | `0x930B51DE`           |

**Observación:** aunque el nombre contiene `915`, la frecuencia configurada sigue siendo **902.2 MHz**.

Esta verificación confirma la definición del preset en Python, no su aplicación efectiva en el transceptor ni su funcionamiento OTA.

---

## Paso 23 — Ejecutar la quinta prueba OTA

Continuaremos con el procedimiento experimental utilizado en las pruebas anteriores.

### Condiciones

| Parámetro              | Valor                        |
| ---------------------- | ---------------------------- |
| TX                     | CatSniffer #2 — `COM33`      |
| RX                     | CatSniffer #3 — `COM88`      |
| Preset                 | `wisun_915_fsk_300k`         |
| Frecuencia configurada | 902.2 MHz                    |
| Potencia solicitada    | 0 dBm                        |
| Transmisiones          | 10                           |
| Marcador               | `A5 05 00 01`                |
| Identificador          | `campaign-wisun-300k-run-01` |

El control positivo intermedio y la prueba anterior obtuvieron **PASS-RF 10/10**.

No necesitamos repetir `boot`, `exit` ni `band2` mientras ambos dispositivos conserven su estado de preparación.

### Comando

Confirma que los dos CatSniffer siguen conectados y que los puertos COM están disponibles.

Ejecuta **una sola vez**:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_300k `
    A5050001 `
    campaign-wisun-300k-run-01
```

**Este comando solicitará diez transmisiones RF a 902.2 MHz.**

### Criterios de evaluación

Revisaremos:

* Diez respuestas `TX_ATTEMPT_ACK`.
* Diez paquetes `RX_PACKET` con `crc_ok=True` y `marker_present=True`.
* Incremento de `rx_ok` consistente con los paquetes recibidos.
* Ausencia de errores asíncronos y excepciones.
* Ejecución correcta de `RX_STOP` y desconexión de ambos dispositivos.

**Estado actual:** 4/4 presets nuevos y 2/2 controles positivos con PASS-RF.

Si esta quinta prueba obtiene PASS-RF, avanzaremos al sexto preset. Después de completar tres presets nuevos desde el último control, repetiremos el control positivo.

Envíame la salida completa desde `TEST_BEGIN` hasta `TEST_END`. No repitas la prueba ni avances al siguiente preset antes de revisar el resultado.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_300k `
>>     A5050001 `
>>     campaign-wisun-300k-run-01
2026-10-15T22:24:23.655441Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wisun-300k-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_300k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 300000, 'deviation': 200, 'rx_bw': 94, 'sync_word': 2466992606} marker='A5 05 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:24:23.655580Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:24:24.774898Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:24:24.775016Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:24:25.902303Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:24:25.902401Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:24:25.936913Z TX_SET_PHY_ACK
2026-10-15T22:24:25.937008Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:24:25.969835Z RX_SET_PHY_ACK
2026-10-15T22:24:25.969943Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 300000, 'deviation': 200, 'rx_bw': 94, 'sync_word': 2466992606}
2026-10-15T22:24:26.002651Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:24:26.002751Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 300000, 'deviation': 200, 'rx_bw': 94, 'sync_word': 2466992606}
2026-10-15T22:24:26.035869Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:24:26.036282Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:24:26.065646Z TX_SET_POWER_ACK
2026-10-15T22:24:26.065744Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:24:26.238638Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:24:26.238737Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:24:26.407765Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:24:26.407966Z RX_START_BEGIN
2026-10-15T22:24:26.445595Z RX_START_ACK
2026-10-15T22:24:26.445763Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:24:26.746206Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.3004371999995783
2026-10-15T22:24:26.746318Z TX_ATTEMPT_BEGIN index=1 payload='A5 05 00 01'
2026-10-15T22:24:26.780985Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:24:26.881194Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10008739999830141
2026-10-15T22:24:26.881344Z TX_ATTEMPT_BEGIN index=2 payload='A5 05 00 01'
2026-10-15T22:24:26.918176Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:24:27.018573Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10024100000009639
2026-10-15T22:24:27.018711Z TX_ATTEMPT_BEGIN index=3 payload='A5 05 00 01'
2026-10-15T22:24:27.048664Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:24:27.149128Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10034640000230866
2026-10-15T22:24:27.149252Z TX_ATTEMPT_BEGIN index=4 payload='A5 05 00 01'
2026-10-15T22:24:27.183968Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:24:27.284531Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10042780000003404
2026-10-15T22:24:27.284674Z TX_ATTEMPT_BEGIN index=5 payload='A5 05 00 01'
2026-10-15T22:24:27.322401Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:24:27.422937Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10040000000299187
2026-10-15T22:24:27.423093Z TX_ATTEMPT_BEGIN index=6 payload='A5 05 00 01'
2026-10-15T22:24:27.458112Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:24:27.558842Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10047690000283183
2026-10-15T22:24:27.558959Z TX_ATTEMPT_BEGIN index=7 payload='A5 05 00 01'
2026-10-15T22:24:27.591187Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:24:27.691558Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10021989999950165
2026-10-15T22:24:27.692070Z TX_ATTEMPT_BEGIN index=8 payload='A5 05 00 01'
2026-10-15T22:24:27.735473Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:24:27.836437Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10044959999868297
2026-10-15T22:24:27.836950Z TX_ATTEMPT_BEGIN index=9 payload='A5 05 00 01'
2026-10-15T22:24:27.871182Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:24:27.971452Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10014680000313092
2026-10-15T22:24:27.971610Z TX_ATTEMPT_BEGIN index=10 payload='A5 05 00 01'
2026-10-15T22:24:28.005738Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:24:28.106507Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10059890000047744
2026-10-15T22:24:28.107091Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:24:29.107967Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0008883000009519
2026-10-15T22:24:29.108125Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:24:29.220014Z RX_PACKET ordinal=1 device_timestamp_us=2011215 channel=37 rssi_dbm=-25 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:29.339023Z RX_PACKET ordinal=2 device_timestamp_us=2011742 channel=37 rssi_dbm=-27 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:29.456226Z RX_PACKET ordinal=3 device_timestamp_us=2012279 channel=37 rssi_dbm=-71 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:29.569786Z RX_PACKET ordinal=4 device_timestamp_us=2012789 channel=37 rssi_dbm=-28 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:29.684795Z RX_PACKET ordinal=5 device_timestamp_us=2013317 channel=37 rssi_dbm=-84 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:29.801790Z RX_PACKET ordinal=6 device_timestamp_us=2013859 channel=37 rssi_dbm=53 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:29.911692Z RX_PACKET ordinal=7 device_timestamp_us=2014389 channel=37 rssi_dbm=126 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:30.028362Z RX_PACKET ordinal=8 device_timestamp_us=2014909 channel=37 rssi_dbm=102 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:30.148892Z RX_PACKET ordinal=9 device_timestamp_us=2015475 channel=37 rssi_dbm=-100 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:30.263481Z RX_PACKET ordinal=10 device_timestamp_us=2016001 channel=37 rssi_dbm=-108 lqi=0 crc_ok=True payload_hex='04 A5 05 00 01 61 91' marker_present=True
2026-10-15T22:24:31.120577Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0124228000022413 packets=10 async_errors=0
2026-10-15T22:24:31.120988Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:24:31.634283Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5132699999994657 async_errors=0
2026-10-15T22:24:31.634725Z RX_STOP_BEGIN
2026-10-15T22:24:31.675095Z RX_STOP_ACK
2026-10-15T22:24:31.675205Z TX_STATS_AFTER_BEGIN
2026-10-15T22:24:31.848393Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:24:31.848587Z RX_STATS_AFTER_BEGIN
2026-10-15T22:24:32.009298Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:24:32.009397Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:24:32.010167Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:24:32.010301Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:24:32.010856Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:24:32.010934Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:24:32.010990Z TEST_CLASSIFICATION label='campaign-wisun-300k-run-01' preset='wisun_915_fsk_300k' classification='PASS-RF'
2026-10-15T22:24:32.011044Z TEST_END exit_code=0

---

## Prueba 5 — `wisun_915_fsk_300k`: PASS-RF (10/10) ✅

La quinta prueba OTA terminó correctamente. **CatSniffer #3 recibió diez paquetes con CRC válido y el marcador esperado**, mientras que CatSniffer #2 obtuvo diez respuestas satisfactorias de la API de transmisión.

Con este resultado, llevamos **cinco presets nuevos consecutivos con PASS-RF**.

### 1. Resultado experimental

| Parámetro                      | Resultado observado          |
| ------------------------------ | ---------------------------- |
| Identificador                  | `campaign-wisun-300k-run-01` |
| Inicio                         | 2026-10-15, 22:24:23 UTC     |
| TX                             | CatSniffer #2 — `COM33`      |
| RX                             | CatSniffer #3 — `COM88`      |
| Preset                         | `wisun_915_fsk_300k`         |
| Frecuencia configurada         | 902.2 MHz                    |
| Modulación solicitada          | FSK (`mod_type=0`)           |
| Symbol rate                    | 300 000 símbolos/s           |
| `deviation`                    | 200                          |
| `rx_bw`                        | 94                           |
| Potencia solicitada            | 0 dBm                        |
| Marcador                       | `A5 05 00 01`                |
| Transmisiones aceptadas        | **10/10**                    |
| Paquetes CRC-good con marcador | **10/10**                    |
| Errores asíncronos RX/TX       | **0/0**                      |
| Excepciones fatales            | **0**                        |
| Limpieza                       | Correcta                     |
| **Clasificación**              | **PASS-RF**                  |

### 2. Evidencia de recepción

Los diez paquetes contienen:

```text
04 A5 05 00 01 61 91
```

En todos se observa:

```text
crc_ok=True
marker_present=True
```

Los diez `device_timestamp_us` son distintos y crecientes.

Los contadores RX evolucionaron así:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se confirmó `RX_STOP_ACK` y la desconexión correcta de ambos dispositivos.

**Conclusión experimental:** la configuración solicitada de FSK a 300 000 símbolos/s permitió recibir diez paquetes CRC-good entre ambos CatSniffer bajo las condiciones ensayadas.

No obstante, la recepción correcta no constituye una medición independiente del symbol rate, la desviación de frecuencia ni la modulación realmente emitida.

### 3. Anomalías persistentes

Se repiten las anomalías observadas en las pruebas anteriores:

* `channel=37`, aunque se configuró `channel=0`.
* `lqi=0` en los diez paquetes.
* RSSI entre −108 y +126 dBm, incluyendo valores positivos físicamente poco plausibles.

Estos metadatos continúan **sin validar** y no deben emplearse para comparar el rendimiento RF de los presets.

---

## 4. Estado actualizado de la campaña

| Ejecución          | Preset               | Resultado   |
| ------------------ | -------------------- | ----------- |
| Control inicial    | `wisun_915_fsk_50k`  | ✅ 10/10     |
| Prueba 1           | `gfsk_902_50k`       | ✅ 10/10     |
| Prueba 2           | `wisun_915_fsk_100k` | ✅ 10/10     |
| Prueba 3           | `wisun_915_fsk_150k` | ✅ 10/10     |
| Control intermedio | `wisun_915_fsk_50k`  | ✅ 10/10     |
| Prueba 4           | `wisun_915_fsk_200k` | ✅ 10/10     |
| Prueba 5           | `wisun_915_fsk_300k` | ✅ 10/10     |
| Prueba 6           | `gfsk_915_50k`       | ⏳ Pendiente |

**Balance acumulado observado:**

* **5/5 presets nuevos:** PASS-RF.
* **2/2 controles positivos:** PASS-RF.
* **70/70 paquetes:** CRC-good con marcador correcto, distribuidos en siete ejecuciones.

Desde el último control positivo hemos ejecutado dos presets nuevos. Si el siguiente obtiene PASS-RF, corresponderá repetir el control.

---

## Paso 24 — Verificar `gfsk_915_50k`

El siguiente preset introduce un cambio importante: **pasaremos de las configuraciones a 902.2 MHz a un preset denominado `gfsk_915_50k`**.

Antes de realizar cualquier transmisión, debemos comprobar su frecuencia exacta. Si efectivamente utiliza 915 MHz, también confirmaremos que las antenas, la ruta RF y las condiciones regulatorias permiten realizar la prueba.

### Ejecuta únicamente:

```powershell
$PRESET = 'gfsk_915_50k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

**Este comando no transmite RF.**

Envíame la salida para verificar los parámetros.

**No ejecutes todavía la prueba OTA.** Primero confirmaremos la frecuencia y las condiciones de transmisión antes de avanzar al sexto preset.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
gfsk_915_50k {'frequency_hz': 915000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}

---

## Paso 24 — Configuración de `gfsk_915_50k` verificada ✅

La salida confirma que el preset está definido en `PROP_PRESETS` con los siguientes parámetros:

| Parámetro              | Valor               |
| ---------------------- | ------------------- |
| Preset                 | `gfsk_915_50k`      |
| Frecuencia configurada | **915 MHz**         |
| Modulación solicitada  | GFSK (`mod_type=1`) |
| Symbol rate            | 50 000 símbolos/s   |
| `deviation`            | 100                 |
| `rx_bw`                | 82                  |
| `sync_word`            | `0x930B51DE`        |

### ¿Qué cambia respecto a las pruebas anteriores?

Hasta ahora, todos los presets evaluados utilizaban **902.2 MHz**. Este será el primero de la campaña en solicitar **915 MHz**.

También cambiaremos respecto al preset anterior de FSK a GFSK, aunque ya validamos previamente un preset GFSK a 902.2 MHz.

**Lo verificado:** la definición del preset en Python.

**Lo pendiente:** comprobar que la configuración se aplica correctamente al transceptor y que existe recepción OTA a 915 MHz.

---

## Paso 25 — Confirmación previa a la transmisión

Antes de ejecutar la sexta prueba, necesitamos confirmar las condiciones físicas y regulatorias.

Por favor, confirma:

1. **Antenas:** ¿ambos CatSniffer siguen utilizando antenas compatibles con 902–928 MHz?
2. **Ruta RF:** ¿ambos dispositivos conservan la configuración `band2` de la preparación anterior, sin reinicios ni desconexiones?
3. **Autorización:** ¿confirmas que podemos realizar diez transmisiones de prueba a **915 MHz**, con potencia solicitada de 0 dBm, en tu entorno de laboratorio y conforme a las condiciones regulatorias aplicables?

La confirmación de `band2` se refiere al estado lógico configurado; todavía no constituye una medición independiente de la ruta RF.

**No ejecutes todavía ningún comando de transmisión.**

Una vez confirmados estos tres puntos, continuaremos con:

* **Preset:** `gfsk_915_50k`
* **Marcador:** `A5 06 00 01`
* **Identificador:** `campaign-gfsk-915-50k-run-01`
* **Cantidad:** 10 transmisiones

Si obtenemos `PASS-RF`, repetiremos el control positivo, ya que habremos completado tres presets nuevos desde el último control.

---

2/3 confirmaciones son válidas, excepto por band2 ¿Puedes darme el comando para verificar ello?

---

Sí. Podemos verificar el estado lógico de `band2` mediante el **Shell de FeralRF**, sin transmitir RF ni modificar la configuración.

En CatSniffer #2 (TX), el puerto Shell es `COM35`; en CatSniffer #3 (RX), es `COM87`.

### Paso 25.1 — Consultar el estado de banda

Ejecuta este comando en PowerShell:

```powershell
python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'band\r\n'),time.sleep(0.4),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
```

**Importante:** el comando envía `band` como consulta. No envía `band2`, por lo que no debería cambiar la selección de banda.

**Resultado esperado:** una respuesta del Shell que permita identificar la banda activa en cada dispositivo.

En las pruebas anteriores, después de ejecutar `band2`, el Shell respondió `Band: 1`. Por tanto, debemos interpretar la respuesta según el comportamiento real del firmware, no asumir que el número reportado coincide con el sufijo del comando.

Envíame la salida de ambos puertos. Si `band` no está implementado como consulta, no ejecutaremos comandos adicionales a ciegas: revisaremos primero la interfaz disponible.

**Límite de la comprobación:** consultar el Shell permite verificar el estado lógico reportado por el firmware, pero no confirma físicamente la posición del RF switch.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'band\r\n'),time.sleep(0.4),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
COM35 "band\rUnknown command. Type 'help'\r\n\n"
COM87 "band\rUnknown command. Type 'help'\r\n\n"

---

## Paso 25.1 — Consulta de banda no disponible

La respuesta es idéntica en ambos CatSniffer:

```text
COM35 "band\rUnknown command. Type 'help'\r\n\n"
COM87 "band\rUnknown command. Type 'help'\r\n\n"
```

**Resultado: INCONCLUSO.** El comando `band` no está reconocido por el Shell de ninguno de los dispositivos.

Esto significa que:

* No hemos verificado el estado actual de la selección de banda.
* No hay evidencia de que esta consulta haya modificado la configuración.
* Tampoco podemos concluir que `band2` se haya perdido.

La confirmación anterior de `band2` sigue siendo evidencia histórica, pero no una lectura del estado actual.

### Paso 25.2 — Consultar los comandos disponibles

El propio firmware recomienda utilizar `help`. Vamos a consultar esa información en ambos dispositivos.

Ejecuta únicamente:

```powershell
python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'help\r\n'),time.sleep(0.5),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
```

Este comando **no solicita transmisiones RF ni cambios de banda**.

Envíame las respuestas de `COM35` y `COM87`. Revisaremos si existe algún comando de consulta antes de decidir cómo verificar o restablecer la selección de banda.

**La prueba OTA a 915 MHz permanece pendiente.**

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'help\r\n'),time.sleep(0.5),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
COM35 'help\rCommands:\r\n  help           - Show available commands\r\n  boot           - CC1352 bootloader mode\r\n  exit           - Return to passthrough\r\n  band1          - 2.4GHz band\r\n  band2          - SUB-GHz band\r\n  band3          - LoRa band\r\n  reboot         - RP2040 USB bootloader\r\n  status         - Device status\r\n  loss_reset     - Reset CC1352 loss counters\r\n  fw_version     - Show firmware build version\r\n  lora_freq      - Set frequency (Hz)\r\n  lora_sf        - Set spreading factor\r\n  lora_bw        - Set bandwidth (kHz)\r\n  lora_cr        - Set coding rate\r\n  lora_power     - Set TX power (dBm)\r\n  lora_mode      - stream|command mode\r\n  lora_preamble  - Set preamble length\r\n  lora_syncword  - Set sync word (private|public|0xNN)\r\n  lora_iq        - normal|inverted IQ\r\n  lora_config    - Show LoRa config\r\n  lora_apply     - Apply pending config\r\n  cc1352_fw_id   - set|get|clear|list CC1352 FW ID\r\n  fsk_freq       - Set FSK frequency (Hz)\r\n  fsk_bitrate    - Set FSK bitrate (bps)\r\n  fsk_fdev       - Set FSK freq deviation (Hz)\r\n  fsk_bw         - Set FSK RX bandwidth\r\n  fsk_power      - Set FSK TX power (dBm)\r\n  fsk_preamble   - Set FSK preamble len\r\n  fsk_syncword   - Set FSK sync word (hex)\r\n  fsk_crc        - Enable/disable CRC\r\n  fsk_whitening  - Enable/disable whitening\r\n  fsk_pktlen     - fixed|variable packet length\r\n  fsk_payload    - Set payload/max len (1-255)\r\n  fsk_bt         - Set GFSK BT (off|0.3|0.5|0.7|1.0)\r\n  fsk_config     - Show FSK config\r\n  fsk_apply      - Apply FSK config\r\n  modulation     - lora|fsk modulation\r\n  radio          - Forward radio cmd (TEST/FSKRX/FSKTX/TX)\r\n  identify       - Blink all LEDs to identify this board\r\n\r\nFSK bandwidths (kHz):\r\n  4.8, 5.8, 7.3, 9.7, 11.7, 14.6, 19.5, 23.4,\r\n  29.3, 39.0, 46.9, 58.6, 78.2, 93.8, 117.3,\r\n  156.2, 187.2, 234.3, 312.0, 373.6, 467.0\r\n\n'
COM87 'help\rCommands:\r\n  help           - Show available commands\r\n  boot           - CC1352 bootloader mode\r\n  exit           - Return to passthrough\r\n  band1          - 2.4GHz band\r\n  band2          - SUB-GHz band\r\n  band3          - LoRa band\r\n  reboot         - RP2040 USB bootloader\r\n  status         - Device status\r\n  loss_reset     - Reset CC1352 loss counters\r\n  fw_version     - Show firmware build version\r\n  lora_freq      - Set frequency (Hz)\r\n  lora_sf        - Set spreading factor\r\n  lora_bw        - Set bandwidth (kHz)\r\n  lora_cr        - Set coding rate\r\n  lora_power     - Set TX power (dBm)\r\n  lora_mode      - stream|command mode\r\n  lora_preamble  - Set preamble length\r\n  lora_syncword  - Set sync word (private|public|0xNN)\r\n  lora_iq        - normal|inverted IQ\r\n  lora_config    - Show LoRa config\r\n  lora_apply     - Apply pending config\r\n  cc1352_fw_id   - set|get|clear|list CC1352 FW ID\r\n  fsk_freq       - Set FSK frequency (Hz)\r\n  fsk_bitrate    - Set FSK bitrate (bps)\r\n  fsk_fdev       - Set FSK freq deviation (Hz)\r\n  fsk_bw         - Set FSK RX bandwidth\r\n  fsk_power      - Set FSK TX power (dBm)\r\n  fsk_preamble   - Set FSK preamble len\r\n  fsk_syncword   - Set FSK sync word (hex)\r\n  fsk_crc        - Enable/disable CRC\r\n  fsk_whitening  - Enable/disable whitening\r\n  fsk_pktlen     - fixed|variable packet length\r\n  fsk_payload    - Set payload/max len (1-255)\r\n  fsk_bt         - Set GFSK BT (off|0.3|0.5|0.7|1.0)\r\n  fsk_config     - Show FSK config\r\n  fsk_apply      - Apply FSK config\r\n  modulation     - lora|fsk modulation\r\n  radio          - Forward radio cmd (TEST/FSKRX/FSKTX/TX)\r\n  identify       - Blink all LEDs to identify this board\r\n\r\nFSK bandwidths (kHz):\r\n  4.8, 5.8, 7.3, 9.7, 11.7, 14.6, 19.5, 23.4,\r\n  29.3, 39.0, 46.9, 58.6, 78.2, 93.8, 117.3,\r\n  156.2, 187.2, 234.3, 312.0, 373.6, 467.0\r\n\n'

---

## Paso 25.2 — Comandos del Shell identificados ✅

Ambos CatSniffer presentan la misma lista de comandos. Hemos confirmado que el Shell dispone de:

* `band1` — 2.4 GHz
* `band2` — SUB-GHz
* `band3` — LoRa
* `status` — Device status

**Hallazgo:** `help` no muestra un comando `band` para consultar directamente la banda activa. Sin embargo, `status` podría proporcionar esa información.

Antes de modificar el estado de los dispositivos, comprobaremos qué reporta.

### Paso 25.3 — Consultar `status` en ambos CatSniffer

Ejecuta únicamente:

```powershell
python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'status\r\n'),time.sleep(0.5),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
```

Este comando consulta el estado del dispositivo; no solicita transmisiones RF ni cambios de banda.

### Qué buscamos

Queremos determinar si `status` informa la selección actual del RF switch.

Si reporta la banda, contrastaremos su valor con el comportamiento previamente observado al ejecutar `band2`.

Si no la reporta, podremos establecer explícitamente `band2` en ambos dispositivos y comprobar sus respuestas, pero documentaremos esa operación como **configuración aplicada**, no como lectura independiente del estado anterior.

**Estado de la validación:** las antenas y la autorización para transmitir a 915 MHz están confirmadas; la selección de banda continúa pendiente.

Envíame las respuestas de `COM35` y `COM87`. No ejecutes todavía `band2` ni la prueba OTA.

---

Un detallazo mi amigazo, no sé cómo funcionan las "bandas" ¿Qué son y que relación específica tienen con Sub y 2.4Ghz? PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'status\r\n'),time.sleep(0.5),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
COM35 'status\rMode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)\r\nCC1352 loss: uart_overrun=0, ring_dropped=0 bytes\r\n\n'
COM87 'status\rMode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)\r\nCC1352 loss: uart_overrun=0, ring_dropped=0 bytes\r\n\n'

---

¡Buena pregunta! De hecho, entender qué es una **banda de frecuencia** y cómo CatSniffer selecciona su ruta de radiofrecuencia es fundamental para explicar por qué estamos utilizando `band2` en las pruebas de FeralRF.

Además, tu última ejecución nos proporciona información importante: **ambos CatSniffer reportan `Band: 1`**, que corresponde al estado lógico asociado con Sub-GHz en el firmware examinado.

Vamos por partes.

## 1. ¿Qué es una banda de frecuencia?

Una **banda de frecuencia** es un intervalo del espectro electromagnético.

Las ondas de radio se distinguen, entre otras propiedades, por su frecuencia: cuántas oscilaciones realizan por segundo.

Por ejemplo:

| Frecuencia | Equivalencia                         |
| ---------- | ------------------------------------ |
| 433 MHz    | 433 millones de ciclos por segundo   |
| 868 MHz    | 868 millones de ciclos por segundo   |
| 915 MHz    | 915 millones de ciclos por segundo   |
| 2.4 GHz    | 2 400 millones de ciclos por segundo |

La diferencia entre MHz y GHz es únicamente la unidad:

**1 GHz = 1 000 MHz.**

Así, cuando decimos que un dispositivo trabaja en la banda de 2.4 GHz, nos referimos a una región del espectro cercana a esa frecuencia.

En cambio, cuando hablamos de **Sub-GHz**, nos referimos a frecuencias inferiores a 1 GHz.

Por ejemplo:

* 433 MHz pertenece a Sub-GHz.
* 868 MHz pertenece a Sub-GHz.
* 902.2 MHz pertenece a Sub-GHz.
* 915 MHz pertenece a Sub-GHz.
* 2.4 GHz **no** pertenece a Sub-GHz.

Un detalle importante: Sub-GHz no es una única banda regulatoria ni significa que todas sus frecuencias estén permitidas en cualquier país.

---

## 2. ¿Por qué CatSniffer tiene diferentes bandas?

Porque CatSniffer está diseñado para trabajar con distintas tecnologías y frecuencias de radio.

Sin embargo, hay dos conceptos que debemos separar:

**La frecuencia que configura el transceptor no es necesariamente lo mismo que la ruta RF que selecciona la placa.**

Para entenderlo, imaginemos una transmisión.

El recorrido conceptual es:

```text
           PC
            │
            │ USB
            ▼
         CatSniffer
            │
            ▼
        Firmware
            │
            ▼
       Transceptor RF
            │
            │ Señal de radio
            ▼
       RF switch
            │
            ├── Ruta para 2.4 GHz
            │
            └── Ruta para Sub-GHz
                     │
                     ▼
                  Antena
```

*Este esquema es conceptual; no representa todavía todas las conexiones físicas de la revisión de PCB.*

Aquí aparecen dos responsabilidades diferentes.

### A. Configurar el transceptor

El transceptor genera o recibe señales de radio.

Por ejemplo, FeralRF solicita:

```python
frequency_hz = 915000000
```

Esto significa:

> Configura el transceptor para operar a 915 MHz.

También configura otros parámetros, como modulación, velocidad de símbolos y desviación.

### B. Seleccionar la ruta RF

La placa puede contener circuitos diferentes para conducir las señales según su frecuencia.

Para seleccionar entre esas rutas se utiliza un componente denominado **RF switch**.

Piensa en él como un conmutador electrónico que conecta una señal RF con una de varias rutas disponibles.

El comando `band2` actúa sobre este segundo aspecto: la selección de la ruta correspondiente a Sub-GHz.

Esta distinción es esencial:

**Configurar 915 MHz en el transceptor no garantiza, por sí solo, que la señal esté conectada a la ruta RF adecuada.**

---

## 3. ¿Qué significan `band1`, `band2` y `band3` en CatSniffer?

Tu comando `help` nos proporcionó esta información directamente desde el firmware:

| Comando Shell | Descripción reportada |
| ------------- | --------------------- |
| `band1`       | 2.4GHz band           |
| `band2`       | SUB-GHz band          |
| `band3`       | LoRa band             |

No debemos interpretar estos números como frecuencias.

Por ejemplo, `band2` **no significa 2 GHz**.

Son nombres de comandos utilizados por el firmware para seleccionar una ruta de hardware.

### ¿Qué ocurre internamente con `band2`?

La auditoría previa del control de banda documentó este recorrido:

```text
PC
 │
 │ Comando "band2"
 ▼
RP2040
 │
 │ change_band(SUBGIG_1)
 ▼
GPIO8 / GPIO9 / GPIO10
 │
 │ Señales CTF1 / CTF2 / CTF3
 ▼
Circuito de conmutación RF
 │
 ▼
Ruta Sub-GHz
```

Según el código examinado en esa auditoría:

| Comando | Selección lógica | CTF1 | CTF2 | CTF3 |
| ------- | ---------------- | ---: | ---: | ---: |
| `band1` | 2.4 GHz          |    0 |    1 |    0 |
| `band2` | Sub-GHz          |    0 |    0 |    1 |

**Fuente documental del proyecto:** `Seguimiento documental OTA Sub-GHz - ruta RF y control de banda.md`, sección de control de banda. También está documentado en `Guía enfocada OTA FeralRF - TX RX con dos CatSniffer(1).md`.

Hay que distinguir dos niveles de evidencia:

* **Código auditado:** muestra qué valores intenta establecer el RP2040.
* **Hardware:** todavía no hemos medido directamente esos niveles eléctricos ni la posición física del RF switch.

Por tanto, el diagrama describe la intención de control identificada en el firmware, no una medición eléctrica realizada durante esta prueba.

---

## 4. Entonces, ¿qué significa tu resultado `Band: 1`?

Tus dos dispositivos respondieron:

**CatSniffer #2 — TX (`COM35`)**

```text
Mode: 0, Band: 1, Radio: LoRa,
LoRa: initialized, LoRa Mode: Stream,
FW: v3.1.0.1,
CC1352 FW: feralrf_cc1352 (custom)
```

**CatSniffer #3 — RX (`COM87`)**

```text
Mode: 0, Band: 1, Radio: LoRa,
LoRa: initialized, LoRa Mode: Stream,
FW: v3.1.0.1,
CC1352 FW: feralrf_cc1352 (custom)
```

La correspondencia documentada es:

| Estado reportado | Interpretación              |
| ---------------- | --------------------------- |
| `Band: 0`        | Selección lógica de 2.4 GHz |
| `Band: 1`        | Selección lógica de Sub-GHz |

Por eso **`band2` produce `Band: 1`**. No existe una contradicción: el nombre del comando y el valor de la enumeración interna son distintos.

### ¿Qué podemos concluir?

✅ Ambos dispositivos reportan actualmente la selección lógica correspondiente a Sub-GHz.

✅ Esta condición es coherente con la preparación de nuestra campaña OTA a 902.2 MHz.

✅ No necesitamos ejecutar nuevamente `band2` para establecer el estado lógico que ya reportan.

⚠️ `Band: 1` es un estado almacenado o reportado por el firmware; no es una lectura eléctrica independiente del RF switch.

### ¿Y por qué aparece `Radio: LoRa`?

Es otra pregunta interesante.

CatSniffer tiene más de una interfaz y subsistema de radio. El campo `Radio: LoRa` pertenece al estado que expone el Shell del RP2040.

**No debemos concluir automáticamente que FeralRF está utilizando LoRa.**

Nuestras pruebas FeralRF configuran el `CC1352` mediante Cat-Bridge y solicitan `PHY.PROPRIETARY_GFSK`.

Además, ya hemos observado recepción OTA de los paquetes de prueba.

Por tanto, el campo `Radio: LoRa` no invalida los resultados, aunque todavía conviene verificar en el firmware del RP2040 qué variable representa exactamente y cómo se relaciona con las distintas rutas.

---

## 5. ¿Qué relación tiene esto con nuestra prueba de 915 MHz?

Hasta ahora trabajamos a **902.2 MHz**.

El siguiente preset utiliza:

```python
{
    'frequency_hz': 915000000,
    'mod_type': 1,
    'symbol_rate': 50000,
    'deviation': 100,
    'rx_bw': 82,
    'sync_word': 2466992606
}
```

Tanto 902.2 MHz como 915 MHz pertenecen a Sub-GHz.

Por eso, desde el punto de vista de la selección general de ruta, **no necesitamos cambiar de `band2` a otra banda**.

Lo que cambia es la frecuencia solicitada al transceptor:

```text
Pruebas anteriores                 Próxima prueba

     902.2 MHz                        915 MHz
         │                                │
         ▼                                ▼
      Sub-GHz                          Sub-GHz
         │                                │
         └──────── band2 ────────────────┘
```

Esto no implica que el rendimiento sea idéntico. La respuesta de la antena, los filtros y otros componentes puede variar dentro de una misma banda.

---

## 6. Estado de nuestra validación

| Elemento                              | Estado                     |
| ------------------------------------- | -------------------------- |
| Antenas compatibles con 902–928 MHz   | ✅ Confirmado por ti        |
| Autorización para la prueba a 915 MHz | ✅ Confirmada por ti        |
| Preset `gfsk_915_50k`                 | ✅ Definición inspeccionada |
| Shell TX `Band: 1`                    | ✅ Observado                |
| Shell RX `Band: 1`                    | ✅ Observado                |
| Posición física del RF switch         | ⚠️ No medida               |
| Transmisión y recepción OTA a 915 MHz | ⏳ Pendiente                |

**Conclusión:** tenemos confirmadas las tres condiciones operativas que establecimos para continuar. La selección lógica Sub-GHz está verificada en ambos dispositivos, aunque la conmutación física sigue pendiente de validación independiente.

### Siguiente paso — Prueba 6

Ejecuta:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    gfsk_915_50k `
    A5060001 `
    campaign-gfsk-915-50k-run-01
```

La prueba solicitará diez transmisiones a **915 MHz**, con potencia configurada de **0 dBm**.

Envíame el registro completo hasta `TEST_END` para comprobar si los diez paquetes llegan correctamente.

**Una idea clave para recordar:** en CatSniffer, seleccionar la banda mediante `band2` y configurar la frecuencia mediante FeralRF son operaciones diferentes, pero ambas forman parte del recorrido necesario para una comunicación RF correcta.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'status\r\n'),time.sleep(0.5),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
COM35 'status\rMode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)\r\nCC1352 loss: uart_overrun=0, ring_dropped=0 bytes\r\n\n'
COM87 'status\rMode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)\r\nCC1352 loss: uart_overrun=0, ring_dropped=0 bytes\r\n\n'
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     gfsk_915_50k `
>>     A5060001 `
>>     campaign-gfsk-915-50k-run-01
2026-10-15T22:35:46.444058Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-gfsk-915-50k-run-01' tx_port='COM33' rx_port='COM88' preset_name='gfsk_915_50k' preset={'frequency_hz': 915000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='A5 06 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:35:46.444204Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:35:47.559279Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:35:47.559387Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:35:48.687124Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:35:48.687223Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:35:48.719972Z TX_SET_PHY_ACK
2026-10-15T22:35:48.720070Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:35:48.753636Z RX_SET_PHY_ACK
2026-10-15T22:35:48.753738Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 915000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:35:48.785183Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:35:48.785284Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 915000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:35:48.818892Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:35:48.818984Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:35:48.856746Z TX_SET_POWER_ACK
2026-10-15T22:35:48.856837Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:35:49.025741Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:35:49.025836Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:35:49.199782Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:35:49.199898Z RX_START_BEGIN
2026-10-15T22:35:49.228111Z RX_START_ACK
2026-10-15T22:35:49.229658Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:35:49.529916Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30169110000133514
2026-10-15T22:35:49.530024Z TX_ATTEMPT_BEGIN index=1 payload='A5 06 00 01'
2026-10-15T22:35:49.563279Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:35:49.663567Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10014719999890076
2026-10-15T22:35:49.663678Z TX_ATTEMPT_BEGIN index=2 payload='A5 06 00 01'
2026-10-15T22:35:49.697793Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:35:49.798176Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10023500000170316
2026-10-15T22:35:49.798680Z TX_ATTEMPT_BEGIN index=3 payload='A5 06 00 01'
2026-10-15T22:35:49.842306Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:35:49.942960Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10009899999931804
2026-10-15T22:35:49.943387Z TX_ATTEMPT_BEGIN index=4 payload='A5 06 00 01'
2026-10-15T22:35:49.985038Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:35:50.085562Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10016080000059446
2026-10-15T22:35:50.085682Z TX_ATTEMPT_BEGIN index=5 payload='A5 06 00 01'
2026-10-15T22:35:50.121173Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:35:50.221415Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.1001276000024518
2026-10-15T22:35:50.221538Z TX_ATTEMPT_BEGIN index=6 payload='A5 06 00 01'
2026-10-15T22:35:50.254860Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:35:50.355347Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10032030000002123
2026-10-15T22:35:50.355862Z TX_ATTEMPT_BEGIN index=7 payload='A5 06 00 01'
2026-10-15T22:35:50.398675Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:35:50.499267Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10017399999924237
2026-10-15T22:35:50.499405Z TX_ATTEMPT_BEGIN index=8 payload='A5 06 00 01'
2026-10-15T22:35:50.531667Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:35:50.632298Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10046480000164593
2026-10-15T22:35:50.632445Z TX_ATTEMPT_BEGIN index=9 payload='A5 06 00 01'
2026-10-15T22:35:50.667954Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:35:50.768633Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10056130000157282
2026-10-15T22:35:50.768766Z TX_ATTEMPT_BEGIN index=10 payload='A5 06 00 01'
2026-10-15T22:35:50.798724Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:35:50.899448Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10051659999953699
2026-10-15T22:35:50.899603Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:35:51.900493Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0008454999988317
2026-10-15T22:35:51.900994Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:35:52.030996Z RX_PACKET ordinal=1 device_timestamp_us=484044 channel=37 rssi_dbm=107 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.145801Z RX_PACKET ordinal=2 device_timestamp_us=484565 channel=37 rssi_dbm=75 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.260946Z RX_PACKET ordinal=3 device_timestamp_us=485093 channel=37 rssi_dbm=110 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.375309Z RX_PACKET ordinal=4 device_timestamp_us=485658 channel=37 rssi_dbm=65 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.488000Z RX_PACKET ordinal=5 device_timestamp_us=486213 channel=37 rssi_dbm=-32 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.608091Z RX_PACKET ordinal=6 device_timestamp_us=486744 channel=37 rssi_dbm=127 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.721676Z RX_PACKET ordinal=7 device_timestamp_us=487269 channel=37 rssi_dbm=-74 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.832355Z RX_PACKET ordinal=8 device_timestamp_us=487829 channel=37 rssi_dbm=1 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:52.942498Z RX_PACKET ordinal=9 device_timestamp_us=488349 channel=37 rssi_dbm=-95 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:53.057067Z RX_PACKET ordinal=10 device_timestamp_us=488882 channel=37 rssi_dbm=-124 lqi=0 crc_ok=True payload_hex='04 A5 06 00 01 61 AD' marker_present=True
2026-10-15T22:35:53.904392Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0034064999999828 packets=10 async_errors=0
2026-10-15T22:35:53.904539Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:35:54.412325Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.507766000002448 async_errors=0
2026-10-15T22:35:54.412477Z RX_STOP_BEGIN
2026-10-15T22:35:54.441185Z RX_STOP_ACK
2026-10-15T22:35:54.441279Z TX_STATS_AFTER_BEGIN
2026-10-15T22:35:54.616352Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:35:54.616461Z RX_STATS_AFTER_BEGIN
2026-10-15T22:35:54.783774Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:35:54.783875Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:35:54.784596Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:35:54.784656Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:35:54.785154Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:35:54.785228Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:35:54.785283Z TEST_CLASSIFICATION label='campaign-gfsk-915-50k-run-01' preset='gfsk_915_50k' classification='PASS-RF'
2026-10-15T22:35:54.785361Z TEST_END exit_code=0

---

## Prueba 6 — `gfsk_915_50k`: PASS-RF (10/10) ✅

**Primera prueba OTA de la campaña a 915 MHz completada satisfactoriamente.**

El registro muestra que CatSniffer #2 solicitó diez transmisiones y CatSniffer #3 recibió diez paquetes con el marcador esperado y `crc_ok=True`.

Además, ambos dispositivos reportaron `Band: 1` antes de la prueba, coherente con la selección lógica de la ruta Sub-GHz.

### 1. Resultado experimental

| Parámetro                    | Resultado                      |
| ---------------------------- | ------------------------------ |
| Identificador                | `campaign-gfsk-915-50k-run-01` |
| Fecha                        | 2026-10-15, 22:35 UTC          |
| TX                           | CatSniffer #2 — `COM33`        |
| RX                           | CatSniffer #3 — `COM88`        |
| Preset                       | `gfsk_915_50k`                 |
| Frecuencia solicitada        | **915 MHz**                    |
| Modulación solicitada        | GFSK (`mod_type=1`)            |
| Symbol rate solicitado       | 50 000 símbolos/s              |
| `deviation`                  | 100                            |
| `rx_bw`                      | 82                             |
| Potencia solicitada          | 0 dBm                          |
| Marcador                     | `A5 06 00 01`                  |
| TX ACK                       | **10/10**                      |
| RX con CRC válido y marcador | **10/10**                      |
| Errores asíncronos TX/RX     | **0/0**                        |
| Excepciones                  | Ninguna                        |
| Limpieza                     | Correcta                       |
| **Clasificación**            | **PASS-RF**                    |

### 2. Evidencia de recepción

Los diez paquetes recibidos contienen:

```text
04 A5 06 00 01 61 AD
```

En todos los casos:

```text
crc_ok=True
marker_present=True
```

También se observa:

```text
RX_STATS_BEFORE: rx_ok=0
RX_STATS_AFTER:  rx_ok=10

rx_crc_err=0
rx_drop=0
rx_overflow=0
```

La ejecución terminó correctamente:

```text
TEST_CLASSIFICATION ... classification='PASS-RF'
TEST_END exit_code=0
```

**Interpretación:** existe evidencia de comunicación OTA funcional entre ambos CatSniffer bajo la configuración solicitada de 915 MHz.

No obstante, seguimos sin disponer de una medición independiente de la frecuencia emitida, la modulación efectiva o la potencia radiada.

### 3. Anomalías que permanecen abiertas

Se repiten tres comportamientos observados anteriormente:

| Campo      | Observación                   | Estado       |
| ---------- | ----------------------------- | ------------ |
| `channel`  | Siempre 37                    | Sin validar  |
| `lqi`      | Siempre 0                     | Sin validar  |
| `rssi_dbm` | Valores entre −124 y +127 dBm | No confiable |

Especialmente, los valores positivos de RSSI no deben interpretarse como mediciones físicas válidas.

Estas anomalías no impiden clasificar la prueba como PASS-RF según los criterios actuales, pero limitan lo que podemos concluir sobre la calidad del enlace.

---

## 4. Estado acumulado de la campaña

| Prueba             | Preset               | Frecuencia solicitada | Resultado   |
| ------------------ | -------------------- | --------------------: | ----------- |
| Control inicial    | `wisun_915_fsk_50k`  |             902.2 MHz | ✅ 10/10     |
| 1                  | `gfsk_902_50k`       |             902.2 MHz | ✅ 10/10     |
| 2                  | `wisun_915_fsk_100k` |             902.2 MHz | ✅ 10/10     |
| 3                  | `wisun_915_fsk_150k` |             902.2 MHz | ✅ 10/10     |
| Control intermedio | `wisun_915_fsk_50k`  |             902.2 MHz | ✅ 10/10     |
| 4                  | `wisun_915_fsk_200k` |             902.2 MHz | ✅ 10/10     |
| 5                  | `wisun_915_fsk_300k` |             902.2 MHz | ✅ 10/10     |
| **6**              | **`gfsk_915_50k`**   |           **915 MHz** | **✅ 10/10** |

**Balance:**

* **6/6 presets nuevos:** PASS-RF.
* **2/2 controles positivos:** PASS-RF.
* **80/80 paquetes recibidos:** CRC-good y marcador correcto.

Hemos completado tres presets nuevos desde el último control positivo: `wisun_915_fsk_200k`, `wisun_915_fsk_300k` y `gfsk_915_50k`.

Por tanto, corresponde repetir el control antes de avanzar.

---

## Paso 26 — Control positivo después de seis presets

Volveremos al preset de referencia:

* **Preset:** `wisun_915_fsk_50k`
* **Frecuencia:** 902.2 MHz
* **Marcador:** `DE AD BE EF`
* **Transmisiones:** 10
* **Objetivo:** comprobar que el enlace de referencia sigue funcionando después de los últimos tres cambios de configuración.

Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_50k `
    DEADBEEF `
    control-after-6-presets-run-01
```

**Criterio de aceptación:** diez TX ACK, diez paquetes RX con `crc_ok=True` y marcador correcto, sin errores asíncronos ni fallos de limpieza.

Envíame el registro completo hasta `TEST_END`.

Si el control vuelve a pasar, continuaremos con la inspección del preset `sidewalk_915_fsk_50k`.

**Conclusión provisional:** ya tenemos evidencia experimental de recepción OTA exitosa con configuraciones solicitadas tanto a 902.2 MHz como a 915 MHz, pero aún no de sus parámetros RF mediante instrumentación independiente.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_50k `
>>     DEADBEEF `
>>     control-after-6-presets-run-01
2026-10-15T22:36:43.364358Z TEST_BEGIN checkpoint='manual-campaign' label='control-after-6-presets-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_50k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='DE AD BE EF' tx_power_dbm=0 tx_count=10
2026-10-15T22:36:43.364495Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:36:44.484826Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:36:44.484921Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:36:45.595382Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:36:45.595479Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:36:45.628995Z TX_SET_PHY_ACK
2026-10-15T22:36:45.629135Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:36:45.662414Z RX_SET_PHY_ACK
2026-10-15T22:36:45.662585Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:36:45.696529Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:36:45.696641Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:36:45.730503Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:36:45.730619Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:36:45.762972Z TX_SET_POWER_ACK
2026-10-15T22:36:45.763121Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:36:47.778910Z TEST_FATAL_EXCEPTION exception_type='TimeoutError' exception="TimeoutError('Response timeout')"
Traceback (most recent call last):
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\experimental_artifacts\manual_preset_runner.py", line 106, in main
    tx_stats_before = tx.get_stats()
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 556, in get_stats
    cmd_id, seq, payload = self._read_response(
                           ~~~~~~~~~~~~~~~~~~~^
        timeout=timeout, expected={Response.STATS, Response.ERROR}
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 336, in _read_response
    raise TimeoutError("Response timeout")
feralrf.exceptions.TimeoutError: Response timeout
2026-10-15T22:36:47.800807Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:36:47.802027Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:36:47.802260Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:36:47.803725Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:36:47.803988Z TEST_SUMMARY tx_accepted=0 tx_requested=10 total_peer_packets=0 peer_crc_good_marker=0 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception="TimeoutError('Response timeout')"
2026-10-15T22:36:47.804176Z TEST_CLASSIFICATION label='control-after-6-presets-run-01' preset='wisun_915_fsk_50k' classification='BLOCKED'
2026-10-15T22:36:47.804332Z TEST_END exit_code=2

---

## Control después de seis presets — BLOCKED ⚠️

El control positivo **no pudo ejecutarse completamente**. Se produjo un `TimeoutError` al consultar las estadísticas del transmisor.

Es importante distinguirlo de un fallo de comunicación RF: **no se llegó a transmitir ningún paquete**.

### 1. Resultado observado

| Etapa                       | Resultado    |
| --------------------------- | ------------ |
| Inicialización TX (`COM33`) | ✅            |
| Inicialización RX (`COM88`) | ✅            |
| `set_phy()` TX/RX           | ✅ ACK        |
| `configure_prop()` TX/RX    | ✅ ACK        |
| `set_power(0)` TX           | ✅ ACK        |
| `tx.get_stats()` inicial    | ❌ Timeout    |
| `rx.start_rx()`             | No ejecutado |
| Transmisiones               | **0/10**     |
| Paquetes recibidos          | **0**        |
| Desconexión TX/RX           | ✅            |
| **Clasificación**           | **BLOCKED**  |

El punto exacto del fallo fue:

```text
TX_STATS_BEFORE_BEGIN
TEST_FATAL_EXCEPTION exception_type='TimeoutError'
exception="TimeoutError('Response timeout')"
```

El traceback identifica:

* `experimental_artifacts/manual_preset_runner.py`, línea 106: `tx.get_stats()`.
* `python/feralrf/radio.py`, línea 556: espera de respuesta `Response.STATS` o `Response.ERROR`.
* `python/feralrf/radio.py`, línea 336: se agota el tiempo de espera.

### 2. ¿Qué significa?

El Cat-Bridge del transmisor respondió correctamente a varios comandos antes del fallo.

Sin embargo, **no se recibió una respuesta válida a `get_stats()` dentro del tiempo establecido**.

Esto permite identificar un problema de comunicación o procesamiento de comandos, pero todavía no su causa.

Entre las hipótesis pendientes están:

* Una respuesta de estadísticas retrasada.
* Un problema transitorio de comunicación entre PC y firmware.
* Un estado interno del firmware que impide responder a `get_stats()`.
* Un problema de sincronización en el manejo de respuestas.

No tenemos evidencia suficiente para elegir una de ellas.

Tampoco podemos atribuir el timeout al cambio previo a 915 MHz: la relación temporal no demuestra causalidad.

### 3. Impacto sobre la campaña

Los seis presets nuevos que terminaron en PASS-RF **conservan sus resultados**.

Sin embargo, el control posterior a esos seis presets queda bloqueado.

| Indicador                                  | Estado      |
| ------------------------------------------ | ----------- |
| Presets nuevos PASS-RF                     | 6/6         |
| Controles PASS-RF                          | 2           |
| Control posterior a seis presets           | **BLOCKED** |
| Pruebas OTA completadas satisfactoriamente | 8           |
| Paquetes CRC-good con marcador             | 80/80       |
| Transmisiones en el control bloqueado      | 0           |

**No avanzaremos a `sidewalk_915_fsk_50k` hasta recuperar el control positivo.**

---

## Paso 27 — Diagnóstico no invasivo del estado

Antes de reiniciar dispositivos o repetir la prueba, comprobaremos qué estado reporta cada Shell.

Ejecuta únicamente:

```powershell
python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'status\r\n'),time.sleep(0.5),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
```

Queremos comprobar:

1. Que ambos Shell siguen respondiendo.
2. Que ambos reportan `Band: 1`.
3. Que los contadores `uart_overrun` y `ring_dropped` permanecen en cero o muestran cambios.

**No ejecutes todavía `boot`, `exit`, `band2` ni repitas el control.**

Primero conservaremos la evidencia del estado posterior al timeout. Después decidiremos si conviene repetir la consulta, restablecer el CC1352 o investigar el manejo de respuestas de `get_stats()`.

**Conclusión:** este resultado es un fallo de ejecución del procedimiento de validación, no una demostración de fallo del enlace RF.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import serial,time; ports=['COM35','COM87']; [(lambda s,p: (s.write(b'status\r\n'),time.sleep(0.5),print(p,repr(s.read_all().decode(errors='replace'))),s.close()))(serial.Serial(p,115200,timeout=1),p) for p in ports]"
COM35 'status\rMode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)\r\nCC1352 loss: uart_overrun=0, ring_dropped=0 bytes\r\n\n'
COM87 'status\rMode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)\r\nCC1352 loss: uart_overrun=0, ring_dropped=0 bytes\r\n\n'

---

## Paso 27 — Diagnóstico posterior al timeout completado ✅

Ambos CatSniffer siguen respondiendo correctamente a través del Shell.

| Indicador                 | CatSniffer #2 (TX)        | CatSniffer #3 (RX)        |
| ------------------------- | ------------------------- | ------------------------- |
| Puerto Shell              | `COM35`                   | `COM87`                   |
| `Mode`                    | 0                         | 0                         |
| `Band`                    | **1 (Sub-GHz)**           | **1 (Sub-GHz)**           |
| Firmware RP2040           | v3.1.0.1                  | v3.1.0.1                  |
| Firmware CC1352 reportado | `feralrf_cc1352 (custom)` | `feralrf_cc1352 (custom)` |
| `uart_overrun`            | 0                         | 0                         |
| `ring_dropped`            | 0                         | 0                         |
| Respuesta a `status`      | ✅                         | ✅                         |

### ¿Qué hemos comprobado?

**Observaciones:**

* El Shell de ambos dispositivos continúa operativo.
* Ambos reportan la selección lógica Sub-GHz.
* No se registran pérdidas en los contadores de UART que expone el RP2040.

**Lo que todavía no sabemos:**

El comando `status` consulta el estado del RP2040, pero **no demuestra que el CC1352 pueda responder correctamente a `get_stats()` a través de Cat-Bridge**.

Por tanto, el timeout anterior continúa sin explicación.

Tampoco hay evidencia que justifique reiniciar los dispositivos todavía.

---

## Paso 28 — Comprobar `get_stats()` de forma aislada

Antes de repetir el control OTA, comprobaremos si el comando que falló puede ejecutarse sin configurar el PHY ni transmitir paquetes.

Ejecuta únicamente este comando:

```powershell
python -c "from feralrf import Radio; ports=['COM33','COM88']; exec('for port in ports:\n print(\"PORT\",port,flush=True)\n r=Radio(port=port,baudrate=921600)\n try:\n  print(\"INIT\",r.init(),flush=True)\n  print(\"STATS\",r.get_stats(),flush=True)\n except Exception as e:\n  print(\"ERROR\",type(e).__name__,str(e),flush=True)\n finally:\n  r.disconnect()\n  print(\"DISCONNECTED\",port,flush=True)')"
```

### Objetivo

Comprobar independientemente:

1. Si `COM33` responde a `get_stats()`.
2. Si `COM88` responde a `get_stats()`.
3. Si el problema persiste después de cerrar la ejecución anterior.

Este diagnóstico **no solicita transmisiones RF ni modifica la selección de banda**.

### Cómo interpretaremos el resultado

* **Ambos responden:** el fallo anterior pudo ser transitorio, pero no quedará demostrada su causa. Podremos considerar una nueva ejecución controlada del preset de referencia.
* **Falla únicamente `COM33`:** investigaremos el camino de respuesta de estadísticas del transmisor.
* **Fallan ambos:** ampliaremos el diagnóstico hacia el protocolo Cat-Bridge o el estado compartido del firmware.

**Importante:** este diagnóstico no sustituye al control positivo OTA. El control `control-after-6-presets-run-01` permanece clasificado como **BLOCKED** y debe conservarse así en el registro histórico.

Envíame la salida completa de este comando antes de continuar.

---

## Paso 28 — Error de sintaxis en el comando de diagnóstico

El error se debe al comando que te proporcioné, no a FeralRF ni a los CatSniffer.

PowerShell interpretó incorrectamente las comillas dentro de `python -c`, provocando:

```text
SyntaxError: unterminated string literal
```

**No se ejecutó el diagnóstico:** Python no llegó a inicializar ningún dispositivo ni a llamar a `get_stats()`.

### Paso 28.1 — Repetir el diagnóstico con sintaxis compatible con PowerShell

Utilizaremos un bloque de Python enviado mediante una cadena multilínea de PowerShell. Esto evita los problemas de comillas anidadas.

Copia y ejecuta el bloque completo:

```powershell
@'
from feralrf import Radio

for port in ["COM33", "COM88"]:
    print(f"\nPORT: {port}", flush=True)

    radio = Radio(port=port, baudrate=921600)

    try:
        info = radio.init()
        print(f"INIT_OK: {info}", flush=True)

        stats = radio.get_stats()
        print(f"STATS_OK: {stats}", flush=True)

    except Exception as exc:
        print(
            f"ERROR: {type(exc).__name__}: {exc}",
            flush=True
        )

    finally:
        radio.disconnect()
        print(f"DISCONNECTED: {port}", flush=True)
'@ | python -
```

### ¿Qué comprobará?

| Dispositivo        | Puerto  | Operación                |
| ------------------ | ------- | ------------------------ |
| CatSniffer #2 (TX) | `COM33` | `init()` → `get_stats()` |
| CatSniffer #3 (RX) | `COM88` | `init()` → `get_stats()` |

Este procedimiento no solicita transmisiones RF ni modifica la banda seleccionada.

**El control OTA sigue clasificado como `BLOCKED`.** No repetiremos la transmisión hasta conocer el resultado de este diagnóstico.

Envíame la salida completa y continuaremos con un solo paso a la vez.

---

PORT: COM33
INIT_OK: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS_OK: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
DISCONNECTED: COM33

PORT: COM88
INIT_OK: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS_OK: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
DISCONNECTED: COM88

---

## Paso 28 — Diagnóstico de `get_stats()` superado ✅

**Ambos CatSniffer responden correctamente a `get_stats()` después del timeout anterior.**

Esto es una buena señal: hemos recuperado la comunicación con ambos dispositivos sin necesidad de reiniciarlos.

### 1. Resultados observados

| Comprobación        | CatSniffer #2 (TX) | CatSniffer #3 (RX) |
| ------------------- | ------------------ | ------------------ |
| Puerto Cat-Bridge   | `COM33`            | `COM88`            |
| `Radio.init()`      | ✅                  | ✅                  |
| `Radio.get_stats()` | ✅                  | ✅                  |
| `rx_ok`             | 0                  | 0                  |
| `rx_crc_err`        | 0                  | 0                  |
| `rx_drop`           | 0                  | 0                  |
| `rx_overflow`       | 0                  | 0                  |
| `disconnect()`      | ✅                  | ✅                  |

No aparecieron excepciones.

### 2. ¿Qué significa para el timeout anterior?

Recordemos que el control `control-after-6-presets-run-01` quedó bloqueado aquí:

```text
TX_STATS_BEFORE_BEGIN
TimeoutError: Response timeout
```

Ahora hemos ejecutado correctamente:

```text
COM33 → init() → get_stats() → disconnect()
COM88 → init() → get_stats() → disconnect()
```

**Conclusión:** el fallo anterior no es permanente ni se reproduce con una consulta aislada de estadísticas.

Sin embargo, todavía no podemos afirmar que fuera un problema transitorio de USB, UART o firmware. Esas siguen siendo hipótesis.

Además, el diagnóstico aislado no reproduce toda la secuencia que precedió al timeout:

```text
init()
  ↓
set_phy()
  ↓
configure_prop()
  ↓
set_power()
  ↓
get_stats()  ← Falló anteriormente
```

Por tanto, necesitamos comprobar nuevamente el procedimiento completo.

---

## 3. Estado de la campaña

Los resultados anteriores permanecen intactos:

* **6/6 presets nuevos:** PASS-RF.
* **2 controles positivos:** PASS-RF.
* **80 paquetes recibidos correctamente** en las ocho ejecuciones exitosas.
* **1 control bloqueado:** timeout antes de transmitir.
* **Diagnóstico posterior:** `get_stats()` responde en ambos dispositivos.

El control bloqueado debe conservarse en el registro; una nueva ejecución no borra ese incidente.

---

## Paso 29 — Repetir el control positivo

Como ambos dispositivos responden y no hemos realizado reinicios ni cambios de banda, podemos repetir el control.

Utilizaremos un identificador diferente para conservar la trazabilidad.

**Condiciones:**

| Parámetro             | Valor                            |
| --------------------- | -------------------------------- |
| Preset                | `wisun_915_fsk_50k`              |
| Frecuencia solicitada | 902.2 MHz                        |
| Marcador              | `DE AD BE EF`                    |
| TX                    | `COM33`                          |
| RX                    | `COM88`                          |
| Potencia solicitada   | 0 dBm                            |
| Transmisiones         | 10                               |
| Identificador         | `control-after-6-presets-run-02` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_50k `
    DEADBEEF `
    control-after-6-presets-run-02
```

### Criterios de aceptación

Esperamos observar diez `TX_ATTEMPT_ACK`, diez `RX_PACKET` con `crc_ok=True` y `marker_present=True`, incremento de `rx_ok` de 0 a 10, ausencia de errores asíncronos y desconexión correcta.

**Si obtenemos PASS-RF**, consideraremos recuperado el control positivo y podremos avanzar al siguiente preset.

**Si vuelve a fallar**, detendremos la campaña para investigar específicamente la secuencia de comandos y el manejo de respuestas.

Envíame el registro completo hasta `TEST_END`.

**Importante:** aunque esta repetición resulte exitosa, el timeout original seguirá documentado como una anomalía de comunicación todavía sin causa raíz identificada.

---

S C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_50k `
>>     DEADBEEF `
>>     control-after-6-presets-run-02
2026-10-15T22:41:11.362479Z TEST_BEGIN checkpoint='manual-campaign' label='control-after-6-presets-run-02' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_50k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='DE AD BE EF' tx_power_dbm=0 tx_count=10
2026-10-15T22:41:11.362621Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:41:12.474030Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:41:12.474131Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:41:13.589312Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:41:13.589430Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:41:13.620351Z TX_SET_PHY_ACK
2026-10-15T22:41:13.620526Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:41:13.649263Z RX_SET_PHY_ACK
2026-10-15T22:41:13.649380Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:41:13.681159Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:41:13.681270Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:41:13.713856Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:41:13.713957Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:41:13.747829Z TX_SET_POWER_ACK
2026-10-15T22:41:13.747951Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:41:13.916904Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:41:13.917056Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:41:14.078712Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:41:14.078823Z RX_START_BEGIN
2026-10-15T22:41:14.108303Z RX_START_ACK
2026-10-15T22:41:14.108521Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:41:14.408869Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30034470000100555
2026-10-15T22:41:14.409115Z TX_ATTEMPT_BEGIN index=1 payload='DE AD BE EF'
2026-10-15T22:41:14.442677Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:41:14.543363Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.1005726000003051
2026-10-15T22:41:14.543502Z TX_ATTEMPT_BEGIN index=2 payload='DE AD BE EF'
2026-10-15T22:41:14.576492Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:41:14.677098Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10041099999943981
2026-10-15T22:41:14.677303Z TX_ATTEMPT_BEGIN index=3 payload='DE AD BE EF'
2026-10-15T22:41:14.710104Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:41:14.810923Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10061570000107167
2026-10-15T22:41:14.811547Z TX_ATTEMPT_BEGIN index=4 payload='DE AD BE EF'
2026-10-15T22:41:14.854840Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:41:14.955897Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10064009999769041
2026-10-15T22:41:14.956418Z TX_ATTEMPT_BEGIN index=5 payload='DE AD BE EF'
2026-10-15T22:41:14.998799Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:41:15.099484Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10025500000119791
2026-10-15T22:41:15.099756Z TX_ATTEMPT_BEGIN index=6 payload='DE AD BE EF'
2026-10-15T22:41:15.139627Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:41:15.240190Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10016240000186372
2026-10-15T22:41:15.240316Z TX_ATTEMPT_BEGIN index=7 payload='DE AD BE EF'
2026-10-15T22:41:15.272916Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:41:15.373517Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10047280000071623
2026-10-15T22:41:15.373657Z TX_ATTEMPT_BEGIN index=8 payload='DE AD BE EF'
2026-10-15T22:41:15.405173Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:41:15.505574Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10022879999814904
2026-10-15T22:41:15.506092Z TX_ATTEMPT_BEGIN index=9 payload='DE AD BE EF'
2026-10-15T22:41:15.541961Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:41:15.642372Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10023069999806467
2026-10-15T22:41:15.642891Z TX_ATTEMPT_BEGIN index=10 payload='DE AD BE EF'
2026-10-15T22:41:15.686761Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:41:15.787948Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10066250000090804
2026-10-15T22:41:15.788439Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:41:16.789028Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0006063000000722
2026-10-15T22:41:16.789194Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:41:16.906693Z RX_PACKET ordinal=1 device_timestamp_us=1753106 channel=37 rssi_dbm=-122 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.027778Z RX_PACKET ordinal=2 device_timestamp_us=1753631 channel=37 rssi_dbm=-47 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.140349Z RX_PACKET ordinal=3 device_timestamp_us=1754154 channel=37 rssi_dbm=0 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.255145Z RX_PACKET ordinal=4 device_timestamp_us=1754678 channel=37 rssi_dbm=-53 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.369348Z RX_PACKET ordinal=5 device_timestamp_us=1755244 channel=37 rssi_dbm=-41 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.489655Z RX_PACKET ordinal=6 device_timestamp_us=1755803 channel=37 rssi_dbm=-10 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.607592Z RX_PACKET ordinal=7 device_timestamp_us=1756352 channel=37 rssi_dbm=-43 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.725313Z RX_PACKET ordinal=8 device_timestamp_us=1756874 channel=37 rssi_dbm=-60 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.843468Z RX_PACKET ordinal=9 device_timestamp_us=1757391 channel=37 rssi_dbm=-29 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:17.958035Z RX_PACKET ordinal=10 device_timestamp_us=1757926 channel=37 rssi_dbm=-61 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:41:18.796239Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0070507999989786 packets=10 async_errors=0
2026-10-15T22:41:18.796418Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:41:19.301723Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5052971000004618 async_errors=0
2026-10-15T22:41:19.301861Z RX_STOP_BEGIN
2026-10-15T22:41:19.331756Z RX_STOP_ACK
2026-10-15T22:41:19.331903Z TX_STATS_AFTER_BEGIN
2026-10-15T22:41:19.513005Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:41:19.513119Z RX_STATS_AFTER_BEGIN
2026-10-15T22:41:19.688226Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:41:19.688334Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:41:19.689136Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:41:19.689233Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:41:19.690090Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:41:19.690176Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:41:19.690232Z TEST_CLASSIFICATION label='control-after-6-presets-run-02' preset='wisun_915_fsk_50k' classification='PASS-RF'
2026-10-15T22:41:19.690286Z TEST_END exit_code=0

---


## Paso 29 — Control positivo recuperado: PASS-RF (10/10) ✅

**El control de referencia volvió a funcionar correctamente después del timeout anterior.** Podemos continuar la campaña OTA.

La ejecución `control-after-6-presets-run-02` completó todas las etapas, incluidas las consultas de estadísticas que habían provocado el bloqueo.

### 1. Resultado experimental

| Parámetro                | Resultado                        |
| ------------------------ | -------------------------------- |
| Identificador            | `control-after-6-presets-run-02` |
| Fecha                    | 2026-10-15, 22:41 UTC            |
| Preset                   | `wisun_915_fsk_50k`              |
| Frecuencia solicitada    | 902.2 MHz                        |
| Modulación solicitada    | FSK (`mod_type=0`)               |
| Symbol rate solicitado   | 50 000 símbolos/s                |
| Marcador                 | `DE AD BE EF`                    |
| TX                       | CatSniffer #2 — `COM33`          |
| RX                       | CatSniffer #3 — `COM88`          |
| TX ACK                   | **10/10**                        |
| RX CRC-good con marcador | **10/10**                        |
| Errores asíncronos       | **0**                            |
| Excepciones fatales      | **0**                            |
| Limpieza                 | Correcta                         |
| **Clasificación**        | **PASS-RF**                      |

### 2. Evidencia de recepción

Los diez paquetes contienen el marcador esperado:

```text
04 DE AD BE EF 33 C8
```

En todos los casos:

```text
crc_ok=True
marker_present=True
```

Los contadores del receptor confirman:

```text
RX_STATS_BEFORE: rx_ok=0
RX_STATS_AFTER:  rx_ok=10

rx_crc_err=0
rx_drop=0
rx_overflow=0
```

También se completaron correctamente `RX_STOP_ACK` y las desconexiones de ambos dispositivos.

**Conclusión experimental:** el enlace de referencia volvió a entregar diez paquetes CRC-good con el marcador esperado, después de ejecutar los presets 4, 5 y 6.

Esto permite continuar, pero no demuestra que el sistema esté libre de fallos intermitentes.

---

## 3. ¿Qué ocurrió con el timeout?

Ahora tenemos tres observaciones consecutivas:

| Ejecución                         | Resultado                                  |
| --------------------------------- | ------------------------------------------ |
| `control-after-6-presets-run-01`  | ⚠️ BLOCKED por timeout en `tx.get_stats()` |
| Diagnóstico aislado `get_stats()` | ✅ Ambos dispositivos respondieron          |
| `control-after-6-presets-run-02`  | ✅ PASS-RF 10/10                            |

**Interpretación:** el timeout no persistió y tampoco volvió a manifestarse durante la repetición completa.

Sin embargo, **su causa raíz continúa sin identificar**.

No debemos eliminar la ejecución bloqueada del registro ni atribuir el problema a USB, UART o firmware sin evidencia adicional.

La anomalía de metadatos RF también permanece abierta: `channel=37`, `lqi=0` y valores de RSSI que todavía no hemos validado.

---

## 4. Estado actualizado de la campaña

| Indicador                      |           Resultado |
| ------------------------------ | ------------------: |
| Presets nuevos evaluados       |                   6 |
| Presets nuevos PASS-RF         |             **6/6** |
| Controles positivos PASS-RF    | **3/3 completados** |
| Ejecuciones bloqueadas         |                   1 |
| Ejecuciones OTA satisfactorias |               **9** |
| Paquetes CRC-good con marcador |           **90/90** |

El total de 90/90 corresponde únicamente a las nueve ejecuciones OTA completadas; la ejecución bloqueada terminó antes de transmitir.

**Estado:** campaña habilitada para continuar con el séptimo preset.

---

## Paso 30 — Inspeccionar `sidewalk_915_fsk_50k`

El siguiente preset es:

```text
sidewalk_915_fsk_50k
```

Antes de transmitir, verificaremos su definición real en `PROP_PRESETS`.

Esto es especialmente importante porque el nombre contiene `sidewalk`, pero **un nombre de preset no demuestra compatibilidad con el protocolo Amazon Sidewalk**.

Por ahora, únicamente estamos evaluando configuraciones de radio propietarias.

### Ejecuta únicamente:

```powershell
$PRESET = 'sidewalk_915_fsk_50k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

**Este comando no transmite RF.**

Envíame la salida para verificar frecuencia, modulación, symbol rate, desviación, ancho de banda y sync word.

Después prepararemos la prueba OTA con el marcador `A5 07 00 01`.

**Conclusión de esta etapa:** el control positivo está recuperado y los seis presets nuevos mantienen sus resultados PASS-RF. El timeout intermitente sigue registrado como una anomalía pendiente de investigación.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
sidewalk_915_fsk_50k {'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}

---

## Paso 30 — Preset `sidewalk_915_fsk_50k` verificado ✅

La salida confirma que el preset existe en `PROP_PRESETS` y solicita una configuración FSK a **915 MHz**.

### 1. Parámetros identificados

| Parámetro   | Valor                  |
| ----------- | ---------------------- |
| Preset      | `sidewalk_915_fsk_50k` |
| Frecuencia  | **915 MHz**            |
| Modulación  | FSK (`mod_type=0`)     |
| Symbol rate | 50 000 símbolos/s      |
| `deviation` | 100                    |
| `rx_bw`     | 82                     |
| `sync_word` | `0x930B51DE`           |

**Fuente:** definición de `PROP_PRESETS` consultada mediante la biblioteca Python de FeralRF.

### 2. Comparación con los presets anteriores

Hay un detalle importante:

| Parámetro   | `gfsk_915_50k` | `sidewalk_915_fsk_50k` |
| ----------- | -------------: | ---------------------: |
| Frecuencia  |        915 MHz |                915 MHz |
| `mod_type`  |       1 (GFSK) |                0 (FSK) |
| Symbol rate |         50 000 |                 50 000 |
| `deviation` |            100 |                    100 |
| `rx_bw`     |             82 |                     82 |
| `sync_word` |   `0x930B51DE` |           `0x930B51DE` |

**La única diferencia declarada entre estos dos presets es `mod_type`.**

Esto nos permite evaluar una configuración FSK a 915 MHz después de haber obtenido PASS-RF con GFSK a la misma frecuencia.

Sin embargo, debemos evitar dos conclusiones prematuras:

1. Un resultado PASS-RF no demostraría compatibilidad con dispositivos Amazon Sidewalk.
2. Tampoco demostraría por sí solo que el transceptor está generando exactamente la modulación solicitada; eso requiere evidencia adicional del firmware o medición RF independiente.

---

## Paso 31 — Prueba OTA `sidewalk_915_fsk_50k`

Ya contamos con las condiciones necesarias para continuar:

* Antenas compatibles con 902–928 MHz: confirmadas.
* Autorización de transmisión a 915 MHz: confirmada.
* Selección lógica Sub-GHz (`Band: 1`): observada en ambos dispositivos.
* Control positivo posterior a seis presets: **PASS-RF 10/10**.

### Condiciones de la prueba

| Elemento              | Valor                                  |
| --------------------- | -------------------------------------- |
| TX                    | CatSniffer #2 — `COM33`                |
| RX                    | CatSniffer #3 — `COM88`                |
| Preset                | `sidewalk_915_fsk_50k`                 |
| Frecuencia solicitada | 915 MHz                                |
| Potencia solicitada   | 0 dBm                                  |
| Marcador              | `A5 07 00 01`                          |
| Transmisiones         | 10                                     |
| Identificador         | `campaign-sidewalk-915-fsk-50k-run-01` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    sidewalk_915_fsk_50k `
    A5070001 `
    campaign-sidewalk-915-fsk-50k-run-01
```

### Criterio de aceptación

Clasificaremos la prueba como **PASS-RF** si observamos:

* 10 respuestas `TX_ATTEMPT_ACK`.
* 10 paquetes recibidos con `crc_ok=True` y `marker_present=True`.
* Incremento de `rx_ok` de 0 a 10.
* Ningún error asíncrono ni excepción fatal.
* Finalización correcta de `RX_STOP` y desconexión de ambos dispositivos.

También prestaremos atención a cualquier nuevo `TimeoutError`, dado el incidente observado durante el control anterior.

**Envíame el registro completo hasta `TEST_END`.**

Si la prueba termina satisfactoriamente, continuaremos con `sidewalk_915_fsk_250k`, manteniendo el mismo procedimiento de validación.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     sidewalk_915_fsk_50k `
>>     A5070001 `
>>     campaign-sidewalk-915-fsk-50k-run-01
2026-10-15T22:43:01.410420Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-sidewalk-915-fsk-50k-run-01' tx_port='COM33' rx_port='COM88' preset_name='sidewalk_915_fsk_50k' preset={'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='A5 07 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:43:01.410576Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:43:02.528926Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:43:02.529022Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:43:03.645170Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:43:03.645273Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:43:03.677387Z TX_SET_PHY_ACK
2026-10-15T22:43:03.677485Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:43:03.706615Z RX_SET_PHY_ACK
2026-10-15T22:43:03.706715Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:43:03.740320Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:43:03.740422Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:43:03.775413Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:43:03.775515Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:43:03.810066Z TX_SET_POWER_ACK
2026-10-15T22:43:03.810194Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:43:03.980413Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:43:03.980524Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:43:04.145064Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:43:04.145164Z RX_START_BEGIN
2026-10-15T22:43:04.180747Z RX_START_ACK
2026-10-15T22:43:04.180849Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:43:04.481358Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.3004712000001746
2026-10-15T22:43:04.481873Z TX_ATTEMPT_BEGIN index=1 payload='A5 07 00 01'
2026-10-15T22:43:04.521754Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:43:04.622017Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10014879999653203
2026-10-15T22:43:04.622132Z TX_ATTEMPT_BEGIN index=2 payload='A5 07 00 01'
2026-10-15T22:43:04.654017Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:43:04.754391Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10022699999899487
2026-10-15T22:43:04.754839Z TX_ATTEMPT_BEGIN index=3 payload='A5 07 00 01'
2026-10-15T22:43:04.794830Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:43:04.895203Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10022160000153235
2026-10-15T22:43:04.895797Z TX_ATTEMPT_BEGIN index=4 payload='A5 07 00 01'
2026-10-15T22:43:04.934419Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:43:05.034787Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10021860000051674
2026-10-15T22:43:05.035242Z TX_ATTEMPT_BEGIN index=5 payload='A5 07 00 01'
2026-10-15T22:43:05.074778Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:43:05.175234Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10017979999975068
2026-10-15T22:43:05.175752Z TX_ATTEMPT_BEGIN index=6 payload='A5 07 00 01'
2026-10-15T22:43:05.217604Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:43:05.318300Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10054010000021663
2026-10-15T22:43:05.318748Z TX_ATTEMPT_BEGIN index=7 payload='A5 07 00 01'
2026-10-15T22:43:05.353546Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:43:05.453812Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10013899999830755
2026-10-15T22:43:05.453938Z TX_ATTEMPT_BEGIN index=8 payload='A5 07 00 01'
2026-10-15T22:43:05.487577Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:43:05.588010Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10028559999773279
2026-10-15T22:43:05.588480Z TX_ATTEMPT_BEGIN index=9 payload='A5 07 00 01'
2026-10-15T22:43:05.624924Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:43:05.725424Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10032359999968321
2026-10-15T22:43:05.725547Z TX_ATTEMPT_BEGIN index=10 payload='A5 07 00 01'
2026-10-15T22:43:05.754973Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:43:05.855574Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10048380000080215
2026-10-15T22:43:05.855714Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:43:06.856489Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0007293000016944
2026-10-15T22:43:06.856947Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:43:06.986435Z RX_PACKET ordinal=1 device_timestamp_us=2183081 channel=37 rssi_dbm=12 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.097020Z RX_PACKET ordinal=2 device_timestamp_us=2183627 channel=37 rssi_dbm=79 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.212893Z RX_PACKET ordinal=3 device_timestamp_us=2184146 channel=37 rssi_dbm=94 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.329302Z RX_PACKET ordinal=4 device_timestamp_us=2184696 channel=37 rssi_dbm=-7 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.444778Z RX_PACKET ordinal=5 device_timestamp_us=2185241 channel=37 rssi_dbm=-116 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.559304Z RX_PACKET ordinal=6 device_timestamp_us=2185790 channel=37 rssi_dbm=-29 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.676509Z RX_PACKET ordinal=7 device_timestamp_us=2186349 channel=37 rssi_dbm=3 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.793300Z RX_PACKET ordinal=8 device_timestamp_us=2186875 channel=37 rssi_dbm=-98 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:07.900143Z RX_PACKET ordinal=9 device_timestamp_us=2187402 channel=37 rssi_dbm=-111 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:08.016098Z RX_PACKET ordinal=10 device_timestamp_us=2187937 channel=37 rssi_dbm=93 lqi=0 crc_ok=True payload_hex='04 A5 07 00 01 E1 BA' marker_present=True
2026-10-15T22:43:08.865908Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0089685000020836 packets=10 async_errors=0
2026-10-15T22:43:08.866088Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:43:09.374450Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5083395000001474 async_errors=0
2026-10-15T22:43:09.374603Z RX_STOP_BEGIN
2026-10-15T22:43:09.404997Z RX_STOP_ACK
2026-10-15T22:43:09.405111Z TX_STATS_AFTER_BEGIN
2026-10-15T22:43:09.580990Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:43:09.581106Z RX_STATS_AFTER_BEGIN
2026-10-15T22:43:09.751784Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:43:09.751889Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:43:09.752612Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:43:09.752673Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:43:09.753307Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:43:09.753386Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:43:09.753442Z TEST_CLASSIFICATION label='campaign-sidewalk-915-fsk-50k-run-01' preset='sidewalk_915_fsk_50k' classification='PASS-RF'
2026-10-15T22:43:09.753503Z TEST_END exit_code=0

---

## Prueba 7 — `sidewalk_915_fsk_50k`: PASS-RF (10/10) ✅

**La séptima prueba de la campaña OTA terminó satisfactoriamente.** Ambos CatSniffer completaron la configuración solicitada y el receptor entregó diez paquetes con CRC válido y el marcador esperado.

Este resultado confirma comunicación OTA funcional bajo el preset `sidewalk_915_fsk_50k`, pero **no demuestra interoperabilidad con Amazon Sidewalk**.

### 1. Resultado experimental

| Parámetro                | Resultado                              |
| ------------------------ | -------------------------------------- |
| Identificador            | `campaign-sidewalk-915-fsk-50k-run-01` |
| Fecha                    | 2026-10-15, 22:43 UTC                  |
| TX                       | CatSniffer #2 — `COM33`                |
| RX                       | CatSniffer #3 — `COM88`                |
| Frecuencia solicitada    | **915 MHz**                            |
| Modulación solicitada    | FSK (`mod_type=0`)                     |
| Symbol rate solicitado   | 50 000 símbolos/s                      |
| `deviation`              | 100                                    |
| `rx_bw`                  | 82                                     |
| Sync word                | `0x930B51DE`                           |
| Potencia solicitada      | 0 dBm                                  |
| Marcador                 | `A5 07 00 01`                          |
| TX ACK                   | **10/10**                              |
| RX CRC-good con marcador | **10/10**                              |
| Errores asíncronos TX/RX | **0/0**                                |
| Excepciones fatales      | **0**                                  |
| Limpieza                 | Correcta                               |
| **Clasificación**        | **PASS-RF**                            |

### 2. Evidencia de recepción

Los diez paquetes recibidos contienen:

```text
04 A5 07 00 01 E1 BA
```

En todos se observa:

```text
crc_ok=True
marker_present=True
```

Además, los contadores del receptor evolucionaron correctamente:

| Contador RX   | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se completaron `RX_STOP_ACK` y las desconexiones de ambos dispositivos.

**Conclusión experimental:** bajo las condiciones de esta ejecución, el enlace entregó diez paquetes con el marcador esperado y CRC reportado como válido.

Los ACK de configuración y transmisión, por sí solos, no prueban los parámetros físicos de la señal. Tampoco hemos medido independientemente la frecuencia, la desviación ni la potencia radiada.

### 3. Anomalías persistentes

Los metadatos continúan mostrando comportamientos que debemos investigar posteriormente:

| Campo      | Observación              | Interpretación                              |
| ---------- | ------------------------ | ------------------------------------------- |
| `channel`  | 37 en todos los paquetes | No coincide con el canal solicitado         |
| `lqi`      | 0 en todos los paquetes  | No validado                                 |
| `rssi_dbm` | Entre −116 y +94 dBm     | Incluye valores físicamente poco plausibles |

No utilizaremos estos campos para comparar la calidad RF entre presets hasta verificar cómo los produce el firmware.

---

## 4. Estado acumulado de la campaña

| Ejecución                        | Preset                     | Resultado   |
| -------------------------------- | -------------------------- | ----------- |
| Control inicial                  | `wisun_915_fsk_50k`        | ✅ 10/10     |
| Prueba 1                         | `gfsk_902_50k`             | ✅ 10/10     |
| Prueba 2                         | `wisun_915_fsk_100k`       | ✅ 10/10     |
| Prueba 3                         | `wisun_915_fsk_150k`       | ✅ 10/10     |
| Control después de 3             | `wisun_915_fsk_50k`        | ✅ 10/10     |
| Prueba 4                         | `wisun_915_fsk_200k`       | ✅ 10/10     |
| Prueba 5                         | `wisun_915_fsk_300k`       | ✅ 10/10     |
| Prueba 6                         | `gfsk_915_50k`             | ✅ 10/10     |
| Control después de 6 — intento 1 | `wisun_915_fsk_50k`        | ⚠️ BLOCKED  |
| Control después de 6 — intento 2 | `wisun_915_fsk_50k`        | ✅ 10/10     |
| **Prueba 7**                     | **`sidewalk_915_fsk_50k`** | **✅ 10/10** |

**Balance actualizado:**

* **7/7 presets nuevos:** PASS-RF.
* **3/3 controles positivos completados:** PASS-RF.
* **100/100 paquetes CRC-good con marcador correcto**, entre las diez ejecuciones exitosas.
* **1 ejecución BLOCKED**, por timeout en `tx.get_stats()` antes de transmitir.

El control positivo más reciente sigue vigente. Hemos ejecutado un preset nuevo desde entonces.

---

## Paso 32 — Inspeccionar `sidewalk_915_fsk_250k`

El siguiente preset de la campaña es:

```text
sidewalk_915_fsk_250k
```

Antes de transmitir, verificaremos su configuración exacta.

### Ejecuta únicamente:

```powershell
$PRESET = 'sidewalk_915_fsk_250k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

**Este comando no transmite RF.**

Envíame la salida para comprobar frecuencia, modulación, symbol rate, desviación, ancho de banda y sync word.

Después prepararemos la prueba OTA con el marcador `A5 08 00 01`.

**Conclusión:** el preset `sidewalk_915_fsk_50k` ha superado nuestra validación básica de entrega OTA entre dos CatSniffer. La interoperabilidad con Amazon Sidewalk, los parámetros físicos reales de la señal y la confiabilidad de los metadatos RF permanecen fuera de lo demostrado.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
sidewalk_915_fsk_250k {'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 250000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}

---

## Paso 32 — Preset `sidewalk_915_fsk_250k` verificado ✅

La consulta confirma que el preset existe en `PROP_PRESETS` y solicita una configuración **FSK a 915 MHz, con 250 000 símbolos/s**.

### 1. Parámetros identificados

| Parámetro             | Valor                   |
| --------------------- | ----------------------- |
| Preset                | `sidewalk_915_fsk_250k` |
| Frecuencia solicitada | **915 MHz**             |
| Modulación solicitada | FSK (`mod_type=0`)      |
| Symbol rate           | **250 000 símbolos/s**  |
| `deviation`           | 200                     |
| `rx_bw`               | 90                      |
| Sync word             | `0x930B51DE`            |
| Potencia prevista     | 0 dBm                   |

**Fuente:** definición de `PROP_PRESETS` consultada mediante la biblioteca Python de FeralRF.

### 2. Comparación con el preset anterior

| Parámetro   | `sidewalk_915_fsk_50k` | `sidewalk_915_fsk_250k` |
| ----------- | ---------------------: | ----------------------: |
| Frecuencia  |                915 MHz |                 915 MHz |
| Modulación  |                    FSK |                     FSK |
| Symbol rate |                 50 000 |             **250 000** |
| `deviation` |                    100 |                 **200** |
| `rx_bw`     |                     82 |                  **90** |
| Sync word   |           `0x930B51DE` |            `0x930B51DE` |

El cambio principal es el aumento del symbol rate de 50 000 a 250 000 símbolos/s.

También cambian `deviation` y `rx_bw`, por lo que **no estamos aislando una sola variable experimental**.

Si obtenemos PASS-RF, podremos concluir que esta combinación de parámetros permite recibir paquetes entre nuestros dos CatSniffer, pero no atribuir el resultado exclusivamente al aumento del symbol rate.

Además, el nombre `sidewalk` no implica que estemos validando interoperabilidad con Amazon Sidewalk.

---

## Paso 33 — Prueba OTA `sidewalk_915_fsk_250k`

Las condiciones de preparación siguen siendo compatibles con esta prueba:

* Antenas compatibles con 902–928 MHz: confirmadas.
* Autorización para transmitir a 915 MHz: confirmada.
* Selección lógica Sub-GHz: `Band: 1` en ambos dispositivos.
* Último control positivo: **PASS-RF 10/10**.

### Condiciones de ejecución

| Elemento              | Valor                                   |
| --------------------- | --------------------------------------- |
| TX                    | CatSniffer #2 — `COM33`                 |
| RX                    | CatSniffer #3 — `COM88`                 |
| Frecuencia solicitada | 915 MHz                                 |
| Potencia solicitada   | 0 dBm                                   |
| Marcador              | `A5 08 00 01`                           |
| Transmisiones         | 10                                      |
| Identificador         | `campaign-sidewalk-915-fsk-250k-run-01` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    sidewalk_915_fsk_250k `
    A5080001 `
    campaign-sidewalk-915-fsk-250k-run-01
```

### Criterios de aceptación

Esperamos observar diez `TX_ATTEMPT_ACK`, diez paquetes RX con `crc_ok=True` y `marker_present=True`, incremento de `rx_ok` de 0 a 10, ausencia de errores asíncronos y limpieza correcta.

También revisaremos si reaparece el `TimeoutError` observado anteriormente en `get_stats()`.

**Envíame el registro completo hasta `TEST_END`.**

Si obtenemos PASS-RF, habremos completado ocho presets nuevos. El siguiente paso será revisar las condiciones de la campaña antes de pasar a los presets de **868 MHz**, porque esa transición requiere una nueva comprobación de compatibilidad de antenas y autorización de transmisión.

**Estado actual:** 7/7 presets nuevos con PASS-RF, tres controles positivos satisfactorios y un timeout intermitente pendiente de investigación.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     sidewalk_915_fsk_250k `
>>     A5080001 `
>>     campaign-sidewalk-915-fsk-250k-run-01
2026-10-15T22:46:04.460856Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-sidewalk-915-fsk-250k-run-01' tx_port='COM33' rx_port='COM88' preset_name='sidewalk_915_fsk_250k' preset={'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 250000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606} marker='A5 08 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:46:04.460990Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:46:05.579902Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:46:05.580003Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:46:06.693313Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:46:06.693454Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:46:06.725815Z TX_SET_PHY_ACK
2026-10-15T22:46:06.725919Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:46:06.759045Z RX_SET_PHY_ACK
2026-10-15T22:46:06.759148Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 250000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}
2026-10-15T22:46:06.790870Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:46:06.790978Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 915000000, 'mod_type': 0, 'symbol_rate': 250000, 'deviation': 200, 'rx_bw': 90, 'sync_word': 2466992606}
2026-10-15T22:46:06.821746Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:46:06.821843Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:46:06.851532Z TX_SET_POWER_ACK
2026-10-15T22:46:06.851633Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:46:07.031011Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:46:07.031134Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:46:07.201161Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:46:07.201267Z RX_START_BEGIN
2026-10-15T22:46:07.234847Z RX_START_ACK
2026-10-15T22:46:07.234960Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:46:07.535395Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30043029999797
2026-10-15T22:46:07.535558Z TX_ATTEMPT_BEGIN index=1 payload='A5 08 00 01'
2026-10-15T22:46:07.566694Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:46:07.666945Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10008269999889308
2026-10-15T22:46:07.667081Z TX_ATTEMPT_BEGIN index=2 payload='A5 08 00 01'
2026-10-15T22:46:07.699896Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:46:07.800325Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10025780000069062
2026-10-15T22:46:07.800505Z TX_ATTEMPT_BEGIN index=3 payload='A5 08 00 01'
2026-10-15T22:46:07.833166Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:46:07.933435Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10008989999914775
2026-10-15T22:46:07.933557Z TX_ATTEMPT_BEGIN index=4 payload='A5 08 00 01'
2026-10-15T22:46:07.965928Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:46:08.066257Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10018149999814341
2026-10-15T22:46:08.066439Z TX_ATTEMPT_BEGIN index=5 payload='A5 08 00 01'
2026-10-15T22:46:08.097779Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:46:08.198248Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10027350000018487
2026-10-15T22:46:08.198401Z TX_ATTEMPT_BEGIN index=6 payload='A5 08 00 01'
2026-10-15T22:46:08.231948Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:46:08.332304Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10018829999899026
2026-10-15T22:46:08.332501Z TX_ATTEMPT_BEGIN index=7 payload='A5 08 00 01'
2026-10-15T22:46:08.365024Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:46:08.465712Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10049629999775789
2026-10-15T22:46:08.465851Z TX_ATTEMPT_BEGIN index=8 payload='A5 08 00 01'
2026-10-15T22:46:08.499551Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:46:08.600168Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10047280000071623
2026-10-15T22:46:08.600297Z TX_ATTEMPT_BEGIN index=9 payload='A5 08 00 01'
2026-10-15T22:46:08.632243Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:46:08.732769Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10038759999952163
2026-10-15T22:46:08.732895Z TX_ATTEMPT_BEGIN index=10 payload='A5 08 00 01'
2026-10-15T22:46:08.766950Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:46:08.867470Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10037279999960447
2026-10-15T22:46:08.867647Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:46:09.868173Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0005108000004839
2026-10-15T22:46:09.868369Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:46:09.988080Z RX_PACKET ordinal=1 device_timestamp_us=2898136 channel=37 rssi_dbm=5 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.102033Z RX_PACKET ordinal=2 device_timestamp_us=2898648 channel=37 rssi_dbm=-100 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.216673Z RX_PACKET ordinal=3 device_timestamp_us=2899170 channel=37 rssi_dbm=-89 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.340453Z RX_PACKET ordinal=4 device_timestamp_us=2899689 channel=37 rssi_dbm=-126 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.456423Z RX_PACKET ordinal=5 device_timestamp_us=2900208 channel=37 rssi_dbm=-59 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.569610Z RX_PACKET ordinal=6 device_timestamp_us=2900724 channel=37 rssi_dbm=12 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.684992Z RX_PACKET ordinal=7 device_timestamp_us=2901249 channel=37 rssi_dbm=63 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.801722Z RX_PACKET ordinal=8 device_timestamp_us=2901769 channel=37 rssi_dbm=127 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:10.915920Z RX_PACKET ordinal=9 device_timestamp_us=2902294 channel=37 rssi_dbm=118 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:11.026287Z RX_PACKET ordinal=10 device_timestamp_us=2902812 channel=37 rssi_dbm=-19 lqi=0 crc_ok=True payload_hex='04 A5 08 00 01 E1 76' marker_present=True
2026-10-15T22:46:11.885767Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0173723000007158 packets=10 async_errors=0
2026-10-15T22:46:11.885982Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:46:12.390859Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5048630999990564 async_errors=0
2026-10-15T22:46:12.391009Z RX_STOP_BEGIN
2026-10-15T22:46:12.423880Z RX_STOP_ACK
2026-10-15T22:46:12.424003Z TX_STATS_AFTER_BEGIN
2026-10-15T22:46:12.586832Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:46:12.586971Z RX_STATS_AFTER_BEGIN
2026-10-15T22:46:12.748553Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:46:12.748809Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:46:12.752401Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:46:12.753242Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:46:12.754037Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:46:12.754200Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:46:12.754344Z TEST_CLASSIFICATION label='campaign-sidewalk-915-fsk-250k-run-01' preset='sidewalk_915_fsk_250k' classification='PASS-RF'
2026-10-15T22:46:12.754473Z TEST_END exit_code=0

---

## Paso 33 — `sidewalk_915_fsk_250k`: PASS-RF (10/10) ✅

**La octava prueba de la campaña OTA terminó satisfactoriamente.** El receptor obtuvo diez paquetes con el marcador esperado y CRC reportado como válido, sin errores asíncronos ni excepciones.

### 1. Resultado experimental

| Parámetro                | Resultado                               |
| ------------------------ | --------------------------------------- |
| Ejecución                | `campaign-sidewalk-915-fsk-250k-run-01` |
| Fecha                    | 2026-10-15, 22:46 UTC                   |
| TX                       | CatSniffer #2 — `COM33`                 |
| RX                       | CatSniffer #3 — `COM88`                 |
| Frecuencia configurada   | 915 MHz                                 |
| Modulación solicitada    | FSK (`mod_type=0`)                      |
| Symbol rate solicitado   | 250 000 símbolos/s                      |
| `deviation` / `rx_bw`    | 200 / 90                                |
| Potencia solicitada      | 0 dBm                                   |
| Marcador                 | `A5 08 00 01`                           |
| TX ACK                   | **10/10**                               |
| RX CRC-good con marcador | **10/10**                               |
| Errores asíncronos       | 0                                       |
| Excepciones fatales      | 0                                       |
| **Clasificación**        | **PASS-RF**                             |

**Fuente:** registro de ejecución proporcionado, desde `TEST_BEGIN` hasta `TEST_END`.

### 2. Evidencia de recepción

Los diez paquetes presentan:

```text
payload_hex='04 A5 08 00 01 E1 76'
crc_ok=True
marker_present=True
```

Los contadores del receptor evolucionaron de la siguiente manera:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se observaron:

* `RX_STOP_ACK`.
* Desconexión correcta de TX y RX.
* `TEST_END exit_code=0`.

**Conclusión:** existe evidencia de entrega OTA satisfactoria entre ambos CatSniffer bajo esta configuración solicitada.

Esto no demuestra interoperabilidad con Amazon Sidewalk ni constituye una medición independiente de los parámetros físicos de transmisión.

### 3. Anomalías que permanecen abiertas

Los metadatos siguen presentando inconsistencias:

| Campo      | Observación                   |
| ---------- | ----------------------------- |
| `channel`  | Siempre 37                    |
| `lqi`      | Siempre 0                     |
| `rssi_dbm` | Valores entre −126 y +127 dBm |

Especialmente el valor `rssi_dbm=127` refuerza la sospecha de que el campo no está siendo interpretado correctamente.

**No debemos utilizar estos valores para comparar sensibilidad, alcance o calidad de recepción.**

---

## 4. Estado acumulado de la campaña

| Indicador                       |       Resultado |
| ------------------------------- | --------------: |
| Presets nuevos evaluados        |           **8** |
| Presets nuevos PASS-RF          |         **8/8** |
| Controles positivos completados | **3/3 PASS-RF** |
| Ejecuciones OTA satisfactorias  |          **11** |
| Paquetes CRC-good con marcador  |     **110/110** |
| Ejecuciones BLOCKED             |           **1** |

El timeout anterior en `tx.get_stats()` permanece documentado como una anomalía sin causa raíz identificada.

### Punto importante: transición a 868 MHz

Hemos completado los ocho presets nuevos de la campaña correspondientes a 902.2 y 915 MHz.

El siguiente grupo contempla:

* `gfsk_868_50k`
* `gfsk_868_100k`
* `wireless_mbus_s_868`
* `wireless_mbus_t_868`
* `wireless_mbus_c_868`

Sin embargo, **no debemos ejecutar todavía ninguna transmisión a 868 MHz**.

Las antenas confirmadas hasta ahora son para **902–928 MHz**. Su compatibilidad con 868 MHz no está demostrada y tampoco hemos confirmado las condiciones regulatorias aplicables a esa frecuencia.

---

## Paso 34 — Verificación previa a las pruebas de 868 MHz

Antes de continuar, necesito confirmar dos aspectos:

**1. Antenas:** ¿Dispones de dos antenas especificadas para 868 MHz, o de antenas cuya hoja de datos indique explícitamente que cubren tanto 868 como 915 MHz?

**2. Autorización:** ¿Tienes permitido realizar transmisiones de prueba a 868 MHz en tu laboratorio, bajo las condiciones de potencia y duración previstas?

Por ahora, **no cambies las antenas ni ejecutes nuevos comandos RF**.

Con estas respuestas determinaremos si podemos continuar la campaña de 868 MHz o si debemos dejar ese grupo pendiente por falta de condiciones de validación.

**Estado actual: 8/8 presets nuevos PASS-RF, 110/110 paquetes recibidos correctamente en las ejecuciones exitosas y transición a 868 MHz pendiente de autorización y verificación de antenas.**

---

## Paso 34 — Condiciones para 868 MHz confirmadas ✅

Con tu confirmación, podemos continuar la campaña de validación OTA en **868 MHz**.

Quedan establecidas las siguientes condiciones:

| Condición                                                        | Estado                      |
| ---------------------------------------------------------------- | --------------------------- |
| Antenas compatibles con 868 MHz en ambos CatSniffer              | ✅ Confirmado por el usuario |
| Autorización para realizar las transmisiones de prueba previstas | ✅ Confirmado por el usuario |
| CatSniffer #2 — TX                                               | `COM33`                     |
| CatSniffer #3 — RX                                               | `COM88`                     |
| Último resultado OTA                                             | PASS-RF 10/10               |
| Presets nuevos completados                                       | **8/8 PASS-RF**             |

**Importante:** esta confirmación establece las condiciones declaradas para continuar, pero no sustituye la verificación documental de las antenas ni una medición independiente de potencia radiada.

### Cambio de frecuencia: ¿qué debemos considerar?

Pasaremos de 915 MHz a 868 MHz, pero ambas frecuencias pertenecen al rango Sub-GHz.

Por tanto, no necesitamos seleccionar una nueva familia de radio mediante `band2` si ambos dispositivos mantienen la ruta Sub-GHz previamente establecida.

Lo que sí cambia es la frecuencia solicitada al transceptor mediante `configure_prop()`.

Además, si las antenas compatibles con 868 MHz son distintas de las utilizadas anteriormente, deben estar instaladas en ambos CatSniffer **antes de transmitir**.

---

## Paso 35 — Inspeccionar `gfsk_868_50k`

El siguiente preset de la campaña es:

`gfsk_868_50k`

Antes de transmitir, verificaremos sus parámetros reales en la biblioteca Python.

### Ejecuta únicamente:

```powershell
$PRESET = 'gfsk_868_50k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

Este comando **no transmite RF ni modifica la configuración de los dispositivos**.

Con la salida comprobaremos la frecuencia, `mod_type`, `symbol_rate`, `deviation`, `rx_bw` y `sync_word`.

Después prepararemos la primera prueba OTA a 868 MHz con el marcador `A5 09 00 01`.

**Estado:** transición a 868 MHz autorizada según las condiciones confirmadas; primera configuración pendiente de inspección.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
gfsk_868_50k {'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}

---

## Paso 35 — Preset `gfsk_868_50k` verificado ✅

La consulta confirma que `gfsk_868_50k` está definido en `PROP_PRESETS` y solicita una configuración **GFSK a 868 MHz, con 50 000 símbolos/s**.

### 1. Parámetros identificados

| Parámetro             | Valor               |
| --------------------- | ------------------- |
| Preset                | `gfsk_868_50k`      |
| Frecuencia solicitada | **868 MHz**         |
| Modulación solicitada | GFSK (`mod_type=1`) |
| Symbol rate           | 50 000 símbolos/s   |
| `deviation`           | 100                 |
| `rx_bw`               | 82                  |
| Sync word             | `0x930B51DE`        |
| Potencia prevista     | 0 dBm               |

**Fuente:** consulta directa de `PROP_PRESETS` mediante la biblioteca Python de FeralRF.

Los valores `deviation=100` y `rx_bw=82` son los parámetros que recibe `configure_prop()`. No debemos asignarles unidades físicas sin verificar su interpretación en el firmware.

### 2. Comparación con `gfsk_915_50k`

| Parámetro     | `gfsk_915_50k` | `gfsk_868_50k` |
| ------------- | -------------: | -------------: |
| Frecuencia    |        915 MHz |    **868 MHz** |
| `mod_type`    |              1 |              1 |
| `symbol_rate` |         50 000 |         50 000 |
| `deviation`   |            100 |            100 |
| `rx_bw`       |             82 |             82 |
| `sync_word`   |   `0x930B51DE` |   `0x930B51DE` |

La única diferencia entre las configuraciones declaradas es la frecuencia.

Esto resulta útil para evaluar si el enlace sigue funcionando al cambiar de 915 a 868 MHz, manteniendo los demás parámetros solicitados.

No obstante, si utilizamos antenas distintas, también cambiarán las condiciones físicas de la prueba.

---

## Paso 36 — Primera prueba OTA a 868 MHz

Ya contamos con:

* Confirmación de antenas compatibles con 868 MHz.
* Confirmación de autorización para las transmisiones previstas.
* Preset identificado y revisado.
* Ocho presets anteriores con resultado PASS-RF.

**Antes de ejecutar, verifica físicamente que ambos CatSniffer tengan instaladas las antenas compatibles con 868 MHz.** No conectes ni desconectes antenas mientras estén transmitiendo.

### Condiciones de ejecución

| Elemento              | Valor                          |
| --------------------- | ------------------------------ |
| TX                    | CatSniffer #2 — `COM33`        |
| RX                    | CatSniffer #3 — `COM88`        |
| Preset                | `gfsk_868_50k`                 |
| Frecuencia solicitada | **868 MHz**                    |
| Potencia solicitada   | 0 dBm                          |
| Marcador              | `A5 09 00 01`                  |
| Transmisiones         | 10                             |
| Identificador         | `campaign-gfsk-868-50k-run-01` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    gfsk_868_50k `
    A5090001 `
    campaign-gfsk-868-50k-run-01
```

### Criterios de aceptación

Clasificaremos la ejecución como **PASS-RF** si observamos:

1. Diez `TX_ATTEMPT_ACK`.
2. Diez paquetes recibidos con `crc_ok=True` y `marker_present=True`.
3. Incremento de `rx_ok` de 0 a 10.
4. Ausencia de errores asíncronos y excepciones fatales.
5. `RX_STOP_ACK` y desconexión correcta de ambos dispositivos.

**Atención especial:** esta es la primera ejecución de la campaña a 868 MHz. Si aparece cualquier error de configuración, transmisión o recepción, detendremos la secuencia para analizarlo antes de repetir.

El timeout anterior de `get_stats()` continúa registrado como anomalía pendiente.

### Estado de la campaña

**8/8 presets nuevos PASS-RF**, con 110/110 paquetes CRC-good con marcador entre las once ejecuciones OTA satisfactorias. La primera prueba a 868 MHz todavía no se ha ejecutado.

**Envíame el registro completo hasta `TEST_END` para evaluar el resultado.**

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     gfsk_868_50k `
>>     A5090001 `
>>     campaign-gfsk-868-50k-run-01
2026-10-15T22:49:50.496807Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-gfsk-868-50k-run-01' tx_port='COM33' rx_port='COM88' preset_name='gfsk_868_50k' preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='A5 09 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:49:50.496974Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:49:51.616853Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:49:51.616952Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:49:52.734658Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:49:52.734847Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:49:52.766035Z TX_SET_PHY_ACK
2026-10-15T22:49:52.766202Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:49:52.799984Z RX_SET_PHY_ACK
2026-10-15T22:49:52.800119Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:49:52.831487Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:49:52.831591Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:49:52.863895Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:49:52.864026Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:49:52.896058Z TX_SET_POWER_ACK
2026-10-15T22:49:52.896183Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:49:53.075352Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:49:53.075457Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:49:53.239190Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:49:53.239289Z RX_START_BEGIN
2026-10-15T22:49:53.275152Z RX_START_ACK
2026-10-15T22:49:53.276915Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:49:53.577295Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30198440000094706
2026-10-15T22:49:53.577812Z TX_ATTEMPT_BEGIN index=1 payload='A5 09 00 01'
2026-10-15T22:49:53.621596Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:49:53.722644Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10052479999649222
2026-10-15T22:49:53.723180Z TX_ATTEMPT_BEGIN index=2 payload='A5 09 00 01'
2026-10-15T22:49:53.765019Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:49:53.865416Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.1002401000005193
2026-10-15T22:49:53.865927Z TX_ATTEMPT_BEGIN index=3 payload='A5 09 00 01'
2026-10-15T22:49:53.908869Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:49:54.009802Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10056169999734266
2026-10-15T22:49:54.009922Z TX_ATTEMPT_BEGIN index=4 payload='A5 09 00 01'
2026-10-15T22:49:54.041489Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:49:54.142333Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10069640000074287
2026-10-15T22:49:54.142779Z TX_ATTEMPT_BEGIN index=5 payload='A5 09 00 01'
2026-10-15T22:49:54.186411Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:49:54.287483Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.1005525000000489
2026-10-15T22:49:54.287910Z TX_ATTEMPT_BEGIN index=6 payload='A5 09 00 01'
2026-10-15T22:49:54.330289Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:49:54.430669Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10015100000236998
2026-10-15T22:49:54.431161Z TX_ATTEMPT_BEGIN index=7 payload='A5 09 00 01'
2026-10-15T22:49:54.475581Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:49:54.576188Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.1001192000003357
2026-10-15T22:49:54.576357Z TX_ATTEMPT_BEGIN index=8 payload='A5 09 00 01'
2026-10-15T22:49:54.610707Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:49:54.711116Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10018649999983609
2026-10-15T22:49:54.711467Z TX_ATTEMPT_BEGIN index=9 payload='A5 09 00 01'
2026-10-15T22:49:54.752933Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:49:54.854031Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10059739999996964
2026-10-15T22:49:54.854386Z TX_ATTEMPT_BEGIN index=10 payload='A5 09 00 01'
2026-10-15T22:49:54.897821Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:49:54.998664Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10033240000120713
2026-10-15T22:49:54.999171Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:49:55.999719Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0005636000023514
2026-10-15T22:49:55.999857Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:49:56.113044Z RX_PACKET ordinal=1 device_timestamp_us=3781116 channel=37 rssi_dbm=-76 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:56.227018Z RX_PACKET ordinal=2 device_timestamp_us=3781683 channel=37 rssi_dbm=43 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:56.338556Z RX_PACKET ordinal=3 device_timestamp_us=3782241 channel=37 rssi_dbm=-58 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:56.448380Z RX_PACKET ordinal=4 device_timestamp_us=3782803 channel=37 rssi_dbm=58 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:56.564523Z RX_PACKET ordinal=5 device_timestamp_us=3783323 channel=37 rssi_dbm=-83 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:56.675265Z RX_PACKET ordinal=6 device_timestamp_us=3783889 channel=37 rssi_dbm=-71 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:56.789860Z RX_PACKET ordinal=7 device_timestamp_us=3784449 channel=37 rssi_dbm=113 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:56.907056Z RX_PACKET ordinal=8 device_timestamp_us=3785015 channel=37 rssi_dbm=-124 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:57.019178Z RX_PACKET ordinal=9 device_timestamp_us=3785543 channel=37 rssi_dbm=59 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:57.132497Z RX_PACKET ordinal=10 device_timestamp_us=3786102 channel=37 rssi_dbm=38 lqi=0 crc_ok=True payload_hex='04 A5 09 00 01 61 61' marker_present=True
2026-10-15T22:49:58.011024Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.011122700001579 packets=10 async_errors=0
2026-10-15T22:49:58.011452Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:49:58.517632Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5061755000024277 async_errors=0
2026-10-15T22:49:58.517806Z RX_STOP_BEGIN
2026-10-15T22:49:58.547563Z RX_STOP_ACK
2026-10-15T22:49:58.547680Z TX_STATS_AFTER_BEGIN
2026-10-15T22:49:58.725716Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:49:58.725894Z RX_STATS_AFTER_BEGIN
2026-10-15T22:49:58.898035Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:49:58.898155Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:49:58.898943Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:49:58.899015Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:49:58.899579Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:49:58.899669Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:49:58.899727Z TEST_CLASSIFICATION label='campaign-gfsk-868-50k-run-01' preset='gfsk_868_50k' classification='PASS-RF'
2026-10-15T22:49:58.899808Z TEST_END exit_code=0

---

## Paso 36 — `gfsk_868_50k`: PASS-RF (10/10) ✅

**La primera prueba OTA a 868 MHz terminó satisfactoriamente.** Los dos CatSniffer completaron la configuración solicitada y el receptor entregó diez paquetes con el marcador esperado y CRC reportado como válido.

Este resultado es importante porque amplía la validación experimental de FeralRF desde las frecuencias de 902.2 y 915 MHz hasta **868 MHz**.

### 1. Resultado experimental

| Parámetro                | Resultado                      |
| ------------------------ | ------------------------------ |
| Ejecución                | `campaign-gfsk-868-50k-run-01` |
| Fecha                    | 2026-10-15, 22:49 UTC          |
| TX                       | CatSniffer #2 — `COM33`        |
| RX                       | CatSniffer #3 — `COM88`        |
| Frecuencia solicitada    | **868 MHz**                    |
| Modulación solicitada    | GFSK (`mod_type=1`)            |
| Symbol rate solicitado   | 50 000 símbolos/s              |
| `deviation`              | 100                            |
| `rx_bw`                  | 82                             |
| Sync word                | `0x930B51DE`                   |
| Potencia solicitada      | 0 dBm                          |
| Marcador                 | `A5 09 00 01`                  |
| TX ACK                   | **10/10**                      |
| RX CRC-good con marcador | **10/10**                      |
| Errores asíncronos       | 0                              |
| Excepciones fatales      | 0                              |
| **Clasificación**        | **PASS-RF**                    |

**Fuente:** registro proporcionado de `manual_preset_runner.py`, desde `TEST_BEGIN` hasta `TEST_END`.

### 2. Evidencia de recepción

Los diez paquetes recibidos contienen:

```text
payload_hex='04 A5 09 00 01 61 61'
crc_ok=True
marker_present=True
```

Los contadores del receptor confirman:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se completaron correctamente `RX_STOP_ACK` y las desconexiones de ambos dispositivos.

**Conclusión experimental:** bajo la configuración solicitada de 868 MHz, se observó una entrega OTA satisfactoria de diez paquetes entre los CatSniffer.

La evidencia demuestra funcionamiento del enlace, pero no verifica independientemente la frecuencia radiada, la potencia de salida ni las características espectrales de GFSK.

---

### 3. Anomalías persistentes

Los metadatos de recepción continúan mostrando inconsistencias:

| Campo      | Observación           |
| ---------- | --------------------- |
| `channel`  | Siempre 37            |
| `lqi`      | Siempre 0             |
| `rssi_dbm` | Entre −124 y +113 dBm |

Estos valores refuerzan la necesidad de auditar posteriormente la generación e interpretación de los metadatos RF.

**No debemos utilizar el RSSI reportado para comparar la calidad del enlace entre 868 y 915 MHz.**

---

## 4. Estado acumulado de la campaña

| Indicador                          |   Resultado |
| ---------------------------------- | ----------: |
| Presets nuevos evaluados           |           9 |
| Presets nuevos PASS-RF             |     **9/9** |
| Controles positivos satisfactorios |           3 |
| Ejecuciones OTA satisfactorias     |      **12** |
| Paquetes CRC-good con marcador     | **120/120** |
| Ejecuciones BLOCKED                |           1 |

La ejecución bloqueada corresponde al timeout anterior en `tx.get_stats()`, antes de transmitir.

### Avance por frecuencia

| Frecuencia solicitada | Presets nuevos con PASS-RF |
| --------------------- | -------------------------: |
| 902.2 MHz             |                          5 |
| 915 MHz               |                          3 |
| 868 MHz               |                      **1** |
| **Total**             |                      **9** |

El resultado de 868 MHz constituye evidencia positiva para esa configuración concreta, no para toda la banda ni para todos los protocolos Sub-GHz.

---

## Paso 37 — Inspeccionar `gfsk_868_100k`

El siguiente preset es:

`gfsk_868_100k`

Antes de transmitir, comprobaremos sus parámetros reales en `PROP_PRESETS`.

### Ejecuta únicamente:

```powershell
$PRESET = 'gfsk_868_100k'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

Este comando **no transmite RF ni modifica la configuración de los CatSniffer**.

Con la salida verificaremos frecuencia, modulación, symbol rate, desviación, ancho de banda y sync word.

Después prepararemos la prueba OTA con el marcador `A5 0A 00 01`.

**Estado actual:** 9/9 presets nuevos PASS-RF, incluida la primera configuración a 868 MHz. El timeout intermitente y las inconsistencias de metadatos RF permanecen como cuestiones abiertas.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
gfsk_868_100k {'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}

---

## Paso 37 — Preset `gfsk_868_100k` verificado ✅

La consulta confirma que `gfsk_868_100k` existe en `PROP_PRESETS` y solicita una configuración **GFSK a 868 MHz, con 100 000 símbolos/s**.

### 1. Parámetros identificados

| Parámetro             | Valor                  |
| --------------------- | ---------------------- |
| Preset                | `gfsk_868_100k`        |
| Frecuencia solicitada | **868 MHz**            |
| Modulación solicitada | GFSK (`mod_type=1`)    |
| Symbol rate           | **100 000 símbolos/s** |
| `deviation`           | 150                    |
| `rx_bw`               | 86                     |
| Sync word             | `0x930B51DE`           |
| Potencia prevista     | 0 dBm                  |

**Fuente:** definición de `PROP_PRESETS` consultada mediante la biblioteca Python de FeralRF.

Los valores `deviation=150` y `rx_bw=86` se conservarán tal como aparecen en la configuración. Sus unidades y su correspondencia con los parámetros físicos del transceptor requieren verificación en el firmware.

### 2. Comparación con `gfsk_868_50k`

| Parámetro     | `gfsk_868_50k` | `gfsk_868_100k` |
| ------------- | -------------: | --------------: |
| Frecuencia    |        868 MHz |         868 MHz |
| `mod_type`    |              1 |               1 |
| `symbol_rate` |         50 000 |     **100 000** |
| `deviation`   |            100 |         **150** |
| `rx_bw`       |             82 |          **86** |
| Sync word     |   `0x930B51DE` |    `0x930B51DE` |

La frecuencia y el tipo de modulación permanecen iguales.

Sin embargo, cambian tres parámetros: `symbol_rate`, `deviation` y `rx_bw`.

Por tanto, esta prueba evaluará una **combinación de parámetros**, no el efecto aislado de duplicar el symbol rate.

---

## Paso 38 — Prueba OTA `gfsk_868_100k`

La ejecución anterior, `gfsk_868_50k`, obtuvo **PASS-RF 10/10**. Las condiciones de antenas y autorización para las pruebas a 868 MHz ya fueron confirmadas.

### Condiciones de ejecución

| Elemento              | Valor                           |
| --------------------- | ------------------------------- |
| TX                    | CatSniffer #2 — `COM33`         |
| RX                    | CatSniffer #3 — `COM88`         |
| Preset                | `gfsk_868_100k`                 |
| Frecuencia solicitada | 868 MHz                         |
| Potencia solicitada   | 0 dBm                           |
| Marcador              | `A5 0A 00 01`                   |
| Transmisiones         | 10                              |
| Identificador         | `campaign-gfsk-868-100k-run-01` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    gfsk_868_100k `
    A50A0001 `
    campaign-gfsk-868-100k-run-01
```

### Criterios de aceptación

Para clasificar la ejecución como **PASS-RF**, comprobaremos:

1. Diez `TX_ATTEMPT_ACK`.
2. Diez paquetes RX con `crc_ok=True` y `marker_present=True`.
3. Incremento de `rx_ok` de 0 a 10.
4. Ausencia de errores asíncronos y excepciones fatales.
5. `RX_STOP_ACK` y desconexión correcta de ambos dispositivos.

Si aparece un timeout o cualquier otro error, detendremos la secuencia para analizarlo antes de repetir.

### Estado de la campaña

Actualmente tenemos **9/9 presets nuevos con PASS-RF**, 120/120 paquetes CRC-good con marcador entre las doce ejecuciones OTA satisfactorias y una ejecución bloqueada por timeout.

**Envíame el registro completo hasta `TEST_END`.**

Si esta prueba resulta satisfactoria, habremos completado diez presets nuevos. Antes de comenzar las pruebas `wireless_mbus`, corresponderá ejecutar el siguiente control positivo, ya que habremos acumulado tres presets nuevos desde el último control.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     gfsk_868_100k `
>>     A50A0001 `
>>     campaign-gfsk-868-100k-run-01
2026-10-15T22:53:00.008550Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-gfsk-868-100k-run-01' tx_port='COM33' rx_port='COM88' preset_name='gfsk_868_100k' preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606} marker='A5 0A 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:53:00.008833Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:53:01.122374Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:53:01.122484Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:53:02.236255Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:53:02.236439Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:53:02.266287Z TX_SET_PHY_ACK
2026-10-15T22:53:02.266430Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:53:02.300726Z RX_SET_PHY_ACK
2026-10-15T22:53:02.300840Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
2026-10-15T22:53:02.335266Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:53:02.335414Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
2026-10-15T22:53:02.367790Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:53:02.367903Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:53:02.395954Z TX_SET_POWER_ACK
2026-10-15T22:53:02.396059Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:53:02.562612Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:53:02.562774Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:53:02.733452Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:53:02.733606Z RX_START_BEGIN
2026-10-15T22:53:02.765793Z RX_START_ACK
2026-10-15T22:53:02.766129Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:53:03.066797Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.300884500000393
2026-10-15T22:53:03.066930Z TX_ATTEMPT_BEGIN index=1 payload='A5 0A 00 01'
2026-10-15T22:53:03.095975Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:53:03.196638Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10023759999967297
2026-10-15T22:53:03.196764Z TX_ATTEMPT_BEGIN index=2 payload='A5 0A 00 01'
2026-10-15T22:53:03.227676Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:53:03.328464Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10051050000038231
2026-10-15T22:53:03.328871Z TX_ATTEMPT_BEGIN index=3 payload='A5 0A 00 01'
2026-10-15T22:53:03.362919Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:53:03.463729Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.1004873999991105
2026-10-15T22:53:03.463883Z TX_ATTEMPT_BEGIN index=4 payload='A5 0A 00 01'
2026-10-15T22:53:03.493579Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:53:03.594209Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10046589999910793
2026-10-15T22:53:03.594389Z TX_ATTEMPT_BEGIN index=5 payload='A5 0A 00 01'
2026-10-15T22:53:03.629068Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:53:03.729835Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10044279999783612
2026-10-15T22:53:03.729948Z TX_ATTEMPT_BEGIN index=6 payload='A5 0A 00 01'
2026-10-15T22:53:03.763180Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:53:03.863957Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10051899999962188
2026-10-15T22:53:03.864201Z TX_ATTEMPT_BEGIN index=7 payload='A5 0A 00 01'
2026-10-15T22:53:03.893930Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:53:03.994569Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10040369999842369
2026-10-15T22:53:03.994733Z TX_ATTEMPT_BEGIN index=8 payload='A5 0A 00 01'
2026-10-15T22:53:04.029596Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:53:04.130149Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10016779999932623
2026-10-15T22:53:04.130282Z TX_ATTEMPT_BEGIN index=9 payload='A5 0A 00 01'
2026-10-15T22:53:04.163402Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:53:04.264118Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.1002282000008563
2026-10-15T22:53:04.264231Z TX_ATTEMPT_BEGIN index=10 payload='A5 0A 00 01'
2026-10-15T22:53:04.295337Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:53:04.395737Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10021400000186986
2026-10-15T22:53:04.395893Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:53:05.396570Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0005816000011691
2026-10-15T22:53:05.396924Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:53:05.512568Z RX_PACKET ordinal=1 device_timestamp_us=327006 channel=37 rssi_dbm=-108 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:05.626807Z RX_PACKET ordinal=2 device_timestamp_us=327512 channel=37 rssi_dbm=71 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:05.742576Z RX_PACKET ordinal=3 device_timestamp_us=328029 channel=37 rssi_dbm=-2 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:05.857406Z RX_PACKET ordinal=4 device_timestamp_us=328556 channel=37 rssi_dbm=42 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:05.970934Z RX_PACKET ordinal=5 device_timestamp_us=329065 channel=37 rssi_dbm=57 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:06.087120Z RX_PACKET ordinal=6 device_timestamp_us=329595 channel=37 rssi_dbm=122 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:06.203926Z RX_PACKET ordinal=7 device_timestamp_us=330119 channel=37 rssi_dbm=39 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:06.315749Z RX_PACKET ordinal=8 device_timestamp_us=330630 channel=37 rssi_dbm=-20 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:06.429464Z RX_PACKET ordinal=9 device_timestamp_us=331159 channel=37 rssi_dbm=31 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:06.541260Z RX_PACKET ordinal=10 device_timestamp_us=331682 channel=37 rssi_dbm=84 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:53:07.408862Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0119217000028584 packets=10 async_errors=0
2026-10-15T22:53:07.408993Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:53:07.915747Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5067344999988563 async_errors=0
2026-10-15T22:53:07.916057Z RX_STOP_BEGIN
2026-10-15T22:53:07.949370Z RX_STOP_ACK
2026-10-15T22:53:07.949572Z TX_STATS_AFTER_BEGIN
2026-10-15T22:53:08.123284Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:53:08.123416Z RX_STATS_AFTER_BEGIN
2026-10-15T22:53:08.286102Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:53:08.286215Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:53:08.286941Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:53:08.287020Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:53:08.287639Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:53:08.287743Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:53:08.287816Z TEST_CLASSIFICATION label='campaign-gfsk-868-100k-run-01' preset='gfsk_868_100k' classification='PASS-RF'
2026-10-15T22:53:08.287871Z TEST_END exit_code=0

---

## Paso 38 — `gfsk_868_100k`: PASS-RF (10/10) ✅

**La décima prueba de la campaña OTA terminó satisfactoriamente.** El receptor obtuvo diez paquetes con el marcador esperado y CRC reportado como válido.

Con este resultado, hemos validado la entrega OTA básica de dos configuraciones GFSK solicitadas a 868 MHz: 50 000 y 100 000 símbolos/s.

### 1. Resultado experimental

| Parámetro                | Resultado                       |
| ------------------------ | ------------------------------- |
| Ejecución                | `campaign-gfsk-868-100k-run-01` |
| Fecha                    | 2026-10-15, 22:53 UTC           |
| TX                       | CatSniffer #2 — `COM33`         |
| RX                       | CatSniffer #3 — `COM88`         |
| Frecuencia solicitada    | 868 MHz                         |
| Modulación solicitada    | GFSK (`mod_type=1`)             |
| Symbol rate solicitado   | 100 000 símbolos/s              |
| `deviation`              | 150                             |
| `rx_bw`                  | 86                              |
| Sync word                | `0x930B51DE`                    |
| Potencia solicitada      | 0 dBm                           |
| Marcador                 | `A5 0A 00 01`                   |
| TX ACK                   | **10/10**                       |
| RX CRC-good con marcador | **10/10**                       |
| Errores asíncronos       | 0                               |
| Excepciones fatales      | 0                               |
| **Clasificación**        | **PASS-RF**                     |

**Fuente:** registro de ejecución proporcionado, desde `TEST_BEGIN` hasta `TEST_END`.

### 2. Evidencia de recepción

Los diez paquetes recibidos contienen:

```text
payload_hex='04 A5 0A 00 01 61 5D'
crc_ok=True
marker_present=True
```

Los contadores del receptor muestran:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se completaron correctamente `RX_STOP_ACK` y las desconexiones de ambos dispositivos.

**Conclusión experimental:** la configuración solicitada `gfsk_868_100k` permitió recibir diez paquetes con el marcador esperado y CRC válido según el firmware.

Esto no constituye una medición independiente de la frecuencia, la potencia radiada ni el symbol rate físico.

### 3. Anomalías persistentes

Los metadatos siguen mostrando inconsistencias:

| Campo      | Observación           |
| ---------- | --------------------- |
| `channel`  | Siempre 37            |
| `lqi`      | Siempre 0             |
| `rssi_dbm` | Entre −108 y +122 dBm |

No utilizaremos estos valores para evaluar la calidad RF.

---

## 4. Estado acumulado de la campaña

| Indicador                          |   Resultado |
| ---------------------------------- | ----------: |
| Presets nuevos evaluados           |          10 |
| Presets nuevos PASS-RF             |   **10/10** |
| Controles positivos satisfactorios |           3 |
| Ejecuciones OTA satisfactorias     |      **13** |
| Paquetes CRC-good con marcador     | **130/130** |
| Ejecuciones BLOCKED                |           1 |

Desde el último control positivo hemos completado:

1. `sidewalk_915_fsk_50k` — PASS-RF.
2. `sidewalk_915_fsk_250k` — PASS-RF.
3. `gfsk_868_50k` — PASS-RF.
4. `gfsk_868_100k` — PASS-RF.

**Corrección del seguimiento:** ya son cuatro presets nuevos desde el último control, no tres. Por tanto, corresponde ejecutar inmediatamente el siguiente control positivo antes de continuar con `wireless_mbus`.

---

## Paso 39 — Control positivo después de los presets 7–10

Utilizaremos nuevamente el preset de referencia:

`wisun_915_fsk_50k`

Este control solicita 902.2 MHz, por lo que **antes de ejecutarlo debemos asegurar que las antenas instaladas también sean compatibles con 902–928 MHz**.

La confirmación anterior de compatibilidad con 868 MHz no demuestra necesariamente esa cobertura.

### Confírmame una sola cosa

**¿Las antenas que tienes instaladas actualmente en ambos CatSniffer también son compatibles con 902–928 MHz?**

Si son antenas multibanda que cubren 868 y 915 MHz, podremos ejecutar el control sin cambiarlas.

Si son exclusivamente de 868 MHz, primero tendremos que volver a instalar las antenas de 902–928 MHz.

**No ejecutes todavía el control positivo.**

El siguiente paso dependerá de esa confirmación.

---

Sí, las antenas son compatibles, podemos proceder, pero antes, me gustaría mostrarte un resultado que tuve después de desconectar, probar transmisión-recepción, y después intentar de nuevo tras reconectar: PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     gfsk_868_100k `
>>     A50A0001 `
>>     campaign-gfsk-868-100k-run-01
2026-10-15T22:54:05.490002Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-gfsk-868-100k-run-01' tx_port='COM33' rx_port='COM88' preset_name='gfsk_868_100k' preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606} marker='A5 0A 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:54:05.490147Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:54:05.490985Z TEST_FATAL_EXCEPTION exception_type='ConnectionError' exception='ConnectionError("Failed to connect to COM33: could not open port \'COM33\': FileNotFoundError(2, \'The system cannot find the file specified.\', None, 2)")'
Traceback (most recent call last):
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 170, in connect
    self._serial = serial.Serial(
                   ~~~~~~~~~~~~~^
        port=self.port,
        ^^^^^^^^^^^^^^^
    ...<2 lines>...
        write_timeout=1.0,
        ^^^^^^^^^^^^^^^^^^
    )
    ^
  File "C:\Users\Support\AppData\Local\Programs\Python\Python314\Lib\site-packages\serial\serialwin32.py", line 33, in __init__
    super(Serial, self).__init__(*args, **kwargs)
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^
  File "C:\Users\Support\AppData\Local\Programs\Python\Python314\Lib\site-packages\serial\serialutil.py", line 244, in __init__
    self.open()
    ~~~~~~~~~^^
  File "C:\Users\Support\AppData\Local\Programs\Python\Python314\Lib\site-packages\serial\serialwin32.py", line 64, in open
    raise SerialException("could not open port {!r}: {!r}".format(self.portstr, ctypes.WinError()))
serial.serialutil.SerialException: could not open port 'COM33': FileNotFoundError(2, 'The system cannot find the file specified.', None, 2)

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\experimental_artifacts\manual_preset_runner.py", line 78, in main
    tx_info = tx.init()
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 409, in init
    self.connect()
    ~~~~~~~~~~~~^^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 182, in connect
    raise ConnectionError(f"Failed to connect to {self.port}: {e}")
feralrf.exceptions.ConnectionError: Failed to connect to COM33: could not open port 'COM33': FileNotFoundError(2, 'The system cannot find the file specified.', None, 2)
2026-10-15T22:54:05.516647Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:54:05.516736Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:54:05.516818Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:54:05.516882Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:54:05.516947Z TEST_SUMMARY tx_accepted=0 tx_requested=10 total_peer_packets=0 peer_crc_good_marker=0 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception='ConnectionError("Failed to connect to COM33: could not open port \'COM33\': FileNotFoundError(2, \'The system cannot find the file specified.\', None, 2)")'
2026-10-15T22:54:05.517019Z TEST_CLASSIFICATION label='campaign-gfsk-868-100k-run-01' preset='gfsk_868_100k' classification='BLOCKED'
2026-10-15T22:54:05.517077Z TEST_END exit_code=2
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     gfsk_868_100k `
>>     A50A0001 `
>>     campaign-gfsk-868-100k-run-01
2026-10-15T22:54:12.475528Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-gfsk-868-100k-run-01' tx_port='COM33' rx_port='COM88' preset_name='gfsk_868_100k' preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606} marker='A5 0A 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:54:12.475664Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:54:13.598355Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:54:13.598449Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:54:14.716731Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:54:14.716871Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:54:14.750702Z TX_SET_PHY_ACK
2026-10-15T22:54:14.750855Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:54:14.784018Z RX_SET_PHY_ACK
2026-10-15T22:54:14.784159Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
2026-10-15T22:54:14.815984Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:54:14.816092Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868000000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 150, 'rx_bw': 86, 'sync_word': 2466992606}
2026-10-15T22:54:14.848706Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:54:14.848864Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:54:14.881316Z TX_SET_POWER_ACK
2026-10-15T22:54:14.881514Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:54:15.060388Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:54:15.060504Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:54:15.228698Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:54:15.228812Z RX_START_BEGIN
2026-10-15T22:54:15.262688Z RX_START_ACK
2026-10-15T22:54:15.262797Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:54:15.563399Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30055519999950775
2026-10-15T22:54:15.563925Z TX_ATTEMPT_BEGIN index=1 payload='A5 0A 00 01'
2026-10-15T22:54:15.598366Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:54:15.698839Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10032800000044517
2026-10-15T22:54:15.699055Z TX_ATTEMPT_BEGIN index=2 payload='A5 0A 00 01'
2026-10-15T22:54:15.733401Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:54:15.833916Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10026559999823803
2026-10-15T22:54:15.834078Z TX_ATTEMPT_BEGIN index=3 payload='A5 0A 00 01'
2026-10-15T22:54:15.869394Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:54:15.970238Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10069210000074236
2026-10-15T22:54:15.970748Z TX_ATTEMPT_BEGIN index=4 payload='A5 0A 00 01'
2026-10-15T22:54:16.009572Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:54:16.110139Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.1004508000005444
2026-10-15T22:54:16.110244Z TX_ATTEMPT_BEGIN index=5 payload='A5 0A 00 01'
2026-10-15T22:54:16.146071Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:54:16.246324Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10013569999864558
2026-10-15T22:54:16.246479Z TX_ATTEMPT_BEGIN index=6 payload='A5 0A 00 01'
2026-10-15T22:54:16.280843Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:54:16.381282Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10028890000103274
2026-10-15T22:54:16.381880Z TX_ATTEMPT_BEGIN index=7 payload='A5 0A 00 01'
2026-10-15T22:54:16.420677Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:54:16.520941Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10014609999780077
2026-10-15T22:54:16.521068Z TX_ATTEMPT_BEGIN index=8 payload='A5 0A 00 01'
2026-10-15T22:54:16.557464Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:54:16.657785Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10019509999983711
2026-10-15T22:54:16.657930Z TX_ATTEMPT_BEGIN index=9 payload='A5 0A 00 01'
2026-10-15T22:54:16.688961Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:54:16.789328Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10019539999848348
2026-10-15T22:54:16.789773Z TX_ATTEMPT_BEGIN index=10 payload='A5 0A 00 01'
2026-10-15T22:54:16.830380Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:54:16.930685Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10017460000017309
2026-10-15T22:54:16.930963Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:54:17.931590Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.000625000000582
2026-10-15T22:54:17.931731Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:54:18.055061Z RX_PACKET ordinal=1 device_timestamp_us=610199 channel=37 rssi_dbm=-98 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:18.178603Z RX_PACKET ordinal=2 device_timestamp_us=610726 channel=37 rssi_dbm=61 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:18.298484Z RX_PACKET ordinal=3 device_timestamp_us=611252 channel=37 rssi_dbm=-26 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:18.416304Z RX_PACKET ordinal=4 device_timestamp_us=611788 channel=37 rssi_dbm=-63 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:18.533935Z RX_PACKET ordinal=5 device_timestamp_us=612332 channel=37 rssi_dbm=-108 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:18.650047Z RX_PACKET ordinal=6 device_timestamp_us=612864 channel=37 rssi_dbm=57 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:18.772275Z RX_PACKET ordinal=7 device_timestamp_us=613393 channel=37 rssi_dbm=0 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:18.891983Z RX_PACKET ordinal=8 device_timestamp_us=613937 channel=37 rssi_dbm=93 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:19.008952Z RX_PACKET ordinal=9 device_timestamp_us=614470 channel=37 rssi_dbm=60 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:19.124634Z RX_PACKET ordinal=10 device_timestamp_us=614987 channel=37 rssi_dbm=-110 lqi=0 crc_ok=True payload_hex='04 A5 0A 00 01 61 5D' marker_present=True
2026-10-15T22:54:19.950525Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0187492999975802 packets=10 async_errors=0
2026-10-15T22:54:19.951029Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:54:20.467303Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5162849000007554 async_errors=0
2026-10-15T22:54:20.467427Z RX_STOP_BEGIN
2026-10-15T22:54:20.499931Z RX_STOP_ACK
2026-10-15T22:54:20.500059Z TX_STATS_AFTER_BEGIN
2026-10-15T22:54:20.672103Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:54:20.672373Z RX_STATS_AFTER_BEGIN
2026-10-15T22:54:20.843525Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:54:20.843694Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:54:20.844408Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:54:20.844500Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:54:20.845253Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:54:20.845349Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:54:20.845427Z TEST_CLASSIFICATION label='campaign-gfsk-868-100k-run-01' preset='gfsk_868_100k' classification='PASS-RF'
2026-10-15T22:54:20.845488Z TEST_END exit_code=0

---

## Paso 39 — Análisis del incidente tras la desconexión

Los registros muestran **un fallo de disponibilidad del puerto COM33 seguido de una ejecución OTA satisfactoria**.

Es importante conservar ambos resultados porque aportan información distinta sobre el comportamiento de FeralRF.

### 1. Comparación de las ejecuciones

| Hora (UTC) | Resultado  | Evidencia                                   |
| ---------- | ---------- | ------------------------------------------- |
| 22:53:00   | ✅ PASS-RF  | Primera ejecución de `gfsk_868_100k`: 10/10 |
| 22:54:05   | ⚠️ BLOCKED | Windows no encuentra `COM33`                |
| 22:54:12   | ✅ PASS-RF  | Nueva ejecución de `gfsk_868_100k`: 10/10   |

**Observación importante:** las tres ejecuciones utilizan el mismo identificador:

`campaign-gfsk-868-100k-run-01`

Para conservar una trazabilidad reproducible, debemos distinguirlas por su hora de inicio. En futuras ejecuciones utilizaremos identificadores únicos.

---

### 2. ¿Qué falló exactamente?

El fallo ocurrió durante:

```text
TX_INIT_BEGIN port='COM33'
```

La excepción principal fue:

```text
FileNotFoundError(2, 'The system cannot find the file specified.')
```

La secuencia de llamadas fue:

```text
manual_preset_runner.py
    main()
        ↓
Radio.init()
        ↓
Radio.connect()
        ↓
serial.Serial(port="COM33")
        ↓
Windows: puerto no encontrado
```

**Fuente de código:** `python/feralrf/radio.py`, funciones `init()` y `connect()`, según el traceback proporcionado.

Esto significa que la ejecución se detuvo **antes de establecer comunicación con el dispositivo TX**.

Por tanto:

* No se configuró el transceptor.
* No se inició RX.
* No hubo solicitudes de transmisión.
* No se evaluó el enlace RF.

La clasificación **BLOCKED** es correcta.

#### ¿Es un fallo de FeralRF?

No podemos afirmarlo.

La evidencia inmediata apunta a que Windows no tenía disponible el puerto `COM33` cuando Python intentó abrirlo.

Entre las posibles explicaciones están la reenumeración USB, el cambio de número de puerto o que el dispositivo todavía no hubiera terminado de conectarse.

Son hipótesis, no causas demostradas.

Además, este incidente es diferente del timeout anterior en `get_stats()`:

| Incidente        | Punto de fallo                                                 |
| ---------------- | -------------------------------------------------------------- |
| Timeout anterior | Puerto abierto; no llegó la respuesta esperada a `get_stats()` |
| Incidente actual | Windows no pudo abrir `COM33`                                  |

No debemos agruparlos automáticamente bajo una misma causa.

---

### 3. ¿Qué demuestra la ejecución posterior?

La ejecución iniciada a las **22:54:12 UTC** completó:

| Comprobación                  | Resultado |
| ----------------------------- | --------- |
| `Radio.init()` TX/RX          | ✅         |
| `set_phy()` TX/RX             | ✅         |
| `configure_prop()` TX/RX      | ✅         |
| `set_power()`                 | ✅         |
| `get_stats()` antes y después | ✅         |
| TX ACK                        | **10/10** |
| RX CRC-good con marcador      | **10/10** |
| Errores asíncronos            | 0         |
| `RX_STOP_ACK`                 | ✅         |
| Desconexión                   | ✅         |

El receptor obtuvo diez veces:

```text
04 A5 0A 00 01 61 5D
```

Con:

```text
crc_ok=True
marker_present=True
```

Y sus estadísticas pasaron de `rx_ok=0` a `rx_ok=10`.

**Conclusión:** después de que `COM33` volvió a estar disponible, el procedimiento OTA funcionó nuevamente sin que el registro mostrara errores de comunicación o recepción.

Esto constituye evidencia positiva de recuperación operativa tras la reconexión, pero no demuestra que todos los ciclos de desconexión y reconexión sean confiables.

---

## 4. Estado actualizado de la campaña

| Indicador                          |   Resultado |
| ---------------------------------- | ----------: |
| Presets nuevos evaluados           |          10 |
| Presets nuevos PASS-RF             |   **10/10** |
| Controles positivos satisfactorios |           3 |
| Ejecuciones OTA satisfactorias     |      **14** |
| Paquetes CRC-good con marcador     | **140/140** |
| Ejecuciones BLOCKED                |       **2** |

Las dos ejecuciones bloqueadas corresponden a:

1. Timeout en `get_stats()`.
2. Puerto `COM33` inexistente durante `Radio.init()`.

Ambas deben permanecer en el registro histórico.

**Nota metodológica:** la repetición de `gfsk_868_100k` no constituye un nuevo preset. Seguimos teniendo diez configuraciones nuevas evaluadas.

---

## Paso 40 — Control positivo posterior a la reconexión

Ahora corresponde ejecutar el control positivo de referencia.

Este control tiene especial valor porque permitirá comprobar que, después de la desconexión y reconexión, el enlace sigue funcionando también con la configuración de referencia a 902.2 MHz.

Ya confirmaste que las antenas instaladas son compatibles.

### Condiciones

| Parámetro              | Valor                                            |
| ---------------------- | ------------------------------------------------ |
| Preset                 | `wisun_915_fsk_50k`                              |
| Frecuencia solicitada  | 902.2 MHz                                        |
| Modulación solicitada  | FSK                                              |
| Symbol rate solicitado | 50 000 símbolos/s                                |
| TX                     | `COM33`                                          |
| RX                     | `COM88`                                          |
| Potencia solicitada    | 0 dBm                                            |
| Marcador               | `DE AD BE EF`                                    |
| Transmisiones          | 10                                               |
| Identificador          | `control-after-10-presets-post-reconnect-run-01` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_50k `
    DEADBEEF `
    control-after-10-presets-post-reconnect-run-01
```

**Criterio de aceptación:** diez TX ACK, diez paquetes CRC-good con marcador correcto, incremento de `rx_ok` a 10, ausencia de errores asíncronos y limpieza correcta.

Si aparece nuevamente un error de apertura del puerto, no repetiremos automáticamente: primero comprobaremos la enumeración USB.

Si obtenemos PASS-RF, podremos continuar con la inspección de `wireless_mbus_s_868`.

**Conclusión del incidente:** observamos una indisponibilidad temporal de `COM33` y una ejecución OTA satisfactoria posterior. No hay evidencia de un fallo RF en la ejecución bloqueada, pero la disponibilidad de los puertos USB debe considerarse una condición explícita de las pruebas de reconexión.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_50k `
>>     DEADBEEF `
>>     control-after-10-presets-post-reconnect-run-01
2026-10-15T22:56:29.920068Z TEST_BEGIN checkpoint='manual-campaign' label='control-after-10-presets-post-reconnect-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_50k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='DE AD BE EF' tx_power_dbm=0 tx_count=10
2026-10-15T22:56:29.920214Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:56:31.030182Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:56:31.030298Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:56:32.157336Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:56:32.157433Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:56:32.187574Z TX_SET_PHY_ACK
2026-10-15T22:56:32.187818Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:56:32.219518Z RX_SET_PHY_ACK
2026-10-15T22:56:32.219686Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:56:32.251192Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:56:32.251293Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-15T22:56:32.280802Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:56:32.280928Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:56:32.315247Z TX_SET_POWER_ACK
2026-10-15T22:56:32.315345Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:56:32.482085Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:56:32.482191Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:56:32.645020Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:56:32.645119Z RX_START_BEGIN
2026-10-15T22:56:32.675277Z RX_START_ACK
2026-10-15T22:56:32.675451Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T22:56:32.975729Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30027830000108224
2026-10-15T22:56:32.975890Z TX_ATTEMPT_BEGIN index=1 payload='DE AD BE EF'
2026-10-15T22:56:33.010006Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T22:56:33.110867Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10060990000056336
2026-10-15T22:56:33.111364Z TX_ATTEMPT_BEGIN index=2 payload='DE AD BE EF'
2026-10-15T22:56:33.150288Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T22:56:33.250553Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10014180000143824
2026-10-15T22:56:33.250807Z TX_ATTEMPT_BEGIN index=3 payload='DE AD BE EF'
2026-10-15T22:56:33.288369Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T22:56:33.388887Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10035799999968731
2026-10-15T22:56:33.389345Z TX_ATTEMPT_BEGIN index=4 payload='DE AD BE EF'
2026-10-15T22:56:33.430858Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T22:56:33.531439Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10022810000009486
2026-10-15T22:56:33.531972Z TX_ATTEMPT_BEGIN index=5 payload='DE AD BE EF'
2026-10-15T22:56:33.566989Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T22:56:33.667371Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10022989999924903
2026-10-15T22:56:33.667873Z TX_ATTEMPT_BEGIN index=6 payload='DE AD BE EF'
2026-10-15T22:56:33.708567Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T22:56:33.808997Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.1002184999997553
2026-10-15T22:56:33.809486Z TX_ATTEMPT_BEGIN index=7 payload='DE AD BE EF'
2026-10-15T22:56:33.845066Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T22:56:33.945352Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10016199999881792
2026-10-15T22:56:33.945505Z TX_ATTEMPT_BEGIN index=8 payload='DE AD BE EF'
2026-10-15T22:56:33.978662Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T22:56:34.079412Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10058800000115298
2026-10-15T22:56:34.079521Z TX_ATTEMPT_BEGIN index=9 payload='DE AD BE EF'
2026-10-15T22:56:34.112316Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T22:56:34.213024Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10050870000122814
2026-10-15T22:56:34.213179Z TX_ATTEMPT_BEGIN index=10 payload='DE AD BE EF'
2026-10-15T22:56:34.244980Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T22:56:34.345327Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10018470000068191
2026-10-15T22:56:34.345451Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T22:56:35.345868Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.000389400000131
2026-10-15T22:56:35.346062Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T22:56:35.464966Z RX_PACKET ordinal=1 device_timestamp_us=1146964 channel=37 rssi_dbm=-70 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:35.579528Z RX_PACKET ordinal=2 device_timestamp_us=1147494 channel=37 rssi_dbm=117 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:35.696194Z RX_PACKET ordinal=3 device_timestamp_us=1148038 channel=37 rssi_dbm=124 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:35.815467Z RX_PACKET ordinal=4 device_timestamp_us=1148581 channel=37 rssi_dbm=-92 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:35.931938Z RX_PACKET ordinal=5 device_timestamp_us=1149138 channel=37 rssi_dbm=83 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:36.050553Z RX_PACKET ordinal=6 device_timestamp_us=1149668 channel=37 rssi_dbm=6 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:36.162834Z RX_PACKET ordinal=7 device_timestamp_us=1150221 channel=37 rssi_dbm=-75 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:36.285733Z RX_PACKET ordinal=8 device_timestamp_us=1150752 channel=37 rssi_dbm=-8 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:36.409804Z RX_PACKET ordinal=9 device_timestamp_us=1151276 channel=37 rssi_dbm=117 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:36.530596Z RX_PACKET ordinal=10 device_timestamp_us=1151798 channel=37 rssi_dbm=-92 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-15T22:56:37.358577Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0124924999981886 packets=10 async_errors=0
2026-10-15T22:56:37.358758Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T22:56:37.866071Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5072973999995156 async_errors=0
2026-10-15T22:56:37.866226Z RX_STOP_BEGIN
2026-10-15T22:56:37.898680Z RX_STOP_ACK
2026-10-15T22:56:37.898796Z TX_STATS_AFTER_BEGIN
2026-10-15T22:56:38.066704Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:56:38.066814Z RX_STATS_AFTER_BEGIN
2026-10-15T22:56:38.235860Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:56:38.235956Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:56:38.236666Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:56:38.236739Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:56:38.237219Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:56:38.237292Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T22:56:38.237357Z TEST_CLASSIFICATION label='control-after-10-presets-post-reconnect-run-01' preset='wisun_915_fsk_50k' classification='PASS-RF'
2026-10-15T22:56:38.237411Z TEST_END exit_code=0

---

## Paso 40 — Control positivo posterior a la reconexión: PASS-RF ✅

**El control positivo terminó satisfactoriamente: 10/10 paquetes recibidos con CRC válido según el firmware y marcador correcto.**

Este resultado confirma que, después del incidente de disponibilidad de `COM33` y de las pruebas a 868 MHz, ambos CatSniffer pudieron volver a comunicarse utilizando la configuración de referencia a **902.2 MHz**.

### 1. Resultado experimental

| Parámetro                | Resultado                                        |
| ------------------------ | ------------------------------------------------ |
| Identificador            | `control-after-10-presets-post-reconnect-run-01` |
| Fecha                    | 2026-10-15, 22:56 UTC                            |
| TX                       | CatSniffer #2 — `COM33`                          |
| RX                       | CatSniffer #3 — `COM88`                          |
| Preset                   | `wisun_915_fsk_50k`                              |
| Frecuencia solicitada    | 902.2 MHz                                        |
| Modulación solicitada    | FSK (`mod_type=0`)                               |
| Symbol rate solicitado   | 50 000 símbolos/s                                |
| Potencia solicitada      | 0 dBm                                            |
| Marcador                 | `DE AD BE EF`                                    |
| TX ACK                   | **10/10**                                        |
| RX CRC-good con marcador | **10/10**                                        |
| Errores asíncronos       | 0                                                |
| Excepciones fatales      | 0                                                |
| Clasificación            | **PASS-RF**                                      |

**Fuente:** registro proporcionado de `manual_preset_runner.py`, desde `TEST_BEGIN` hasta `TEST_END`.

### 2. Evidencia de recepción

Los diez paquetes contienen:

```text
payload_hex='04 DE AD BE EF 33 C8'
crc_ok=True
marker_present=True
```

Las estadísticas del receptor muestran:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se completaron correctamente `RX_STOP_ACK`, `TX_DISCONNECT_COMPLETE` y `RX_DISCONNECT_COMPLETE`.

**Interpretación:** el enlace OTA de referencia volvió a funcionar después de la reconexión y del cambio de frecuencia solicitado.

Esto no demuestra que el puerto USB sea siempre estable ni constituye una medición independiente de la señal RF.

### 3. Estado acumulado

| Indicador                      |   Resultado |
| ------------------------------ | ----------: |
| Presets nuevos evaluados       |          10 |
| Presets nuevos PASS-RF         |   **10/10** |
| Controles positivos PASS-RF    |       **4** |
| Ejecuciones OTA satisfactorias |      **15** |
| Paquetes CRC-good con marcador | **150/150** |
| Ejecuciones BLOCKED            |       **2** |

Los incidentes pendientes siguen siendo el timeout de `get_stats()` y la indisponibilidad temporal de `COM33`.

También persisten las inconsistencias en `channel`, `lqi` y `rssi_dbm`. Por ello, estos campos no se utilizarán para comparar la calidad del enlace.

---

## Paso 41 — Inspección del preset `wireless_mbus_s_868`

El siguiente preset previsto en la campaña es:

**`wireless_mbus_s_868`**

Antes de transmitir debemos revisar sus parámetros, especialmente porque cambiaremos de una configuración GFSK genérica a otra identificada en la biblioteca como Wireless M-Bus.

**Una recepción satisfactoria con este preset no demostraría por sí sola interoperabilidad con dispositivos Wireless M-Bus reales.** Para eso sería necesario verificar el formato de trama y los requisitos específicos del protocolo.

### Ejecuta únicamente este comando

```powershell
$PRESET = 'wireless_mbus_s_868'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

Este comando solamente consulta la definición del preset; no transmite ni modifica el estado de los CatSniffer.

Con el resultado podremos comparar su configuración con `gfsk_868_50k` y `gfsk_868_100k` antes de preparar la siguiente prueba OTA.

**Estado actual:** control positivo posterior a la reconexión completado satisfactoriamente. La campaña puede continuar con el siguiente preset.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wireless_mbus_s_868 {'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840}

---

## Paso 41 — Preset `wireless_mbus_s_868` verificado ✅

La consulta confirma que `wireless_mbus_s_868` está definido en `PROP_PRESETS`.

A diferencia de los dos presets GFSK anteriores, este solicita una frecuencia de **868.3 MHz**, un symbol rate de **32 768 símbolos/s** y un sync word diferente.

### 1. Parámetros identificados

| Parámetro             | Valor                 |
| --------------------- | --------------------- |
| Preset                | `wireless_mbus_s_868` |
| Frecuencia solicitada | **868.3 MHz**         |
| Modulación solicitada | GFSK (`mod_type=1`)   |
| Symbol rate           | **32 768 símbolos/s** |
| `deviation`           | 75                    |
| `rx_bw`               | 82                    |
| Sync word             | `0x543C0000`          |
| Potencia prevista     | 0 dBm                 |

**Fuente:** consulta directa de `PROP_PRESETS` mediante la biblioteca Python de FeralRF.

Los valores `deviation` y `rx_bw` se presentan sin unidades hasta verificar su interpretación en el firmware.

### 2. Comparación con las configuraciones anteriores

| Parámetro     | `gfsk_868_50k` | `gfsk_868_100k` | `wireless_mbus_s_868` |
| ------------- | -------------: | --------------: | --------------------: |
| Frecuencia    |        868 MHz |         868 MHz |         **868.3 MHz** |
| `mod_type`    |              1 |               1 |                     1 |
| `symbol_rate` |         50 000 |         100 000 |            **32 768** |
| `deviation`   |            100 |             150 |                **75** |
| `rx_bw`       |             82 |              86 |                **82** |
| Sync word     |   `0x930B51DE` |    `0x930B51DE` |      **`0x543C0000`** |

Esta prueba permitirá comprobar si dos CatSniffer pueden intercambiar paquetes usando la configuración específica del preset.

Sin embargo, **no demostrará compatibilidad con Wireless M-Bus Mode S**. Para establecerla tendríamos que verificar, entre otros aspectos, la codificación, el formato de trama y la sincronización exigidos por el protocolo.

---

## Paso 42 — Prueba OTA `wireless_mbus_s_868`

El control positivo anterior obtuvo **PASS-RF 10/10**. Las antenas compatibles con 868 MHz están confirmadas y no hay una anomalía activa que obligue a detener la campaña.

### Condiciones de ejecución

| Elemento              | Valor                                 |
| --------------------- | ------------------------------------- |
| TX                    | CatSniffer #2 — `COM33`               |
| RX                    | CatSniffer #3 — `COM88`               |
| Preset                | `wireless_mbus_s_868`                 |
| Frecuencia solicitada | 868.3 MHz                             |
| Potencia solicitada   | 0 dBm                                 |
| Marcador              | `A5 0B 00 01`                         |
| Transmisiones         | 10                                    |
| Identificador         | `campaign-wireless-mbus-s-868-run-01` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wireless_mbus_s_868 `
    A50B0001 `
    campaign-wireless-mbus-s-868-run-01
```

### Criterios de aceptación

La ejecución se clasificará como **PASS-RF** si presenta diez TX ACK, diez paquetes recibidos con `crc_ok=True` y `marker_present=True`, incremento de `rx_ok` a 10, ausencia de errores asíncronos y limpieza correcta de ambos dispositivos.

Si aparece un error, conservaremos el resultado original y analizaremos su punto de fallo antes de repetir.

**Estado actual de la campaña:** 10/10 presets nuevos PASS-RF, cuatro controles positivos satisfactorios y 150/150 paquetes recibidos con CRC válido reportado por el firmware.

Envíame el registro completo hasta `TEST_END` para evaluar esta primera prueba del grupo `wireless_mbus`.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wireless_mbus_s_868 {'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840}
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wireless_mbus_s_868 `
>>     A50B0001 `
>>     campaign-wireless-mbus-s-868-run-01
2026-10-15T22:58:27.710922Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wireless-mbus-s-868-run-01' tx_port='COM33' rx_port='COM88' preset_name='wireless_mbus_s_868' preset={'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840} marker='A5 0B 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T22:58:27.711056Z TX_INIT_BEGIN port='COM33'
2026-10-15T22:58:28.827423Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:58:28.827534Z RX_INIT_BEGIN port='COM88'
2026-10-15T22:58:29.937906Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T22:58:29.938070Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:58:29.971458Z TX_SET_PHY_ACK
2026-10-15T22:58:29.971830Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T22:58:30.003992Z RX_SET_PHY_ACK
2026-10-15T22:58:30.004136Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840}
2026-10-15T22:58:30.039870Z TX_CONFIGURE_PROP_ACK
2026-10-15T22:58:30.040057Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840}
2026-10-15T22:58:30.072959Z RX_CONFIGURE_PROP_ACK
2026-10-15T22:58:30.073082Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T22:58:30.107884Z TX_SET_POWER_ACK
2026-10-15T22:58:30.108167Z TX_STATS_BEFORE_BEGIN
2026-10-15T22:58:30.275004Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T22:58:30.275119Z RX_STATS_BEFORE_BEGIN
2026-10-15T22:58:32.292701Z TEST_FATAL_EXCEPTION exception_type='TimeoutError' exception="TimeoutError('Response timeout')"
Traceback (most recent call last):
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\experimental_artifacts\manual_preset_runner.py", line 110, in main
    rx_stats_before = rx.get_stats()
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 556, in get_stats
    cmd_id, seq, payload = self._read_response(
                           ~~~~~~~~~~~~~~~~~~~^
        timeout=timeout, expected={Response.STATS, Response.ERROR}
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 336, in _read_response
    raise TimeoutError("Response timeout")
feralrf.exceptions.TimeoutError: Response timeout
2026-10-15T22:58:32.313730Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T22:58:32.315575Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T22:58:32.316010Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T22:58:32.317794Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T22:58:32.318292Z TEST_SUMMARY tx_accepted=0 tx_requested=10 total_peer_packets=0 peer_crc_good_marker=0 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception="TimeoutError('Response timeout')"
2026-10-15T22:58:32.318699Z TEST_CLASSIFICATION label='campaign-wireless-mbus-s-868-run-01' preset='wireless_mbus_s_868' classification='BLOCKED'
2026-10-15T22:58:32.319134Z TEST_END exit_code=2

---

## Paso 42 — `wireless_mbus_s_868`: BLOCKED ⚠️

**La prueba no llegó a transmitir ningún paquete.** Se produjo un timeout al consultar las estadísticas del receptor (`COM88`), antes de iniciar RX.

Este resultado **no demuestra un fallo de la configuración RF `wireless_mbus_s_868`**. Debemos investigar primero la comunicación entre Python y el firmware del receptor.

### 1. Resultado observado

| Parámetro                             | Resultado                             |
| ------------------------------------- | ------------------------------------- |
| Ejecución                             | `campaign-wireless-mbus-s-868-run-01` |
| Fecha                                 | 2026-10-15, 22:58 UTC                 |
| TX                                    | CatSniffer #2 — `COM33`               |
| RX                                    | CatSniffer #3 — `COM88`               |
| Frecuencia solicitada                 | 868.3 MHz                             |
| Modulación solicitada                 | GFSK                                  |
| Symbol rate solicitado                | 32 768 símbolos/s                     |
| `TX_CONFIGURE_PROP_ACK`               | ✅                                     |
| `RX_CONFIGURE_PROP_ACK`               | ✅                                     |
| `TX_STATS_BEFORE`                     | ✅                                     |
| `RX_STATS_BEFORE`                     | ❌ Timeout                             |
| `RX_START_ACK`                        | No ejecutado                          |
| Transmisiones solicitadas al firmware | **0/10**                              |
| Paquetes recibidos                    | 0                                     |
| Clasificación                         | **BLOCKED**                           |

**Fuente:** registro proporcionado de `manual_preset_runner.py`, iniciado a las 22:58:27 UTC.

### 2. ¿Dónde ocurrió el fallo?

La última operación iniciada fue:

```text
2026-10-15T22:58:30.275119Z RX_STATS_BEFORE_BEGIN
```

Dos segundos después:

```text
TEST_FATAL_EXCEPTION
exception_type='TimeoutError'
exception="TimeoutError('Response timeout')"
```

El traceback identifica este recorrido:

```text
manual_preset_runner.py:110
    rx.get_stats()
        ↓
python/feralrf/radio.py:556
    Radio.get_stats()
        ↓
python/feralrf/radio.py:336
    Radio._read_response()
        ↓
    TimeoutError("Response timeout")
```

El programa esperaba una respuesta `Response.STATS` o `Response.ERROR`, pero no recibió una respuesta válida dentro del tiempo establecido.

**Importante:** esto no demuestra que el firmware haya dejado de funcionar. También podría tratarse de una respuesta perdida, retrasada o no reconocida por la biblioteca Python.

### 3. Relación con los incidentes anteriores

Ahora tenemos dos casos de timeout durante `get_stats()`:

| Incidente                       | Dispositivo  | Operación        |
| ------------------------------- | ------------ | ---------------- |
| Control después de seis presets | TX — `COM33` | `tx.get_stats()` |
| `wireless_mbus_s_868`           | RX — `COM88` | `rx.get_stats()` |

En ambos casos:

* La inicialización había terminado correctamente.
* `set_phy()` recibió ACK.
* `configure_prop()` recibió ACK.
* El fallo ocurrió antes de transmitir.
* El procedimiento de desconexión terminó sin excepciones.

Esto sugiere que debemos investigar la **fiabilidad de la consulta `get_stats()`**, en lugar de atribuir inmediatamente el problema a una frecuencia o modulación determinada.

Todavía no podemos determinar si ambos incidentes tienen la misma causa.

---

## 4. Estado actualizado de la campaña

| Indicador                               |                          Resultado |
| --------------------------------------- | ---------------------------------: |
| Presets nuevos PASS-RF                  |                             **10** |
| Presets nuevos pendientes de validación | `wireless_mbus_s_868` y siguientes |
| Controles positivos PASS-RF             |                              **4** |
| Ejecuciones OTA satisfactorias          |                             **15** |
| Paquetes CRC-good con marcador          |                        **150/150** |
| Ejecuciones BLOCKED                     |                              **3** |

Las tres ejecuciones bloqueadas corresponden a dos timeouts de `get_stats()` y una indisponibilidad de `COM33`.

El preset `wireless_mbus_s_868` permanece **sin resultado RF concluyente**.

---

## Paso 43 — Diagnóstico aislado de `get_stats()` en RX

Antes de repetir la transmisión, comprobaremos si el receptor puede responder a una consulta independiente de estadísticas.

No reinicies ni desconectes físicamente los CatSniffer todavía: queremos observar el estado actual después del timeout.

### Ejecuta únicamente este comando

```powershell
@'
from feralrf import Radio

rx = Radio(port="COM88", baudrate=921600)

try:
    print("RX_INIT_BEGIN", flush=True)
    print("RX_INIT_OK", rx.init(), flush=True)

    print("RX_STATS_BEGIN", flush=True)
    print("RX_STATS_RESULT", rx.get_stats(), flush=True)

except Exception as exc:
    print("RX_DIAGNOSTIC_ERROR", type(exc).__name__, repr(exc), flush=True)

finally:
    print("RX_DISCONNECT_BEGIN", flush=True)
    rx.disconnect()
    print("RX_DISCONNECT_COMPLETE", flush=True)
'@ | python -u -
```

**Este diagnóstico no transmite RF, no modifica la configuración del transceptor y no solicita un reset.**

### ¿Qué buscamos?

Si `rx.get_stats()` responde correctamente, tendremos evidencia de que el receptor volvió a atender comandos después del timeout. Eso justificaría preparar una nueva ejecución OTA con un identificador distinto, conservando el intento bloqueado.

Si vuelve a fallar, detendremos la campaña para investigar la comunicación con `COM88`.

**Por ahora, no repitas `wireless_mbus_s_868`.** Primero envíame el resultado de esta consulta aislada.

---

RX_INIT_BEGIN
RX_INIT_OK DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
RX_STATS_BEGIN
RX_STATS_RESULT DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
RX_DISCONNECT_BEGIN
RX_DISCONNECT_COMPLETE

---

## Paso 43 — Diagnóstico aislado de `get_stats()`: PASS ✅

El receptor CatSniffer #3 (`COM88`) respondió correctamente a la consulta de estadísticas después del timeout observado durante `wireless_mbus_s_868`.

### 1. Resultados observados

| Operación            | Resultado  |
| -------------------- | ---------- |
| `Radio.init()`       | ✅ Correcto |
| Firmware reportado   | `1.0.0`    |
| `Radio.get_stats()`  | ✅ Correcto |
| `rx_ok`              | 0          |
| `rx_crc_err`         | 0          |
| `rx_drop`            | 0          |
| `rx_overflow`        | 0          |
| `Radio.disconnect()` | ✅ Correcto |

**Fuente:** salida del diagnóstico aislado proporcionada.

### 2. Interpretación técnica

Este resultado demuestra que:

1. Windows permitió abrir `COM88`.
2. La biblioteca Python estableció comunicación con el dispositivo.
3. El firmware respondió correctamente a `get_stats()`.
4. La conexión se cerró sin errores.

Sin embargo, **no identifica la causa del timeout anterior**.

La hipótesis de un problema intermitente en el intercambio de comandos sigue abierta.

Tampoco podemos concluir que el firmware haya recuperado automáticamente un estado interno defectuoso: el diagnóstico abrió una conexión nueva, por lo que no reproduce exactamente las condiciones de la ejecución bloqueada.

### 3. Decisión de validación

Tenemos evidencia suficiente para realizar un nuevo intento controlado de `wireless_mbus_s_868`.

Conservaremos dos registros independientes:

| Ejecución                             | Clasificación                         |
| ------------------------------------- | ------------------------------------- |
| `campaign-wireless-mbus-s-868-run-01` | BLOCKED — timeout en `rx.get_stats()` |
| `campaign-wireless-mbus-s-868-run-02` | Pendiente                             |

No modificaremos el preset ni reiniciaremos los dispositivos.

---

## Paso 44 — Segundo intento OTA `wireless_mbus_s_868`

### Condiciones

| Parámetro              | Valor                                 |
| ---------------------- | ------------------------------------- |
| TX                     | `COM33`                               |
| RX                     | `COM88`                               |
| Preset                 | `wireless_mbus_s_868`                 |
| Frecuencia solicitada  | 868.3 MHz                             |
| Modulación solicitada  | GFSK                                  |
| Symbol rate solicitado | 32 768 símbolos/s                     |
| Potencia solicitada    | 0 dBm                                 |
| Marcador               | `A5 0B 00 01`                         |
| Transmisiones          | 10                                    |
| Identificador          | `campaign-wireless-mbus-s-868-run-02` |

### Ejecuta únicamente:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wireless_mbus_s_868 `
    A50B0001 `
    campaign-wireless-mbus-s-868-run-02
```

**Criterios de aceptación:** diez TX ACK, diez paquetes recibidos con CRC válido reportado y marcador correcto, incremento de `rx_ok` a 10, ausencia de errores asíncronos y desconexión correcta.

Si vuelve a aparecer un timeout, detendremos la campaña para investigar la secuencia de comandos y respuestas, en lugar de continuar repitiendo pruebas.

**Estado actual:** 10 presets nuevos PASS-RF, cuatro controles positivos PASS-RF y tres ejecuciones BLOCKED. `wireless_mbus_s_868` sigue pendiente de validación OTA.

Envíame el registro completo hasta `TEST_END` para evaluar el segundo intento.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wireless_mbus_s_868 `
>>     A50B0001 `
>>     campaign-wireless-mbus-s-868-run-02
2026-10-15T23:00:14.994555Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wireless-mbus-s-868-run-02' tx_port='COM33' rx_port='COM88' preset_name='wireless_mbus_s_868' preset={'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840} marker='A5 0B 00 01' tx_power_dbm=0 tx_count=10
2026-10-15T23:00:14.994685Z TX_INIT_BEGIN port='COM33'
2026-10-15T23:00:16.109798Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T23:00:16.109896Z RX_INIT_BEGIN port='COM88'
2026-10-15T23:00:17.221913Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-15T23:00:17.222010Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T23:00:17.257636Z TX_SET_PHY_ACK
2026-10-15T23:00:17.257752Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-15T23:00:17.290875Z RX_SET_PHY_ACK
2026-10-15T23:00:17.290974Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840}
2026-10-15T23:00:17.321948Z TX_CONFIGURE_PROP_ACK
2026-10-15T23:00:17.322044Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868300000, 'mod_type': 1, 'symbol_rate': 32768, 'deviation': 75, 'rx_bw': 82, 'sync_word': 1413283840}
2026-10-15T23:00:17.353590Z RX_CONFIGURE_PROP_ACK
2026-10-15T23:00:17.353686Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-15T23:00:17.383876Z TX_SET_POWER_ACK
2026-10-15T23:00:17.383970Z TX_STATS_BEFORE_BEGIN
2026-10-15T23:00:17.548049Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T23:00:17.548155Z RX_STATS_BEFORE_BEGIN
2026-10-15T23:00:17.712864Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T23:00:17.712980Z RX_START_BEGIN
2026-10-15T23:00:17.742062Z RX_START_ACK
2026-10-15T23:00:17.743706Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-15T23:00:18.043954Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30178520000117715
2026-10-15T23:00:18.044065Z TX_ATTEMPT_BEGIN index=1 payload='A5 0B 00 01'
2026-10-15T23:00:18.077406Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-15T23:00:18.177847Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10024340000018128
2026-10-15T23:00:18.178410Z TX_ATTEMPT_BEGIN index=2 payload='A5 0B 00 01'
2026-10-15T23:00:18.214562Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-15T23:00:18.315441Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10063050000098883
2026-10-15T23:00:18.315953Z TX_ATTEMPT_BEGIN index=3 payload='A5 0B 00 01'
2026-10-15T23:00:18.355353Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-15T23:00:18.455793Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.1003108999975666
2026-10-15T23:00:18.456137Z TX_ATTEMPT_BEGIN index=4 payload='A5 0B 00 01'
2026-10-15T23:00:18.495175Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-15T23:00:18.595489Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10011239999948884
2026-10-15T23:00:18.596045Z TX_ATTEMPT_BEGIN index=5 payload='A5 0B 00 01'
2026-10-15T23:00:18.639296Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-15T23:00:18.740176Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10014719999890076
2026-10-15T23:00:18.740314Z TX_ATTEMPT_BEGIN index=6 payload='A5 0B 00 01'
2026-10-15T23:00:18.773588Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-15T23:00:18.873974Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10023500000170316
2026-10-15T23:00:18.874485Z TX_ATTEMPT_BEGIN index=7 payload='A5 0B 00 01'
2026-10-15T23:00:18.918419Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-15T23:00:19.019280Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10033549999934621
2026-10-15T23:00:19.019793Z TX_ATTEMPT_BEGIN index=8 payload='A5 0B 00 01'
2026-10-15T23:00:19.055862Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-15T23:00:19.156251Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10023419999924954
2026-10-15T23:00:19.156676Z TX_ATTEMPT_BEGIN index=9 payload='A5 0B 00 01'
2026-10-15T23:00:19.200493Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-15T23:00:19.301503Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10064959999726852
2026-10-15T23:00:19.301953Z TX_ATTEMPT_BEGIN index=10 payload='A5 0B 00 01'
2026-10-15T23:00:19.344378Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-15T23:00:19.445173Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10027960000297753
2026-10-15T23:00:19.445676Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-15T23:00:20.446688Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.000998299998173
2026-10-15T23:00:20.447367Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-15T23:00:20.577544Z RX_PACKET ordinal=1 device_timestamp_us=2026140 channel=37 rssi_dbm=115 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:20.700808Z RX_PACKET ordinal=2 device_timestamp_us=2026666 channel=37 rssi_dbm=125 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:20.813533Z RX_PACKET ordinal=3 device_timestamp_us=2027203 channel=37 rssi_dbm=3 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:20.933655Z RX_PACKET ordinal=4 device_timestamp_us=2027750 channel=37 rssi_dbm=86 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:21.051966Z RX_PACKET ordinal=5 device_timestamp_us=2028297 channel=37 rssi_dbm=-88 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:21.168953Z RX_PACKET ordinal=6 device_timestamp_us=2028860 channel=37 rssi_dbm=4 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:21.285767Z RX_PACKET ordinal=7 device_timestamp_us=2029385 channel=37 rssi_dbm=24 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:21.402178Z RX_PACKET ordinal=8 device_timestamp_us=2029952 channel=37 rssi_dbm=-16 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:21.518843Z RX_PACKET ordinal=9 device_timestamp_us=2030487 channel=37 rssi_dbm=48 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:21.636817Z RX_PACKET ordinal=10 device_timestamp_us=2031054 channel=37 rssi_dbm=111 lqi=0 crc_ok=True payload_hex='04 A5 0B 00 01 E1 4A' marker_present=True
2026-10-15T23:00:22.465247Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.017853400000604 packets=10 async_errors=0
2026-10-15T23:00:22.465896Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-15T23:00:22.979292Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5133930000010878 async_errors=0
2026-10-15T23:00:22.979695Z RX_STOP_BEGIN
2026-10-15T23:00:23.013818Z RX_STOP_ACK
2026-10-15T23:00:23.013917Z TX_STATS_AFTER_BEGIN
2026-10-15T23:00:23.189153Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T23:00:23.189370Z RX_STATS_AFTER_BEGIN
2026-10-15T23:00:23.361201Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-15T23:00:23.361298Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-15T23:00:23.362006Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-15T23:00:23.362065Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-15T23:00:23.362631Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-15T23:00:23.362704Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-15T23:00:23.362761Z TEST_CLASSIFICATION label='campaign-wireless-mbus-s-868-run-02' preset='wireless_mbus_s_868' classification='PASS-RF'
2026-10-15T23:00:23.362816Z TEST_END exit_code=0

---

## Paso 44 — `wireless_mbus_s_868`: PASS-RF (10/10) ✅

**El segundo intento OTA de `wireless_mbus_s_868` terminó satisfactoriamente.** Los dos CatSniffer completaron la configuración solicitada y el receptor entregó diez paquetes con el marcador esperado y CRC reportado como válido.

El timeout del primer intento permanece registrado como un incidente independiente.

### 1. Resultado experimental

| Parámetro                | Resultado                             |
| ------------------------ | ------------------------------------- |
| Ejecución                | `campaign-wireless-mbus-s-868-run-02` |
| Fecha                    | 2026-10-15, 23:00 UTC                 |
| TX                       | CatSniffer #2 — `COM33`               |
| RX                       | CatSniffer #3 — `COM88`               |
| Preset                   | `wireless_mbus_s_868`                 |
| Frecuencia solicitada    | **868.3 MHz**                         |
| Modulación solicitada    | GFSK (`mod_type=1`)                   |
| Symbol rate solicitado   | 32 768 símbolos/s                     |
| `deviation`              | 75                                    |
| `rx_bw`                  | 82                                    |
| Sync word                | `0x543C0000`                          |
| Potencia solicitada      | 0 dBm                                 |
| Marcador                 | `A5 0B 00 01`                         |
| TX ACK                   | **10/10**                             |
| RX CRC-good con marcador | **10/10**                             |
| Errores asíncronos       | 0                                     |
| Excepciones fatales      | 0                                     |
| **Clasificación**        | **PASS-RF**                           |

**Fuente:** registro de ejecución proporcionado, desde `TEST_BEGIN` hasta `TEST_END`.

### 2. Evidencia de recepción

Los diez paquetes presentan:

```text
payload_hex='04 A5 0B 00 01 E1 4A'
crc_ok=True
marker_present=True
```

Los contadores del receptor confirman:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se completaron correctamente `RX_STOP_ACK` y las desconexiones de ambos dispositivos.

**Conclusión experimental:** se observó comunicación OTA satisfactoria entre los dos CatSniffer utilizando la configuración solicitada por `wireless_mbus_s_868`.

Esto **no demuestra interoperabilidad con Wireless M-Bus Mode S**. El ensayo utiliza el mecanismo de paquetes de FeralRF y no verifica independientemente una trama Wireless M-Bus conforme al estándar.

### 3. Relación con el timeout anterior

Tenemos dos resultados distintos para este preset:

| Ejecución | Resultado   | Motivo                                                 |
| --------- | ----------- | ------------------------------------------------------ |
| `run-01`  | BLOCKED     | Timeout en `rx.get_stats()`, antes de transmitir       |
| `run-02`  | **PASS-RF** | Diez transmisiones aceptadas y diez paquetes recibidos |

Entre ambos intentos se realizó una consulta aislada de `get_stats()` que respondió correctamente.

Esto indica que el timeout no fue persistente en las operaciones posteriores. **No identifica su causa raíz ni garantiza que no vuelva a ocurrir.**

---

## 4. Estado actualizado de la campaña

| Indicador                          |   Resultado |
| ---------------------------------- | ----------: |
| Presets nuevos evaluados con éxito |      **11** |
| Presets nuevos PASS-RF             |   **11/11** |
| Controles positivos PASS-RF        |           4 |
| Ejecuciones OTA satisfactorias     |      **16** |
| Paquetes CRC-good con marcador     | **160/160** |
| Ejecuciones BLOCKED                |       **3** |

Los tres incidentes bloqueados permanecen documentados:

* Timeout en `tx.get_stats()` durante un control positivo.
* Puerto `COM33` no disponible después de una desconexión.
* Timeout en `rx.get_stats()` durante `wireless_mbus_s_868`, primer intento.

Los metadatos `channel=37`, `lqi=0` y los valores de RSSI inconsistentes siguen siendo cuestiones abiertas.

---

## Paso 45 — Inspección de `wireless_mbus_t_868`

El siguiente preset previsto es `wireless_mbus_t_868`.

Antes de transmitir, verificaremos sus parámetros para identificar las diferencias respecto de `wireless_mbus_s_868`, especialmente frecuencia, symbol rate, modulación y sync word.

### Ejecuta únicamente:

```powershell
$PRESET = 'wireless_mbus_t_868'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

Este comando no transmite RF ni modifica la configuración de los dispositivos.

**Estado actual:** 11 presets nuevos PASS-RF, cuatro controles positivos satisfactorios y 160 paquetes recibidos con CRC válido reportado por el firmware.

Envíame la salida de `PROP_PRESETS` para revisar la siguiente configuración antes de ejecutar su prueba OTA.

---

Para retomar la campaña OTA de FeralRF después de cerrar PowerShell, necesitamos **restaurar el entorno de Python y seleccionar la banda Sub-GHz en ambos CatSniffer**.

No es necesario repetir las consultas de información del firmware ni las pruebas de conectividad.

## 1. Abrir PowerShell y entrar al proyecto

```powershell
cd 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF'
```

## 2. Configurar `PYTHONPATH`

La biblioteca `feralrf` está dentro del directorio `python` del repositorio.

```powershell
$env:PYTHONPATH = (Resolve-Path '.\python').Path
```

Esta variable debe configurarse nuevamente porque la sesión anterior de PowerShell se cerró.

## 3. Seleccionar la banda Sub-GHz en ambos CatSniffer

Según la interfaz Shell utilizada en nuestras pruebas, el comando correspondiente es **`band2`**.

Los puertos utilizados anteriormente fueron:

| Dispositivo   | Función | Shell   | Cat-Bridge |
| ------------- | ------- | ------- | ---------- |
| CatSniffer #2 | TX      | `COM35` | `COM33`    |
| CatSniffer #3 | RX      | `COM87` | `COM88`    |

Ejecuta el siguiente bloque:

```powershell
@'
import serial
import time

shell_ports = {
    "CatSniffer #2 (TX)": "COM35",
    "CatSniffer #3 (RX)": "COM87",
}

for name, port in shell_ports.items():
    print(f"\n{name} - {port}")

    with serial.Serial(port, 115200, timeout=1) as shell:
        time.sleep(0.3)

        shell.reset_input_buffer()
        shell.write(b"band2\r\n")
        shell.flush()

        time.sleep(0.5)

        response = shell.read_all().decode(errors="replace")
        print(response)

print("\nComandos band2 enviados.")
'@ | python -u -
```

**Importante:** el script envía `band2` a ambos dispositivos, pero el mensaje final solamente confirma que terminó de enviar los comandos. Para considerar confirmada la selección de banda, debemos observar una respuesta adecuada del Shell.

Además, estos puertos corresponden a la asignación anterior. Si Windows los cambió después de reconectar los dispositivos, será necesario actualizar sus valores.

No es necesario ejecutar `boot`, `exit` ni reiniciar los CC1352 únicamente por haber cerrado PowerShell.

## 4. Retomar la inspección del siguiente preset

Una vez seleccionada la banda Sub-GHz, ejecuta:

```powershell
$PRESET = 'wireless_mbus_t_868'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

**Nota:** esta consulta solo lee los parámetros de `PROP_PRESETS`; técnicamente no necesita que los CatSniffer estén conectados ni configurados en Sub-GHz. La selección de banda es necesaria para preparar las posteriores pruebas OTA.

---

### Estado de la campaña al reanudar

* **11 presets nuevos:** PASS-RF.
* **4 controles positivos:** PASS-RF.
* **160/160 paquetes:** recibidos con CRC válido reportado por el firmware.
* **3 ejecuciones:** BLOCKED por problemas de comunicación o disponibilidad de puertos.
* **Siguiente preset:** `wireless_mbus_t_868`.

Cuando tengas la salida del último comando, continuaremos con la inspección de parámetros antes de transmitir.

---

## Paso 45 — Inspección de `wireless_mbus_t_868`: completada ✅

La consulta confirma que el preset **`wireless_mbus_t_868`** está disponible en `PROP_PRESETS`.

Su configuración solicita una frecuencia de **868.95 MHz**, modulación GFSK y una velocidad de **100 000 símbolos/s**.

### 1. Parámetros identificados

| Parámetro      | Valor                  |
| -------------- | ---------------------- |
| Preset         | `wireless_mbus_t_868`  |
| `frequency_hz` | **868 950 000 Hz**     |
| `mod_type`     | 1 — GFSK               |
| `symbol_rate`  | **100 000 símbolos/s** |
| `deviation`    | 200                    |
| `rx_bw`        | 87                     |
| `sync_word`    | `0x543C0000`           |

**Fuente:** salida de la consulta directa a `feralrf.PROP_PRESETS`.

Los valores `deviation` y `rx_bw` se mantienen como parámetros de la API; todavía no hemos confirmado sus unidades en el firmware.

### 2. Comparación con `wireless_mbus_s_868`

| Parámetro     | `wireless_mbus_s_868` | `wireless_mbus_t_868` |
| ------------- | --------------------: | --------------------: |
| Frecuencia    |             868.3 MHz |        **868.95 MHz** |
| Modulación    |                  GFSK |                  GFSK |
| Symbol rate   |                32 768 |           **100 000** |
| `deviation`   |                    75 |               **200** |
| `rx_bw`       |                    82 |                **87** |
| Sync word     |          `0x543C0000` |          `0x543C0000` |
| Resultado OTA |     **PASS-RF 10/10** |             Pendiente |

La diferencia principal es que `wireless_mbus_t_868` solicita una velocidad de símbolos aproximadamente **3.05 veces mayor**.

Ambos presets utilizan el mismo sync word, pero eso no demuestra que implementen correctamente los modos S y T del estándar Wireless M-Bus.

### 3. Decisión de validación

Podemos continuar con la prueba OTA, manteniendo las condiciones de la campaña.

**Condición previa:** dado que abriste una nueva sesión de PowerShell, debemos haber enviado correctamente `band2` a ambos CatSniffer. La consulta de `PROP_PRESETS` no confirma por sí misma la selección de la ruta Sub-GHz.

Si ya ejecutaste el bloque `band2` y ambos Shell respondieron correctamente, podemos avanzar.

---

## Paso 46 — Prueba OTA `wireless_mbus_t_868`

| Parámetro             | Valor                                 |
| --------------------- | ------------------------------------- |
| TX                    | CatSniffer #2 — `COM33`               |
| RX                    | CatSniffer #3 — `COM88`               |
| Preset                | `wireless_mbus_t_868`                 |
| Frecuencia solicitada | 868.95 MHz                            |
| Potencia solicitada   | 0 dBm                                 |
| Marcador              | `A5 0C 00 01`                         |
| Transmisiones         | 10                                    |
| Identificador         | `campaign-wireless-mbus-t-868-run-01` |

### Ejecuta únicamente este comando

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wireless_mbus_t_868 `
    A50C0001 `
    campaign-wireless-mbus-t-868-run-01
```

### Criterios de aceptación

Para clasificar la prueba como **PASS-RF** necesitamos:

* 10/10 comandos TX aceptados.
* 10/10 paquetes recibidos con `crc_ok=True` y `marker_present=True`.
* Incremento de `rx_ok` de 0 a 10.
* Ausencia de errores asíncronos y excepciones fatales.
* Finalización correcta de RX y desconexión de ambos dispositivos.

**Importante:** el resultado permitirá evaluar la comunicación OTA entre los CatSniffer utilizando este preset, pero no certificará interoperabilidad con Wireless M-Bus Mode T.

**Estado acumulado:** 11 presets nuevos PASS-RF, cuatro controles positivos satisfactorios, 160/160 paquetes recibidos y tres ejecuciones BLOCKED.

Envíame el registro completo hasta `TEST_END` para evaluar el resultado.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wireless_mbus_t_868 `
>>     A50C0001 `
>>     campaign-wireless-mbus-t-868-run-01
2026-10-16T14:52:01.172945Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wireless-mbus-t-868-run-01' tx_port='COM33' rx_port='COM88' preset_name='wireless_mbus_t_868' preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 200, 'rx_bw': 87, 'sync_word': 1413283840} marker='A5 0C 00 01' tx_power_dbm=0 tx_count=10
2026-10-16T14:52:01.173126Z TX_INIT_BEGIN port='COM33'
2026-10-16T14:52:02.297006Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:52:02.297121Z RX_INIT_BEGIN port='COM88'
2026-10-16T14:52:03.421951Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:52:03.422061Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:52:03.455602Z TX_SET_PHY_ACK
2026-10-16T14:52:03.455696Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:52:03.491210Z RX_SET_PHY_ACK
2026-10-16T14:52:03.491315Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 200, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:52:03.524283Z TX_CONFIGURE_PROP_ACK
2026-10-16T14:52:03.524374Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 200, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:52:03.554769Z RX_CONFIGURE_PROP_ACK
2026-10-16T14:52:03.554868Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-16T14:52:03.589823Z TX_SET_POWER_ACK
2026-10-16T14:52:03.589913Z TX_STATS_BEFORE_BEGIN
2026-10-16T14:52:03.753508Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:52:03.753604Z RX_STATS_BEFORE_BEGIN
2026-10-16T14:52:03.936560Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:52:03.936664Z RX_START_BEGIN
2026-10-16T14:52:03.965851Z RX_START_ACK
2026-10-16T14:52:03.966320Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-16T14:52:04.266902Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.300899500012747
2026-10-16T14:52:04.267473Z TX_ATTEMPT_BEGIN index=1 payload='A5 0C 00 01'
2026-10-16T14:52:04.308241Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-16T14:52:04.409130Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10050300000875723
2026-10-16T14:52:04.409706Z TX_ATTEMPT_BEGIN index=2 payload='A5 0C 00 01'
2026-10-16T14:52:04.451373Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-16T14:52:04.552230Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.1005250000016531
2026-10-16T14:52:04.552517Z TX_ATTEMPT_BEGIN index=3 payload='A5 0C 00 01'
2026-10-16T14:52:04.586976Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-16T14:52:04.687235Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10015460000431631
2026-10-16T14:52:04.687341Z TX_ATTEMPT_BEGIN index=4 payload='A5 0C 00 01'
2026-10-16T14:52:04.718273Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-16T14:52:04.818585Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10014620001311414
2026-10-16T14:52:04.818844Z TX_ATTEMPT_BEGIN index=5 payload='A5 0C 00 01'
2026-10-16T14:52:04.851637Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-16T14:52:04.952030Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10022799999569543
2026-10-16T14:52:04.952518Z TX_ATTEMPT_BEGIN index=6 payload='A5 0C 00 01'
2026-10-16T14:52:04.984152Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-16T14:52:05.084768Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10045880000689067
2026-10-16T14:52:05.085386Z TX_ATTEMPT_BEGIN index=7 payload='A5 0C 00 01'
2026-10-16T14:52:05.127368Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-16T14:52:05.228113Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10018660000059754
2026-10-16T14:52:05.228558Z TX_ATTEMPT_BEGIN index=8 payload='A5 0C 00 01'
2026-10-16T14:52:05.263298Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-16T14:52:05.364024Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10060879999946337
2026-10-16T14:52:05.364167Z TX_ATTEMPT_BEGIN index=9 payload='A5 0C 00 01'
2026-10-16T14:52:05.398490Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-16T14:52:05.499010Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10038779999013059
2026-10-16T14:52:05.499130Z TX_ATTEMPT_BEGIN index=10 payload='A5 0C 00 01'
2026-10-16T14:52:05.530433Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-16T14:52:05.631162Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10058030000072904
2026-10-16T14:52:05.634647Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-16T14:52:06.635375Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0007482999935746
2026-10-16T14:52:06.635828Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-16T14:52:06.756399Z RX_PACKET ordinal=1 device_timestamp_us=1999532 channel=37 rssi_dbm=-8 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:06.873554Z RX_PACKET ordinal=2 device_timestamp_us=2000087 channel=37 rssi_dbm=-22 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.001216Z RX_PACKET ordinal=3 device_timestamp_us=2000644 channel=37 rssi_dbm=-71 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.117547Z RX_PACKET ordinal=4 device_timestamp_us=2001170 channel=37 rssi_dbm=-44 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.234595Z RX_PACKET ordinal=5 device_timestamp_us=2001684 channel=37 rssi_dbm=-105 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.350326Z RX_PACKET ordinal=6 device_timestamp_us=2002207 channel=37 rssi_dbm=-72 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.477808Z RX_PACKET ordinal=7 device_timestamp_us=2002726 channel=37 rssi_dbm=-53 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.587994Z RX_PACKET ordinal=8 device_timestamp_us=2003285 channel=37 rssi_dbm=-105 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.708866Z RX_PACKET ordinal=9 device_timestamp_us=2003815 channel=37 rssi_dbm=27 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:07.825067Z RX_PACKET ordinal=10 device_timestamp_us=2004342 channel=37 rssi_dbm=64 lqi=0 crc_ok=True payload_hex='04 A5 0C 00 01 61 25' marker_present=True
2026-10-16T14:52:08.653577Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0178306000016164 packets=10 async_errors=0
2026-10-16T14:52:08.653732Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-16T14:52:09.172195Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5184386000037193 async_errors=0
2026-10-16T14:52:09.172654Z RX_STOP_BEGIN
2026-10-16T14:52:09.210059Z RX_STOP_ACK
2026-10-16T14:52:09.210153Z TX_STATS_AFTER_BEGIN
2026-10-16T14:52:09.381226Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:52:09.381362Z RX_STATS_AFTER_BEGIN
2026-10-16T14:52:09.560711Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:52:09.560810Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-16T14:52:09.561624Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-16T14:52:09.561686Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-16T14:52:09.562375Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-16T14:52:09.562448Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-16T14:52:09.562504Z TEST_CLASSIFICATION label='campaign-wireless-mbus-t-868-run-01' preset='wireless_mbus_t_868' classification='PASS-RF'
2026-10-16T14:52:09.562594Z TEST_END exit_code=0

---

## Paso 46 — `wireless_mbus_t_868`: PASS-RF ✅

**La prueba OTA de `wireless_mbus_t_868` terminó satisfactoriamente: 10/10 transmisiones aceptadas y 10/10 paquetes recibidos con el marcador correcto y CRC válido según el firmware.**

Es especialmente relevante que el resultado se obtuvo después de abrir una nueva sesión de PowerShell y reanudar la campaña.

### 1. Resultado experimental

| Parámetro                | Resultado                             |
| ------------------------ | ------------------------------------- |
| Ejecución                | `campaign-wireless-mbus-t-868-run-01` |
| Fecha                    | 2026-10-16, 14:52 UTC                 |
| TX                       | CatSniffer #2 — `COM33`               |
| RX                       | CatSniffer #3 — `COM88`               |
| Preset                   | `wireless_mbus_t_868`                 |
| Frecuencia solicitada    | **868.95 MHz**                        |
| Modulación solicitada    | GFSK (`mod_type=1`)                   |
| Symbol rate solicitado   | 100 000 símbolos/s                    |
| `deviation`              | 200                                   |
| `rx_bw`                  | 87                                    |
| Sync word                | `0x543C0000`                          |
| Potencia solicitada      | 0 dBm                                 |
| Marcador                 | `A5 0C 00 01`                         |
| TX ACK                   | **10/10**                             |
| RX CRC-good con marcador | **10/10**                             |
| Errores asíncronos       | 0                                     |
| Excepciones fatales      | 0                                     |
| **Clasificación**        | **PASS-RF**                           |

**Fuente:** registro proporcionado de `manual_preset_runner.py`, desde `TEST_BEGIN` hasta `TEST_END`.

### 2. Evidencia de recepción

Los diez paquetes contienen:

```text
payload_hex='04 A5 0C 00 01 61 25'
crc_ok=True
marker_present=True
```

Las estadísticas del receptor muestran:

| Contador      | Antes | Después |
| ------------- | ----: | ------: |
| `rx_ok`       |     0 |      10 |
| `rx_crc_err`  |     0 |       0 |
| `rx_drop`     |     0 |       0 |
| `rx_overflow` |     0 |       0 |

También se completaron correctamente `RX_STOP_ACK` y las desconexiones de ambos dispositivos.

**Conclusión experimental:** los dos CatSniffer intercambiaron correctamente paquetes mediante la configuración solicitada por `wireless_mbus_t_868`.

Esto no certifica interoperabilidad con Wireless M-Bus Mode T ni constituye una medición independiente de frecuencia, modulación o potencia radiada.

### 3. Anomalías que permanecen abiertas

Los paquetes siguen reportando `channel=37`, aunque se configuró `channel=0`.

Asimismo, `lqi=0` aparece en todas las recepciones y existen valores de RSSI positivos, como `27` y `64 dBm`, que no son coherentes con las condiciones esperadas.

Por tanto, **no utilizaremos esos metadatos para evaluar la calidad del enlace** hasta revisar su implementación en el firmware.

---

## 4. Estado actualizado de la campaña

| Indicador                      |   Resultado |
| ------------------------------ | ----------: |
| Presets nuevos PASS-RF         |      **12** |
| Controles positivos PASS-RF    |       **4** |
| Ejecuciones OTA satisfactorias |      **17** |
| Paquetes CRC-good con marcador | **170/170** |
| Ejecuciones BLOCKED            |       **3** |
| Ejecuciones FAIL-RF            |       **0** |

Dentro del grupo Wireless M-Bus:

| Preset                | Resultado         |
| --------------------- | ----------------- |
| `wireless_mbus_s_868` | PASS-RF 10/10     |
| `wireless_mbus_t_868` | **PASS-RF 10/10** |
| `wireless_mbus_c_868` | Pendiente         |

Los dos primeros presets del grupo funcionan para el intercambio de paquetes FeralRF entre nuestros dispositivos.

---

## Paso 47 — Inspección de `wireless_mbus_c_868`

El siguiente preset es `wireless_mbus_c_868`.

Antes de transmitir, verificaremos sus parámetros para identificar las diferencias respecto de los modos S y T.

### Ejecuta únicamente:

```powershell
$PRESET = 'wireless_mbus_c_868'

python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
```

Esta consulta no transmite RF ni modifica la configuración de los CatSniffer.

**Estado actual:** 12 presets nuevos PASS-RF, cuatro controles positivos satisfactorios y 170/170 paquetes recibidos con CRC válido reportado por el firmware.

Después de inspeccionar `wireless_mbus_c_868`, prepararemos su prueba OTA y, si resulta satisfactoria, ejecutaremos el control positivo correspondiente al tercer preset nuevo desde el último control.

---

**PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> $PRESET = 'wireless_mbus_c_868'
**
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF>
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -c "import sys; from feralrf import PROP_PRESETS; name=sys.argv[1]; print(name, PROP_PRESETS[name])" $PRESET
wireless_mbus_c_868 {'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840}

---

## Paso 47 — Inspección de `wireless_mbus_c_868`: completada ✅

La consulta confirma que **`wireless_mbus_c_868` está definido en `PROP_PRESETS`** y utiliza una configuración muy similar a `wireless_mbus_t_868`.

La diferencia identificada es el valor de `deviation`: **180 frente a 200**.

### 1. Parámetros identificados

| Parámetro      | Valor                  |
| -------------- | ---------------------- |
| Preset         | `wireless_mbus_c_868`  |
| `frequency_hz` | **868 950 000 Hz**     |
| `mod_type`     | 1 — GFSK               |
| `symbol_rate`  | **100 000 símbolos/s** |
| `deviation`    | 180                    |
| `rx_bw`        | 87                     |
| `sync_word`    | `0x543C0000`           |

**Fuente:** salida de la consulta directa a `feralrf.PROP_PRESETS`.

Seguimos sin asignar unidades físicas a `deviation` y `rx_bw`, porque su interpretación todavía requiere comprobar el firmware.

### 2. Comparación de los tres presets Wireless M-Bus

| Parámetro             |       Mode S |       Mode T |       Mode C |
| --------------------- | -----------: | -----------: | -----------: |
| Frecuencia solicitada |    868.3 MHz |   868.95 MHz |   868.95 MHz |
| `mod_type`            |            1 |            1 |            1 |
| `symbol_rate`         |       32 768 |      100 000 |      100 000 |
| `deviation`           |           75 |          200 |      **180** |
| `rx_bw`               |           82 |           87 |           87 |
| Sync word             | `0x543C0000` | `0x543C0000` | `0x543C0000` |
| Resultado OTA         |      PASS-RF |      PASS-RF |    Pendiente |

**Observación importante:** los nombres Mode S, Mode T y Mode C son identificadores de presets en FeralRF. No debemos interpretarlos como evidencia de implementación completa de esos modos del estándar Wireless M-Bus.

La próxima prueba verificará únicamente la comunicación entre los dos CatSniffer utilizando los parámetros solicitados.

---

## Paso 48 — Prueba OTA `wireless_mbus_c_868`

El preset anterior, `wireless_mbus_t_868`, terminó con **PASS-RF 10/10**.

Podemos continuar con la siguiente configuración sin cambiar las condiciones del ensayo.

### Condiciones de ejecución

| Parámetro              | Valor                                 |
| ---------------------- | ------------------------------------- |
| TX                     | CatSniffer #2 — `COM33`               |
| RX                     | CatSniffer #3 — `COM88`               |
| Preset                 | `wireless_mbus_c_868`                 |
| Frecuencia solicitada  | **868.95 MHz**                        |
| Modulación solicitada  | GFSK                                  |
| Symbol rate solicitado | 100 000 símbolos/s                    |
| Potencia solicitada    | 0 dBm                                 |
| Marcador               | `A5 0D 00 01`                         |
| Transmisiones          | 10                                    |
| Identificador          | `campaign-wireless-mbus-c-868-run-01` |

### Ejecuta únicamente este comando

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wireless_mbus_c_868 `
    A50D0001 `
    campaign-wireless-mbus-c-868-run-01
```

### Criterios de aceptación

La ejecución será **PASS-RF** si cumple:

* 10/10 comandos TX aceptados.
* 10/10 paquetes recibidos con `crc_ok=True` y `marker_present=True`.
* Incremento de `rx_ok` de 0 a 10.
* Ausencia de errores asíncronos y excepciones fatales.
* Finalización correcta de RX y desconexión de ambos dispositivos.

### ¿Qué sigue después?

Si obtenemos PASS-RF, habremos completado los tres presets Wireless M-Bus previstos en esta etapa.

**Antes de continuar con otra familia de configuraciones, ejecutaremos el control positivo `wisun_915_fsk_50k`**, siguiendo la regla de un control cada tres presets nuevos.

**Estado actual:** 12 presets nuevos PASS-RF, cuatro controles positivos satisfactorios, 170/170 paquetes recibidos y tres ejecuciones BLOCKED.

Envíame el registro completo hasta `TEST_END` para evaluar `wireless_mbus_c_868`.

---

He realizado 3 pruebas, la primera parece haber funcionado, y las 2 fallaron en cierto punto intencionalmente, ésto ya que yo en cada una presione el botón físico de RESET de cada CatSniffer (Una diferente por pueba) para verificar que realmente existe comunicación, es decir, hice una contraprueba para verificar que debía fallar en algún punto el procedimiento tras interrumpir la comunicación con el botón RESET: PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wireless_mbus_c_868 `
>>     A50D0001 `
>>     campaign-wireless-mbus-c-868-run-01
2026-10-16T14:54:19.615546Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wireless-mbus-c-868-run-01' tx_port='COM33' rx_port='COM88' preset_name='wireless_mbus_c_868' preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840} marker='A5 0D 00 01' tx_power_dbm=0 tx_count=10
2026-10-16T14:54:19.615698Z TX_INIT_BEGIN port='COM33'
2026-10-16T14:54:20.750955Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:54:20.751052Z RX_INIT_BEGIN port='COM88'
2026-10-16T14:54:21.878625Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:54:21.878720Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:54:21.915074Z TX_SET_PHY_ACK
2026-10-16T14:54:21.915167Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:54:21.951789Z RX_SET_PHY_ACK
2026-10-16T14:54:21.951882Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:54:21.986215Z TX_CONFIGURE_PROP_ACK
2026-10-16T14:54:21.986301Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:54:22.021023Z RX_CONFIGURE_PROP_ACK
2026-10-16T14:54:22.021169Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-16T14:54:22.051335Z TX_SET_POWER_ACK
2026-10-16T14:54:22.051543Z TX_STATS_BEFORE_BEGIN
2026-10-16T14:54:22.229263Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:54:22.229360Z RX_STATS_BEFORE_BEGIN
2026-10-16T14:54:22.408358Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:54:22.408454Z RX_START_BEGIN
2026-10-16T14:54:22.441183Z RX_START_ACK
2026-10-16T14:54:22.441270Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-16T14:54:22.742036Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.3007394000014756
2026-10-16T14:54:22.742530Z TX_ATTEMPT_BEGIN index=1 payload='A5 0D 00 01'
2026-10-16T14:54:22.784473Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-16T14:54:22.885648Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10050919999775942
2026-10-16T14:54:22.886106Z TX_ATTEMPT_BEGIN index=2 payload='A5 0D 00 01'
2026-10-16T14:54:22.927462Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-16T14:54:23.028525Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10053370000969153
2026-10-16T14:54:23.029082Z TX_ATTEMPT_BEGIN index=3 payload='A5 0D 00 01'
2026-10-16T14:54:23.063172Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-16T14:54:23.164154Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10083709999162238
2026-10-16T14:54:23.164664Z TX_ATTEMPT_BEGIN index=4 payload='A5 0D 00 01'
2026-10-16T14:54:23.206470Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-16T14:54:23.307044Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10034740000264719
2026-10-16T14:54:23.307181Z TX_ATTEMPT_BEGIN index=5 payload='A5 0D 00 01'
2026-10-16T14:54:23.338809Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-16T14:54:23.439189Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10022910000407137
2026-10-16T14:54:23.439691Z TX_ATTEMPT_BEGIN index=6 payload='A5 0D 00 01'
2026-10-16T14:54:23.481433Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-16T14:54:23.582227Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10032769999816082
2026-10-16T14:54:23.582333Z TX_ATTEMPT_BEGIN index=7 payload='A5 0D 00 01'
2026-10-16T14:54:23.612626Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-16T14:54:23.713841Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10072960000252351
2026-10-16T14:54:23.714341Z TX_ATTEMPT_BEGIN index=8 payload='A5 0D 00 01'
2026-10-16T14:54:23.756400Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-16T14:54:23.857141Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.1002435999980662
2026-10-16T14:54:23.857643Z TX_ATTEMPT_BEGIN index=9 payload='A5 0D 00 01'
2026-10-16T14:54:23.899322Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-16T14:54:23.999784Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.10020539999823086
2026-10-16T14:54:24.000199Z TX_ATTEMPT_BEGIN index=10 payload='A5 0D 00 01'
2026-10-16T14:54:24.039152Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-16T14:54:24.139527Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10022929999104235
2026-10-16T14:54:24.140019Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-16T14:54:25.140570Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.000567900002352
2026-10-16T14:54:25.141077Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-16T14:54:25.276839Z RX_PACKET ordinal=1 device_timestamp_us=2540470 channel=37 rssi_dbm=-103 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:25.395210Z RX_PACKET ordinal=2 device_timestamp_us=2541031 channel=37 rssi_dbm=0 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:25.520405Z RX_PACKET ordinal=3 device_timestamp_us=2541589 channel=37 rssi_dbm=106 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:25.639315Z RX_PACKET ordinal=4 device_timestamp_us=2542119 channel=37 rssi_dbm=-54 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:25.764400Z RX_PACKET ordinal=5 device_timestamp_us=2542675 channel=37 rssi_dbm=-21 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:25.879401Z RX_PACKET ordinal=6 device_timestamp_us=2543193 channel=37 rssi_dbm=102 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:25.998101Z RX_PACKET ordinal=7 device_timestamp_us=2543750 channel=37 rssi_dbm=97 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:26.104087Z RX_PACKET ordinal=8 device_timestamp_us=2544266 channel=37 rssi_dbm=-12 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:26.218770Z RX_PACKET ordinal=9 device_timestamp_us=2544826 channel=37 rssi_dbm=-71 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:26.333748Z RX_PACKET ordinal=10 device_timestamp_us=2545382 channel=37 rssi_dbm=-90 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:54:27.150904Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0098915999988094 packets=10 async_errors=0
2026-10-16T14:54:27.151211Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-16T14:54:27.666981Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5157611000031466 async_errors=0
2026-10-16T14:54:27.667480Z RX_STOP_BEGIN
2026-10-16T14:54:27.709434Z RX_STOP_ACK
2026-10-16T14:54:27.709902Z TX_STATS_AFTER_BEGIN
2026-10-16T14:54:27.886111Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:54:27.886321Z RX_STATS_AFTER_BEGIN
2026-10-16T14:54:28.066184Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:54:28.066282Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-16T14:54:28.067087Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-16T14:54:28.067151Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-16T14:54:28.067806Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-16T14:54:28.067879Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-16T14:54:28.067934Z TEST_CLASSIFICATION label='campaign-wireless-mbus-c-868-run-01' preset='wireless_mbus_c_868' classification='PASS-RF'
2026-10-16T14:54:28.068021Z TEST_END exit_code=0
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wireless_mbus_c_868 `
>>     A50D0001 `
>>     campaign-wireless-mbus-c-868-run-01
2026-10-16T14:54:52.452786Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wireless-mbus-c-868-run-01' tx_port='COM33' rx_port='COM88' preset_name='wireless_mbus_c_868' preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840} marker='A5 0D 00 01' tx_power_dbm=0 tx_count=10
2026-10-16T14:54:52.452970Z TX_INIT_BEGIN port='COM33'
2026-10-16T14:54:53.585485Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:54:53.585582Z RX_INIT_BEGIN port='COM88'
2026-10-16T14:54:54.709977Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:54:54.710073Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:54:54.744617Z TX_SET_PHY_ACK
2026-10-16T14:54:54.744722Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:54:54.779996Z RX_SET_PHY_ACK
2026-10-16T14:54:54.780087Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:54:54.808707Z TX_CONFIGURE_PROP_ACK
2026-10-16T14:54:54.808915Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:54:54.842237Z RX_CONFIGURE_PROP_ACK
2026-10-16T14:54:54.842444Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-16T14:54:54.876590Z TX_SET_POWER_ACK
2026-10-16T14:54:54.876866Z TX_STATS_BEFORE_BEGIN
2026-10-16T14:54:55.039920Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:54:55.040018Z RX_STATS_BEFORE_BEGIN
2026-10-16T14:54:55.204569Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:54:55.204693Z RX_START_BEGIN
2026-10-16T14:54:55.236779Z RX_START_ACK
2026-10-16T14:54:55.236894Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-16T14:54:55.537487Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30059920001076534
2026-10-16T14:54:55.537681Z TX_ATTEMPT_BEGIN index=1 payload='A5 0D 00 01'
2026-10-16T14:54:55.567877Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-16T14:54:55.668700Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10055669999564998
2026-10-16T14:54:55.668863Z TX_ATTEMPT_BEGIN index=2 payload='A5 0D 00 01'
2026-10-16T14:54:55.701402Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-16T14:54:55.802211Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10065680000116117
2026-10-16T14:54:55.802710Z TX_ATTEMPT_BEGIN index=3 payload='A5 0D 00 01'
2026-10-16T14:54:55.844858Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-16T14:54:55.945991Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10066470000310801
2026-10-16T14:54:55.946184Z TX_ATTEMPT_BEGIN index=4 payload='A5 0D 00 01'
2026-10-16T14:54:55.981389Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-16T14:54:56.081741Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10021699999924749
2026-10-16T14:54:56.081879Z TX_ATTEMPT_BEGIN index=5 payload='A5 0D 00 01'
2026-10-16T14:54:56.113587Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-16T14:54:56.213959Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10016519999771845
2026-10-16T14:54:56.214545Z TX_ATTEMPT_BEGIN index=6 payload='A5 0D 00 01'
2026-10-16T14:54:56.256564Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-16T14:54:56.357295Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10025899999891408
2026-10-16T14:54:56.357444Z TX_ATTEMPT_BEGIN index=7 payload='A5 0D 00 01'
2026-10-16T14:54:56.386131Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-16T14:54:56.486789Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10043459999724291
2026-10-16T14:54:56.487003Z TX_ATTEMPT_BEGIN index=8 payload='A5 0D 00 01'
2026-10-16T14:54:56.518565Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-16T14:54:56.619337Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10048889998870436
2026-10-16T14:54:56.619449Z TX_ATTEMPT_BEGIN index=9 payload='A5 0D 00 01'
2026-10-16T14:54:56.649860Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-16T14:54:56.750326Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.1002714000060223
2026-10-16T14:54:56.751133Z TX_ATTEMPT_BEGIN index=10 payload='A5 0D 00 01'
2026-10-16T14:54:56.786185Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-16T14:54:56.886543Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10022749999188818
2026-10-16T14:54:56.886948Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-16T14:54:57.887434Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0004956000047969
2026-10-16T14:54:57.887928Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-16T14:54:57.891775Z TEST_FATAL_EXCEPTION exception_type='SerialException' exception='SerialException("Cannot configure port, something went wrong. Original message: PermissionError(13, \'The device does not recognize the command.\', None, 22)")'
Traceback (most recent call last):
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\experimental_artifacts\manual_preset_runner.py", line 153, in main
    for item in rx.read_packets(timeout=2.0):
                ~~~~~~~~~~~~~~~^^^^^^^^^^^^^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 662, in read_packets
    cmd_id, seq, payload = self._read_response(read_timeout)
                           ~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 317, in _read_response
    self._serial.timeout = min(remaining, 0.1)
    ^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\Support\AppData\Local\Programs\Python\Python314\Lib\site-packages\serial\serialutil.py", line 372, in timeout
    self._reconfigure_port()
    ~~~~~~~~~~~~~~~~~~~~~~^^
  File "C:\Users\Support\AppData\Local\Programs\Python\Python314\Lib\site-packages\serial\serialwin32.py", line 222, in _reconfigure_port
    raise SerialException(
        'Cannot configure port, something went wrong. '
        'Original message: {!r}'.format(ctypes.WinError()))
serial.serialutil.SerialException: Cannot configure port, something went wrong. Original message: PermissionError(13, 'The device does not recognize the command.', None, 22)
2026-10-16T14:54:57.927108Z RX_STOP_CLEANUP_BEGIN
2026-10-16T14:54:57.927353Z RX_STOP_CLEANUP_EXCEPTION exception_type='SerialException' exception='SerialException("WriteFile failed (PermissionError(13, \'The device does not recognize the command.\', None, 22))")'
2026-10-16T14:54:57.927601Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-16T14:54:57.928468Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-16T14:54:57.928605Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-16T14:54:57.928861Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-16T14:54:57.929072Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=0 peer_crc_good_marker=0 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception='SerialException("Cannot configure port, something went wrong. Original message: PermissionError(13, \'The device does not recognize the command.\', None, 22)")'
2026-10-16T14:54:57.929222Z TEST_CLASSIFICATION label='campaign-wireless-mbus-c-868-run-01' preset='wireless_mbus_c_868' classification='BLOCKED'
2026-10-16T14:54:57.929360Z TEST_END exit_code=2
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wireless_mbus_c_868 `
>>     A50D0001 `
>>     campaign-wireless-mbus-c-868-run-01
2026-10-16T14:55:05.620309Z TEST_BEGIN checkpoint='manual-campaign' label='campaign-wireless-mbus-c-868-run-01' tx_port='COM33' rx_port='COM88' preset_name='wireless_mbus_c_868' preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840} marker='A5 0D 00 01' tx_power_dbm=0 tx_count=10
2026-10-16T14:55:05.620454Z TX_INIT_BEGIN port='COM33'
2026-10-16T14:55:06.733556Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:55:06.733669Z RX_INIT_BEGIN port='COM88'
2026-10-16T14:55:07.867872Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T14:55:07.867977Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:55:07.901859Z TX_SET_PHY_ACK
2026-10-16T14:55:07.901939Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T14:55:07.938169Z RX_SET_PHY_ACK
2026-10-16T14:55:07.938266Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:55:07.975159Z TX_CONFIGURE_PROP_ACK
2026-10-16T14:55:07.975254Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 868950000, 'mod_type': 1, 'symbol_rate': 100000, 'deviation': 180, 'rx_bw': 87, 'sync_word': 1413283840}
2026-10-16T14:55:08.012217Z RX_CONFIGURE_PROP_ACK
2026-10-16T14:55:08.012304Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-16T14:55:08.043070Z TX_SET_POWER_ACK
2026-10-16T14:55:08.043156Z TX_STATS_BEFORE_BEGIN
2026-10-16T14:55:08.216258Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:55:08.216367Z RX_STATS_BEFORE_BEGIN
2026-10-16T14:55:08.388930Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T14:55:08.389028Z RX_START_BEGIN
2026-10-16T14:55:08.424499Z RX_START_ACK
2026-10-16T14:55:08.424594Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-16T14:55:08.724917Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.30029599998670164
2026-10-16T14:55:08.725436Z TX_ATTEMPT_BEGIN index=1 payload='A5 0D 00 01'
2026-10-16T14:55:08.768102Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-16T14:55:08.868856Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10024830000475049
2026-10-16T14:55:08.869341Z TX_ATTEMPT_BEGIN index=2 payload='A5 0D 00 01'
2026-10-16T14:55:08.911063Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-16T14:55:09.012021Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10052029999496881
2026-10-16T14:55:09.012152Z TX_ATTEMPT_BEGIN index=3 payload='A5 0D 00 01'
2026-10-16T14:55:09.043292Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-16T14:55:09.144044Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.10052239999640733
2026-10-16T14:55:09.144555Z TX_ATTEMPT_BEGIN index=4 payload='A5 0D 00 01'
2026-10-16T14:55:09.185890Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-16T14:55:09.286631Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.10023609999916516
2026-10-16T14:55:09.287165Z TX_ATTEMPT_BEGIN index=5 payload='A5 0D 00 01'
2026-10-16T14:55:09.323817Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-16T14:55:09.424610Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.1004735000024084
2026-10-16T14:55:09.424940Z TX_ATTEMPT_BEGIN index=6 payload='A5 0D 00 01'
2026-10-16T14:55:09.425242Z TX_ATTEMPT_EXCEPTION index=6 exception_type='SerialException' exception='SerialException("WriteFile failed (PermissionError(13, \'The device does not recognize the command.\', None, 22))")'
2026-10-16T14:55:09.525607Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.1001850999891758
2026-10-16T14:55:09.526023Z TX_ATTEMPT_BEGIN index=7 payload='A5 0D 00 01'
2026-10-16T14:55:09.526380Z TX_ATTEMPT_EXCEPTION index=7 exception_type='SerialException' exception='SerialException("WriteFile failed (PermissionError(13, \'The device does not recognize the command.\', None, 22))")'
2026-10-16T14:55:09.626956Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10044950000883546
2026-10-16T14:55:09.627078Z TX_ATTEMPT_BEGIN index=8 payload='A5 0D 00 01'
2026-10-16T14:55:09.627236Z TX_ATTEMPT_EXCEPTION index=8 exception_type='SerialException' exception='SerialException("WriteFile failed (PermissionError(13, \'The device does not recognize the command.\', None, 22))")'
2026-10-16T14:55:09.727881Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10049959999741986
2026-10-16T14:55:09.728132Z TX_ATTEMPT_BEGIN index=9 payload='A5 0D 00 01'
2026-10-16T14:55:09.728396Z TX_ATTEMPT_EXCEPTION index=9 exception_type='SerialException' exception='SerialException("WriteFile failed (PermissionError(13, \'The device does not recognize the command.\', None, 22))")'
2026-10-16T14:55:09.829147Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.1004802999959793
2026-10-16T14:55:09.829778Z TX_ATTEMPT_BEGIN index=10 payload='A5 0D 00 01'
2026-10-16T14:55:09.830574Z TX_ATTEMPT_EXCEPTION index=10 exception_type='SerialException' exception='SerialException("WriteFile failed (PermissionError(13, \'The device does not recognize the command.\', None, 22))")'
2026-10-16T14:55:09.931351Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10037129999545868
2026-10-16T14:55:09.931644Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-16T14:55:10.932232Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.00060559999838
2026-10-16T14:55:10.932593Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-16T14:55:11.055076Z RX_PACKET ordinal=1 device_timestamp_us=49144 channel=37 rssi_dbm=29 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:55:11.163944Z RX_PACKET ordinal=2 device_timestamp_us=49706 channel=37 rssi_dbm=71 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:55:11.283893Z RX_PACKET ordinal=3 device_timestamp_us=50263 channel=37 rssi_dbm=68 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:55:11.392800Z RX_PACKET ordinal=4 device_timestamp_us=50780 channel=37 rssi_dbm=49 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:55:11.515668Z RX_PACKET ordinal=5 device_timestamp_us=51338 channel=37 rssi_dbm=46 lqi=0 crc_ok=True payload_hex='04 A5 0D 00 01 E1 32' marker_present=True
2026-10-16T14:55:12.943319Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0107637999899453 packets=5 async_errors=0
2026-10-16T14:55:12.943764Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-16T14:55:12.944517Z TEST_FATAL_EXCEPTION exception_type='SerialException' exception='SerialException("Cannot configure port, something went wrong. Original message: PermissionError(13, \'The device does not recognize the command.\', None, 22)")'
Traceback (most recent call last):
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\experimental_artifacts\manual_preset_runner.py", line 184, in main
    for item in tx.read_packets(timeout=0.5):
                ~~~~~~~~~~~~~~~^^^^^^^^^^^^^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 662, in read_packets
    cmd_id, seq, payload = self._read_response(read_timeout)
                           ~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 317, in _read_response
    self._serial.timeout = min(remaining, 0.1)
    ^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\Support\AppData\Local\Programs\Python\Python314\Lib\site-packages\serial\serialutil.py", line 372, in timeout
    self._reconfigure_port()
    ~~~~~~~~~~~~~~~~~~~~~~^^
  File "C:\Users\Support\AppData\Local\Programs\Python\Python314\Lib\site-packages\serial\serialwin32.py", line 222, in _reconfigure_port
    raise SerialException(
        'Cannot configure port, something went wrong. '
        'Original message: {!r}'.format(ctypes.WinError()))
serial.serialutil.SerialException: Cannot configure port, something went wrong. Original message: PermissionError(13, 'The device does not recognize the command.', None, 22)
2026-10-16T14:55:12.952341Z RX_STOP_CLEANUP_BEGIN
2026-10-16T14:55:12.980445Z RX_STOP_CLEANUP_ACK
2026-10-16T14:55:12.980547Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-16T14:55:12.980726Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-16T14:55:12.980842Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-16T14:55:12.981492Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-16T14:55:12.981572Z TEST_SUMMARY tx_accepted=5 tx_requested=10 total_peer_packets=5 peer_crc_good_marker=5 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception='SerialException("Cannot configure port, something went wrong. Original message: PermissionError(13, \'The device does not recognize the command.\', None, 22)")'
2026-10-16T14:55:12.981762Z TEST_CLASSIFICATION label='campaign-wireless-mbus-c-868-run-01' preset='wireless_mbus_c_868' classification='BLOCKED'
2026-10-16T14:55:12.981854Z TEST_END exit_code=2

---

## Paso 48 — `wireless_mbus_c_868`: PASS-RF y dos contrapruebas por RESET

Las tres ejecuciones aportan evidencia interesante porque permiten comparar el comportamiento normal del enlace con dos interrupciones físicas provocadas deliberadamente.

**El resultado principal es positivo: `wireless_mbus_c_868` obtuvo PASS-RF 10/10.** Además, las dos contrapruebas muestran que interrumpir uno de los CatSniffer afecta al procedimiento de forma diferente según el dispositivo reiniciado.

Sin embargo, debemos distinguir dos cuestiones:

1. **Comunicación funcional:** los paquetes enviados mediante FeralRF aparecen en el receptor con el marcador esperado.
2. **Dependencia del hardware físico:** al reiniciar un dispositivo, las operaciones asociadas a él dejan de completarse normalmente.

Las contrapruebas refuerzan la segunda observación, pero no demuestran por sí solas todos los detalles del recorrido RF.

---

## 1. Resultados de las tres ejecuciones

Aunque las tres utilizan el mismo identificador `campaign-wireless-mbus-c-868-run-01`, podemos distinguirlas mediante sus timestamps UTC.

| Ejecución | Inicio UTC | Intervención | TX ACK |    RX válido | Resultado                    |
| --------- | ---------- | ------------ | -----: | -----------: | ---------------------------- |
| 1         | 14:54:19   | Sin RESET    |  10/10 |        10/10 | **PASS-RF**                  |
| 2         | 14:54:52   | RESET en RX  |  10/10 | 0 observados | **BLOCKED — RESET inducido** |
| 3         | 14:55:05   | RESET en TX  |   5/10 |         5/10 | **BLOCKED — RESET inducido** |

La asignación de RX y TX a las contrapruebas es consistente con los errores registrados y con tu descripción de haber reiniciado un CatSniffer diferente en cada ejecución.

**Observación metodológica:** los registros no incluyen un evento explícito que indique el instante exacto en que presionaste RESET. Para documentar las contrapruebas, debemos conservar esa información como una intervención reportada por el operador.

---

## 2. Primera ejecución: comunicación normal

**Resultado: PASS-RF.**

La configuración solicitada fue:

| Parámetro             | Valor                 |
| --------------------- | --------------------- |
| Preset                | `wireless_mbus_c_868` |
| Frecuencia solicitada | 868.95 MHz            |
| Modulación solicitada | GFSK                  |
| Symbol rate           | 100 000 símbolos/s    |
| `deviation`           | 180                   |
| `rx_bw`               | 87                    |
| Sync word             | `0x543C0000`          |
| Potencia solicitada   | 0 dBm                 |
| Marcador              | `A5 0D 00 01`         |

El receptor entregó diez paquetes con el mismo contenido:

```text
payload_hex='04 A5 0D 00 01 E1 32'
crc_ok=True
marker_present=True
```

Los contadores también son consistentes:

```text
RX_STATS_BEFORE: rx_ok=0
RX_STATS_AFTER:  rx_ok=10
```

No hubo excepciones ni errores asíncronos.

**Conclusión:** la comunicación entre los dos CatSniffer funciona con los parámetros solicitados por este preset.

Con ello, los tres presets Wireless M-Bus previstos en esta etapa cuentan con una ejecución PASS-RF:

| Preset                | Resultado         |
| --------------------- | ----------------- |
| `wireless_mbus_s_868` | PASS-RF 10/10     |
| `wireless_mbus_t_868` | PASS-RF 10/10     |
| `wireless_mbus_c_868` | **PASS-RF 10/10** |

Esto no equivale a certificar interoperabilidad con dispositivos Wireless M-Bus reales.

---

## 3. Segunda ejecución: RESET del receptor

**Resultado: BLOCKED por interrupción física inducida.**

En esta ejecución, los diez comandos de transmisión fueron aceptados:

```text
TX_ATTEMPT_ACK index=1
...
TX_ATTEMPT_ACK index=10
```

Después, durante la lectura de paquetes del receptor, apareció:

```text
RX_OBSERVATION_BEGIN
TEST_FATAL_EXCEPTION exception_type='SerialException'
```

El traceback identifica:

```text
manual_preset_runner.py:153
    rx.read_packets(timeout=2.0)

python/feralrf/radio.py:662
    self._read_response(read_timeout)

python/feralrf/radio.py:317
    self._serial.timeout = min(remaining, 0.1)
```

Windows devolvió:

```text
PermissionError(13, 'The device does not recognize the command.')
```

### Interpretación

El programa había establecido correctamente la comunicación con ambos dispositivos y había recibido `RX_START_ACK`.

Posteriormente, perdió la capacidad de operar normalmente sobre el puerto serial del receptor.

Esto es coherente con la intervención física reportada: **presionar RESET en el CatSniffer receptor**.

Hay un detalle importante: el registro muestra cero paquetes observados porque la lectura falló antes de entregar resultados. **No significa que podamos afirmar que el receptor no recibió ningún paquete antes del RESET.**

Tampoco demuestra una pérdida de señal RF: el error ocurrió en la interfaz serial USB utilizada por Python.

---

## 4. Tercera ejecución: RESET del transmisor

Esta contraprueba es especialmente interesante porque permite observar una interrupción durante la secuencia de transmisión.

### Antes del RESET

Los primeros cinco comandos terminaron correctamente:

```text
TX_ATTEMPT_ACK index=1
TX_ATTEMPT_ACK index=2
TX_ATTEMPT_ACK index=3
TX_ATTEMPT_ACK index=4
TX_ATTEMPT_ACK index=5
```

### Después de la interrupción

A partir del sexto intento:

```text
TX_ATTEMPT_BEGIN index=6
TX_ATTEMPT_EXCEPTION index=6
```

El mismo error se repitió hasta el intento 10:

```text
SerialException(
    "WriteFile failed (... The device does not recognize the command ...)"
)
```

### ¿Qué observó el receptor?

A diferencia de la segunda contraprueba, aquí el receptor permaneció operativo.

El resultado fue:

```text
RX_OBSERVATION_COMPLETE packets=5 async_errors=0
```

Los cinco paquetes tenían:

```text
payload_hex='04 A5 0D 00 01 E1 32'
crc_ok=True
marker_present=True
```

Por tanto, observamos:

| Indicador                         | Resultado |
| --------------------------------- | --------: |
| TX ACK antes de la interrupción   |         5 |
| TX ACK después de la interrupción |         0 |
| Paquetes recibidos                |         5 |
| Paquetes CRC-good con marcador    |         5 |
| Intentos TX con excepción         |         5 |

### Interpretación

Esta ejecución aporta una evidencia adicional a la prueba normal:

**Cuando se interrumpió el CatSniffer transmisor, dejaron de aceptarse nuevos comandos TX y el receptor entregó únicamente cinco paquetes, coincidentes con los cinco comandos aceptados antes de la interrupción.**

La correspondencia es consistente con un enlace dependiente del transmisor físico.

No obstante, un TX ACK sigue siendo una confirmación del comando por el firmware, no una medición independiente de emisión RF.

También apareció una excepción al intentar observar respuestas asíncronas del TX. Es esperable que una conexión serial previamente abierta pueda quedar inválida después de reiniciar el dispositivo.

---

## 5. ¿Qué demuestran realmente estas contrapruebas?

Podemos separar los niveles de evidencia:

| Afirmación                                                                                | Estado                               |
| ----------------------------------------------------------------------------------------- | ------------------------------------ |
| El preset `wireless_mbus_c_868` permite intercambiar paquetes entre los dos CatSniffer    | **Validado experimentalmente**       |
| Reiniciar el receptor interrumpe el procedimiento de recepción por Python                 | **Observado**                        |
| Reiniciar el transmisor interrumpe la aceptación de nuevos comandos TX                    | **Observado**                        |
| El receptor recibió cinco paquetes antes de finalizar la tercera prueba                   | **Observado**                        |
| El RESET físico afecta al procedimiento de comunicación                                   | **Respaldado por las contrapruebas** |
| Las contrapruebas demuestran por sí solas que los paquetes viajaron exclusivamente por RF | **No demostrado**                    |
| La señal transmitida cumple todos los parámetros físicos solicitados                      | **No demostrado**                    |
| El preset implementa Wireless M-Bus Mode C conforme al estándar                           | **No demostrado**                    |

### Una distinción importante

El RESET del CatSniffer puede afectar simultáneamente varios elementos:

* La conexión USB y sus interfaces seriales.
* El firmware de control.
* El estado del transceptor.
* La configuración RF previamente aplicada.

Por ello, **la excepción serial no permite aislar cuál de esos efectos causó la interrupción de recepción o transmisión**.

Para una contraprueba RF más específica, en otra etapa podríamos mantener las conexiones USB activas y modificar únicamente una condición de recepción, como la frecuencia configurada en RX.

Eso permitiría estudiar la dependencia de los parámetros RF sin introducir el efecto adicional del RESET.

---

## 6. Estado actualizado de la campaña

Debemos contabilizar las contrapruebas por separado de las ejecuciones ordinarias.

| Indicador                                             |   Resultado |
| ----------------------------------------------------- | ----------: |
| Presets nuevos PASS-RF                                |      **13** |
| Controles positivos PASS-RF                           |           4 |
| Ejecuciones OTA satisfactorias                        |      **18** |
| Paquetes CRC-good con marcador en ejecuciones PASS-RF | **180/180** |
| Ejecuciones BLOCKED anteriores                        |           3 |
| Contrapruebas BLOCKED por RESET inducido              |       **2** |
| Total de ejecuciones BLOCKED                          |       **5** |
| FAIL-RF no inducidos                                  |       **0** |

Los cinco paquetes de la tercera contraprueba son evidencia adicional y **no se incorporan al acumulado de 180/180**, porque pertenecen a una ejecución interrumpida deliberadamente.

Es importante conservar los tres registros originales, aunque compartan identificador, distinguiéndolos por su hora de inicio.

---

## Paso 49 — Control positivo después de las contrapruebas

El procedimiento de campaña establece un control positivo después de cada tres presets nuevos y también después de intervenciones físicas como RESET.

En este caso se cumplen ambas condiciones.

**Antes de continuar con otras configuraciones, debemos comprobar que los dos CatSniffer volvieron a operar correctamente.**

Dado que presionaste RESET en ambos dispositivos durante las contrapruebas, primero debemos restaurar explícitamente la selección de Sub-GHz mediante `band2` en ambos Shell, si todavía no lo hiciste después de las interrupciones.

Una vez restaurada esa selección, ejecuta el control:

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    wisun_915_fsk_50k `
    DEADBEEF `
    control-after-13-presets-post-reset-run-01
```

Este control vuelve a solicitar **902.2 MHz**, por lo que también comprueba el retorno desde la última configuración de 868.95 MHz.

**No necesitamos repetir `wireless_mbus_c_868`: su ejecución normal ya obtuvo PASS-RF.**

Con el resultado del control positivo podremos cerrar este bloque de la campaña, dejando las dos contrapruebas documentadas como experimentos de interrupción inducida, no como fallos de la implementación RF.

---

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF> python -u '.\experimental_artifacts\manual_preset_runner.py' `
>>     wisun_915_fsk_50k `
>>     DEADBEEF `
>>     control-after-13-presets-post-reset-run-01
2026-10-16T15:01:00.610817Z TEST_BEGIN checkpoint='manual-campaign' label='control-after-13-presets-post-reset-run-01' tx_port='COM33' rx_port='COM88' preset_name='wisun_915_fsk_50k' preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606} marker='DE AD BE EF' tx_power_dbm=0 tx_count=10
2026-10-16T15:01:00.610956Z TX_INIT_BEGIN port='COM33'
2026-10-16T15:01:01.733516Z TX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T15:01:01.733625Z RX_INIT_BEGIN port='COM88'
2026-10-16T15:01:02.859967Z RX_INIT_OK info=DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
2026-10-16T15:01:02.860083Z TX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T15:01:02.890696Z TX_SET_PHY_ACK
2026-10-16T15:01:02.890785Z RX_SET_PHY_BEGIN phy='PROPRIETARY_GFSK' channel=0
2026-10-16T15:01:02.923686Z RX_SET_PHY_ACK
2026-10-16T15:01:02.923786Z TX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-16T15:01:02.952357Z TX_CONFIGURE_PROP_ACK
2026-10-16T15:01:02.952457Z RX_CONFIGURE_PROP_BEGIN preset={'frequency_hz': 902200000, 'mod_type': 0, 'symbol_rate': 50000, 'deviation': 100, 'rx_bw': 82, 'sync_word': 2466992606}
2026-10-16T15:01:02.990316Z RX_CONFIGURE_PROP_ACK
2026-10-16T15:01:02.990424Z TX_SET_POWER_BEGIN power_dbm=0
2026-10-16T15:01:03.021787Z TX_SET_POWER_ACK
2026-10-16T15:01:03.021881Z TX_STATS_BEFORE_BEGIN
2026-10-16T15:01:03.198780Z TX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T15:01:03.198880Z RX_STATS_BEFORE_BEGIN
2026-10-16T15:01:03.364568Z RX_STATS_BEFORE stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T15:01:03.364672Z RX_START_BEGIN
2026-10-16T15:01:03.399102Z RX_START_ACK
2026-10-16T15:01:03.399662Z PRE_TX_WAIT_BEGIN requested_seconds=0.3
2026-10-16T15:01:03.700342Z PRE_TX_WAIT_COMPLETE elapsed_seconds=0.3006852999969851
2026-10-16T15:01:03.700864Z TX_ATTEMPT_BEGIN index=1 payload='DE AD BE EF'
2026-10-16T15:01:03.741385Z TX_ATTEMPT_ACK index=1 return_value=None
2026-10-16T15:01:03.842160Z TX_SPACING_COMPLETE index=1 elapsed_seconds=0.10044840000045951
2026-10-16T15:01:03.842604Z TX_ATTEMPT_BEGIN index=2 payload='DE AD BE EF'
2026-10-16T15:01:03.879812Z TX_ATTEMPT_ACK index=2 return_value=None
2026-10-16T15:01:03.980377Z TX_SPACING_COMPLETE index=2 elapsed_seconds=0.10032399999909103
2026-10-16T15:01:03.980877Z TX_ATTEMPT_BEGIN index=3 payload='DE AD BE EF'
2026-10-16T15:01:04.019649Z TX_ATTEMPT_ACK index=3 return_value=None
2026-10-16T15:01:04.120025Z TX_SPACING_COMPLETE index=3 elapsed_seconds=0.1002332999923965
2026-10-16T15:01:04.120534Z TX_ATTEMPT_BEGIN index=4 payload='DE AD BE EF'
2026-10-16T15:01:04.150721Z TX_ATTEMPT_ACK index=4 return_value=None
2026-10-16T15:01:04.251500Z TX_SPACING_COMPLETE index=4 elapsed_seconds=0.1006381000042893
2026-10-16T15:01:04.252055Z TX_ATTEMPT_BEGIN index=5 payload='DE AD BE EF'
2026-10-16T15:01:04.293866Z TX_ATTEMPT_ACK index=5 return_value=None
2026-10-16T15:01:04.394385Z TX_SPACING_COMPLETE index=5 elapsed_seconds=0.10021150000102352
2026-10-16T15:01:04.394829Z TX_ATTEMPT_BEGIN index=6 payload='DE AD BE EF'
2026-10-16T15:01:04.436911Z TX_ATTEMPT_ACK index=6 return_value=None
2026-10-16T15:01:04.537569Z TX_SPACING_COMPLETE index=6 elapsed_seconds=0.10017029999289662
2026-10-16T15:01:04.538093Z TX_ATTEMPT_BEGIN index=7 payload='DE AD BE EF'
2026-10-16T15:01:04.578131Z TX_ATTEMPT_ACK index=7 return_value=None
2026-10-16T15:01:04.678673Z TX_SPACING_COMPLETE index=7 elapsed_seconds=0.10016009998798836
2026-10-16T15:01:04.679172Z TX_ATTEMPT_BEGIN index=8 payload='DE AD BE EF'
2026-10-16T15:01:04.715455Z TX_ATTEMPT_ACK index=8 return_value=None
2026-10-16T15:01:04.815943Z TX_SPACING_COMPLETE index=8 elapsed_seconds=0.10034760000417009
2026-10-16T15:01:04.816416Z TX_ATTEMPT_BEGIN index=9 payload='DE AD BE EF'
2026-10-16T15:01:04.850775Z TX_ATTEMPT_ACK index=9 return_value=None
2026-10-16T15:01:04.951620Z TX_SPACING_COMPLETE index=9 elapsed_seconds=0.1005903000041144
2026-10-16T15:01:04.952126Z TX_ATTEMPT_BEGIN index=10 payload='DE AD BE EF'
2026-10-16T15:01:04.994312Z TX_ATTEMPT_ACK index=10 return_value=None
2026-10-16T15:01:05.095307Z TX_SPACING_COMPLETE index=10 elapsed_seconds=0.10049149999395013
2026-10-16T15:01:05.095819Z POST_TX_WAIT_BEGIN requested_seconds=1.0
2026-10-16T15:01:06.096684Z POST_TX_WAIT_COMPLETE elapsed_seconds=1.0008820000075502
2026-10-16T15:01:06.097119Z RX_OBSERVATION_BEGIN requested_seconds=2.0
2026-10-16T15:01:06.234233Z RX_PACKET ordinal=1 device_timestamp_us=1435818 channel=37 rssi_dbm=-90 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:06.348735Z RX_PACKET ordinal=2 device_timestamp_us=1436371 channel=37 rssi_dbm=17 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:06.470058Z RX_PACKET ordinal=3 device_timestamp_us=1436911 channel=37 rssi_dbm=-40 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:06.587438Z RX_PACKET ordinal=4 device_timestamp_us=1437457 channel=37 rssi_dbm=59 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:06.697902Z RX_PACKET ordinal=5 device_timestamp_us=1437971 channel=37 rssi_dbm=90 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:06.807311Z RX_PACKET ordinal=6 device_timestamp_us=1438528 channel=37 rssi_dbm=118 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:06.931678Z RX_PACKET ordinal=7 device_timestamp_us=1439088 channel=37 rssi_dbm=-51 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:07.042390Z RX_PACKET ordinal=8 device_timestamp_us=1439639 channel=37 rssi_dbm=96 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:07.163612Z RX_PACKET ordinal=9 device_timestamp_us=1440175 channel=37 rssi_dbm=27 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:07.274434Z RX_PACKET ordinal=10 device_timestamp_us=1440705 channel=37 rssi_dbm=2 lqi=0 crc_ok=True payload_hex='04 DE AD BE EF 33 C8' marker_present=True
2026-10-16T15:01:08.105815Z RX_OBSERVATION_COMPLETE elapsed_seconds=2.0087352999980794 packets=10 async_errors=0
2026-10-16T15:01:08.106300Z TX_ASYNC_OBSERVATION_BEGIN requested_seconds=0.5
2026-10-16T15:01:08.617732Z TX_ASYNC_OBSERVATION_COMPLETE elapsed_seconds=0.5114354000106687 async_errors=0
2026-10-16T15:01:08.618243Z RX_STOP_BEGIN
2026-10-16T15:01:08.650018Z RX_STOP_ACK
2026-10-16T15:01:08.650110Z TX_STATS_AFTER_BEGIN
2026-10-16T15:01:08.822258Z TX_STATS_AFTER stats=DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T15:01:08.822355Z RX_STATS_AFTER_BEGIN
2026-10-16T15:01:09.000432Z RX_STATS_AFTER stats=DeviceStats(rx_ok=10, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
2026-10-16T15:01:09.000528Z TX_DISCONNECT_BEGIN port='COM33'
2026-10-16T15:01:09.001283Z TX_DISCONNECT_COMPLETE port='COM33'
2026-10-16T15:01:09.001343Z RX_DISCONNECT_BEGIN port='COM88'
2026-10-16T15:01:09.001936Z RX_DISCONNECT_COMPLETE port='COM88'
2026-10-16T15:01:09.002007Z TEST_SUMMARY tx_accepted=10 tx_requested=10 total_peer_packets=10 peer_crc_good_marker=10 peer_expected=10 rx_async_errors=0 tx_async_errors=0 fatal_exception=None
2026-10-16T15:01:09.002061Z TEST_CLASSIFICATION label='control-after-13-presets-post-reset-run-01' preset='wisun_915_fsk_50k' classification='PASS-RF'
2026-10-16T15:01:09.002145Z TEST_END exit_code=0

---

Ok, entonces, es hora de crear un reporte de todos los resultados obtenidos hasta ahora. Antes de hacer algo ¿Puedes retomar la información desde el inicio del Chat? Entiendo que normalmente pierdes contexto, pero me gustaría saber si lo puedes hacer, conservando además los comandos y salidas intactas a nivel código de los comandos de ejecución y salida de ellos.

