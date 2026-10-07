# EV-12 — Control preliminar y preparación de EV-13

Registro histórico complementario de EV-12. Resultado actualizado en [[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]]. Su segunda mitad es planificación de EV-13, no evidencia de su ejecución.

## 1. Contexto de evaluación

La etapa inicial revisó comandos TX y STOP por control, antes de disponer del montaje posterior de dos placas y observación por aire.

## 2. Objetivo de validación

Ejecutar RAW, FRAME, BURST, CONTINUOUS/STOP y comprobar respuestas de API sin afirmar todavía emisiones RF.

## 3. Capacidad o requisito FeralRF evaluado

API de transmisión y programación; [[Protocolo y API Python]]. RAW/FRAME no son pruebas de CW; CONTINUOUS repite paquetes y no equivale a portadora continua.

## 4. Precondiciones y condiciones

COM88 como DUT inicial; IEEE canal 25, potencia 0 dBm, payload `01020304`; burst de cinco con intervalo 5000 µs según comandos; continuous durante 1 s con intervalo 5000 µs. No existe observador RF de esta etapa. Fecha experimental ausente.

## 5. Resultado esperado

ACK de los cuatro modos y de STOP; posibilidad de nueva inicialización/estadísticas. El propio registro no define conteo OTA de repeticiones como comprobación realizada.

## 6. Procedimiento y ejecución cronológica

1. RAW y nueva inicialización/estadísticas.
2. FRAME conforme al comando original.
3. BURST de cinco y nueva inicialización/estadísticas.
4. CONTINUOUS durante un segundo, STOP y nueva inicialización/estadísticas.
5. Documentar una propuesta de EV-13; no se ejecutan sus pasos por el hecho de estar escritos.

## 7. Resultado observado

Cuatro modos aceptados por control (4/4 declarado), STOP con ACK y estadísticas posteriores registradas. No hay conteo RF ni evidencia física de cese.

## 8. Evidencia

Comandos/salidas y preparación de EV-13 íntegros en §17. La segunda H1 EV-13 de la fuente es una sección de planificación, no un segundo registro ejecutado.

## 9. Comparación entre lo esperado y lo observado

Control cumple el objetivo inicial. La misma familia de modos mostró después discrepancias de repetición por aire; aquella observación no invalida que estos ACK ocurrieran.

## 10. Interpretación técnica

ACK no equivale a transmisión completada. Un `INIT` previo a leer estadísticas puede reiniciar métricas; esas lecturas no son una fotografía pasiva del estado posterior al modo.

## 11. Anomalías, desviaciones y limitaciones

Sin observador, sin bytes por aire, sin cese medido y sin hash binario. Reinicializaciones limitan inferencias de continuidad. Propuesta EV-13 contiene condiciones de disponibilidad que deben leerse como históricas.

## 12. Resultado de la evaluación

PASS de control inicial; PARTIAL para los modos RF. Resultado global actualizado en el registro canónico, no en esta etapa histórica.

## 13. Confianza

High para ACK documentados; Low para emisiones/repetición/cese no medidos.

## 14. Preguntas abiertas

¿Se emite realmente cada paquete programado? ¿Los intervalos y STOP son efectivos por aire?

## 15. Acciones de seguimiento

Consultar etapa OTA y sus anomalías; mantener la preparación como procedencia de EV-13, sin contabilizarla como ejecución adicional.

## 16. Trazabilidad

Guía EV-12/EV-13; [[FeralRF - Guía de validación experimental]]; [[FeralRF - Matriz de pruebas]]; [[EV-12 — TX RAW FRAME BURST CONTINUOUS por aire]]; [[EV-13 — CW PRBS y TX_TEST_STOP por control]]; [[Matriz de capacidades]]; [[Registro de validación FeralRF]].

