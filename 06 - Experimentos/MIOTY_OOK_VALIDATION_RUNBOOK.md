# Runbook práctico: validación del preset MIOTY y OOK

**Documento de procedimiento; no contiene resultados experimentales.**  Este runbook prepara dos ensayos OTA de paquete propietario con dos CatSniffer: `mioty_868_tsunb`, tratado exclusivamente como FSK genérico de baja velocidad, y `ook_868_4k8`, tratado como OOK propietario con recuperación obligatoria. No valida TS-UNB ni interoperabilidad MIOTY.

## 1. Alcance, estado de fuente y reglas de parada

La revisión de este procedimiento se realizó sobre:

- rama esperada: `feature/hardware-validation`;
- revisión auditada: `fbff1b7f5077964418fb63a3fdf346a27c35a003`;
- TI SimpleLink SDK: `8.30.01.01` según la documentación del proyecto.

Antes de ejecutar, el operador debe volver a registrar rama, revisión y estado de trabajo. Un árbol con cambios no impide el ensayo, pero esos cambios deben quedar identificados y no deben mezclarse con el resultado.

Reglas obligatorias:

1. Ejecutar una sola operación `OP-MO-*` a la vez, revisar su log y decidir si se continúa.
2. No inferir Cat-Shell a partir del número de Cat-Bridge. Cada interfaz se identifica explícitamente.
3. No usar `Radio.reset_device()`, `--auto-reset` ni un reset de software como sustituto de retirar alimentación.
4. Un `ACK` demuestra aceptación del comando por el protocolo host; no demuestra que la forma de onda solicitada haya ocurrido ni que el paquete haya llegado al peer.
5. Después de configurar OOK no cambiar PHY, frecuencia ni preset. La única operación RF restante es el propio ensayo OOK; después se retira toda alimentación.
6. Si falla `RX_STOP`, desaparece un puerto, aparece un error asíncrono, se bloquea el proceso o no puede verificarse la recuperación, detener la sesión.

## 2. Configuraciones verificadas

Las definiciones están en [`python/feralrf/presets.py`](../python/feralrf/presets.py).

| Preset | Frecuencia | `mod_type` | Tasa solicitada | Desviación | `rx_bw` | Sync word | `format_conf` |
|---|---:|---:|---:|---:|---:|---:|---:|
| `gfsk_868_50k` (control) | 868000000 Hz | 1, GFSK | 50000 baud | 100 | `0x52` | `0x930B51DE` | 0, valor predeterminado |
| `mioty_868_tsunb` | 868000000 Hz | 0, FSK | 396 baud | 1 | `0x4A` | `0x930B51DE` | 0, valor predeterminado |
| `ook_868_4k8` | 868000000 Hz | 2, OOK/ASK | 4800 baud | 0 | `0x4C` | `0x930B51DE` | 0, valor predeterminado |

### 2.1 Ruta del comando y límite de interpretación

`Radio.configure_prop()` llama a `CommandBuilder.set_prop_config()`, que serializa `SET_PROP_CONFIG` (`0x08`). El procesador de comandos de firmware entrega los campos a `RadioIF_setPropConfig()`, que actualiza el RF setup, TX, RX, sync word y frecuencia del backend propietario de TI RF Driver.

El `ACK` de `SET_PROP_CONFIG` no incluye lectura de vuelta de registros ni medición RF. Por tanto, se informan por separado:

- configuración aceptada;
- `TX_RAW` aceptado;
- paquete observado por el peer;
- metadato CRC informado por firmware;
- limpieza completada;
- medición física, que este procedimiento no realiza.

### 2.2 Qué significa el preset rotulado MIOTY

El cálculo del firmware usa `preScale = 15` y:

```text
rateWord = floor(symbol_rate * 1048576 * 15 / 24000000)
```

Para 396 baud resulta `rateWord = 259`, equivalente teóricamente a aproximadamente `395.203 baud`. El backend sigue siendo el modo propietario de paquete FSK: preámbulo, sync word, longitud, payload y CRC. No existen en esta ruta la división de telegramas, codificación, interleaving, temporización de subpaquetes ni procedimientos TS-UNB. Un intercambio positivo prueba sólo entrega de paquetes FSK propietarios con esta implementación.

### 2.3 Ciclo de vida OOK

Con `mod_type=2`, `RadioIF_setPropConfig()` selecciona los overrides OOK, marca `s_prop_ook_active` y el backend usa `Prop0_modeOok`, que carga `rf_patch_cpe_prop`, `rf_patch_mce_genook` y `rf_patch_rfe_genook`. El código documenta que el parche del RF core no puede descargarse con seguridad durante la misma alimentación; detener RX/TX o cerrar el puerto no revierte ese estado. Cambiar posteriormente de PHY o de frecuencia puede bloquear el RF core.

Consecuencia operativa: la primera configuración OOK fija modo y frecuencia para esa alimentación. Tras el ensayo se debe retirar físicamente toda alimentación de ambas placas durante al menos 10 segundos; se recomienda 15 segundos. Después se vuelven a identificar los puertos, se ejecuta preflight y se repite el control conocido. Un `RX_STOP` aceptado no elimina esta obligación.

## 3. Evaluación de herramientas existentes

