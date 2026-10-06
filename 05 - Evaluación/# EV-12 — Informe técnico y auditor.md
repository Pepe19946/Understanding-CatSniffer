# EV-12 — Informe técnico y auditoría de TX RAW / FRAME / BURST / CONTINUOUS en FeralRF

## 1. Identificación de la validación

**Proyecto:** CatSniffer - FeralRF
**EV:** EV-12 — TX raw/frame/burst/continuous y stop
**Fecha de campaña:** 6 de octubre de 2026
**Repositorio evaluado:** `FeralRF`
**Rama:** `main`
**HEAD de referencia:** `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`
**Firmware reportado por la API:** `1.0.0`
**PHY empleado:** `PHY 4 = IEEE 802.15.4`
**Canal:** 25
**Potencia solicitada:** `0 dBm`

La Wiki técnica identifica este `HEAD` como la línea base examinada y registra además que el submódulo TI estaba modificado respecto a su referencia; por ello, el commit identifica el código FeralRF bajo análisis pero no constituye por sí solo un hash reproducible del binario efectivamente flasheado.

### 1.1 Hardware utilizado

Después del reemplazo de la segunda CatSniffer, el inventario experimental vigente quedó:

| Rol           | Dispositivo   | Hardware               | Cat-Bridge | Cat-LoRa | Cat-Shell |
| ------------- | ------------- | ---------------------- | ---------- | -------- | --------- |
| DUT / TX      | CatSniffer #1 | v3 — RP2040 + CC1352P7 | **COM33**  | COM34    | COM35     |
| Observer / RX | CatSniffer #2 | v3 — RP2040 + CC1352P7 | **COM88**  | COM86    | COM87     |

En esta campaña:

* `COM33` fue el endpoint FeralRF del transmisor bajo prueba.
* `COM88` fue utilizado como receptor OTA independiente a nivel de placa, aunque ejecutando la misma implementación FeralRF.

El RP2040 no ejecuta FeralRF. En runtime actúa principalmente como interfaz USB CDC y passthrough hacia el CC1352P7; FeralRF se ejecuta en el Cortex-M4F del CC1352P7 y utiliza su RF Core mediante TI RF Driver.

---

# 2. Objetivo de EV-12

La guía define EV-12 para comprobar:

* `TX_RAW`;
* `TX_FRAME`;
* `TX_BURST`;
* `TX_CONTINUOUS`;
* `TX_STOP`;

primero a nivel de aceptación/control y, cuando existe observador, a nivel RF/OTA. La propia guía advierte que el ACK no equivale a `TX_DONE` y que PASS-RF requiere observación física.

La matriz original describía EV-12 como una validación estable con evidencia esperada `ACK + captura`, señalando ya la ambigüedad de los PASS históricos exclusivamente basados en scripts.

Por tanto, el criterio adoptado fue separar explícitamente:

**PASS-control:** el comando es aceptado por FeralRF y devuelve ACK.

**PASS-OTA funcional:** un segundo receptor observa por aire el marcador enviado.

**PASS cuantitativo / temporal:** el comportamiento repetitivo solicitado —por ejemplo cinco o cuarenta transmisiones— se corresponde con múltiples emisiones observables.

**PASS instrumentado:** potencia, frecuencia, espectro y temporización medidos con instrumentación RF independiente. Este nivel no forma parte de la evidencia obtenida en EV-12.

---

# 3. Arquitectura relevante para interpretar los resultados

La ruta de esta validación fue:

```text
script Python
    ↓
feralrf.Radio
    ↓
protocolo host FeralRF
COBS + CRC16
    ↓
USB CDC / Cat-Bridge
    ↓
RP2040
passthrough USB ↔ UART
    ↓
UART0 921600 8N1
    ↓
CC1352P7 / FeralRF
HostIFTask
    ↓
CommandProcessor
    ↓
ControlTask / DataTask
    ↓
RadioIF
    ↓
TI RF Driver
    ↓
RF Core CC1352P7
    ↓
IEEE 802.15.4 PHY
canal 25
    ↓
aire
    ↓
CC1352P7 observer
    ↓
COM88 / Python
```

Esta ruta está documentada en la Wiki técnica.

Las interfaces concretas son:

* PC → RP2040: USB CDC.
* RP2040 → CC1352P7: UART 921600 8N1.
* Host FeralRF: frames COBS terminados en `0x00`, con CRC.
* CC1352P7 → RF Core: operaciones TI `RF_Op` / estructuras SmartRF.

Los IDs TX relevantes son:

```text
0x20 CMD_TX_RAW
0x21 CMD_TX_CONTINUOUS
0x22 CMD_TX_BURST
0x23 CMD_TX_FRAME
0x24 CMD_TX_STOP
```

y `0x90` corresponde a `RSP_RX_PACKET`.

---

# 4. Contexto de protocolo RF

En esta EV se utilizó:

```text
PHY 4
IEEE 802.15.4
canal 25
2.4 GHz
```

Esto **no significa que se estuviera ejecutando un stack Zigbee completo**.

