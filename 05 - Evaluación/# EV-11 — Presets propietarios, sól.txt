# EV-11 — Presets propietarios, sólo control

## Resultado parcial: bloque de 868 MHz

### 1. Alcance del bloque

La guía de EV-11 establece que deben probarse los presets propietarios alcanzables mediante `smoke_prop_phase1.py`, excluyendo inicialmente `ook_*` y `mioty_868_tsunb`. También establece que la aceptación en esta fase es **PASS-control**, no PASS-RF, y que un preset aceptado no equivale a validar el protocolo asociado.

Para 868 MHz, los presets no excluidos identificados previamente en `PROP_PRESETS` son:

```text
gfsk_868_50k
gfsk_868_100k
msk_868_50k
wireless_mbus_s_868
wireless_mbus_t_868
wireless_mbus_c_868
4fsk_868_50k
4gfsk_868_50k
```

Los presets excluidos de esta fase son:

```text
ook_868_4k8
mioty_868_tsunb
```

En la evidencia recibida se realizaron siete ejecuciones, pero una corresponde a una repetición de `gfsk_868_50k`. Por tanto, el estado real del bloque es:

```text
6 presets únicos completados
2 presets pendientes
1 ejecución adicional repetida de gfsk_868_50k
```

---

## 2. Resultado por preset

### `gfsk_868_50k`

#### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset gfsk_868_50k --power 0
```

#### Resultado obtenido — primera ejecución

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=gfsk_868_50k freq=868000000 mod_type=1 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

#### Segunda ejecución

Se ejecutó accidental o deliberadamente el mismo comando una segunda vez y produjo nuevamente:

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=gfsk_868_50k freq=868000000 mod_type=1 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

#### Análisis

Ambas ejecuciones completaron la cadena de control sin timeout, error de protocolo o pérdida de comunicación.

La repetición aporta una pequeña evidencia adicional de reproducibilidad inmediata, pero **no cuenta como un preset adicional cubierto**.

Resultado:

**PASS-control — `gfsk_868_50k`**

---

### `msk_868_50k`

#### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset msk_868_50k --power 0
```

#### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=msk_868_50k freq=868000000 mod_type=4 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**PASS-control — `msk_868_50k`**

---

### `wireless_mbus_s_868`

#### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset wireless_mbus_s_868 --power 0
```

#### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=wireless_mbus_s_868 freq=868300000 mod_type=1 symbol_rate=32768 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**PASS-control — `wireless_mbus_s_868`**

El nombre del preset no permite afirmar validación del protocolo Wireless M-Bus S. En esta prueba sólo se verificó que la configuración asociada al preset fue aceptada por la ruta de control.

---

### `wireless_mbus_t_868`

#### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset wireless_mbus_t_868 --power 0
```

#### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=wireless_mbus_t_868 freq=868950000 mod_type=1 symbol_rate=100000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**PASS-control — `wireless_mbus_t_868`**

De nuevo, esto valida el preset como configuración alcanzable, no Wireless M-Bus T como stack o implementación interoperable.

---

### `4fsk_868_50k`

#### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset 4fsk_868_50k --power 0
```

#### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=4fsk_868_50k freq=868000000 mod_type=5 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**PASS-control — `4fsk_868_50k`**

---

### `4gfsk_868_50k`

#### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset 4gfsk_868_50k --power 0
```

#### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=4gfsk_868_50k freq=868000000 mod_type=6 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**PASS-control — `4gfsk_868_50k`**

---

# 3. Resumen provisional del bloque 868 MHz