| Herramienta | Comportamiento relevante | Decisión para este runbook |
|---|---|---|
| `python/examples/smoke_ota_txrx.py` | Requiere `--tx-port` y `--rx-port`; admite `--baudrate`, `--phy` o `--preset`, `--channel`, `--power`, `--count` y `--min-markers`. Usa `DEADBEEF`, inicia RX 0.3 s antes, separa TX por 0.1 s y observa 2 s. Salidas: 0 alcanza el umbral, 1 no lo alcanza, 2 falta PHY/preset, 4 excepción. Omite `RxStreamError`, oculta el fallo de `stop_rx()` y llama siempre a un reset que infiere Cat-Shell por `Bridge + 2`. | **No usar.** El reset es inseguro, los errores pueden ocultarse y el espaciado es menor que el airtime del preset lento. |
| `python/examples/smoke_prop_phase1.py` | Es de una placa; admite `--port`, `--baudrate`, `--preset`, `--power`, `--duration` (3 s), `--tx-timeout`, `--packet-hex`, `--min-packets` (0) y `--auto-reset`. Hace RX, después un TX local; 0 indica que el smoke terminó, 2 argumento, 3 conexión, 4 timeout, 5 protocolo, 6 inesperado y 7 no se alcanzó `min-packets`. Con el mínimo predeterminado puede salir 0 con cero RX. `--auto-reset` llama `Radio.reset_device()`. | No sirve como criterio OTA entre dos placas. No usar `--auto-reset`. |
| `python/examples/smoke_phase2.py` | Admite `--port`, `--baudrate`, `--phy`, `--channel`, `--power`; ejecuta SET de PHY/canal/potencia, abre RX y lo detiene. Salidas: 0 PASS de comandos, 2 argumento, 3 conexión, 4 timeout, 5 protocolo, 6 inesperado. | No se usa como preflight porque modifica RF e introduce una operación RX innecesaria. |
| `python/examples/lab/demo_mioty_listen.py` | Requiere `--port`; admite `--preset` MIOTY, `--baudrate`, `--max-hex` y `--duration` (30 s). Es RX-only, omite `RxStreamError`, no llama explícitamente `stop_rx()` y retorna 0 aunque no reciba paquetes. | Es un stub de escucha, no un criterio peer-to-peer. No usar para aceptación. |
| `python/examples/lab/ota_rx_probe.py` y `ota_tx_burst.py` | Tienen conteo, marcador y códigos de error útiles, pero seleccionan sólo `PHY`/canal y no aceptan `--preset` ni `SET_PROP_CONFIG`; además requieren dos procesos coordinados. | No pueden aplicar directamente ninguna de las dos configuraciones solicitadas. |
| Otros archivos en `python/examples/lab/` | No hay un runner que resuelva conjuntamente temporización MIOTY-labeled, conteo único, errores asíncronos y aislamiento OOK. | No se invocan. |

Se usa abajo un harness temporal en memoria de PowerShell. No crea ni modifica un script: llama a la API pública actual y ejecuta exactamente un preset por invocación, con puertos explícitos, payloads numerados, logs, limpieza visible y código de salida conservado.

## 4. Hardware, autorización y criterio GO/NO-GO

Se requieren:

- dos CatSniffer con CC1352P7 y firmware FeralRF compatible;
- dos conexiones USB identificables de manera independiente;
- Cat-Bridge de cada placa a 921600 baud y Cat-Shell de cada placa a 115200 baud;
- revisión de hardware y ruta Sub-GHz documentadas;
- antenas adecuadas para 868 MHz o fixture conducido previamente verificado;
- entorno RF controlado, blindado o con autorización aplicable;
- capacidad de retirar completamente alimentación de ambas placas;
- conocimiento de límites del receptor, pérdidas, atenuación, aislamiento y conectores si se usa un montaje conducido.

Los mapeos históricos `COM33`/`COM35` y `COM88`/`COM87` son únicamente ejemplos y **no deben reutilizarse sin identificación actual**.

### Lista GO/NO-GO antes de cualquier transmisión

Marcar todos los puntos:

- [ ] Se registró la revisión exacta de ambas placas y se confirmó la ruta RF de 868 MHz.
- [ ] Se identificaron por placa Cat-Bridge y Cat-Shell sin aritmética de puertos.
- [ ] Hay antenas correctas o un fixture conducido con atenuación, aislamiento, conectores y potencia máxima del receptor verificados.
- [ ] No existe una conexión coaxial directa sin atenuación e aislamiento calculados.
- [ ] La operación a 868 MHz, la potencia solicitada y el duty cycle están autorizados en la jurisdicción y el laboratorio reales.
- [ ] El entorno es controlado, blindado o de otro modo autorizado.
- [ ] Se dispone de desconexión física de toda fuente de alimentación, incluidos programadores, hubs alimentados y señales auxiliares.
- [ ] El operador ha fijado y registrado la potencia solicitada. Este procedimiento propone `0 dBm`, no high-PA; reducirla si el montaje lo requiere.

Si algún punto no está establecido, la transmisión es **NO-GO**. Sólo se permite completar identificación, inventario, consulta Cat-Shell y preflight sin TX. No se asume que 868 MHz esté permitido en México ni en otra jurisdicción.