IEEE 802.15.4 proporciona la base PHY/MAC sobre la que pueden construirse protocolos superiores. FeralRF permite RX/TX crudo y acceso a funciones de radio, pero actualmente no implementa stacks completos de Zigbee, Thread, Matter, Wi-SUN, Wireless M-Bus o Sidewalk.

Por tanto, para EV-12 la formulación correcta es:

> Se validó transmisión de bytes controlados sobre el backend IEEE 802.15.4 de FeralRF; no se validó una sesión Zigbee ni una pila Zigbee completa.

---

# 5. Uso de markers en la observación OTA

`ota_rx_probe.py` utiliza un patrón de bytes conocido para diferenciar los paquetes generados deliberadamente de tráfico ambiental.

Por ejemplo:

```text
--marker-hex DEADBEEF
```

no configura un canal RF.

El canal es:

```text
--channel 25
```

El marker es únicamente una firma buscada dentro de los bytes recibidos.

De este modo:

```text
packets_total=30
marker_hits=1
```

significa:

* treinta paquetes fueron recibidos durante la ventana;
* uno contenía el patrón controlado.

Esto resulta necesario porque se observó tráfico IEEE 802.15.4 ambiental incluso sin transmisión deliberada.

---

# 6. Ejecución experimental

## 6.1 TX_RAW — validación de control

### Comando

```powershell
python .\examples\smoke_tx_phase1.py --port COM33 --phy 4 --channel 25 --power 0 --packet-hex DEADBEEF
```

### Acción

Configura:

```text
DUT          COM33
PHY          IEEE 802.15.4
canal        25
power req.   0 dBm
payload      DE AD BE EF
```

y solicita `TX_RAW`.

### Salida obtenida

```text
FeralRF TX Raw Smoke Test (Phase 1)
====================================
port=COM33 baudrate=921600 phy=4 channel=25 power=0 len=4

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK

[ OK ] TX SMOKE PASS
```

### Análisis

**PASS-control.**

El comando fue aceptado por FeralRF.

El ACK no constituye por sí solo evidencia de transmisión física, ya que los comandos TX son aceptados a nivel de control antes de que pueda establecerse la finalización RF. Esta limitación de observabilidad ya estaba documentada como KI-14.

---

# 6.2 TX_RAW — control negativo ambiental

Antes de ejecutar el TX sincronizado se dejó al observer COM88 escuchando `DEADBEEF`.

### Comando

```powershell
python .\examples\lab\ota_rx_probe.py --port COM88 --phy 4 --channel 25 --duration 15 --marker-hex DEADBEEF --min-hits 1
```

### Salida

```text
FeralRF OTA RX Probe
====================
port=COM88 baudrate=921600 phy=4 channel=25 duration=15.0s min_hits=1 marker=deadbeef match_mode=contains allow_crc_fail=False

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL
[ OK ] Config ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets_total=23 crc_ok=23
[ OK ] marker_hits=0
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[FAIL] RX PROBE FAIL hits=0 min_hits=1
```

### Análisis

El `FAIL` era esperado porque durante esa ventana no se ejecutó el DUT.

Este resultado constituyó un **control negativo útil**:

```text
23 paquetes ambientales
23 CRC-valid
0 contenían DEADBEEF
```

Por tanto, el marker no estaba apareciendo espontáneamente en la ventana de prueba.

---

# 6.3 TX_RAW — validación OTA sincronizada

### RX observado

```text
[ OK ] packets_total=23 crc_ok=23
[HIT] ts=76657725us ch=25 rssi=-52 crc_ok=True len=6 data=deadbeef1519
[ OK ] marker_hits=1
...
[ OK ] RX PROBE PASS hits=1 min_hits=1
```

### TX sincronizado

```text
FeralRF TX Raw Smoke Test (Phase 1)
====================================
port=COM33 baudrate=921600 phy=4 channel=25 power=0 len=4

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK

[ OK ] TX SMOKE PASS
```

### Resultado

```text
TX_RAW:
PASS-control
PASS-OTA funcional
```

El observer recibió un paquete CRC-válido que contenía el marcador solicitado.

### Límite

No se demostró:

* potencia física exacta de 0 dBm;
* frecuencia calibrada;
* máscara espectral;
* conformidad instrumental IEEE 802.15.4.

Los dos bytes adicionales observados:

```text
15 19
```

no se interpretaron durante EV-12. Debe evitarse afirmar sin análisis específico que sean exactamente FCS/CRC.

---

# 7. TX_FRAME

## 7.1 Primera ejecución OTA inválida por marker no coincidente

Se produjo una ejecución donde:

```text
RX buscaba: DEADBEEF
TX enviaba: A1B2C3D4
```

El observer terminó:

```text
marker_hits=0
```

Este resultado **no se clasificó como fallo RF**, porque TX y RX estaban buscando datos diferentes.

El TX sí produjo:

```text
[ OK ] TX_FRAME ACK
[ OK ] TX FRAME SMOKE PASS
```

por lo que esa ejecución acreditó exclusivamente **PASS-control**.

---

# 7.2 TX_FRAME — ejecución corregida

### Receptor