| Preset                |  Frecuencia | `mod_type` | `symbol_rate` | Estado                          |
| --------------------- | ----------: | ---------: | ------------: | ------------------------------- |
| `gfsk_868_50k`        | 868.000 MHz |          1 |         50000 | PASS-control, ejecutado 2 veces |
| `gfsk_868_100k`       |           — |          — |             — | PENDIENTE                       |
| `msk_868_50k`         | 868.000 MHz |          4 |         50000 | PASS-control                    |
| `wireless_mbus_s_868` | 868.300 MHz |          1 |         32768 | PASS-control                    |
| `wireless_mbus_t_868` | 868.950 MHz |          1 |        100000 | PASS-control                    |
| `wireless_mbus_c_868` |           — |          — |             — | PENDIENTE                       |
| `4fsk_868_50k`        | 868.000 MHz |          5 |         50000 | PASS-control                    |
| `4gfsk_868_50k`       | 868.000 MHz |          6 |         50000 | PASS-control                    |

Estado actual:

```text
6/8 presets únicos de 868 MHz completados
6/6 de los ejecutados → PASS-control
2/8 pendientes
```

---

# 4. Comparación con lo esperado

La guía define como comportamiento sano de EV-11:

```text
ACK
+
salida normal del script
+
PASS-control
```

y deja la evidencia RF como inconclusa sin receptor independiente.

Los seis presets únicos ejecutados hasta ahora coinciden con ese comportamiento esperado:

* `SET_PHY` fue aceptado;
* `SET_PROP_CONFIG` fue aceptado;
* `SET_POWER` fue aceptado;
* `RX_START` y `RX_STOP` funcionaron;
* `TX_RAW` devolvió ACK;
* `GET_STATS` siguió respondiendo;
* el script terminó con `PROP PRESET SMOKE PASS`.

En todas las ejecuciones registradas:

```text
packets=0
rx_ok=0
rx_crc_err=0
rx_drop=0
rx_overflow=0
```

No se observaron timeout, lock-up, pérdida de puerto o error asíncrono visible.

---

# 5. Observaciones técnicas

## Repetición de `gfsk_868_50k`

`gfsk_868_50k` fue ejecutado dos veces y ambas ejecuciones produjeron el mismo resultado satisfactorio.

Esto no incrementa la cobertura funcional del bloque, pero sí constituye evidencia limitada de reproducibilidad inmediata del mismo preset.

No debe contabilizarse como “7 presets validados”.

---

## Wireless M-Bus: preset ≠ protocolo

Los resultados:

```text
wireless_mbus_s_868 → PASS-control
wireless_mbus_t_868 → PASS-control
```

demuestran que los presets correspondientes alcanzaron la ruta de configuración y completaron el smoke.

No demuestran:

```text
framing Wireless M-Bus
codificación completa
timing de protocolo
interoperabilidad
recepción por medidor real
decodificación externa
cumplimiento del estándar
```

La propia guía establece explícitamente como limitación que **“preset no es protocolo”**.

Por tanto, estos resultados no sustituyen las pruebas posteriores específicas de Wireless M-Bus.

---

## ACK de TX

En todos los casos aparece:

```text
[ OK ] TX_RAW ACK
```

Esto demuestra aceptación del comando de transmisión en la ruta de control.

No demuestra físicamente:

```text
frecuencia correcta
potencia real
modulación correcta
ocupación espectral
payload recibido OTA
```

Por ello los resultados permanecen clasificados como **PASS-control**.

---

# 6. Estado del bloque

El bloque de 868 MHz todavía **no debe cerrarse como 8/8**.

Faltan ejecutar:

```text
gfsk_868_100k
wireless_mbus_c_868
```

Sólo después de obtener evidencia de ambos podrá cerrarse formalmente 868 MHz y realizar el reset requerido antes del siguiente cambio de banda.


## `gfsk_868_100k`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset gfsk_868_100k --power 0
```

### Acción realizada

Se configuró el backend propietario mediante el preset `gfsk_868_100k`, con los parámetros mostrados por el propio script:

```text
frequency_hz = 868000000
mod_type = 1
symbol_rate = 100000
power = 0 dBm
duration = 3.0 s
min_packets = 0
```

La secuencia ejecutada fue la habitual de EV-11:

```text
RADIO_INIT
SET_PHY PROPRIETARY_GFSK
SET_PROP_CONFIG
SET_POWER
RX_START
Read packets
RX_STOP
TX_RAW
GET_STATS
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=gfsk_868_100k freq=868000000 mod_type=1 symbol_rate=100000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