## 5. Preparación común en Windows PowerShell

Abrir una consola PowerShell ordinaria. No instalar dependencias durante esta sesión.

```powershell
$Repo = 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF'
Set-Location (Join-Path $Repo 'python')

python --version
python -c "import sys, importlib.metadata as m; import feralrf, serial, serial_asyncio, cobs; print('python=',sys.executable); print('feralrf=',feralrf.__file__); print('feralrf_version=',feralrf.__version__); print('pyserial=',m.version('pyserial')); print('pyserial-asyncio=',m.version('pyserial-asyncio')); print('cobs=',m.version('cobs'))"

git -C .. branch --show-current
git -C .. rev-parse HEAD
git -C .. status --short

$LogDir = Join-Path $env:TEMP ('feralrf_mioty_ook_' + (Get-Date -Format 'yyyyMMdd_HHmmss'))
New-Item -ItemType Directory -Path $LogDir | Out-Null
$LogDir
```

Resultado esperado: Python 3.9 o posterior; importación sin error de `feralrf`, `serial`, `serial_asyncio` y `cobs`; rama y revisión registradas. Si falta un módulo, clasificar **BLOCKED** y resolverlo fuera de esta sesión; no instalar nada como parte del ensayo.

En una canalización `python ... 2>&1 | Tee-Object`, capturar `$LASTEXITCODE` inmediatamente después: `Tee-Object` es un cmdlet y el valor conserva el código del último proceso nativo, Python. No ejecutar otro programa nativo antes de guardar ese valor.

## 6. OP-MO-01 — Identificación de hardware

### Prerrequisitos

- No transmitir.
- Conectar una sola placa al inicio para relacionar físicamente el conjunto de interfaces; repetir con la otra.

### Comandos

Con cada placa conectada por separado:

```powershell
Get-CimInstance Win32_SerialPort |
  Sort-Object DeviceID |
  Select-Object DeviceID, Name, PNPDeviceID |
  Format-List

python -m serial.tools.list_ports -v
```

En las unidades históricamente inspeccionadas, las interfaces USB del mismo dispositivo aparecieron agrupadas por identidad PNP y `MI_00` correspondió a Bridge, `MI_02` a LoRa y `MI_04` a Shell. Usar esa relación sólo si las descripciones e identidad USB actuales la confirman. No derivar un puerto sumando o restando números.

Registrar y después asignar manualmente:

```powershell
$TxBridge = 'COM__'
$TxShell  = 'COM__'
$RxBridge = 'COM__'
$RxShell  = 'COM__'
$TxPowerDbm = 0
```

Consulta Cat-Shell, una interfaz por vez:

```powershell
$ShellQueryCode = @'
import os, time, serial
p = os.environ["FERAL_SHELL_PORT"]
with serial.Serial(p, 115200, timeout=0.5, write_timeout=1.0) as s:
    s.reset_input_buffer()
    s.write(b"status\r\n")
    s.flush()
    time.sleep(1.0)
    print(s.read_all().decode(errors="replace"), end="")
'@

$env:FERAL_SHELL_PORT = $TxShell
$ShellQueryCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-01_tx_shell.txt')
$TxShellExit = $LASTEXITCODE

$env:FERAL_SHELL_PORT = $RxShell
$ShellQueryCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-01_rx_shell.txt')
$RxShellExit = $LASTEXITCODE
```

### Criterio

- **PASS:** ambas placas se distinguen físicamente; Bridge y Shell quedan identificados; Cat-Shell responde en los puertos explícitos; se registran revisión, antena/fixture y alimentación.
- **BLOCKED:** identidad ambigua, puerto ocupado, interfaz faltante o revisión/ruta RF no establecida.

No continuar si existe ambigüedad. La respuesta `status` ayuda a identificar la interfaz lógica, pero por sí sola no prueba la ruta física del switch RF.

## 7. OP-MO-02 — Preflight inicial y estado limpio

### Prerrequisitos y limpieza inicial

1. Cerrar cualquier aplicación que use los cuatro puertos.
2. Retirar **toda** alimentación de ambas placas durante al menos 10 segundos; usar 15 segundos como margen operativo.
3. Reconectar una placa a la vez y repetir la enumeración de `OP-MO-01`; reasignar variables si Windows cambió los COM.

### Preflight de Bridge sin TX

```powershell
$PreflightCode = @'
import os
from feralrf import Radio
p = os.environ["FERAL_PORT"]
r = Radio(port=p, baudrate=921600)
try:
    info = r.init()
    stats = r.get_stats()
    print("PORT=", p)
    print("INFO=", info)
    print("STATS=", stats)
finally:
    r.disconnect()
'@

$env:FERAL_PORT = $TxBridge
$PreflightCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-02_tx_preflight.txt')
$TxPreflightExit = $LASTEXITCODE

$env:FERAL_PORT = $RxBridge
$PreflightCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-02_rx_preflight.txt')
$RxPreflightExit = $LASTEXITCODE
```

### Criterio

- **PASS:** ambos procesos terminan con código 0, informan `DeviceInfo` y `DeviceStats`, y no aparece error de protocolo, timeout ni desconexión.
- **FAIL/BLOCKED:** cualquier error o código distinto de 0. Cerrar procesos, retirar alimentación, reidentificar y repetir como máximo una vez. Si se repite, terminar la sesión.