```powershell
python .\examples\lab\ota_rx_probe.py --port COM88 --phy 4 --channel 25 --duration 15 --marker-hex A1B2C3D4 --min-hits 1
```

### Resultado

```text
[ OK ] packets_total=30 crc_ok=30
[HIT] ts=984904095us ch=25 rssi=-63 crc_ok=True len=6 data=a1b2c3d417f1
[ OK ] marker_hits=1
...
[ OK ] RX PROBE PASS hits=1 min_hits=1
```

El DUT había ejecutado:

```powershell
python .\examples\smoke_tx_frame_phase1.py --port COM33 --phy 4 --channel 25 --power 0 --frame-hex A1B2C3D4
```

### Resultado

```text
TX_FRAME:
PASS-control
PASS-OTA funcional
```

## 7.3 Auditoría de código de TX_FRAME

El nombre `TX_FRAME` puede inducir a pensar que construye una trama Zigbee o IEEE 802.15.4 de nivel superior.

El código actual no implementa ese comportamiento.

`ControlTask_onTxFrame()` termina delegando en:

```c
ControlTask_onTxRaw(...)
```

utilizando la potencia almacenada.

Por tanto, `TX_FRAME` no debe presentarse como constructor de una pila Zigbee o como generador de una trama de protocolo superior. La diferencia respecto de RAW está principalmente en la representación del comando host y manejo de potencia.

---

# 8. TX_BURST

## 8.1 Prueba inicial — count 40 / 25 ms

### TX

```powershell
python .\examples\lab\ota_tx_burst.py --port COM33 --phy 4 --channel 25 --power 0 --payload-hex C0FFEE01 --count 40 --interval-us 25000
```

### Salida TX

```text
FeralRF OTA TX Burst
====================
port=COM33 baudrate=921600 phy=4 channel=25 power=0 len=4 count=40 interval_us=25000 tx_timeout=5.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX burst

[ OK ] TX BURST PASS scheduled=40
```

### RX — criterio funcional mínimo

```text
[ OK ] packets_total=29 crc_ok=29
[HIT] ts=858942589us ch=25 rssi=-62 crc_ok=True len=6 data=c0ffee012a9f
[ OK ] marker_hits=1
...
[ OK ] RX PROBE PASS hits=1 min_hits=1
```

### RX — criterio exploratorio `min-hits=40`

```text
[ OK ] packets_total=24 crc_ok=24
[HIT] ts=898583490us ch=25 rssi=-63 crc_ok=True len=6 data=c0ffee012a9f
[ OK ] marker_hits=1
...
[FAIL] RX PROBE FAIL hits=1 min_hits=40
```

### Interpretación

`scheduled=40` **no significa 40 emisiones físicas**.

Según la implementación, `TX_BURST` almacena:

* payload;
* contador;
* intervalo;
* potencia;
* siguiente vencimiento;

y devuelve ACK indicando que la operación fue programada. Cada `DataTask_poll()` procesa como máximo una iteración vencida.

Por ello no es correcto describir la observación como:

```text
39 paquetes perdidos
```

La evidencia solamente establece:

```text
requested/scheduled = 40
OTA observed        = 1
```

---

# 8.2 BURST — reducción a 5 transmisiones y 250 ms

Se modificó la prueba para separar posibles problemas de tasa.

### TX

```powershell
python .\examples\lab\ota_tx_burst.py --port COM33 --phy 4 --channel 25 --power 0 --payload-hex C0FFEE02 --count 5 --interval-us 250000
```

### Salida

```text
FeralRF OTA TX Burst
====================
port=COM33 baudrate=921600 phy=4 channel=25 power=0 len=4 count=5 interval_us=250000 tx_timeout=5.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX burst

[ OK ] TX BURST PASS scheduled=5
```

### Observer

```text
[ OK ] packets_total=26 crc_ok=26
[HIT] ts=48163833us ch=25 rssi=-63 crc_ok=True len=6 data=c0ffee02b1ad
[ OK ] marker_hits=1
```

Una segunda ejecución con:

```text
--min-hits 5
```

volvió a obtener:

```text
marker_hits=1
```

y el helper terminó:

```text
[FAIL] RX PROBE FAIL hits=1 min_hits=5
```

### Resultado

Aumentar el intervalo de:

```text
25 ms
```

a:

```text
250 ms
```

y reducir count de 40 a 5 no modificó el patrón observado.

---

# 8.3 BURST — prueba de lifecycle del host

Se planteó la hipótesis de que `ota_tx_burst.py` pudiera cerrar la conexión inmediatamente después del ACK antes de completarse el burst.

Para eliminar esa variable se utilizó un harness temporal:

```powershell
python -c "import time; from feralrf import Radio,PHY; r=Radio(port='COM33',baudrate=921600); info=r.init(); r.set_phy(PHY(4),25); r.set_channel(25); r.set_power(0); print('INIT:',info); print('TX_BURST count=5 interval_us=250000'); r.transmit_burst(bytes.fromhex('C0FFEE03'),count=5,interval_us=250000,timeout=5.0); print('ACK received; keeping host connected for 3 s'); time.sleep(3); print('Disconnect'); r.disconnect()"
```

