# Reporte complementario — OP-05: verificación explícita del código de rechazo TX

**Proyecto:** CatSniffer — FeralRF
**Documento:** `06 - Procedimientos IEEE, TX y modos de prueba`
**Procedimiento:** OP-05 — Rechazo de TX durante RX
**Estado experimental:** PASS
**Tipo de corrección:** Documentación y script de reproducción.

## 1. Problema identificado

El procedimiento original captura la excepción generada al solicitar TX durante RX mediante:

```python
except CommandError as exc:
    print("TX REJECTED:", repr(exc))
```

La salida observada fue:

```text
TX REJECTED: CommandError('Transmit failed')
```

Aunque demuestra que la transmisión fue rechazada, no permite comprobar explícitamente que el código corresponda al valor `0x05`, esperado por OP-05.

Esto representa una limitación del procedimiento documentado, no un defecto demostrado del firmware.

## 2. Verificación de la implementación

La inspección de `feralrf.exceptions.CommandError` mostró:

```python
class CommandError(FeralRFError):
    """Command execution error"""

    def __init__(self, message: str, error_code: int = 0):
        super().__init__(message)
        self.error_code = error_code
```

Por tanto, el código de rechazo puede consultarse mediante:

```python
exc.error_code
```

## 3. Corrección propuesta

Sustituir el bloque de captura por:

```python
except CommandError as exc:
    print("TX REJECTED:", str(exc))
    print(f"ERROR CODE: 0x{exc.error_code:02X}")

    if exc.error_code != 0x05:
        raise RuntimeError(
            f"Unexpected TX rejection code: 0x{exc.error_code:02X}"
        )

    print("PASS: TX rejected with expected error code 0x05")
```

La corrección debe conservar el rechazo esperado como resultado satisfactorio, pero fallar cuando el código sea distinto.

Adicionalmente, Codex debe revisar el manejo de `finally` para garantizar que un fallo de `RX_STOP` no quede oculto por una impresión informativa. Debe conservarse la desconexión del dispositivo incluso cuando falle la limpieza.

## 4. Evidencia experimental

**Configuración:**

| Parámetro              | Valor         |
| ---------------------- | ------------- |
| Cat-Bridge             | COM88         |
| PHY                    | IEEE 802.15.4 |
| Canal                  | 25            |
| Frecuencia nominal     | 2475 MHz      |
| Potencia TX solicitada | −20 dBm       |
| Firmware reportado     | `1.0.0`       |

**Salida obtenida tras la corrección:**

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
PHY: IEEE_802_15_4, CHANNEL: 25
RX: STARTED
TX REJECTED: Transmit failed
ERROR CODE: 0x05
PASS: TX rechazado con el codigo esperado 0x05
RX: STOPPED
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

## 5. Resultado y solicitud para Codex

**Resultado: OP-05 — PASS.**

Se verificó que:

1. La sesión RX inició correctamente.
2. La solicitud TX fue rechazada.
3. El código reportado por la API fue `0x05`.
4. `RX_STOP` terminó correctamente.
5. `GET_STATS` devolvió una respuesta válida.

**Acción solicitada a Codex:** actualizar OP-05 del documento 06 para mostrar y validar explícitamente `CommandError.error_code`, conservar las condiciones de limpieza y distinguir el rechazo esperado de un fallo inesperado.

No se solicita modificar el firmware FeralRF, ya que no se ha demostrado un defecto funcional.