`RADIO_INIT`, `GET_INFO` y `GET_STATS` verifican acceso y plano de control. No son evidencia de TX, RX ni frecuencia radiada.

## 8. Preparar la ruta Sub-GHz y el harness OTA manual

Ejecutar esta sección sólo después de obtener **GO** en la lista de seguridad. `band2` selecciona la ruta lógica Sub-GHz de CatSniffer; la respuesta no sustituye la confirmación física del hardware.

```powershell
$Band2Code = @'
import os, time, serial
p = os.environ["FERAL_SHELL_PORT"]
with serial.Serial(p, 115200, timeout=0.5, write_timeout=1.0) as s:
    s.reset_input_buffer()
    for cmd in (b"band2\r\n", b"status\r\n"):
        s.write(cmd)
        s.flush()
        time.sleep(1.0)
    print(s.read_all().decode(errors="replace"), end="")
'@

$env:FERAL_SHELL_PORT = $TxShell
$Band2Code | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'band2_tx.txt')
$TxBandExit = $LASTEXITCODE

$env:FERAL_SHELL_PORT = $RxShell
$Band2Code | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'band2_rx.txt')
$RxBandExit = $LASTEXITCODE
```

Si cualquier comando falla o la respuesta es incoherente, **NO-GO** para RF.

Copiar una vez el siguiente harness en la misma sesión PowerShell. No lo guardar como archivo:

```powershell
$OtaCode = @'
import os
import sys
import time
import traceback
from feralrf import PHY, PROP_PRESETS, Packet, Radio, RxStreamError

def env(name):
    value = os.environ.get(name)
    if value is None or value == "":
        raise RuntimeError(f"missing environment variable: {name}")
    return value

label = env("FERAL_LABEL")
tx_port = env("FERAL_TX_PORT")
rx_port = env("FERAL_RX_PORT")
preset_name = env("FERAL_PRESET")
power_dbm = int(env("FERAL_POWER_DBM"))
marker_prefix = bytes.fromhex(env("FERAL_MARKER_PREFIX_HEX"))
count = int(env("FERAL_COUNT"))
minimum = int(env("FERAL_MIN_HITS"))
spacing = float(env("FERAL_SPACING_SECONDS"))
observe = float(env("FERAL_OBSERVE_SECONDS"))
ook_stage = env("FERAL_OOK_STAGE") == "1"

if preset_name not in PROP_PRESETS:
    raise RuntimeError(f"unknown preset: {preset_name}")
if len(marker_prefix) != 7:
    raise RuntimeError("marker prefix must be exactly 7 bytes")
if count < 1 or count > 255:
    raise RuntimeError("count must be in range 1..255")

preset = dict(PROP_PRESETS[preset_name])
tx = Radio(port=tx_port, baudrate=921600)
rx = Radio(port=rx_port, baudrate=921600)
rx_started = False
cleanup_ok = False
setup_complete = False
tx_acks = 0
unique_hits = set()
crc_good_unique_hits = set()
rx_async_errors = []
tx_async_errors = []
tx_exceptions = []
fatal = None

def consume(item, origin):
    if isinstance(item, RxStreamError):
        event = (item.error_code, item.context)
        (rx_async_errors if origin == "RX" else tx_async_errors).append(event)
        print(f"{origin}_ASYNC_ERROR code={item.error_code} context={item.context}")
        return
    if not isinstance(item, Packet):
        print(f"{origin}_UNKNOWN_ITEM {item!r}")
        return
    data = bytes(item.data)
    pos = data.find(marker_prefix)
    seq = None
    if pos >= 0 and pos + len(marker_prefix) < len(data):
        candidate = data[pos + len(marker_prefix)]
        if 1 <= candidate <= count:
            seq = candidate
            unique_hits.add(seq)
            if item.crc_ok:
                crc_good_unique_hits.add(seq)
    print(
        f"{origin}_PACKET seq={seq} crc_ok={item.crc_ok} "
        f"rssi_dbm={item.rssi_dbm} lqi={item.lqi} data={data.hex()}"
    )

print(f"LABEL={label}")
print(f"TX_PORT={tx_port} RX_PORT={rx_port}")
print(f"PRESET={preset_name} CONFIG={preset}")
print(f"POWER_DBM={power_dbm} COUNT={count} MIN_HITS={minimum}")
print(f"MARKER_PREFIX={marker_prefix.hex()} SEQUENCES=01..{count:02x}")
print(f"SPACING_SECONDS={spacing} OBSERVE_SECONDS={observe}")

try:
    tx_info = tx.init()
    rx_info = rx.init()
    print(f"TX_INFO={tx_info}")
    print(f"RX_INFO={rx_info}")

    tx.set_phy(PHY.PROPRIETARY_GFSK, channel=0)
    rx.set_phy(PHY.PROPRIETARY_GFSK, channel=0)
    tx.configure_prop(**preset)
    rx.configure_prop(**preset)
    tx.set_power(power_dbm)
    setup_complete = True
    print("CONFIG_ACCEPTED=2/2")
    print(f"TX_STATS_BEFORE={tx.get_stats()}")
    print(f"RX_STATS_BEFORE={rx.get_stats()}")

    rx.start_rx()
    rx_started = True
    print("RX_START_ACK=1")
    time.sleep(1.0)

    for seq in range(1, count + 1):
        payload = marker_prefix + bytes([seq])
        try:
            tx.transmit(payload, power_dbm=power_dbm, timeout=5.0)
            tx_acks += 1
            print(f"TX_ACK seq={seq} payload={payload.hex()}")
        except Exception as exc:
            tx_exceptions.append((seq, repr(exc)))
            print(f"TX_EXCEPTION seq={seq} error={exc!r}")
        time.sleep(spacing)

    time.sleep(1.0)
    for item in rx.read_packets(timeout=observe):
        consume(item, "RX")
    for item in tx.read_packets(timeout=1.0):
        consume(item, "TX")
except BaseException as exc:
    fatal = exc
    print(f"FATAL={exc!r}")
    traceback.print_exc()
finally:
    if rx_started:
        try:
            rx.stop_rx(retries=3, timeout=1.0)
            cleanup_ok = True
            print("RX_STOP_ACK=1")
        except BaseException as exc:
            print(f"RX_STOP_ERROR={exc!r}")
    else:
        print("RX_STOP_ACK=NOT_APPLICABLE")
    allow_post_stats = not ook_stage and fatal is None and (not rx_started or cleanup_ok)
    if allow_post_stats:
        try:
            print(f"TX_STATS_AFTER={tx.get_stats()}")
        except BaseException as exc:
            print(f"TX_STATS_AFTER_ERROR={exc!r}")
        try:
            print(f"RX_STATS_AFTER={rx.get_stats()}")
        except BaseException as exc:
            print(f"RX_STATS_AFTER_ERROR={exc!r}")
    else:
        print("STATS_AFTER=SKIPPED_FOR_SAFE_RECOVERY")
    tx.disconnect()
    rx.disconnect()

errors = len(rx_async_errors) + len(tx_async_errors) + len(tx_exceptions)
raw_criterion = (
    fatal is None
    and setup_complete
    and tx_acks == count
    and len(unique_hits) >= minimum
    and len(crc_good_unique_hits) >= minimum
    and errors == 0
    and cleanup_ok
)

if raw_criterion and ook_stage:
    classification = "PARTIAL_AWAITING_MANDATORY_POWER_CYCLE_AND_CONTROL"
elif raw_criterion:
    classification = "PASS_RAW_STAGE"
elif fatal is not None or not setup_complete or not cleanup_ok:
    classification = "BLOCKED_OR_CLEANUP_FAILED"
elif len(unique_hits) >= minimum and len(crc_good_unique_hits) < minimum:
    classification = "INCONCLUSIVE_CRC_METADATA"
elif len(unique_hits) > 0:
    classification = "PARTIAL_RAW_DELIVERY"
elif tx_acks == count:
    classification = "FAIL_RAW_DELIVERY_CRITERION"
else:
    classification = "FAIL_CONTROL_OR_TX_ACCEPTANCE"

print(f"SUMMARY tx_acks={tx_acks}/{count}")
print(f"SUMMARY unique_marker_hits={len(unique_hits)} values={sorted(unique_hits)}")
print(f"SUMMARY crc_good_unique_hits={len(crc_good_unique_hits)} values={sorted(crc_good_unique_hits)}")
print(f"SUMMARY async_or_tx_errors={errors}")
print(f"SUMMARY cleanup_ok={cleanup_ok}")
print(f"CLASSIFICATION={classification}")
sys.exit(0 if raw_criterion else 1)
'@
```