### Salida

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
TX_BURST count=5 interval_us=250000
ACK received; keeping host connected for 3 s
Disconnect
```

### RX

```text
[ OK ] packets_total=25 crc_ok=25
[HIT] ts=452131075us ch=25 rssi=-63 crc_ok=True len=6 data=c0ffee0338bc
[ OK ] marker_hits=1
...
[FAIL] RX PROBE FAIL hits=1 min_hits=5
```

### Conclusión de esta prueba

La hipótesis:

> “El helper Python termina demasiado pronto y por eso sólo sale una transmisión.”

queda fuertemente debilitada.

Incluso manteniendo la conexión activa durante 3 s —más que suficiente frente a los ~1.25 s nominales del burst— sólo se observó un marker.

---

# 9. TX_CONTINUOUS

`TX_CONTINUOUS` no significa Continuous Wave.

Es una repetición programada de paquetes hasta `TX_STOP` o fallo. La Wiki especifica que con `interval_us=0` intenta transmitir una vez por ciclo de polling; CW pertenece a otra función.

---

# 9.1 CONTINUOUS — intervalo 250 ms

### RX

```powershell
python .\examples\lab\ota_rx_probe.py --port COM88 --phy 4 --channel 25 --duration 15 --marker-hex C0FFEE04 --min-hits 1
```

### TX

```powershell
python .\examples\smoke_tx_continuous_phase1.py --port COM33 --phy 4 --channel 25 --power 0 --packet-hex C0FFEE04 --interval-us 250000 --run-seconds 1
```

### TX obtenido

```text
FeralRF TX Continuous Smoke Test (Phase 1)
===========================================
port=COM33 baudrate=921600 phy=4 channel=25 power=0 len=4 interval_us=250000 run_seconds=1.0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_CONTINUOUS
[ OK ] TX_CONTINUOUS ACK
[STEP] TX_STOP
[ OK ] TX_STOP ACK

[ OK ] TX CONTINUOUS SMOKE PASS
```

### RX

```text
[ OK ] packets_total=21 crc_ok=21
[HIT] ts=812149556us ch=25 rssi=-65 crc_ok=True len=6 data=c0ffee0487c8
[ OK ] marker_hits=1
...
[ OK ] RX PROBE PASS hits=1 min_hits=1
```

### Resultado

```text
PASS-control
PASS-OTA mínimo
```

pero sólo se observó una transmisión aunque la operación debía permanecer activa durante un segundo.

---

# 9.2 CONTINUOUS — `interval_us=0`

Se eliminó la espera temporal programada para determinar si la ruta RF podía realmente repetir transmisiones.

### TX

```powershell
python .\examples\smoke_tx_continuous_phase1.py --port COM33 --phy 4 --channel 25 --power 0 --packet-hex C0FFEE05 --interval-us 0 --run-seconds 1
```

### TX obtenido

```text
FeralRF TX Continuous Smoke Test (Phase 1)
===========================================
port=COM33 baudrate=921600 phy=4 channel=25 power=0 len=4 interval_us=0 run_seconds=1.0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_CONTINUOUS
[ OK ] TX_CONTINUOUS ACK
[STEP] TX_STOP
[ OK ] TX_STOP ACK

[ OK ] TX CONTINUOUS SMOKE PASS
```

### RX obtenido

```text
FeralRF OTA RX Probe
====================
port=COM88 baudrate=921600 phy=4 channel=25 duration=15.0s min_hits=2 marker=c0ffee05 match_mode=contains allow_crc_fail=False

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL
[ OK ] Config ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets_total=102 crc_ok=102
[HIT] ts=44636903us ch=25 rssi=-64 crc_ok=True len=6 data=c0ffee050ed9
[HIT] ts=44637948us ch=25 rssi=-64 crc_ok=True len=6 data=c0ffee050ed9
[HIT] ts=44638985us ch=25 rssi=-64 crc_ok=True len=6 data=c0ffee050ed9
[HIT] ts=44640010us ch=25 rssi=-64 crc_ok=True len=6 data=c0ffee050ed9
[HIT] ts=44641050us ch=25 rssi=-64 crc_ok=True len=6 data=c0ffee050ed9
[ OK ] marker_hits=99
[STEP] RX_STOP
[FAIL] Timeout waiting for response: Response timeout (ignored 9 unexpected response(s), last=0x90)
```

### Resultado

Este resultado fue decisivo.

```text
interval_us = 0
marker_hits = 99
```

demuestra que:

> La ruta RF es capaz de ejecutar transmisiones repetidas del mismo payload.

Por tanto, quedó fuertemente debilitada la hipótesis previa de que la segunda llamada a `RadioIF_transmitRaw()` necesariamente fallara.

---

# 9.3 CONTINUOUS — frontera `interval_us=1`

Se utilizó un intervalo positivo mínimo:

```text
1 µs
```

### RX

```powershell
python .\examples\lab\ota_rx_probe.py --port COM88 --phy 4 --channel 25 --duration 15 --marker-hex C0FFEE06 --min-hits 2
```

### TX

```powershell
python .\examples\smoke_tx_continuous_phase1.py --port COM33 --phy 4 --channel 25 --power 0 --packet-hex C0FFEE06 --interval-us 1 --run-seconds 1
```

### RX preservado

```text
FeralRF OTA RX Probe
====================
port=COM88 baudrate=921600 phy=4 channel=25 duration=15.0s min_hits=2 marker=c0ffee06 match_mode=contains allow_crc_fail=False

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL
[ OK ] Config ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets_total=24 crc_ok=24
[HIT] ts=617657554us ch=25 rssi=-59 crc_ok=True len=6 data=c0ffee0695eb
[ OK ] marker_hits=1
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[FAIL] RX PROBE FAIL hits=1 min_hits=2
```

### TX

```text
FeralRF TX Continuous Smoke Test (Phase 1)
===========================================
port=COM33 baudrate=921600 phy=4 channel=25 power=0 len=4 interval_us=1 run_seconds=1.0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_CONTINUOUS
[ OK ] TX_CONTINUOUS ACK
[STEP] TX_STOP
[ OK ] TX_STOP ACK

