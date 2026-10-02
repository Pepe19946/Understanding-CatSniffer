# EV-11 — Presets propietarios, sólo control

## Cierre: 169 MHz, 902/915 MHz y 2.4 GHz

La guía define EV-11 como una evaluación de **configuración del backend**, usando `PROP_PRESETS`, `smoke_prop_phase1.py` y la ruta `SET_PROP_CONFIG → SmartRF/patch`, con aceptación `PASS-control`. El propio plan advierte que un preset aceptado **no equivale a validar RF ni un protocolo**, y excluye inicialmente `ook_*` y `mioty_868_tsunb`.

---

# 1. Bloque 169 MHz

Se evaluaron los dos presets Wireless M-Bus N definidos para 169.45 MHz.

## `wireless_mbus_n_169_2k4`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset wireless_mbus_n_169_2k4 --power 0
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=wireless_mbus_n_169_2k4 freq=169450000 mod_type=1 symbol_rate=2400 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**Resultado: PASS-control.**

---

## `wireless_mbus_n_169_4k8`

### Comando realizado explícitamente

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset wireless_mbus_n_169_4k8 --power 0
```

### Resultado obtenido

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=wireless_mbus_n_169_4k8 freq=169450000 mod_type=1 symbol_rate=4800 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**Resultado: PASS-control.**

### Análisis del bloque 169 MHz

Los dos presets completaron `SET_PROP_CONFIG`, RX, TX y `GET_STATS` sin timeout, pérdida de comunicación ni error visible.

Esto es relevante frente a la nota histórica de la guía, que indicaba **“W-MBus N no validado”**. El resultado actual permite actualizar esa afirmación únicamente en este sentido:

> Los presets `wireless_mbus_n_169_2k4` y `wireless_mbus_n_169_4k8` han sido validados actualmente a nivel de control.

No permite afirmar que Wireless M-Bus N funcione OTA ni que la implementación sea interoperable.

**169 MHz: PASS-control 2/2.**

---

# 2. Reset 169 → 902/915 MHz

Se ejecutó explícitamente el procedimiento manual mediante `COM87`.

```powershell
python -c "import serial; s=serial.Serial('COM87',115200,timeout=1); print('OPEN COM87'); s.write(b'boot\r\n'); print('SENT boot'); s.close(); print('CLOSED')"
```

Salida:

```text
OPEN COM87
SENT boot
CLOSED
```

Posteriormente:

```powershell
python -c "import serial; s=serial.Serial('COM87',115200,timeout=1); print('OPEN COM87'); s.write(b'exit\r\n'); print('SENT exit'); s.close(); print('CLOSED')"
```

Salida:

```text
OPEN COM87
SENT exit
CLOSED
```

Esto conserva el procedimiento acordado después de EV-04 y evita utilizar `Radio.reset_device()`, cuya aritmética de puertos no coincide con el Shell real de esta unidad.

---

# 3. Bloque 902/915 MHz

Se ejecutaron nueve presets distintos.

| Preset                  | Frecuencia declarada | `mod_type` | `symbol_rate` | Resultado    |
| ----------------------- | -------------------: | ---------: | ------------: | ------------ |
| `gfsk_915_50k`          |          915.000 MHz |          1 |         50000 | PASS-control |
| `gfsk_902_50k`          |          902.200 MHz |          1 |         50000 | PASS-control |
| `sidewalk_915_fsk_50k`  |          915.000 MHz |          0 |         50000 | PASS-control |
| `sidewalk_915_fsk_250k` |          915.000 MHz |          0 |        250000 | PASS-control |
| `wisun_915_fsk_50k`     |          902.200 MHz |          0 |         50000 | PASS-control |
| `wisun_915_fsk_100k`    |          902.200 MHz |          0 |        100000 | PASS-control |
| `wisun_915_fsk_150k`    |          902.200 MHz |          0 |        150000 | PASS-control |
| `wisun_915_fsk_200k`    |          902.200 MHz |          0 |        200000 | PASS-control |
| `wisun_915_fsk_300k`    |          902.200 MHz |          0 |        300000 | PASS-control |

En los nueve casos se observó el mismo patrón funcional:

```text
[ OK ] SET_PHY ACK
[ OK ] SET_PROP_CONFIG ACK
[ OK ] SET_POWER ACK
[ OK ] RX_START ACK
[ OK ] packets=0
[ OK ] RX_STOP ACK
[ OK ] TX_RAW ACK
[ OK ] stats: rx_ok=0 rx_crc_err=0 rx_drop=0 rx_overflow=0