El harness nunca llama `reset_device()`. El código de salida 0 significa únicamente que se cumplieron sus criterios de etapa; el operador debe inspeccionar `SUMMARY`, errores y log. Para OOK, aun con código 0 la clasificación permanece `PARTIAL_AWAITING_MANDATORY_POWER_CYCLE_AND_CONTROL`.

## 9. Temporización y payload

Cada payload lleva un prefijo ASCII de 7 bytes y una secuencia de 1 byte. Con el framing propietario predeterminado, la estimación nominal es:

```text
4 bytes preámbulo + 4 bytes sync + 1 byte longitud + 8 bytes payload + 2 bytes CRC
= 19 bytes = 152 bits
```

| Preset | Tasa efectiva usada para estimar | Airtime nominal de 152 bits | Separación del runbook |
|---|---:|---:|---:|
| `gfsk_868_50k` | 50000 baud | 3.04 ms | 0.25 s |
| `mioty_868_tsunb` | 395.203 baud teóricos | 384.6 ms | 1.0 s |
| `ook_868_4k8` | 4800 baud | 31.7 ms | 0.25 s |

La estimación no incluye arranque de sintetizador, latencia del scheduler, USB ni RF Driver. Para el preset lento, 1.0 s supera 2.5 veces el airtime calculado. El receptor inicia 1 s antes, permanece activo durante los diez envíos y se observa 5 s adicionales. No usar el espaciado de 0.1 s de `smoke_ota_txrx.py` para el preset lento.

## 10. OP-MO-03 — Control RF conocido

### Prerrequisitos