[ OK ] TX CONTINUOUS SMOKE PASS
```

Esta prueba fue repetida tres veces por el operador y las tres finalizaron con el mismo comportamiento general:

```text
interval_us=1
marker_hits=1
```

Sólo una de las tres salidas completas quedó preservada literalmente en este registro, por lo que no deben fabricarse logs para las otras dos ejecuciones.

---

# 10. Hallazgo principal de EV-12

El conjunto experimental presenta una frontera muy clara:

```text
TX_BURST count=40 interval=25 000 us
→ 1 marker

TX_BURST count=5 interval=250 000 us
→ 1 marker

TX_BURST count=5 interval=250 000 us
+ host conectado 3 s
→ 1 marker

TX_CONTINUOUS interval=250 000 us
→ 1 marker

TX_CONTINUOUS interval=1 us
→ 1 marker
→ reproducido 3/3

TX_CONTINUOUS interval=0 us
→ 99 markers
```

## Conclusión experimental

**La repetición RF funciona cuando no existe una espera temporal positiva.**

**BURST y CONTINUOUS presentan una anomalía reproducible cuando `interval_us > 0`.**

No existe evidencia experimental de que aumentar o reducir el intervalo positivo corrija el comportamiento.

---

# 11. Auditoría de código del scheduler TX

La Wiki documenta que:

* `TX_BURST` conserva contador, intervalo y próxima fecha.
* `TX_CONTINUOUS` conserva intervalo y próxima fecha sin contador.
* el tiempo de scheduling se integra mediante SysTick.

Los archivos prioritarios son:

```text
firmware/cc1352/src/control_task.c
firmware/cc1352/src/data_task.c
firmware/cc1352/src/radio_if.c
```

## 11.1 `ControlTask_onTxBurst()`

Conceptualmente:

```c
s_tx_burst_remaining = count;
s_tx_burst_interval_us = interval_us;
s_tx_burst_next_due_us = ControlTask_getTimeUs();
s_tx_burst_pending = true;
```

La primera transmisión puede ejecutarse inmediatamente porque:

```text
next_due = now
```

---

# 11.2 `ControlTask_processTxBurst()`

Su comportamiento relevante es:

```text
if now < next_due:
    return

transmitRaw()

remaining--

next_due = now + interval_us
```

Si la base de tiempo no alcanza nuevamente `next_due`, la segunda transmisión nunca se procesa.

---

# 11.3 `ControlTask_processTxContinuous()`

Utiliza la misma idea:

```text
if now < next_due:
    return

transmitRaw()

next_due = now + interval_us
```

La coincidencia del fallo en BURST y CONTINUOUS apunta por tanto a una dependencia compartida.

---

# 11.4 `ControlTask_getTimeUs()`

El scheduler utiliza una timebase software basada en:

```text
SysTickValueGet()
s_tx_systick_last
s_tx_cycles_per_us
s_tx_cycles_carry
s_tx_time_us
```

El comportamiento experimental es compatible con:

```text
interval = 0
next_due = now
→ condición satisfecha repetidamente

