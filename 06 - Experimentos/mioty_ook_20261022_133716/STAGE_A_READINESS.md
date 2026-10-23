# Stage A — Readiness estático

**Inicio del registro:** 2026-10-22T13:37:16-06:00  
**Cierre de revisión estática:** 2026-10-22T13:38:04-06:00  
**Alcance:** inspección de archivos, revisión de dependencias y preparación de evidencia. No se enumeraron ni abrieron puertos serie y no se ejecutó RF.

## Fuente

- Repositorio: `C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF`
- Rama: `feature/hardware-validation`
- HEAD actual: `f2744e400f743df30499e20f03f156f08ad15bd8`
- Commit previamente auditado: `fbff1b7f5077964418fb63a3fdf346a27c35a003`
- Commit actual: `docs: Final technical documentation of Hardware Validation was added in docs.`
- Estado del árbol antes de crear este registro: limpio.
- Diferencia relevante de implementación entre ambos commits: ninguna en presets, API Python, runners revisados ni backend RF. El commit actual añade los ocho documentos bajo `docs/HARDWARE_VALIDATION_RESULTS/`.

## Ubicaciones reales consultadas

- Runbook: `Understanding-CatSniffer/06 - Experimentos/MIOTY_OOK_VALIDATION_RUNBOOK.md`
- Referencias: `FeralRF/docs/HARDWARE_VALIDATION_RESULTS/`

La ruta indicada originalmente como `FeralRF/docs/MIOTY_OOK_VALIDATION_RUNBOOK.md` ya no existe. El enlace relativo del runbook movido a `../python/feralrf/presets.py` tampoco resuelve desde su ubicación actual. Esto es una inconsistencia documental, no un bloqueo de ejecución: los comandos usan la ruta absoluta correcta de FeralRF y los presets se verificaron directamente en source.

## Entorno Python

- Python: `3.14.7`
- Ejecutable: `C:\Users\Support\AppData\Local\Programs\Python\Python314\python.exe`
- `feralrf`: `0.3.0`, importado desde el checkout actual.
- `pyserial`: `3.5`
- `pyserial-asyncio`: `0.6`
- `cobs`: `1.2.2`

Los imports requeridos terminaron correctamente; no se instaló ninguna dependencia. `pyproject.toml` exige Python `>=3.9`, pero sus classifiers sólo enumeran 3.9–3.12. Python 3.14 no está declarado como versión probada; el import estático pasa, pero la ejecución HIL seguirá siendo evidencia necesaria de compatibilidad efectiva.

## Presets verificados

```text
gfsk_868_50k    = frequency_hz 868000000, mod_type 1, symbol_rate 50000,
                   deviation 100, rx_bw 82/0x52, sync_word 0x930B51DE
mioty_868_tsunb = frequency_hz 868000000, mod_type 0, symbol_rate 396,
                   deviation 1, rx_bw 74/0x4A, sync_word 0x930B51DE
ook_868_4k8     = frequency_hz 868000000, mod_type 2, symbol_rate 4800,
                   deviation 0, rx_bw 76/0x4C, sync_word 0x930B51DE
```

## API y harness

Se verificaron las firmas actuales de:

- `Radio.init()`
- `Radio.set_phy(phy, channel=0, frequency_hz=0)`
- `Radio.configure_prop(frequency_hz, mod_type=1, symbol_rate=50000, deviation=100, rx_bw=82, sync_word=0x930B51DE, format_conf=0)`
- `Radio.set_power(power_dbm)`
- `Radio.start_rx()` / `Radio.stop_rx(retries=3, timeout=1.0)`
- `Radio.read_packets(timeout=1.0)`
- `Radio.transmit(packet, power_dbm=-128, timeout=5.0)`
- `Radio.get_stats(timeout=2.0)` / `Radio.disconnect()`

Los cuatro bloques Python embebidos en el runbook (`ShellQueryCode`, `PreflightCode`, `Band2Code` y `OtaCode`) pasaron `ast.parse()`. No fueron ejecutados.

## Riesgos y ajustes de supervisión

1. El criterio original del harness exige al menos nueve marcadores con `crc_ok=True`. Debido al defecto conocido del metadata proprietary, se reportarán por separado el resultado literal del runbook y la observación estrecha de payloads/secuencias coincidentes.
2. `TX_RAW ACK` acredita aceptación, no finalización física ni radiación.
3. `SET_PROP_CONFIG ACK` no incluye readback ni resultado del RF core.
4. Existen antecedentes de `RX_STOP` timeout y descarte host-visible de eventos asíncronos; cualquier error obliga a detener la etapa.
5. `Radio.reset_device()` y los runners que infieren Shell como `Bridge + 2` quedan excluidos.
6. OOK carga patches `genook`; no se cambiará de PHY después y se exigirá power-cycle físico completo.
7. La identificación del binario instalado probablemente quedará limitada a `DeviceInfo` genérico salvo evidencia externa adicional.
8. El runbook propone logs en `%TEMP%`; para esta campaña se mantendrá `$LogDir` en este directorio de evidencia, sin modificar el runbook.

## Comandos estáticos ejecutados

```powershell
git -c safe.directory=<repo> -C FeralRF branch --show-current
git -c safe.directory=<repo> -C FeralRF rev-parse HEAD
git -c safe.directory=<repo> -C FeralRF status --short
git -c safe.directory=<repo> -C FeralRF diff fbff1b7f...HEAD -- <archivos RF seleccionados>
python --version
python -c "import feralrf, serial, serial_asyncio, cobs; ..."
python -c "import inspect; from feralrf import Radio; ..."
python -c "from feralrf import PROP_PRESETS; ..."
python -c "import ast,sys; ast.parse(sys.stdin.read())"
```

No se ejecutaron `Get-CimInstance Win32_SerialPort`, `serial.tools.list_ports`, Cat-Shell, `Radio.init()`, `GET_INFO`, `GET_STATS`, RX ni TX.

## Estado de Stage A

**READY para solicitar Stage B**, condicionado a aprobación explícita del operador. Stage B será sólo identificación y preflight sin RF TX.