[ OK ] PROP PRESET SMOKE PASS
```

No se observó timeout, error de conexión, error de protocolo ni bloqueo al cambiar entre los distintos symbol rates.

**902/915 MHz: PASS-control 9/9.**

### Sidewalk y Wi-SUN

Los nombres:

```text
sidewalk_915_fsk_*
wisun_915_fsk_*
```

no deben interpretarse como validación de Amazon Sidewalk ni Wi-SUN.

Lo demostrado aquí es:

```text
preset → SET_PROP_CONFIG → backend aceptado → RX/TX smoke → respuesta posterior
```

No se demostró:

```text
stack Sidewalk
stack Wi-SUN
framing interoperable
networking
association
decodificación externa
OTA
```

Por tanto, la evidencia sigue siendo estrictamente de PHY/configuración.

---

# 4. Reset 902/915 → 2.4 GHz

También quedó registrado explícitamente:

```powershell
python -c "import serial; s=serial.Serial('COM87',115200,timeout=1); print('OPEN COM87'); s.write(b'boot\r\n'); print('SENT boot'); s.close(); print('CLOSED')"
```

Salida:

```text
OPEN COM87
SENT boot
CLOSED
```

y:

```powershell
python -c "import serial; s=serial.Serial('COM87',115200,timeout=1); print('OPEN COM87'); s.write(b'exit\r\n'); print('SENT exit'); s.close(); print('CLOSED')"
```

Salida:

```text
OPEN COM87
SENT exit
CLOSED
```

---

# 5. Bloque propietario 2.4 GHz

## `gfsk_2440_250k`

Comando:

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset gfsk_2440_250k --power 0
```

Resultado:

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=gfsk_2440_250k freq=2440000000 mod_type=1 symbol_rate=250000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**Resultado: PASS-control.**

---

## `gfsk_2440_50k`

Comando:

```powershell
python examples\smoke_prop_phase1.py --port COM88 --preset gfsk_2440_50k --power 0
```

Resultado:

```text
FeralRF Proprietary Preset Smoke Test
=====================================
port=COM88 baudrate=921600 preset=gfsk_2440_50k freq=2440000000 mod_type=1 symbol_rate=50000 power=0 duration=3.0s min_packets=0 auto_reset=no

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

**Resultado: PASS-control.**

### Análisis del bloque 2.4 GHz

Ambos presets propietarios de 2.44 GHz fueron aceptados y completaron el smoke test.

Esto confirma actualmente la existencia y alcanzabilidad de la ruta de **configuración propietaria 2.4 GHz**, pero no resuelve por sí misma posibles discrepancias históricas entre README, matriz o evidencia OTA. Para ello se necesita la evaluación RF posterior correspondiente.

**2.4 GHz: PASS-control 2/2.**

---

# 6. Resultado global de EV-11

El inventario inicial contenía:

```text
31 PROP_PRESETS
```

Se excluyeron conforme a la propia guía:

```text
ook_433_4k8
ook_433_2k4
ook_868_4k8
mioty_868_tsunb
```

Por tanto:

```text
31 totales
- 3 OOK
- 1 MIOTY
= 27 presets objetivo para EV-11
```

Resultado final:

| Banda       | Presets objetivo | PASS-control |
| ----------- | ---------------: | -----------: |
| 433 MHz     |                6 |          6/6 |
| 868 MHz     |                8 |          8/8 |
| 169 MHz     |                2 |          2/2 |
| 902/915 MHz |                9 |          9/9 |
| 2.4 GHz     |                2 |          2/2 |
| **TOTAL**   |           **27** |    **27/27** |

## Conclusión

**EV-11 = PASS-control 27/27.**

En los 27 presets incluidos en la evaluación:

```text
RADIO_INIT          → satisfactorio
SET_PHY             → ACK
SET_PROP_CONFIG     → ACK
SET_POWER           → ACK
RX_START            → ACK
RX_STOP             → ACK
TX_RAW              → ACK
GET_STATS           → respondió
script final        → PROP PRESET SMOKE PASS
```

No se registró entre estos presets evaluados ningún:

```text
timeout
ConnectionError
Protocol/command error
lock-up visible
pérdida de COM88
rx_crc_err
rx_drop
rx_overflow
```

La observación repetida de:

```text
packets=0
```

no representa un fallo de EV-11 porque `smoke_prop_phase1.py` utiliza `min_packets=0`.

---

# 7. Qué permite afirmar EV-11

La evidencia obtenida permite concluir:

> En la revisión de FeralRF evaluada, los 27 presets incluidos en EV-11 pudieron configurar el backend mediante la API pública y completar la secuencia de control RX/TX prevista por el script original del repositorio sin errores visibles de comunicación o estado.

Esto supone evidencia positiva de:

```text
API Python
    ↓