interval > 0
next_due = now + delta
→ condición aparentemente no vuelve a cumplirse
```

## Estado de esta afirmación

**Observación demostrada:**

```text
interval_us=0  → repetición
interval_us>0 → sólo primera TX observable
```

**Hipótesis fuerte:**

> La base temporal utilizada por `ControlTask_getTimeUs()` o su integración con TI-RTOS/SYSBIOS no progresa como supone el scheduler.

**No demostrado todavía:**

> Que `SysTickValueGet()` sea específicamente la línea defectuosa.

No debe calificarse aún como causa raíz hasta observar directamente la evolución de la timebase o demostrar estáticamente una incompatibilidad con la configuración del RTOS.

---

# 12. Hipótesis descartadas o debilitadas

## H1 — “El receptor no puede manejar múltiples paquetes.”

**Debilitada fuertemente.**

Con `interval_us=0` el observer recibió:

```text
marker_hits=99
```

Por tanto, COM88 puede recibir y entregar múltiples paquetes durante esta configuración.

---

## H2 — “RadioIF sólo puede transmitir una vez.”

**Debilitada fuertemente.**

Los 99 markers de CONTINUOUS con intervalo cero demuestran múltiples emisiones físicas observables.

---

## H3 — “El script `ota_tx_burst.py` desconecta demasiado pronto.”

**Debilitada fuertemente.**

Mantener abierta la sesión durante 3 s no modificó el resultado:

```text
count=5
interval=250 ms
marker_hits=1
```

---

## H4 — “25 ms es demasiado rápido.”

**Descartada como explicación suficiente.**

El mismo comportamiento apareció con:

```text
250 ms
1 µs
```

---

# 13. Hallazgo secundario — RX_STOP bajo alta tasa

Durante la prueba:

```text
TX_CONTINUOUS
interval_us=0
```

el observer recibió 99 markers y posteriormente:

```text
[STEP] RX_STOP
[FAIL] Timeout waiting for response: Response timeout (ignored 9 unexpected response(s), last=0x90)
```

`0x90` es:

```text
RSP_RX_PACKET
```

Por tanto, mientras la API esperaba la respuesta síncrona de `RX_STOP`, continuaban entrando respuestas asíncronas RX.

## Interpretación

Esto no demuestra que `RX_STOP` físico haya fallado.

Sí demuestra una debilidad de observabilidad/flujo host bajo carga:

> Una alta tasa de `RSP_RX_PACKET` puede interferir con la espera de una respuesta síncrona y producir timeout en la API.

La arquitectura documenta una cola host de 32 frames y un drenaje limitado por vuelta, además de hasta ocho paquetes RX emitidos por `DataTask` en cada poll.

Este comportamiento deberá correlacionarse posteriormente con pruebas de presión RX/colas y no debe corregirse de forma aislada sin estudiar su interacción con las EV dedicadas a throughput.

---

# 14. Observación secundaria — latencia variable antes de `Read packets`

Se reportó ocasionalmente una espera perceptible alrededor del inicio de RX.

La revisión de:

```text
python/examples/lab/ota_rx_probe.py
```

muestra que el flujo es:

```python
step("RX_START")
radio.start_rx()
ok("RX_START ACK")