- `OP-MO-01` y `OP-MO-02`: PASS.
- Lista GO/NO-GO completa y `band2` aceptado en ambas placas.
- RX y TX físicamente separados o unidos sólo mediante fixture conducido verificado.

### Comando

```powershell
$env:FERAL_LABEL = 'OP-MO-03_CONTROL'
$env:FERAL_TX_PORT = $TxBridge
$env:FERAL_RX_PORT = $RxBridge
$env:FERAL_PRESET = 'gfsk_868_50k'
$env:FERAL_POWER_DBM = [string]$TxPowerDbm
$env:FERAL_MARKER_PREFIX_HEX = '4D4F3033433030'
$env:FERAL_COUNT = '10'
$env:FERAL_MIN_HITS = '9'
$env:FERAL_SPACING_SECONDS = '0.25'
$env:FERAL_OBSERVE_SECONDS = '5.0'
$env:FERAL_OOK_STAGE = '0'

$OtaCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-03_control.txt')
$MO03Exit = $LASTEXITCODE
"python_exit=$MO03Exit" | Tee-Object -Append -FilePath (Join-Path $LogDir 'OP-MO-03_control.txt')
```

El marcador es el prefijo hexadecimal `4D4F3033433030` (`MO03C00`) seguido por `01` a `0A`.

### Criterio y decisión

- **PASS de etapa raw:** configuración 2/2; 10/10 `TX_ACK`; al menos 9 secuencias únicas coincidentes y marcadas `crc_ok=True`; cero errores asíncronos/errores TX; `RX_STOP_ACK=1`; código 0.
- **INCONCLUSIVE:** se observan marcadores suficientes pero el campo `crc_ok` no alcanza 9. El parser host/firmware de CRC tiene limitaciones documentadas; esto no borra la observación del payload, pero tampoco satisface el criterio provisional CRC-valid.
- **PARTIAL/FAIL:** usar literalmente `CLASSIFICATION` y los conteos. Un 10/10 de ACK con 0 hits no prueba transmisión radiada ni identifica la causa.
- **STOP:** cualquier fallo de cleanup, error asíncrono, puerto perdido o proceso bloqueado. Retirar alimentación antes de diagnosticar.

No continuar a `OP-MO-04` salvo que el control sea PASS o exista una decisión explícita del responsable de laboratorio que documente por qué un resultado inconcluso no invalida el fixture.

## 11. OP-MO-04 — Preset `mioty_868_tsunb` como FSK propietario

### Pregunta experimental

¿Pueden dos dispositivos FeralRF actuales intercambiar paquetes raw con la configuración nominal del preset `mioty_868_tsunb`?

No se evalúan TS-UNB, telegram splitting, coding, scheduling ni interoperabilidad MIOTY.

### Comando

```powershell
$env:FERAL_LABEL = 'OP-MO-04_MIOTY_LABELED_GENERIC_FSK'
$env:FERAL_TX_PORT = $TxBridge
$env:FERAL_RX_PORT = $RxBridge
$env:FERAL_PRESET = 'mioty_868_tsunb'
$env:FERAL_POWER_DBM = [string]$TxPowerDbm
$env:FERAL_MARKER_PREFIX_HEX = '4D4F30344D3030'
$env:FERAL_COUNT = '10'
$env:FERAL_MIN_HITS = '9'
$env:FERAL_SPACING_SECONDS = '1.0'
$env:FERAL_OBSERVE_SECONDS = '5.0'
$env:FERAL_OOK_STAGE = '0'

$OtaCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-04_mioty_labeled_fsk.txt')
$MO04Exit = $LASTEXITCODE
"python_exit=$MO04Exit" | Tee-Object -Append -FilePath (Join-Path $LogDir 'OP-MO-04_mioty_labeled_fsk.txt')
```

El marcador es `4D4F30344D3030` (`MO04M00`) seguido por `01` a `0A`.

### Interpretación obligatoria

| Observación | Conclusión permitida |
|---|---|
| `CONFIG_ACCEPTED=2/2` | El plano de control aceptó los parámetros nominales. No prueba RF. |
| 10 `TX_ACK` | Firmware aceptó diez solicitudes `TX_RAW`. No es prueba de TX completado en aire. |
| Al menos 9 payloads únicos en el peer | Entrega raw FSK propietaria observada entre estas dos implementaciones y este montaje. |
| `RX_STOP_ACK=1` | Comando de limpieza aceptado. No es una medición del RF core. |
| Resultado raw positivo | No valida MIOTY TS-UNB ni interoperabilidad MIOTY. |
| Resultado raw negativo | No demuestra que CC1352P7 sea incapaz de soportar MIOTY; sólo falla esta configuración/backend/procedimiento. |

Para atribuir frecuencia, desviación, ancho ocupado o forma de onda se necesita instrumentación RF independiente. Para interoperabilidad se requiere una implementación MIOTY independiente y la pila TS-UNB ausente de FeralRF.

## 12. OP-MO-05 — Limpieza y reconfirmación del baseline

Si `OP-MO-04` informó cualquier fallo de `RX_STOP`, error asíncrono, pérdida de puerto o hang, no ejecutar un cambio de PHY en caliente. Retirar toda alimentación de ambas placas por 15 segundos, reconectar, reidentificar y repetir `OP-MO-02` antes de intentar el control.

