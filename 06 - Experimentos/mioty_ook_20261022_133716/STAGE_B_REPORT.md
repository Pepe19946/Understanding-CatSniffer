# Stage B — Identificación y preflight

## Resultado

**PASS**, limitado al plano de control y a la asociación USB lógica. No se ejecutó RX ni TX.

## Asociación

| Unidad | Identidad USB | Bridge | LoRa no usado | Shell |
|---|---|---:|---:|---:|
| A | `E661A89753565932` / PNP `1E70E7AF` | COM33 | COM34 | COM35 |
| B | `E661A89753656A35` / PNP `106D987F` | COM88 | COM86 | COM87 |

La asociación se estableció mediante identidad USB y `MI_00`/`MI_02`/`MI_04`, no por aritmética de números COM. Las designaciones A/B identifican inequívocamente los conjuntos USB; la etiqueta física externa de cada placa debe confirmarse por el operador antes de Stage C.

## Observaciones

- Ambos RP2040 informaron `v3.1.0.1`, `feralrf_cc1352 (custom)` y contadores `uart_overrun=0`, `ring_dropped=0`.
- Ambos Bridge respondieron a `RADIO_INIT`, `GET_INFO` y `GET_STATS` con código de proceso 0.
- Ambos CC1352 informaron firmware `1.0.0`, capabilities `0x07` y serial lógico `FERALRF1`.
- Todas las estadísticas iniciales fueron cero.
- El hash del binario CC1352 instalado no puede determinarse con esos campos.
- `Band: 0, Radio: LoRa` se conserva como estado lógico inicial. No se emitió `band2`.

## Comandos ejecutados

```powershell
Get-CimInstance Win32_SerialPort |
  Sort-Object DeviceID |
  Select-Object DeviceID, Name, PNPDeviceID |
  Format-List

python -m serial.tools.list_ports -v
```

Después se abrió cada Shell explícito a 115200, se envió únicamente `status` y se cerró. Cada Bridge explícito se abrió a 921600 para `Radio.init()` y `Radio.get_stats()`, seguido por `Radio.disconnect()`.

## Seguridad de continuación

Stage B permite preparar Stage C, pero no autoriza RF. Antes de OP-MO-03 aún deben confirmarse:

1. Correspondencia física de Unidad A y Unidad B y roles TX/RX.
2. Revisión de hardware y ruta 868 MHz.
3. Antenas adecuadas o fixture conducido verificado.
4. Autorización del entorno y restricciones aplicables.
5. Potencia solicitada.
6. Capacidad de retirar por completo alimentación de ambas placas.