step("Read packets")
radio.read_packets(...)
```

No existe un `sleep()` deliberado entre:

```text
RX_START ACK
```

y:

```text
Read packets
```

Por tanto:

* si la demora ocurre antes de `RX_START ACK`, puede relacionarse con `radio.start_rx()` / firmware / transporte;
* si ocurre literalmente después de imprimir `RX_START ACK` y antes de `Read packets`, no está explicada por una espera intencional del script.

## Estado

**Observación no cuantificada.**

No debe considerarse aún defecto de hardware o firmware.

### Recomendación futura

Si vuelve a presentarse:

* agregar timestamps de host antes y después de `radio.start_rx()`;
* no depender de percepción visual;
* medir varias iteraciones;
* correlacionar con RX traffic y estado previo del radio.

---

# 15. Resultado consolidado de EV-12

| Función                       | Control      | OTA                                | Comportamiento solicitado | Estado                            |
| ----------------------------- | ------------ | ---------------------------------- | ------------------------- | --------------------------------- |
| `TX_RAW`                      | PASS         | PASS                               | one-shot                  | **PASS**                          |
| `TX_FRAME`                    | PASS         | PASS                               | one-shot                  | **PASS con aclaración semántica** |
| `TX_BURST`                    | PASS         | ≥1 TX                              | repetición `count > 1`    | **ANÓMALO**                       |
| `TX_CONTINUOUS`, `interval>0` | PASS         | primera TX                         | repetición temporal       | **ANÓMALO**                       |
| `TX_CONTINUOUS`, `interval=0` | PASS         | 99 markers                         | repetición rápida         | **PASS-OTA repetitivo**           |
| `TX_STOP`                     | PASS-control | efecto RF no aislado completamente | limpiar estado programado | **PASS-control / RF parcial**     |
| RX bajo alta tasa             | RX funciona  | 99 markers                         | STOP limpio               | **timeout host observado**        |

---

# 16. Veredicto técnico de EV-12

## Resultado global

**EV-12: PASS PARCIAL CON ANOMALÍA REPRODUCIBLE DE SCHEDULING TX.**

No sería técnicamente correcto marcarla como un PASS simple.

La evidencia actual permite afirmar:

> FeralRF transmite físicamente sobre IEEE 802.15.4 canal 25 utilizando `TX_RAW`, `TX_FRAME`, `TX_BURST` y `TX_CONTINUOUS`. `TX_RAW` y `TX_FRAME` fueron observados OTA como operaciones one-shot. `TX_CONTINUOUS` con intervalo cero produjo repetición física abundante. Sin embargo, `TX_BURST` y `TX_CONTINUOUS` no presentaron la repetición esperada cuando `interval_us` fue positivo; en las condiciones probadas sólo se observó la primera transmisión.

La condición:

```text
interval_us=0 → 99 markers
interval_us=1 → 1 marker
```

es la evidencia más fuerte generada durante EV-12.

---

# 17. Discrepancia con documentación previa

La auditoría técnica anterior indicaba:

```text
EV-12 — PREPARED / NOT TESTED
```

porque en el momento en que se generó no existían comandos, stdout ni evidencia RF de EV-12.

Ese estado se encuentra ahora **obsoleto**.

No era incorrecto para la fecha de aquella auditoría; simplemente quedó superado por esta campaña experimental.

El estado vigente debe ser este informe.

---

# 18. Hallazgos para backlog de diagnóstico y mejoras

## EV12-F01 — Scheduling TX con intervalo positivo

**Severidad funcional:** alta para `TX_BURST` y `TX_CONTINUOUS`.

**Estado:** reproducido.

**Síntoma:**

```text
interval_us > 0
→ sólo primera transmisión OTA observable
```

**Contraprueba:**

```text
interval_us = 0
→ 99 transmisiones OTA observadas
```

**Principal área sospechosa:**

```text
control_task.c
ControlTask_getTimeUs()
ControlTask_processTxBurst()
ControlTask_processTxContinuous()
```

**Hipótesis:**

timebase/scheduling basado en SysTick incompatible o incorrectamente integrado con runtime TI-RTOS/SYSBIOS.

**Pendiente antes de corregir:**

demostrar cómo evoluciona `ControlTask_getTimeUs()`.

---

## EV12-F02 — ACK insuficiente para determinar resultado real de BURST/CONTINUOUS

**Estado:** confirmado por arquitectura.

Actualmente:

```text
scheduled=N
```

significa aceptación, no finalización.

Para BURST/CONTINUOUS, un fallo tardío de `RadioIF_transmitRaw()` puede limpiar estado sin notificación al host.

### Mejora candidata futura

Incorporar alguno de:

```text
TX_DONE
TX_ABORTED
TX_BURST_COMPLETE
TX_BURST_PROGRESS
TX_ERROR
```

o extender estadísticas con:

```text
requested_count
attempted_count
completed_count
failed_count
```

No implementar todavía hasta consolidar arquitectura y protocolo.

---

## EV12-F03 — `TX_FRAME` tiene una semántica potencialmente engañosa

El nombre puede sugerir framing de protocolo superior, pero:

```text
ControlTask_onTxFrame()
→ ControlTask_onTxRaw()
```

### Mejora candidata

Una de:

1. documentar explícitamente que es un alias/raw-frame path;
2. renombrar API;
3. dotarlo de semántica real de frame si existe un requisito concreto.

La opción debe decidirse después de definir qué abstracción quiere ofrecer FeralRF.

---

## EV12-F04 — RX_STOP bajo flujo asíncrono intenso

**Estado:** observado una vez bajo 99 markers.

**Síntoma:**

```text
RX_STOP esperando ACK
RSP_RX_PACKET 0x90 continúa llegando
→ timeout host
```

### Áreas a revisar posteriormente

```text
radio.py::_read_response()
radio.py::stop_rx()
OutputIF
PacketQueue
DataTask_emitRxPacket()
```

### Mejora candidata

La API host debería tolerar y enrutar correctamente eventos asíncronos mientras espera una respuesta síncrona, sin que el stream RF provoque starvation del ACK.

Debe evaluarse conjuntamente con pruebas de throughput, drops y backpressure.

---

## EV12-F05 — scripts smoke pueden producir PASS demasiado optimista

Por ejemplo:

```text
TX BURST PASS scheduled=40
```

es técnicamente sólo:

> ACK recibido para una solicitud que contiene count=40.

La Wiki ya advierte que los helpers de BURST y CONTINUOUS no prueban el conteo físico ni la energía emitida.

### Mejora candidata futura

Separar mensajes:

```text
[ OK ] TX_BURST COMMAND ACK
[INFO] scheduled_count=40
[WARN] RF completion not observed
```

en lugar de:

```text
TX BURST PASS scheduled=40
```

cuando no existe observación RF.

---

# 19. Plan de validación de una corrección futura

Una modificación del scheduler no deberá considerarse corregida exclusivamente porque el código parezca lógico.

Después de una corrección deberán repetirse, como mínimo:

### Caso A — frontera temporal

```text
CONTINUOUS interval_us=0
CONTINUOUS interval_us=1
CONTINUOUS interval_us=1000
CONTINUOUS interval_us=25000
CONTINUOUS interval_us=250000
```

Criterio:

```text
más de una emisión OTA para todos los intervalos positivos
```

---

### Caso B — BURST determinista

Probar:

```text
count=1
count=2
count=5
count=40
```

con intervalos moderados.

No exigir inicialmente 100% de recepción al observer simétrico como criterio RF absoluto, pero verificar que:

```text
count > 1
```

produzca claramente múltiples emisiones.

---

### Caso C — medición de spacing

Con instrumento o captura suficientemente precisa:

```text
timestamp[n+1] - timestamp[n]
```

comparado con `interval_us`.

Esto permitirá diferenciar:

* función correcta;
* jitter esperado;
* precisión temporal real.

---

### Caso D — terminación

Con `TX_CONTINUOUS` funcionando correctamente:

1. observar múltiples paquetes;
2. emitir `TX_STOP`;
3. comprobar ausencia posterior de markers;
4. repetir varias veces.

Sólo entonces podrá declararse **PASS-RF de STOP**.

---

### Caso E — observabilidad

Provocar deliberadamente un fallo TX posterior al ACK y verificar que el host pueda conocerlo.

---

# 20. Aspectos que EV-12 no valida

EV-12 no permite afirmar:

* que `0 dBm` solicitado equivale físicamente a 0 dBm;
* precisión absoluta de frecuencia;
* máscara espectral;
* EVM;
* armónicos;
* sensibilidad RX;
* interoperabilidad Zigbee;
* cumplimiento normativo;
* comportamiento de todas las PHY;
* robustez bajo soak prolongado.

Estas preguntas corresponden a EV posteriores e instrumentación independiente.

---

# 21. Recomendación de auditoría

No corregir todavía directamente `ControlTask_getTimeUs()` basándose únicamente en intuición.

El siguiente análisis de código debería responder, en orden:

1. ¿Qué componente configura realmente SysTick bajo TI-RTOS7/SYSBIOS?
2. ¿Está SysTick habilitado y decrementando como supone `SysTickValueGet()`?
3. ¿Quién más reconfigura SysTick?
4. ¿Cuál es su periodo real?
5. ¿Qué ocurre con el wrap-around?
6. ¿`SysCtrlClockGet()` representa correctamente el clock utilizado por SysTick?
7. ¿Existe una API de tiempo monotónico del RTOS/TI más apropiada?
8. ¿Por qué el scheduler eligió SysTick en lugar de una fuente RTOS/RTC?
9. ¿BURST queda permanentemente pending después del primer TX?
10. ¿Qué valor tienen `now_us` y `next_due_us` después de la primera transmisión?

Sólo después debe elegirse la solución.

---

# 22. Posibles líneas de corrección — no implementadas

## Opción 1 — sustituir la timebase

Usar una fuente monotónica soportada explícitamente por TI-RTOS/SYSBIOS o por el hardware cuya semántica esté documentada.

**Ventaja:** elimina dependencia de supuestos sobre SysTick.

**Riesgo:** resolución, overflow y contexto de ejecución deben analizarse.

---

## Opción 2 — scheduler RTOS

Programar expiraciones mediante servicios temporales del RTOS en lugar de integrar ciclos manualmente.

**Ventaja:** semántica de scheduling más explícita.

**Riesgo:** cambios de arquitectura, concurrencia y overhead.

---

## Opción 3 — mantener scheduler pero corregir timebase

Si SysTick es viable, corregir inicialización/cálculo/wrap.

**Ventaja:** cambio pequeño.

**Riesgo:** conservar una dependencia frágil si SysTick pertenece al kernel.

---

# 23. Prioridad para mejoras posteriores

A partir de EV-12, el orden recomendado es:

```text
P1 — demostrar causa raíz del interval scheduling
P1 — corregir BURST/CONTINUOUS
P1 — añadir validación OTA regresiva
P2 — mejorar observabilidad TX completion/error
P2 — robustecer RX_STOP frente a eventos asíncronos
P2 — corregir lenguaje de PASS en scripts
P3 — caracterizar precisión temporal real
P3 — instrumentar potencia/frecuencia/espectro
```

---

# 24. Conclusión

EV-12 produjo evidencia física positiva y, simultáneamente, descubrió un defecto funcional que un smoke test basado sólo en ACK habría ocultado.

La evidencia demuestra que:

* `TX_RAW` transmite OTA;
* `TX_FRAME` transmite OTA, aunque no representa un stack Zigbee;
* `TX_BURST` produce al menos una transmisión, pero no la secuencia esperada cuando el intervalo es positivo;
* `TX_CONTINUOUS` puede producir muchas transmisiones;
* esa repetición funciona con `interval_us=0`;
* con cualquier intervalo positivo ensayado sólo se observó la primera transmisión;
* `TX_STOP` es aceptado por la capa de control;
* bajo alta tasa RX apareció además un timeout de `RX_STOP` asociado a la llegada continua de `RSP_RX_PACKET 0x90`.

La conclusión técnica más importante de EV-12 es:

> **La capacidad RF de transmisión repetida existe y funciona, pero el mecanismo de scheduling temporal compartido por `TX_BURST` y `TX_CONTINUOUS` presenta una anomalía reproducible para `interval_us > 0`. La evidencia apunta a la timebase/scheduling de `ControlTask` como principal área de investigación, sin que todavía pueda declararse una causa raíz específica.**

En consecuencia, EV-12 debe clasificarse como:

**PASS PARCIAL CON DEFECTO FUNCIONAL REPRODUCIBLE Y ACCIÓN CORRECTIVA PENDIENTE.**

Este hallazgo deberá conservarse como baseline para comparar cualquier corrección futura y formar parte del diagnóstico final de FeralRF.
