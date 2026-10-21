# Observaciones experimentales — OP-08 y preparación de OP-09

**Proyecto:** CatSniffer — FeralRF
**Documento de referencia:** `06 - Procedimientos IEEE, TX y modos de prueba`
**Fecha:** 21 de octubre de 2026
**Dispositivos:** CatSniffer #2 (TX, COM33) y CatSniffer #3 (RX, COM88)
**Directorio de ejecución:** `FeralRF/python`

## 1. OP-08 — IEEE 802.15.4 BURST OTA

**Estado:** PARTIAL — limitación histórica reproducida.

### 1.1. Condiciones experimentales

| Parámetro                      | Valor               |
| ------------------------------ | ------------------- |
| PHY                            | IEEE 802.15.4 (`4`) |
| Canal                          | 25                  |
| Frecuencia nominal             | 2475 MHz            |
| Potencia TX solicitada         | 0 dBm               |
| Marcador                       | `A30133008803`      |
| Cantidad solicitada            | 5                   |
| Intervalo solicitado           | 250 000 µs          |
| Ventana RX                     | 12 segundos         |
| Firmware reportado por FeralRF | `1.0.0`             |

### 1.2. Control negativo

Comando ejecutado:

```powershell
python .\examples\lab\ota_rx_probe.py --port COM88 --phy 4 --channel 25 --duration 5 --marker-hex A30133008803 --match-mode contains --min-hits 0 --print-limit 30
```

Resultado:

```text
packets_total=4 crc_ok=4
marker_hits=0
RX_STOP ACK
RX PROBE PASS hits=0 min_hits=0
```

**Interpretación:** durante la ventana de control no se observó el marcador de OP-08. El resultado PASS del script no sustituye la comprobación explícita de `marker_hits=0`, ya que se configuró `--min-hits 0`.

### 1.3. Transmisión BURST

Comando ejecutado:

```powershell
python .\examples\lab\ota_tx_burst.py --port COM33 --phy 4 --channel 25 --power 0 --payload-hex A30133008803 --count 5 --interval-us 250000 --tx-timeout 2
```

Resultado:

```text
[ OK ] Config ACK
[STEP] TX burst

[ OK ] TX BURST PASS scheduled=5
```

**Interpretación:** el script confirmó que la solicitud BURST fue aceptada y reportó cinco transmisiones programadas. `scheduled=5` no demuestra que los cinco paquetes hayan sido físicamente emitidos.

### 1.4. Recepción OTA

Comando ejecutado:

```powershell
python .\examples\lab\ota_rx_probe.py --port COM88 --phy 4 --channel 25 --duration 12 --marker-hex A30133008803 --match-mode contains --min-hits 5 --print-limit 80
```

Resultado:

```text
packets_total=18 crc_ok=18

[HIT] ts=539107707us ch=25 rssi=-32 crc_ok=True len=8 data=a30133008803539e

marker_hits=1
RX_STOP ACK

[FAIL] RX PROBE FAIL hits=1 min_hits=5
```

**Interpretación:** el receptor independiente observó un paquete con el marcador esperado y CRC válido reportado, pero no alcanzó las cinco coincidencias requeridas.

### 1.5. Comparación histórica

La ejecución actual reproduce el resultado histórico de OP-08:

| Indicador           | Histórico | Actual  |
| ------------------- | --------- | ------- |
| Cantidad programada | 5         | 5       |
| Coincidencias RX    | 1         | 1       |
| Resultado OTA       | Parcial   | Parcial |

La coincidencia justifica investigar la implementación de `TX_BURST`, pero no demuestra todavía que el firmware transmita físicamente un único paquete.

### 1.6. Conclusión

**OP-08 permanece PARTIAL.**

Se demostró recepción OTA de al menos un paquete, pero no se validó la entrega de los cinco paquetes solicitados.

La causa de la discrepancia permanece abierta. Entre las hipótesis están la ejecución incompleta de la ráfaga, la gestión del estado RF, la temporización y la pérdida de recepciones.

No se recomienda modificar el criterio `--min-hits 5` ni cambiar el firmware sin investigar primero el recorrido de implementación.

## 2. Ajustes solicitados a Codex

1. Incorporar la nueva ejecución de OP-08 al registro experimental, conservando el antecedente histórico.
2. Diferenciar explícitamente `scheduled=5` de cinco transmisiones RF verificadas.
3. Registrar `marker_hits=1` y el resultado FAIL del receptor.
4. Mantener OP-08 como PARTIAL, sin presentar el ACK de BURST como validación OTA completa.
5. Conservar el control negativo y sus resultados.
6. Añadir como investigación pendiente el recorrido de `TX_BURST` desde la API Python hasta el firmware CC1352P7.
7. Mantener los comandos estandarizados para ejecutarse desde `FeralRF/python`, sin el prefijo `.\python\`.

## 3. OP-09 — Verificación inicial de TX CONTINUOUS

**Estado:** Preparación completada; prueba RF pendiente.

Se ejecutó:

```powershell
python .\examples\smoke_tx_continuous_phase1.py --help
```

La interfaz del script reconoce los argumentos necesarios:

* `--port`
* `--phy`
* `--channel`
* `--power`
* `--packet-hex`
* `--interval-us`
* `--run-seconds`
* `--tx-timeout`

El argumento `--run-seconds` indica cuánto tiempo debe mantenerse activo el modo continuo antes de solicitar `TX_STOP`.

**Resultado:** la interfaz CLI es compatible con la preparación de OP-09. Esta comprobación no constituye una validación RF ni demuestra todavía el comportamiento de `TX_STOP`.

**Siguiente acción:** ejecutar OP-09 respetando los parámetros del manual y modificando únicamente las rutas relativas necesarias para trabajar desde `FeralRF/python`.