El preset fue aceptado correctamente por `SET_PROP_CONFIG` y todo el recorrido de control terminó sin timeout, error de protocolo ni pérdida de comunicación.

El resultado:

```text
packets=0
```

no constituye un fallo, porque el propio script utiliza `min_packets=0` por defecto.

Las estadísticas posteriores:

```text
rx_ok=0
rx_crc_err=0
rx_drop=0
rx_overflow=0
```

muestran que el dispositivo permaneció operativo después del ciclo RX/TX y que no se registraron errores CRC, drops ni overflow durante la ventana observada.

El `TX_RAW ACK` demuestra aceptación del comando de transmisión por la ruta de control, pero no valida físicamente la emisión RF ni sus características espectrales.

**Resultado: PASS-control — `gfsk_868_100k`.**

---

## `wireless_mbus_c_868`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset wireless_mbus_c_868 --power 0
```

### Acción realizada

Se configuró el backend propietario con el preset `wireless_mbus_c_868`, cuyos parámetros mostrados por el script fueron:

```text
frequency_hz = 868950000
mod_type = 1
symbol_rate = 100000
power = 0 dBm
duration = 3.0 s
min_packets = 0
```

Posteriormente se realizó el mismo smoke de RX, TX y consulta de estadísticas utilizado durante el resto de EV-11.

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=wireless_mbus_c_868 freq=868950000 mod_type=1 symbol_rate=100000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

`wireless_mbus_c_868` completó correctamente toda la cadena de control y terminó con:

```text
[ OK ] PROP PRESET SMOKE PASS
```

No se observaron timeouts, errores de conexión, errores de protocolo ni pérdida del enlace con el DUT.

Al igual que en `wireless_mbus_s_868` y `wireless_mbus_t_868`, este resultado valida únicamente que el **preset correspondiente puede ser configurado y utilizado por la ruta de control de FeralRF**.

No demuestra todavía:

```text
interoperabilidad Wireless M-Bus C
framing conforme al estándar
timing de protocolo
decodificación por un receptor externo
calidad RF
potencia real
frecuencia medida
```

La guía de EV-11 establece expresamente que **“preset no es protocolo”** y que la aceptación de esta fase es `PASS-control`, dejando RF inconcluso sin un receptor independiente.

**Resultado: PASS-control — `wireless_mbus_c_868`.**

---

# Actualización del resumen de 868 MHz

Con estos dos resultados, el bloque queda completo:

| Preset                |  Frecuencia | `mod_type` | `symbol_rate` | Resultado    |
| --------------------- | ----------: | ---------: | ------------: | ------------ |
| `gfsk_868_50k`        | 868.000 MHz |          1 |         50000 | PASS-control |
| `gfsk_868_100k`       | 868.000 MHz |          1 |        100000 | PASS-control |
| `msk_868_50k`         | 868.000 MHz |          4 |         50000 | PASS-control |
| `wireless_mbus_s_868` | 868.300 MHz |          1 |         32768 | PASS-control |
| `wireless_mbus_t_868` | 868.950 MHz |          1 |        100000 | PASS-control |
| `wireless_mbus_c_868` | 868.950 MHz |          1 |        100000 | PASS-control |
| `4fsk_868_50k`        | 868.000 MHz |          5 |         50000 | PASS-control |
| `4gfsk_868_50k`       | 868.000 MHz |          6 |         50000 | PASS-control |

**Resultado del bloque 868 MHz: PASS-control 8/8.**

Los ocho presets no excluidos completaron satisfactoriamente `SET_PROP_CONFIG`, RX, TX y `GET_STATS`, sin timeout ni error visible. La ejecución duplicada de `gfsk_868_50k` se conserva como repetición adicional, pero no altera el conteo de cobertura.

El resultado sigue siendo exclusivamente de control: **no se asigna PASS-RF a ninguno de estos presets con la evidencia actual**.
