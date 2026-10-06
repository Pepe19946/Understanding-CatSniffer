## EV-11 — Presets propietarios, sólo control

### Resultado parcial: bloque de 433 MHz

### 1. Objetivo de la evaluación

EV-11 busca comprobar que los presets propietarios definidos por FeralRF pueden recorrer correctamente la ruta de configuración del backend de radio. La guía identifica como evidencia principal `presets.py:PROP_PRESETS`, `smoke_prop_phase1.py` y `RadioIF_setPropConfig`, y define conceptualmente la ruta:

`host Python → SET_PROP_CONFIG → configuración SmartRF/patch → backend RF`

La aceptación prevista para esta fase es **PASS-control**. Aunque `smoke_prop_phase1.py` realiza un TX breve, la documentación advierte que sin un receptor u observador RF independiente el resultado de transmisión debe permanecer **inconcluso a nivel RF**. También establece que los presets `ook_*` y `mioty_868_tsunb` no deben incluirse inicialmente, y que debe realizarse reset al cambiar de banda.

Para el bloque de 433 MHz se excluyeron, conforme al plan:

```text
ook_433_4k8
ook_433_2k4
```

Por tanto, se evaluaron seis presets:

```text
gfsk_433_50k
gfsk_433_10k
fsk_433_50k
msk_433_50k
4fsk_433_50k
4gfsk_433_50k
```

---

## 2. Comportamiento del script utilizado

Se utilizó sin modificar el script original del repositorio:

```text
examples\smoke_prop_phase1.py
```

con la forma:

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset NOMBRE --power 0
```

La inspección previa del código confirmó que cada ejecución realiza, en este orden:

```text
Radio.init()
    ↓
set_phy(PHY.PROPRIETARY_GFSK, 0)
    ↓
configure_prop(**preset)
    ↓
set_power(0)
    ↓
start_rx()
    ↓
read_packets(timeout=3.0)
    ↓
stop_rx()
    ↓
transmit(0x01020304, power_dbm=0)
    ↓
get_stats()
```

Un aspecto importante es que el script define:

```python
--min-packets
default=0
```

por lo que `packets=0` es un resultado permitido y **no constituye un fallo de EV-11**. Esta característica es coherente con el objetivo de la prueba: validar primero el camino de control/configuración, no exigir todavía recepción OTA.

El script también ejecuta `TX_RAW`; sin embargo, un `TX_RAW ACK` sólo demuestra que la operación fue aceptada por el firmware/protocolo de control. No demuestra por sí solo potencia, frecuencia, espectro, modulación correcta ni recepción por otro dispositivo.

---

# 3. Resultados experimentales — 433 MHz

## 3.1 `gfsk_433_50k`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset gfsk_433_50k --power 0
```

### Acción realizada

Configura el backend propietario con el preset `gfsk_433_50k`, correspondiente a:

```text
frequency_hz = 433920000
mod_type = 1
symbol_rate = 50000
power = 0 dBm
```

Posteriormente abre RX durante 3 s, detiene RX, solicita un `TX_RAW` del payload `01020304` y consulta estadísticas.

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=gfsk_433_50k freq=433920000 mod_type=1 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY PROPRIETARY_GFSK
[ OK ] SET_PHY ACK
[STEP] SET_PROP_CONFIG
[ OK ] SET_PROP_CONFIG ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=0
[STEP] RX_STOP
[ OK ] RX_STOP ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK
[STEP] GET_STATS
[ OK ] stats: rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0

[ OK ] PROP PRESET SMOKE PASS
```

### Análisis

Todos los comandos de control esperados recibieron respuesta satisfactoria. No hubo timeout, error de protocolo, pérdida de conexión ni fallo visible durante `SET_PROP_CONFIG`.

`packets=0` es compatible con el criterio de esta prueba, dado que `min_packets=0`.

Resultado:

**PASS-control — `gfsk_433_50k`**

No se asigna `PASS-RF`.

---

## 3.2 `gfsk_433_10k`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset gfsk_433_10k --power 0
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=gfsk_433_10k freq=433920000 mod_type=1 symbol_rate=10000 power=0 duration=3.0s min_packets=0 auto_reset=no

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY PROPRIETARY_GFSK
[ OK ] SET_PHY ACK
[STEP] SET_PROP_CONFIG
[ OK ] SET_PROP_CONFIG ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=0
[STEP] RX_STOP
[ OK ] RX_STOP ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK
[STEP] GET_STATS
[ OK ] stats: rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0

[ OK ] PROP PRESET SMOKE PASS
```

Resultado:

**PASS-control — `gfsk_433_10k`**

El cambio respecto al preset anterior es principalmente el `symbol_rate`, que pasa de 50000 a 10000, manteniendo frecuencia y `mod_type`. La ruta de configuración siguió respondiendo normalmente.

---