protocolo FeralRF
    ↓
SET_PHY
    ↓
SET_PROP_CONFIG
    ↓
backend propietario
    ↓
aceptación RX/TX
    ↓
recuperación funcional posterior mediante GET_STATS
```

---

# 8. Qué NO permite afirmar EV-11

EV-11 no demuestra que los 27 presets funcionen físicamente por RF.

En particular, `TX_RAW ACK` no demuestra:

```text
frecuencia real emitida
potencia real
desviación
ancho de banda
espectro
modulación correcta
packet-on-air correcto
recepción OTA
sensibilidad
interoperabilidad
```

Tampoco permite afirmar que presets con nombres de protocolos implementen esos protocolos.

Por tanto:

```text
wireless_mbus_*  ≠ Wireless M-Bus validado
wisun_*          ≠ Wi-SUN validado
sidewalk_*       ≠ Sidewalk validado
```

La propia guía establece expresamente que **“preset no es protocolo”** y que RF queda inconcluso sin receptor.

---

# 9. Comparación con observaciones históricas

### 433 MHz marginal

EV-11 obtuvo:

```text
433 MHz → PASS-control 6/6
```

Esto no contradice el antecedente histórico de 433 MHz marginal, porque la marginalidad corresponde a comportamiento RF, mientras EV-11 únicamente verifica control/configuración.

### Wireless M-Bus N

La guía indicaba W-MBus N como no validado.

Ahora podemos refinar esa afirmación:

```text
W-MBus N / preset control → VALIDADO en EV-11
W-MBus N / RF-OTA         → NO VALIDADO por EV-11
W-MBus N / protocolo      → NO VALIDADO por EV-11
```

### Propietario 2.4 GHz

Los dos presets de 2440 MHz completaron correctamente la ruta de control:

```text
gfsk_2440_250k → PASS-control
gfsk_2440_50k  → PASS-control
```

Esto confirma la presencia funcional de la configuración en la API/backend actual, pero todavía no constituye evidencia física de operación RF en 2.4 GHz.

---

# 10. Observación de reproducibilidad

La guía solicita reset entre bandas. En la evidencia acumulada quedaron registrados explícitamente resets manuales mediante `COM87` para diversas transiciones, incluyendo:

```text
433 → 868 MHz
169 → 902/915 MHz
902/915 → 2.4 GHz
```

En el bloque de terminal utilizado para este cierre **no aparece la salida del reset 868 → 169 MHz**.

Esto no invalida los resultados funcionales de los dos presets de 169 MHz, pero debe conservarse como una pequeña limitación documental si el objetivo es reconstruir posteriormente cada transición únicamente a partir del log disponible.

No debe rellenarse suponiendo que ocurrió.

---

# Estado final

```text
EV-11 — Presets propietarios, sólo control

PROP_PRESETS encontrados:        31
Excluidos por plan:               4
Presets evaluados:               27

433 MHz:                          6/6 PASS-control
868 MHz:                          8/8 PASS-control
169 MHz:                          2/2 PASS-control
902/915 MHz:                      9/9 PASS-control
2.4 GHz:                          2/2 PASS-control

TOTAL:                           27/27 PASS-control

PASS-RF:                         NO
Protocolos validados:            NO
Timeouts/hangs observados:       0
Errores de control observados:   0

EV-11:                           CERRADA — PASS-control
```