Definición específica: [[FeralRF - Wiki técnica integral#9. Arquitectura TX]].



## 17. Notas originales preservadas y material pendiente

Fuente: `# EV-12 — Validación de TX rawframe..MD`. SHA-256 previo: `8AC3893D3C3264619169B61F7C682F623C54E8A7DB1E9E5008A1E9F84BFC36A2`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
# EV-12 — Validación de TX raw/frame/burst/continuous y STOP

## 1. Objetivo

EV-12 tiene como objetivo recorrer las principales primitivas de transmisión expuestas por FeralRF y comprobar, en una primera etapa, que el plano de control acepta las operaciones y mantiene un estado recuperable.

La Guía de validación define cuatro rutas:

* `TX_RAW`
* `TX_FRAME`
* `TX_BURST`
* `TX_CONTINUOUS + TX_STOP`

La configuración prescrita para esta evaluación es:

```text
DUT: CatSniffer V3 con FeralRF
Bridge: COM88
PHY: 4
Channel: 25
Power: 0 dBm
Payload inicial: 01 02 03 04
Continuous: 1 s máximo inicial
```

La Guía establece explícitamente que un ACK sólo permite concluir `PASS-control`; `PASS-RF` requiere un observador que confirme las emisiones. También establece que, en continuous, la persistencia de energía después de `STOP` constituye una condición de fallo físico.

La Matriz clasifica EV-12 como una capacidad estable implementada y cubierta por los scripts `smoke_tx_*`, pero señala que el PASS histórico es ambiguo precisamente porque ACK y emisión RF no son equivalentes. La evidencia ideal es `ACK + captura`, y el observador RF es necesario para elevar el resultado a RF físico.

---

## 2. Consideración metodológica derivada de la auditoría

Antes de ejecutar EV-12 se adoptó como regla:

```text
ACK ≠ TX_DONE ≠ emisión RF observada
```

La auditoría determinó que, al menos para la ruta TX raw, el ACK se produce antes de poder confirmar `RadioIF_transmitRaw`, y que un fallo posterior puede aparecer como error asíncrono. Por tanto, expresiones como “TX completado” no deben utilizarse sólo porque el host haya recibido ACK.

En consecuencia, durante EV-12 se aplicaron dos niveles separados:

```text
PASS-control
    =
el comando fue aceptado,
el script terminó correctamente
y no apareció error síncrono visible.

PASS-RF
    =
la transmisión fue observada físicamente
por un receptor o instrumento independiente.
```

En esta campaña sólo se buscó cerrar la primera categoría.

---

# 3. Inspección previa de los scripts

Antes de transmitir se inspeccionaron los cuatro scripts originales del repositorio.

## 3.1 TX_RAW

Script:

```text
examples\smoke_tx_phase1.py
```

Ruta principal:

```text
Radio.init()
→ set_phy()
→ set_channel()
→ set_power()
→ Radio.transmit()
→ TX_RAW ACK
→ disconnect()
```

Configuración usada:

```text
PHY 4
Channel 25
Power 0 dBm
Payload 01 02 03 04
```

El script considera PASS cuando `Radio.transmit()` obtiene ACK.

No ejecuta `GET_STATS` al final ni espera un evento `TX_DONE`.

---

## 3.2 TX_FRAME

Script:

```text
examples\smoke_tx_frame_phase1.py
```

La diferencia principal frente a TX_RAW es que utiliza:

```python
radio.transmit_frame(frame, timeout=args.tx_timeout)
```

en lugar de:

```python
radio.transmit(...)
```

Por tanto, `TX_RAW` y `TX_FRAME` ejercitan primitivas distintas del API.

El script impone además límites locales:

```text
BLE PHY 0–3:
  canales 37/38/39
  payload <= 31 B

IEEE 802.15.4 PHY 4:
  payload <= 125 B
```

Estas validaciones se realizan en el host, por lo que no deben interpretarse como rechazo demostrado por firmware.

La prueba realizada utilizó PHY 4 y sólo 4 bytes, por lo que EV-12 no caracterizó aquí los límites máximos.

---

## 3.3 TX_BURST

Script:

```text
examples\smoke_tx_burst_phase1.py
```

Ruta principal:

```text
Radio.transmit_burst(
    packet,
    count=5,
    interval_us=5000
)
```

Configuración utilizada:

```text
Payload = 01 02 03 04
Count = 5
Interval = 5000 us
```

Conceptualmente solicita una ráfaga finita del mismo payload.

El script valida localmente:

```text
count: 1..65535
interval_us >= 0
```

Un `TX_BURST ACK` demuestra aceptación de la solicitud, pero no demuestra por sí solo que ocurrieran físicamente cinco emisiones ni que el espaciado real fuera exactamente 5000 µs.

---

## 3.4 TX_CONTINUOUS + TX_STOP

Script:

```text
examples\smoke_tx_continuous_phase1.py
```

Configuración usada:

```text
Payload = 01 02 03 04
Interval = 5000 us
Run time = 1 s
```

Secuencia:

```text
TX_CONTINUOUS
→ ACK
→ sleep(1 s)
→ TX_STOP
→ ACK
```

Este modo no debe confundirse con CW.

`TX_CONTINUOUS` representa transmisión repetitiva de paquetes hasta recibir `TX_STOP`.

El script dispone además de cleanup defensivo:

```python
if tx_started:
    try:
        radio.stop_transmit(...)
    except Exception:
        pass
```

Esto intenta detener TX si aparece una excepción después de comenzar continuous.

Sin embargo, cualquier excepción durante ese STOP de emergencia es descartada silenciosamente, por lo que un cierre anómalo no permitiría asumir por sí mismo que RF cesó.

---

# 4. Ejecución experimental

## 4.1 TX_RAW

### Comando realizado explícitamente

```powershell
python examples\smoke_tx_phase1.py --port COM88 --phy 4 --channel 25 --power 0
```

### Acción

Solicita una transmisión raw de:

```text
01 02 03 04
```

usando:

```text
PHY 4
Channel 25
Power 0 dBm
```

### Resultado registrado

```text
FeralRF TX Raw Smoke Test (Phase 1)
====================================
port=COM88 baudrate=921600 phy=4 channel=25 power=0 len=4

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_RAW
[ OK ] TX_RAW ACK

[ OK ] TX SMOKE PASS
```

### Interpretación

La sesión se inicializó correctamente, se aceptó la configuración y `TX_RAW` obtuvo ACK sin timeout ni error síncrono.

**Resultado: `PASS-control`.**

No existe evidencia externa que demuestre la emisión física de los cuatro bytes.

---

## 4.2 Comprobación de estado posterior a TX_RAW

### Comando

```powershell
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:', r.init()); print('STATS:', r.get_stats()); r.disconnect()"
```

### Resultado

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

### Interpretación

El DUT siguió respondiendo después de TX_RAW.

Esto constituye una comprobación favorable del estado posterior, pero no una medición RF.

---

## 4.3 TX_FRAME

### Comando

```powershell
python examples\smoke_tx_frame_phase1.py --port COM88 --phy 4 --channel 25 --power 0
```

### Resultado

```text
FeralRF TX Frame Smoke Test (Phase 1)
======================================
port=COM88 baudrate=921600 phy=4 channel=25 power=0 len=4

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_FRAME
[ OK ] TX_FRAME ACK

[ OK ] TX FRAME SMOKE PASS
```

### Interpretación

La ruta específica `TX_FRAME` fue aceptada y el script terminó normalmente.

**Resultado: `PASS-control`.**

No queda demostrado:

* framing físico correcto;
* FCS correcto;
* recepción por otro equipo;
* emisión RF real.

---

# 4.4 TX_BURST

### Comando

```powershell
python examples\smoke_tx_burst_phase1.py --port COM88 --phy 4 --channel 25 --power 0
```

### Resultado

```text
FeralRF TX Burst Smoke Test (Phase 1)
======================================
port=COM88 baudrate=921600 phy=4 channel=25 power=0 len=4 count=5 interval_us=5000

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY + SET_CHANNEL + SET_POWER
[ OK ] Config ACK
[STEP] TX_BURST
[ OK ] TX_BURST ACK

[ OK ] TX BURST SMOKE PASS
```

### Interpretación

La API/firmware aceptó una solicitud de burst de:

```text
count = 5
interval_us = 5000
```

y el script terminó correctamente.

**Resultado: `PASS-control`.**

No queda establecido que:

```text
se hayan emitido realmente 5 paquetes
```

ni que:

```text
el intervalo físico real haya sido 5000 us.
```

Eso requiere observación externa.

---

## 4.5 Comprobación posterior a TX_BURST

### Comando

```powershell
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:', r.init()); print('STATS:', r.get_stats()); r.disconnect()"
```

### Resultado

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

El DUT permaneció accesible.

---

# 4.6 TX_CONTINUOUS + TX_STOP

### Comando

```powershell
python examples\smoke_tx_continuous_phase1.py --port COM88 --phy 4 --channel 25 --power 0 --run-seconds 1
```

### Resultado

```text
FeralRF TX Continuous Smoke Test (Phase 1)
===========================================
port=COM88 baudrate=921600 phy=4 channel=25 power=0 len=4 interval_us=5000 run_seconds=1.0

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

### Interpretación

Se comprobó que:

```text
TX_CONTINUOUS fue aceptado
```

y posteriormente:

```text
TX_STOP fue aceptado
```

sin timeout ni error síncrono visible.

**Resultado: `PASS-control`.**

El `TX_STOP ACK` demuestra aceptación de la orden STOP por el plano de control.

No demuestra por sí solo que la energía RF haya cesado físicamente en ese instante.

---

## 4.7 Comprobación final posterior a continuous

### Comando

```powershell
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:', r.init()); print('STATS:', r.get_stats()); r.disconnect()"
```

### Resultado

```text
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)
```

### Interpretación

Después del recorrido `TX_CONTINUOUS → TX_STOP`, el DUT volvió a responder normalmente a `INIT` y `GET_STATS`.

Esto constituye una evidencia favorable de recuperación del plano de control.

No constituye prueba directa de cese RF.

---

# 5. Resultado consolidado de EV-12

| Primitiva                 | Resultado          |
| ------------------------- | ------------------ |
| `TX_RAW`                  | **PASS-control**   |
| `TX_FRAME`                | **PASS-control**   |
| `TX_BURST`                | **PASS-control**   |
| `TX_CONTINUOUS + TX_STOP` | **PASS-control**   |
| Estado posterior          | Favorable          |
| TX observado físicamente  | **NO ESTABLECIDO** |

## Veredicto

> **EV-12: PASS-control 4/4, con recuperación posterior favorable.**

Las cuatro primitivas TX evaluadas fueron aceptadas por FeralRF y sus scripts terminaron sin errores síncronos visibles. Después de las rutas más relevantes para estado —BURST y especialmente CONTINUOUS+STOP— el dispositivo continuó respondiendo correctamente.

Este resultado **no debe promoverse a PASS-RF**.

La Auditoría ya establecía que ACK no equivale a TX_DONE y que los resultados TX sin observador siguen siendo RF no establecida.

---

# 6. Qué queda sin validar en EV-12

No existe evidencia actual que demuestre:

* emisión física efectiva de TX_RAW;
* emisión física efectiva de TX_FRAME;
* cinco emisiones reales en TX_BURST;
* intervalo físico de 5000 µs;
* duración física exacta de continuous;
* cese físico de energía tras `TX_STOP`;
* frecuencia RF real;
* potencia RF real de 0 dBm;
* calidad espectral;
* framing sobre el aire;
* recepción por un segundo dispositivo;
* correspondencia byte por byte entre payload solicitado y señal recibida.

Estas cuestiones requieren receptor independiente o instrumentación y algunas se estudian explícitamente en EV posteriores.

---

# 7. Relación con EV-13

EV-12 demuestra principalmente que las primitivas TX son **controlables desde el host**.

EV-13 cambia la pregunta:

```text
EV-12:
“¿FeralRF acepta y controla las operaciones TX?”

EV-13:
“¿Qué señal RF produce físicamente cuando usamos modos de prueba?”
```

La transición es, por tanto:

```text
control lógico
        ↓
observación física/instrumentación
```

---

# EV-13 — CW y PRBS instrumentados

## 8. Objetivo según la Guía

EV-13 tiene como objetivo:

> verificar frecuencia, potencia relativa y patrón, no sólo comportamiento wire-level.

La ruta definida es:

```text
host
→ CMD_TX_TEST
→ RadioIF_runTxTest
→ RF
```

y las modalidades principales son:

```text
CW
PRBS15
PRBS32
STOP
```

La Guía indica expresamente que esta evaluación necesita:

* un segundo receptor para un smoke parcial;
* **analizador / frecuencímetro / power meter** para una validación física propiamente instrumentada;
* potencia inicial ≤ 0 dBm;
* detener cualquier modo continuo en ≤ 1 s;
* no usar PRBS9 BLE DTM en esta evaluación.

---

## 9. Qué debe demostrar EV-13

EV-13 no debería cerrarse simplemente porque:

```text
tx_cw() → ACK
```

o:

```text
tx_prbs() → ACK
```

La aceptación fuerte definida en la Guía es:

> **PASS-instrumento sólo con medición.**

El comportamiento wire-level queda como evidencia parcial.

Debemos comprobar físicamente al menos:

### CW

Durante la ventana TX:

```text
energía RF presente
frecuencia correcta
potencia relativa coherente
```

Después de STOP:

```text
energía desaparece
```

### PRBS

Durante la ventana:

```text
señal/patrón PRBS presente
```

según permita observarlo el instrumento disponible.

### STOP

Debe ser:

```text
efectivo
recuperable
idealmente idempotente
```

y no debe quedar una emisión persistente.

---

# 10. Instrumentación requerida

La Matriz clasifica EV-13 como una evaluación que necesita:

```text
DUT + RF-OBSERVER
```

y marca al observador independiente como **obligatorio** para la validación instrumentada. Una segunda placa FeralRF sólo permite un smoke parcial y no sustituye a un instrumento para medir frecuencia, potencia o pureza.

La Matriz también indica que equipo RF independiente es necesario para EV-13 y distingue:

```text
SDR
→ energía / frecuencia / posible observación compatible

analizador / power meter
→ potencia / espectro
```

---

# 11. Estado actual respecto a EV-13

Con los recursos que están documentados actualmente, no debemos asumir que EV-13 puede cerrarse por completo.

Tenemos:

```text
DUT-V3-FERAL
```

pero no está establecido en la evidencia de esta campaña que dispongamos ya de:

```text
analizador de espectro
frecuencímetro
power meter
RF-OBSERVER independiente apropiado
```

Por tanto, antes de transmitir CW o PRBS, EV-13 debe comenzar con una **fase de preparación**, no directamente con el comando de RF.

---

# 12. Procedimiento recomendado para comenzar EV-13

## Paso 1 — inspeccionar el script original

Antes de transmitir nada, abrir:

```powershell
Get-Content examples\lab\smoke_f22_tx_test.py
```

Debemos identificar exactamente:

* qué PHY/frecuencia configura;
* qué potencia solicita;
* qué modos PRBS utiliza;
* cómo llama `tx_cw()`;
* cómo llama `tx_prbs()`;
* cómo ejecuta STOP;
* cuánto tiempo mantiene las emisiones;
* qué considera PASS;
* qué espera del segundo receptor;
* qué cleanup realiza ante error.

Esto mantiene la misma disciplina que usamos correctamente en EV-12.

---

## Paso 2 — identificar la instrumentación disponible

Antes de ejecutar CW/PRBS debemos establecer qué observador real existe.

Las posibilidades no son equivalentes:

```text
AUX-V3-FERAL
→ smoke parcial

SDR
→ presencia de energía y frecuencia aproximada

Spectrum Analyzer
→ frecuencia, espectro, armónicos, nivel relativo

Power Meter
→ potencia

Frequency Counter
→ frecuencia
```

No debe llamarse `PASS-instrumento` a una prueba realizada únicamente con ACK o con una segunda placa que sólo detecte pérdida/recepción de paquetes.

---

## Paso 3 — preparar banco seguro

Según la Guía:

```text
potencia <= 0 dBm
duración <= 1 s inicialmente
```

CW y PRBS son modos de prueba RF persistentes hasta STOP.

Por ello debe existir antes de empezar:

```text
método de STOP conocido
método de recuperación manual por COM87
power-cycle disponible
```

Recordando además que `Radio.reset_device()` sigue bloqueado por KI-15 en nuestra enumeración.

---

## Paso 4 — ejecutar smoke sólo si tenemos observación adecuada

La Guía proporciona como prueba de dos placas:

```powershell
python examples\lab\smoke_f22_tx_test.py --tx-port COM_TX --rx-port COM_RX
```

pero este comando debe revisarse primero leyendo el script.

Con una sola placa, la Guía permite usar la API únicamente en un banco conducido y detener en ≤1 s.

---

# 13. Qué registrar en EV-13

Para que el resultado sea académicamente útil debemos conservar:

```text
comando exacto
modo CW/PRBS
frecuencia configurada
potencia solicitada
duración
instrumento usado
conexión/antena
span
RBW
frecuencia observada
pico observado
potencia medida o relativa
momento de STOP
estado después de STOP
stdout/stderr del host
```

La Guía menciona específicamente como evidencia:

```text
span / RBW
pico
potencia
tiempo
```

---

# 14. Criterio de aceptación de EV-13

No utilizaremos simplemente:

```text
PASS-control
```

como cierre de la evaluación.

Los posibles estados deberían interpretarse como:

```text
ACK solamente
→ PARTIAL / wire-level

ACK + segundo receptor
→ smoke físico parcial

medición con instrumento
→ candidato a PASS-instrumento
```

Para `PASS-instrumento` debemos comprobar al menos:

```text
frecuencia correcta
energía únicamente durante la ventana
STOP efectivo
potencia relativa coherente con la configuración
ausencia de anomalías graves observables
```

La Guía advierte además:

* posible problema de High-PA;
* discrepancia documental entre +5 y +14 dBm;
* posibilidad de armónicos;
* potencia no monótona;
* emisión persistente después de STOP.

---

# 15. Decisión antes de ejecutar EV-13

EV-12 puede considerarse cerrada como:

> **PASS-control 4/4; RF física pendiente.**

EV-13 **no debería empezar directamente transmitiendo CW**.

El siguiente paso correcto es:

```powershell
Get-Content examples\lab\smoke_f22_tx_test.py
```

y después determinar qué instrumentación real tenemos disponible.

Si no existe `RF-OBSERVER`/instrumentación suficiente, EV-13 podrá prepararse e incluso recorrerse parcialmente a nivel wire/control, pero no debe cerrarse como `PASS-instrumento`.

La Matriz es explícita en que EV-13 requiere `D+I` y que el observador independiente es obligatorio para la parte instrumentada.

````