## 3.3 `fsk_433_50k`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset fsk_433_50k --power 0
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=fsk_433_50k freq=433920000 mod_type=0 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY PROPRIETARY_GFSK
[ OK ] SET_PHY ACK
[STEP] SET_PROP_CONFIG
[ OK ] SET_PROP_CONFIG ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=0
[STEP] RX_STOP
[ OK ] RX_STOP ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK
[STEP] GET_STATS
[ OK ] stats: rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0

[ OK ] PROP PRESET SMOKE PASS
```

Resultado:

**PASS-control — `fsk_433_50k`**

Es relevante que el script siga imprimiendo:

```text
SET_PHY PROPRIETARY_GFSK
```

aunque el preset declare:

```text
mod_type=0
```

Esto no debe interpretarse inmediatamente como incoherencia. `PROPRIETARY_GFSK` es el PHY genérico seleccionado previamente por la API, mientras que la configuración concreta se suministra posteriormente mediante `SET_PROP_CONFIG`. No obstante, esta arquitectura merece conservarse como observación para el análisis de implementación.

---

## 3.4 `msk_433_50k`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset msk_433_50k --power 0
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=msk_433_50k freq=433920000 mod_type=4 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY PROPRIETARY_GFSK
[ OK ] SET_PHY ACK
[STEP] SET_PROP_CONFIG
[ OK ] SET_PROP_CONFIG ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=0
[STEP] RX_STOP
[ OK ] RX_STOP ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK
[STEP] GET_STATS
[ OK ] stats: rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0

[ OK ] PROP PRESET SMOKE PASS
```

Resultado:

**PASS-control — `msk_433_50k`**

---

## 3.5 `4fsk_433_50k`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset 4fsk_433_50k --power 0
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=4fsk_433_50k freq=433920000 mod_type=5 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY PROPRIETARY_GFSK
[ OK ] SET_PHY ACK
[STEP] SET_PROP_CONFIG
[ OK ] SET_PROP_CONFIG ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=0
[STEP] RX_STOP
[ OK ] RX_STOP ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK
[STEP] GET_STATS
[ OK ] stats: rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0

[ OK ] PROP PRESET SMOKE PASS
```

Resultado:

**PASS-control — `4fsk_433_50k`**

---

## 3.6 `4gfsk_433_50k`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset 4gfsk_433_50k --power 0
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=4gfsk_433_50k freq=433920000 mod_type=6 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY PROPRIETARY_GFSK
[ OK ] SET_PHY ACK
[STEP] SET_PROP_CONFIG
[ OK ] SET_PROP_CONFIG ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=0
[STEP] RX_STOP
[ OK ] RX_STOP ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK
[STEP] GET_STATS
[ OK ] stats: rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0

[ OK ] PROP PRESET SMOKE PASS
```

Resultado:

**PASS-control — `4gfsk_433_50k`**

---

# 4. Resumen del bloque de 433 MHz

| Preset          | Frecuencia | `mod_type` | `symbol_rate` | Resultado    |
| --------------- | ---------: | ---------: | ------------: | ------------ |
| `gfsk_433_50k`  | 433.92 MHz |          1 |         50000 | PASS-control |
| `gfsk_433_10k`  | 433.92 MHz |          1 |         10000 | PASS-control |
| `fsk_433_50k`   | 433.92 MHz |          0 |         50000 | PASS-control |
| `msk_433_50k`   | 433.92 MHz |          4 |         50000 | PASS-control |
| `4fsk_433_50k`  | 433.92 MHz |          5 |         50000 | PASS-control |
| `4gfsk_433_50k` | 433.92 MHz |          6 |         50000 | PASS-control |

**Resultado provisional del bloque 433 MHz: PASS-control 6/6.**

Todos los presets alcanzaron el backend mediante `SET_PROP_CONFIG`, permitieron iniciar y detener RX, aceptaron `TX_RAW`, respondieron posteriormente a `GET_STATS` y terminaron con:

```text
[ OK ] PROP PRESET SMOKE PASS
```

No se observaron:

```text
Timeout
ConnectionError
Protocol/command error
rx_crc_err
rx_drop
rx_overflow
```

en las ejecuciones registradas.

---

# 5. Comparación con lo esperado y con la documentación

La guía establece como comportamiento sano de EV-11:

* aceptación/configuración del preset;
* ACK y terminación normal del script;
* `PASS-control`;
* RF inconcluso sin receptor;
* reset entre bandas.

El comportamiento obtenido en 433 MHz coincide con esos criterios.

La documentación también registra **433 MHz como históricamente marginal**. Los resultados actuales no contradicen esa observación, porque ambas evidencias pertenecen a niveles diferentes:

```text
Resultado actual:
SET_PROP_CONFIG + RX/TX control → PASS-control

Advertencia histórica:
calidad/comportamiento RF 433 MHz → marginal
```

Por tanto, sería incorrecto concluir a partir de EV-11 que los problemas históricos de 433 MHz han desaparecido.

Lo que sí podemos afirmar es más limitado:

> En la revisión de FeralRF evaluada, los seis presets no-OOK de 433 MHz incluidos en EV-11 fueron aceptados por la ruta de control y completaron el smoke test original del repositorio sin timeout, error de protocolo ni pérdida de comunicación.

No podemos afirmar todavía:

* potencia real de salida;
* frecuencia RF medida;
* desviación de frecuencia;
* calidad espectral;
* correspondencia física de FSK/GFSK/MSK/4FSK/4GFSK;
* sensibilidad RX;
* interoperabilidad;
* recepción OTA;
* resolución del comportamiento históricamente marginal de 433 MHz.

---

# 6. Observaciones de implementación

### `SET_PHY PROPRIETARY_GFSK` es común a todas las modulaciones

Todos los presets, incluidos `FSK`, `MSK`, `4FSK` y `4GFSK`, pasan inicialmente por:

```text
SET_PHY PROPRIETARY_GFSK
```

y posteriormente por:

```text
SET_PROP_CONFIG
```

La modulación concreta aparece dentro del preset mediante `mod_type`.

Esto indica que `PROPRIETARY_GFSK` funciona aquí como selección de la familia/backend propietario de la API y que `configure_prop()` aporta después los parámetros específicos. Esta observación debe contrastarse posteriormente con `RadioIF_setPropConfig` y la configuración TI antes de convertirla en una conclusión arquitectónica definitiva.

### `TX_RAW ACK` no equivale a TX RF validado

El smoke registra:

```text
[ OK ] TX_RAW ACK
```

para las seis configuraciones.

Esto constituye evidencia del camino de control:

```text
PC → Python API → protocolo FeralRF → firmware
```

pero no demuestra por sí mismo:

```text
firmware → RF core → RF front-end → antena → aire
```

Por ello se conserva la clasificación `PASS-control`.

### Estadísticas posteriores sanas

En todas las salidas preservadas:

```text
rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0
```

El dato importante para EV-11 no es que `rx_ok=0`, sino que la operación posterior `GET_STATS` continuó respondiendo y no aparecieron `drop` u `overflow` durante el smoke.

---

# 7. Cambio de banda 433 → 868 MHz

La guía exige reset entre bandas. Se aplicó el procedimiento manual mediante el Shell real del CatSniffer, `COM87`, evitando `Radio.reset_device()` porque EV-04 ya había demostrado la discrepancia KI-15 entre el Shell calculado y el Shell real.

Hubo primero un error de sintaxis en el comando construido para PowerShell/Python:

```text
File "<string>", line 1
    import serial; s=serial.Serial('COM87',115200,timeout=1); print('OPEN COM87'); s.write(b'boot\r\n'); print(" SENT bboot\\r\\n\);
                                                                                                               ^
SyntaxError: unterminated string literal (detected at line 1)
```

Este evento ocurrió durante el parseo de Python, antes de poder ejecutar la operación serial. Por tanto:

* no constituye un fallo de FeralRF;
* no constituye un fallo del DUT;
* no demuestra que se abriera `COM87`;
* se clasifica como error del comando/harness operativo.

Se corrigió simplificando el quoting.

### Reset ejecutado correctamente

Comando:

```powershell
python -c "import serial; s=serial.Serial('COM87',115200,timeout=1); print('OPEN COM87'); s.write(b'boot\r\n'); print('SENT boot'); s.close(); print('CLOSED')"
```

Salida exacta:

```text
OPEN COM87
SENT boot
CLOSED
```

Posteriormente:

```powershell
python -c "import serial; s=serial.Serial('COM87',115200,timeout=1); print('OPEN COM87'); s.write(b'exit\r\n'); print('SENT exit'); s.close(); print('CLOSED')"
```

Salida exacta:

```text
OPEN COM87
SENT exit
CLOSED
```

Estas salidas demuestran que ambos comandos fueron enviados al Shell por `COM87`.

No deben interpretarse por sí solos como prueba completa de recuperación de `COM88`; para cerrar formalmente la transición antes de iniciar 868 MHz conviene verificar nuevamente la comunicación con FeralRF.

---

# 8. Estado parcial de EV-11

A este punto:

```text
Inventario PROP_PRESETS          → realizado
Exclusiones iniciales            → aplicadas
OOK                              → diferido a EV-24
MIOTY                            → excluido de EV-11 inicial

433 MHz:
  gfsk_433_50k                   → PASS-control
  gfsk_433_10k                   → PASS-control
  fsk_433_50k                    → PASS-control
  msk_433_50k                    → PASS-control
  4fsk_433_50k                   → PASS-control
  4gfsk_433_50k                  → PASS-control

Bloque 433 MHz                   → PASS-control 6/6

Reset 433 → siguiente banda:
  boot por COM87                 → enviado
  exit por COM87                 → enviado

868 MHz                          → pendiente
169 MHz                          → pendiente
902/915 MHz                      → pendiente
2.4 GHz                          → pendiente

EV-11 completa                   → EN PROGRESO
```

La conclusión de EV-11 todavía no debe cerrarse hasta completar los presets restantes previstos por el plan.