Con estado limpio y puertos confirmados, ejecutar:

```powershell
$env:FERAL_LABEL = 'OP-MO-05_CONTROL_AFTER_SLOW_FSK'
$env:FERAL_TX_PORT = $TxBridge
$env:FERAL_RX_PORT = $RxBridge
$env:FERAL_PRESET = 'gfsk_868_50k'
$env:FERAL_POWER_DBM = [string]$TxPowerDbm
$env:FERAL_MARKER_PREFIX_HEX = '4D4F3035433030'
$env:FERAL_COUNT = '10'
$env:FERAL_MIN_HITS = '9'
$env:FERAL_SPACING_SECONDS = '0.25'
$env:FERAL_OBSERVE_SECONDS = '5.0'
$env:FERAL_OOK_STAGE = '0'

$OtaCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-05_control_after_slow_fsk.txt')
$MO05Exit = $LASTEXITCODE
"python_exit=$MO05Exit" | Tee-Object -Append -FilePath (Join-Path $LogDir 'OP-MO-05_control_after_slow_fsk.txt')
```

No continuar a OOK salvo que este control pase, se mantenga GO de seguridad y el operador autorice explícitamente `OP-MO-06`. La autorización debe registrarse en el formulario de resultados.

## 13. OP-MO-06 — Ensayo OOK 868 aislado

### Pregunta experimental

¿Pueden dos dispositivos FeralRF actuales intercambiar paquetes raw con `ook_868_4k8`, y puede restablecerse la operación normal después del power-cycle obligatorio?

El corpus histórico informa 10/10 entregas raw OOK a 868 MHz. Ese reporte es evidencia válida de la observación histórica; no demuestra que el firmware actualmente instalado tenga un ciclo de vida seguro. Este ensayo es una nueva ejecución independiente.

### Prerrequisitos adicionales

- PASS de `OP-MO-05`.
- Autorización explícita del operador para OOK.
- Medio inmediato para retirar toda alimentación.
- Ninguna otra operación RF prevista antes de `OP-MO-07`.

### Comando único de etapa OOK

```powershell
$env:FERAL_LABEL = 'OP-MO-06_OOK_868_ISOLATED'
$env:FERAL_TX_PORT = $TxBridge
$env:FERAL_RX_PORT = $RxBridge
$env:FERAL_PRESET = 'ook_868_4k8'
$env:FERAL_POWER_DBM = [string]$TxPowerDbm
$env:FERAL_MARKER_PREFIX_HEX = '4D4F30364F3030'
$env:FERAL_COUNT = '10'
$env:FERAL_MIN_HITS = '9'
$env:FERAL_SPACING_SECONDS = '0.25'
$env:FERAL_OBSERVE_SECONDS = '5.0'
$env:FERAL_OOK_STAGE = '1'

$OtaCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-06_ook_868.txt')
$MO06Exit = $LASTEXITCODE
"python_exit=$MO06Exit" | Tee-Object -Append -FilePath (Join-Path $LogDir 'OP-MO-06_ook_868.txt')
```

El marcador es `4D4F30364F3030` (`MO06O00`) seguido por `01` a `0A`.

### Criterio provisional

La etapa RF cumple su criterio provisional sólo con:

- diez solicitudes TX aceptadas;
- al menos nueve secuencias peer coincidentes y marcadas `crc_ok=True`;
- ningún error asíncrono inesperado;
- `RX_STOP` aceptado.

Incluso entonces, el resultado es **PARTIAL** hasta completar `OP-MO-07`: power-cycle total, nuevo preflight y control conocido. Estos umbrales son criterios propuestos para este ensayo, no requisitos de producto previamente aprobados.

No declarar PASS por el código de salida ni por ACKs aislados. Si hay suficientes marcadores pero el metadato CRC no los confirma, registrar **INCONCLUSIVE** para el criterio CRC y conservar separadamente la observación de payloads.

Al terminar el comando —haya pasado, fallado o quedado bloqueado— no ejecutar otra orden contra las placas antes de retirar alimentación.

## 14. OP-MO-07 — Recuperación OOK obligatoria

1. Cerrar los handles/procesos si siguen abiertos. Si el proceso está bloqueado, pulsar `Ctrl+C` una vez; si no responde, cerrar esa consola.
2. Desconectar físicamente **ambas** placas de USB, programadores y cualquier alimentación auxiliar.
3. Esperar al menos 10 segundos; usar 15 segundos.
4. Reconectar una placa a la vez.
5. Repetir los comandos de enumeración de `OP-MO-01`; no asumir que conservaron el COM.
6. Actualizar `$TxBridge`, `$TxShell`, `$RxBridge`, `$RxShell`.
7. Repetir `OP-MO-02` para ambas placas.
8. Con GO RF todavía vigente, volver a emitir `band2` en ambos Shell.
9. Ejecutar el control final:

```powershell
$env:FERAL_LABEL = 'OP-MO-07_CONTROL_AFTER_OOK_POWER_CYCLE'
$env:FERAL_TX_PORT = $TxBridge
$env:FERAL_RX_PORT = $RxBridge
$env:FERAL_PRESET = 'gfsk_868_50k'
$env:FERAL_POWER_DBM = [string]$TxPowerDbm
$env:FERAL_MARKER_PREFIX_HEX = '4D4F3037433030'
$env:FERAL_COUNT = '10'
$env:FERAL_MIN_HITS = '9'
$env:FERAL_SPACING_SECONDS = '0.25'
$env:FERAL_OBSERVE_SECONDS = '5.0'
$env:FERAL_OOK_STAGE = '0'

$OtaCode | python -u - 2>&1 | Tee-Object -FilePath (Join-Path $LogDir 'OP-MO-07_control_after_ook.txt')
$MO07Exit = $LASTEXITCODE
"python_exit=$MO07Exit" | Tee-Object -Append -FilePath (Join-Path $LogDir 'OP-MO-07_control_after_ook.txt')
```

Sólo si preflight y control final pasan puede cerrarse el ensayo OOK con PASS raw provisional. Si la recuperación falla, clasificar el ensayo OOK **FAIL** o **INCONCLUSIVE** según las observaciones conservadas y terminar la sesión; no intentar más RF.

## 15. Manejo de fallos

| Condición | Acción inmediata | Clasificación mínima |
|---|---|---|
| `RX_STOP` timeout/error antes de OOK | No cambiar PHY; cerrar proceso, power-cycle ambas placas, reidentificar, preflight y baseline. | BLOCKED o FAIL de cleanup. |
| `RX_STOP` timeout/error durante OOK | No enviar más comandos; power-cycle obligatorio. | OOK no puede ser PASS. |
| Pérdida de un puerto | Detener ambos procesos, no reasignar por número supuesto; power-cycle y reenumerar. | BLOCKED. |
| Cero peer markers con TX ACKs | Conservar ACKs como control aceptado; revisar fixture, ruta, temporización e instrumentación antes de repetir. | FAIL del criterio raw; causa INCONCLUSIVE. |
| 1–8 secuencias únicas | Conservar conteos y CRC; no redondear a éxito. | PARTIAL. |
| Marcadores suficientes pero CRC insuficiente | Conservar ambos hechos; no descartar la recepción ni afirmar CRC-valid. | INCONCLUSIVE para CRC. |
| Error asíncrono de firmware | Registrar `code` y `context`, detener y recuperar. | FAIL/BLOCKED según etapa. |
| Hang del proceso | `Ctrl+C` una vez; cerrar consola si no responde; power-cycle. | BLOCKED. |
| Fallo de preflight o baseline post-OOK | No reentrar en OOK ni continuar RF. | FAIL de recuperación; sesión terminada. |

No repetir indefinidamente. Una repetición sólo es defendible después de identificar y registrar una causa corregida; use un nuevo log y marcador.

## 16. Plantilla compacta de registro

Copiar esta tabla a una bitácora nueva; no actualizar automáticamente la matriz canónica.

| Campo | Valor |
|---|---|
| Fecha/hora y zona | |
| Operador | |
| Rama / commit / `git status` | |
| Firmware informado TX / RX | |
| Identificador y revisión HW TX / RX | |
| TX Bridge / Shell | |
| RX Bridge / Shell | |
| Antena o fixture; atenuación/aislamiento | |
| Base de autorización RF / entorno | |
| Operación `OP-MO-*` | |
| Preset e implementación exacta | |
| Frecuencia solicitada | |
| Potencia solicitada | |
| Marker prefix y rango de secuencias | |
| Paquetes solicitados | |
| TX ACKs | |
| Secuencias peer únicas | |
| Secuencias `crc_ok=True` | |
| RSSI/LQI observados | |
| Errores asíncronos | |
| `RX_STOP` / cleanup | |
| Power-cycle completo y duración | |
| Preflight post-recuperación | |
| Control RF post-recuperación | |
| Código de salida Python | |
| Clasificación: PASS / PARTIAL / FAIL / BLOCKED / NOT RUN / INCONCLUSIVE | |
| Interpretación permitida | |
| Límites: RF física / TS-UNB / interoperabilidad | |
| Ruta de los logs | |

Separar siempre las conclusiones:

- **Control/API:** comando aceptado.
- **Raw OTA:** payload coincidente observado en el peer.
- **CRC:** metadato registrado, sujeto a sus limitaciones conocidas.
- **Física RF:** no establecida sin instrumentación apropiada.
- **Protocolo/interoperabilidad:** no evaluados por estos ensayos.

## 17. Checklist de cierre

- [ ] Los presets usados coinciden con `PROP_PRESETS`; no se cambiaron sus parámetros RF.
- [ ] Todos los puertos se seleccionaron explícitamente.
- [ ] No se usó aritmética Cat-Shell ni `Radio.reset_device()`.
- [ ] Los logs conservan salida completa y el código real de Python.
- [ ] La temporización MIOTY-labeled permitió el airtime calculado.
- [ ] No se interpretó el preset FSK como TS-UNB.
- [ ] No se interpretaron ACKs como medición RF.
- [ ] OOK fue la última modalidad RF antes del power-cycle.
- [ ] Ambas placas quedaron sin ninguna alimentación por el intervalo documentado.
- [ ] Puertos, preflight y control se verificaron después de OOK.
- [ ] No se modificaron reportes históricos ni la matriz de validación.

