# Reporte complementario — Selección de banda, RF Switch y verificación de frecuencia en CatSniffer

**Proyecto:** CatSniffer — FeralRF
**Documento relacionado:** `06 - Procedimientos IEEE, TX y modos de prueba`
**Procedimiento relacionado:** Prerrequisitos comunes y OP-04
**Hardware:** CatSniffer V3
**Firmware RP2040 reportado:** `v3.1.0.1`
**Firmware CC1352P7 reportado:** `feralrf_cc1352 (custom)`
**Estado:** Verificación lógica de bandas completada; verificación física del RF Switch pendiente.

---

## 1. Objetivo

Documentar cómo seleccionar, consultar e interpretar la banda de radiofrecuencia utilizada por CatSniffer durante las pruebas de FeralRF.

En particular, se busca responder:

1. ¿Qué diferencia existe entre frecuencia, banda y ruta RF?
2. ¿Cómo seleccionar las rutas de 2.4 GHz y Sub-GHz?
3. ¿Cómo consultar qué banda está seleccionada?
4. ¿Qué significan los valores `Band: 0` y `Band: 1`?
5. ¿Cómo se relaciona la selección de banda con la configuración del CC1352P7?
6. ¿Qué podemos demostrar mediante comandos y qué requiere instrumentación adicional?

Este reporte complementa los prerrequisitos del documento 06 y registra una comprobación experimental realizada durante su reproducción.

---

## 2. Conceptos fundamentales

### 2.1. Frecuencia de operación

La frecuencia de operación es la frecuencia de la señal de radio utilizada para transmitir o recibir información.

Se expresa normalmente en:

* MHz: megahercios.
* GHz: gigahercios.

Ejemplos:

| Frecuencia | Región de frecuencia | Aplicación posible                 |
| ---------- | -------------------- | ---------------------------------- |
| 433 MHz    | Sub-GHz              | Sistemas de radio de corto alcance |
| 868 MHz    | Sub-GHz              | Sistemas de radio regionales       |
| 915 MHz    | Sub-GHz              | Sistemas de radio regionales       |
| 2405 MHz   | 2.4 GHz              | IEEE 802.15.4, canal 11            |
| 2475 MHz   | 2.4 GHz              | IEEE 802.15.4, canal 25            |
| 2480 MHz   | 2.4 GHz              | IEEE 802.15.4, canal 26            |

Estas frecuencias son ejemplos técnicos; no implican que todas estén habilitadas o validadas en el firmware FeralRF utilizado.

### 2.2. Banda de frecuencia

Una banda es un intervalo de frecuencias.

En el contexto de CatSniffer se utilizan dos categorías principales:

**2.4 GHz**

Agrupa las frecuencias de operación cercanas a 2.4 GHz, como las empleadas por IEEE 802.15.4 y Bluetooth Low Energy.

**Sub-GHz**

Agrupa frecuencias inferiores a 1 GHz, como 433, 868, 902 y 915 MHz.

**Importante:** Sub-GHz no representa una frecuencia única. Seleccionar esta banda no significa que el transceptor esté trabajando automáticamente a 868 o 915 MHz.

### 2.3. Ruta RF y RF Switch

CatSniffer dispone de un sistema de selección de ruta RF que permite dirigir las señales hacia el trayecto de radiofrecuencia correspondiente.

En este sistema deben distinguirse:

* **RP2040:** administra comandos de Cat-Shell y la selección lógica de banda.
* **CC1352P7:** ejecuta el firmware FeralRF y configura el PHY de radio.
* **RF Switch:** selecciona físicamente el trayecto de radiofrecuencia según sus señales de control.
* **Antena:** debe ser compatible con la frecuencia utilizada.

La selección de banda y la configuración del transceptor son operaciones relacionadas, pero diferentes.

**Una ruta RF seleccionada no demuestra, por sí sola, que el transceptor esté transmitiendo o recibiendo en una frecuencia determinada.**

---

## 3. Interfaces utilizadas

Durante las pruebas se identificaron las siguientes interfaces:

| CatSniffer    | Interfaz   | Puerto | Función                  |
| ------------- | ---------- | ------ | ------------------------ |
| CatSniffer #2 | Cat-Shell  | COM35  | Comandos al RP2040       |
| CatSniffer #2 | Cat-Bridge | COM33  | Comunicación con FeralRF |
| CatSniffer #3 | Cat-Shell  | COM87  | Comandos al RP2040       |
| CatSniffer #3 | Cat-Bridge | COM88  | Comunicación con FeralRF |

**Advertencia:** estos puertos corresponden al entorno experimental utilizado. Pueden cambiar después de reconectar los dispositivos o utilizar otra computadora.

No debe asumirse que el número del puerto Cat-Bridge puede calcularse a partir del puerto Cat-Shell.

### 3.1. Velocidades de comunicación

Para las pruebas realizadas:

* Cat-Shell: `115200 baud`.
* Cat-Bridge/FeralRF: `921600 baud`.

Estos valores corresponden a interfaces diferentes.

---

## 4. Selección de banda mediante Cat-Shell

El RP2040 reconoce los siguientes comandos:

| Comando  | Solicitud                  | Respuesta observada |
| -------- | -------------------------- | ------------------- |
| `band1`  | Seleccionar 2.4 GHz        | `2.4GHz Band`       |
| `band2`  | Seleccionar Sub-GHz        | `SUB-GHz Band`      |
| `status` | Consultar estado reportado | Incluye `Band: N`   |

Los comandos se envían mediante la interfaz serial Cat-Shell.

### 4.1. Seleccionar 2.4 GHz

Ejemplo para CatSniffer #2, Cat-Shell COM35:

```powershell
python -c "import serial,time; s=serial.Serial('COM35',115200,timeout=1); s.write(b'band1\r\n'); time.sleep(.3); print(s.read_all().decode(errors='replace')); s.close()"
```

**Salida observada:**

```text
2.4GHz Band
```

**Interpretación:**

El RP2040 recibió y procesó una solicitud de selección de la ruta de 2.4 GHz, según la respuesta del Shell.

Esta respuesta no constituye una medición física del RF Switch.

### 4.2. Seleccionar Sub-GHz

```powershell
python -c "import serial,time; s=serial.Serial('COM35',115200,timeout=1); s.write(b'band2\r\n'); time.sleep(.3); print(s.read_all().decode(errors='replace')); s.close()"
```

**Salida observada:**

```text
SUB-GHz Band
```

**Interpretación:**

El RP2040 reporta haber procesado la solicitud de selección de la ruta Sub-GHz.

Esto no configura automáticamente el CC1352P7 a una frecuencia específica.

---

## 5. Consulta de la banda seleccionada

### 5.1. Comando `status`

Para consultar el estado reportado por el RP2040 sin solicitar un cambio de banda:

```powershell
python -c "import serial,time; s=serial.Serial('COM35',115200,timeout=1); s.reset_input_buffer(); s.write(b'status\r\n'); time.sleep(.5); print(s.read_all().decode(errors='replace')); s.close()"
```

**Salida real observada después de seleccionar `band1`:**

```text
Mode: 0, Band: 0, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes
```

El campo relevante es:

```text
Band: 0
```

La correspondencia de este valor fue comprobada mediante el experimento descrito a continuación.

---

## 6. Experimento de correspondencia entre comandos y estado de banda

### 6.1. Objetivo

Determinar experimentalmente qué valores del campo `Band` corresponden a los comandos `band1` y `band2`.

### 6.2. Condiciones

* Dispositivo: CatSniffer #2.
* Interfaz: Cat-Shell.
* Puerto: COM35.
* Velocidad: 115200 baud.
* Firmware RP2040: `v3.1.0.1`.
* Herramienta: Python con `pyserial`, ejecutado desde PowerShell.

### 6.3. Prueba A — Selección Sub-GHz

**Comando ejecutado:**

```powershell
python -c "import serial,time; s=serial.Serial('COM35',115200,timeout=1); s.reset_input_buffer(); s.write(b'band2\r\n'); time.sleep(.3); print('=== BAND2 ==='); print(s.read_all().decode(errors='replace')); s.write(b'status\r\n'); time.sleep(.5); print('=== STATUS ==='); print(s.read_all().decode(errors='replace')); s.close()"
```

**Salida real:**

```text
=== BAND2 ===
SUB-GHz Band

=== STATUS ===
Mode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes
```

**Interpretación:**

Después de ejecutar `band2`, el RP2040 reportó:

```text
Band: 1
```

Por tanto, en la versión examinada, el valor lógico `1` corresponde a la selección Sub-GHz.

**Resultado:** PASS.

### 6.4. Prueba B — Selección 2.4 GHz

**Comando ejecutado:**

```powershell
python -c "import serial,time; s=serial.Serial('COM35',115200,timeout=1); s.reset_input_buffer(); s.write(b'band1\r\n'); time.sleep(.3); print('=== BAND1 ==='); print(s.read_all().decode(errors='replace')); s.write(b'status\r\n'); time.sleep(.5); print('=== STATUS ==='); print(s.read_all().decode(errors='replace')); s.close()"
```

**Salida real:**

```text
=== BAND1 ===
2.4GHz Band

=== STATUS ===
Mode: 0, Band: 0, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)
CC1352 loss: uart_overrun=0, ring_dropped=0 bytes
```

**Interpretación:**

Después de ejecutar `band1`, el RP2040 reportó:

```text
Band: 0
```

Por tanto, en la versión examinada, el valor lógico `0` corresponde a la selección de 2.4 GHz.

**Resultado:** PASS.

### 6.5. Resultado consolidado

| Selección | Respuesta      | Estado reportado | Verificación |
| --------- | -------------- | ---------------- | ------------ |
| `band1`   | `2.4GHz Band`  | `Band: 0`        | Confirmada   |
| `band2`   | `SUB-GHz Band` | `Band: 1`        | Confirmada   |

**Conclusión experimental:**

En la CatSniffer examinada, con firmware RP2040 `v3.1.0.1`, se verificó que:

* `Band: 0` corresponde a la selección lógica de 2.4 GHz.
* `Band: 1` corresponde a la selección lógica Sub-GHz.

La prueba demuestra la correspondencia entre comandos y estado reportado.

No demuestra directamente las tensiones de control del RF Switch ni la frecuencia de una señal transmitida.

---

## 7. ¿Por qué aparecían ambas bandas en una misma salida?

El procedimiento original del documento 06 utiliza una secuencia como esta:

```python
s.write(b'band2\r\n')
time.sleep(.3)

s.write(b'band1\r\n')
time.sleep(.3)

print(s.read_all().decode(errors='replace'))
```

**Salida real:**

```text
SUB-GHz Band

2.4GHz Band
```

Esto sucede porque se ejecutan dos comandos consecutivos y sus respuestas se leen juntas.

La secuencia es:

1. Solicitar Sub-GHz mediante `band2`.
2. Solicitar 2.4 GHz mediante `band1`.
3. Leer las respuestas acumuladas.

La aparición de ambas respuestas no significa que ambas rutas estén activas simultáneamente.

La última selección solicitada es `band1`.

Para confirmar el estado lógico final, debe utilizarse `status`.

### Recomendación documental

Después de la secuencia `band2` → `band1`, añadir una consulta `status` y comprobar que aparece:

```text
Band: 0
```

De esta forma, el prerrequisito queda explícitamente verificado.

---

## 8. Diferencia entre seleccionar una banda y configurar una frecuencia

Este es el punto más importante para interpretar correctamente las pruebas de FeralRF.

Existen dos niveles de configuración.

### 8.1. Nivel 1 — Selección de ruta RF

**Procesador:** RP2040.
**Interfaz:** Cat-Shell.
**Comandos:** `band1`, `band2`.

Ejemplo:

```text
band1
```

Resultado:

```text
Band: 0
```

Esto indica que el RP2040 reporta la ruta de 2.4 GHz como seleccionada.

**No determina el canal IEEE 802.15.4 ni una frecuencia central exacta.**

### 8.2. Nivel 2 — Configuración del transceptor

**Procesador que ejecuta FeralRF:** CC1352P7.
**Interfaz desde la PC:** Cat-Bridge.
**Comandos del protocolo:** `SET_PHY`, `SET_CHANNEL`.

Ejemplo conceptual:

```text
SET_PHY → IEEE_802_15_4
SET_CHANNEL → 25
```

Estos comandos configuran el PHY y el canal solicitados al firmware del CC1352P7.

Para el PHY IEEE 802.15.4 de 2.4 GHz, el canal 25 corresponde nominalmente a:

**2475 MHz = 2.475 GHz.**

Esta frecuencia proviene de la asignación de canales IEEE 802.15.4; no fue medida directamente durante OP-04.

### 8.3. Fórmula de frecuencia para IEEE 802.15.4 en 2.4 GHz

Para los canales 11 a 26:

$$
f_c = 2405 + 5(c-11)\ \text{MHz}
$$

Donde:

* \(f_c\): frecuencia central nominal.
* \(c\): número de canal IEEE 802.15.4.

Para el canal 25:

$$
f_c = 2405 + 5(25-11)
$$

$$
\boxed{f_c = 2475\ \text{MHz}}
$$

Por tanto:

| Parámetro                  | Valor                  |
| -------------------------- | ---------------------- |
| PHY                        | IEEE 802.15.4, 2.4 GHz |
| Canal                      | 25                     |
| Frecuencia central nominal | 2475 MHz               |
| Banda                      | 2.4 GHz                |
| Selección Cat-Shell        | `band1`                |
| Estado lógico esperado     | `Band: 0`              |

**Advertencia:** esta fórmula es específica de la canalización IEEE 802.15.4 de 2.4 GHz. No debe utilizarse para calcular frecuencias Sub-GHz ni canales de otros PHY.

---

## 9. Ejemplo práctico: verificar la configuración IEEE 802.15.4

Durante la reproducción de OP-04 se ejecutó:

```powershell
python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
```

El comando se ejecutó desde:

```text
FeralRF\python
```

### 9.1. Salida real

```text
FeralRF IEEE 802.15.4 Smoke Test
================================
port=COM88 baudrate=921600 phy=4 channel=25 duration=30.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY IEEE_802_15_4
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=54
[ OK ] first: ts=860363944us ch=25 rssi=-69 lqi=60 crc_ok=True len=52
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS
```

### 9.2. Interpretación

El resultado permite establecer que:

1. La comunicación con el firmware FeralRF funcionó.
2. El firmware aceptó configurar el PHY IEEE 802.15.4.
3. El firmware aceptó configurar el canal 25.
4. La recepción comenzó correctamente.
5. Se reportaron 54 paquetes durante 30 segundos.
6. El primer paquete mostrado tenía `crc_ok=True`.
7. La recepción se detuvo correctamente.

Combinando esta ejecución con la consulta de Cat-Shell, tenemos evidencia de que:

* La ruta de 2.4 GHz estaba seleccionada lógicamente.
* El CC1352P7 aceptó la configuración IEEE 802.15.4, canal 25.
* Se recibieron paquetes con esa configuración.

### 9.3. ¿Qué frecuencia estaba utilizando?

**Frecuencia nominal configurada:** 2475 MHz.

**Banda:** 2.4 GHz.

**Ruta RF lógica:** `band1`, equivalente a `Band: 0` en el firmware RP2040 examinado.

**Frecuencia físicamente medida:** no disponible.

No debe confundirse la frecuencia calculada a partir del canal con una medición instrumental de frecuencia.

---

## 10. ¿Cómo verificar una frecuencia Sub-GHz?

En Sub-GHz, el procedimiento requiere conocer la configuración concreta del PHY o preset utilizado por FeralRF.

No es correcto interpretar que:

```text
Band: 1
```

significa automáticamente:

```text
868 MHz
```

o:

```text
915 MHz
```

El valor `Band: 1` únicamente identifica la ruta Sub-GHz seleccionada.

Para establecer la frecuencia nominal de una prueba Sub-GHz se requiere:

1. Seleccionar `band2`.
2. Confirmar `Band: 1` mediante `status`.
3. Identificar el PHY o preset utilizado por FeralRF.
4. Identificar cómo ese PHY interpreta el canal solicitado.
5. Obtener la frecuencia central configurada o calculada a partir de sus parámetros.
6. Comprobar que la configuración fue aceptada por el firmware.
7. Realizar una medición RF independiente si se necesita confirmar la frecuencia física.

**Limitación actual:** este reporte no incluye una ejecución Sub-GHz que permita validar un comando concreto de frecuencia y su salida.

No debe inventarse una correspondencia entre números de canal y frecuencias Sub-GHz sin verificar el preset y la implementación correspondiente.

Esta comprobación se abordará durante los procedimientos del documento 07.

---

## 11. Interpretación de `Radio: LoRa`

Durante las consultas `status`, ambas CatSniffer reportaron:

```text
Radio: LoRa, LoRa: initialized, LoRa Mode: Stream
```

Esto ocurrió incluso después de una ejecución satisfactoria de recepción IEEE 802.15.4 mediante FeralRF.

No debe concluirse que el CC1352P7 estaba utilizando modulación LoRa.

La interpretación de trabajo es que estos campos pertenecen al estado que mantiene Cat-Shell para otro subsistema de radio.

**Pendiente:** verificar en el código fuente del RP2040 qué variables y componentes representan los campos `Radio`, `LoRa` y `LoRa Mode`.

Hasta completar esa revisión, estos campos no deben utilizarse como evidencia del PHY activo en FeralRF.

---

## 12. Niveles de verificación

Es necesario distinguir tres niveles.

### Nivel A — Estado lógico del RP2040

**Herramienta:** Cat-Shell.

**Comandos:**

```text
band1
band2
status
```

**Permite comprobar:**

* Qué banda solicita el software.
* Qué banda reporta el RP2040.
* Si la selección cambia de manera consistente.

**Estado:** verificado experimentalmente.

### Nivel B — Configuración funcional del CC1352P7

**Herramienta:** FeralRF mediante Cat-Bridge.

**Comandos del protocolo:**

```text
SET_PHY
SET_CHANNEL
RX_START
RX_STOP
```

**Permite comprobar:**

* Qué PHY y canal se solicitaron.
* Si el firmware aceptó la configuración.
* Si existe recepción funcional bajo esas condiciones.

**Estado:** verificado para IEEE 802.15.4, canal 25, durante OP-04.

**Limitación:** los ACK no equivalen a una lectura independiente de los registros RF ni a una medición de frecuencia.

### Nivel C — Verificación física

**Instrumentación posible:**

* Analizador de espectro.
* Receptor SDR calibrado para la banda correspondiente.
* Instrumentación de medición de señales de control del RF Switch.

**Permite comprobar, según el instrumento y el montaje:**

* Frecuencia central real de una transmisión.
* Presencia de energía RF.
* Comportamiento del trayecto de señal.
* Estados eléctricos de las señales de control del switch.

**Estado:** pendiente.

La selección del instrumento dependerá de la magnitud que se quiera comprobar. Un analizador de espectro puede medir la señal RF, pero no demuestra por sí solo qué GPIO activó el RP2040.

---

## 13. Procedimiento recomendado antes de cada experimento

### Para IEEE 802.15.4 en 2.4 GHz

1. Identificar el puerto Cat-Shell correspondiente.
2. Ejecutar `band1`.
3. Consultar `status`.
4. Confirmar `Band: 0`.
5. Abrir Cat-Bridge.
6. Configurar el PHY IEEE 802.15.4.
7. Configurar el canal requerido.
8. Determinar la frecuencia nominal correspondiente.
9. Ejecutar la prueba.
10. Registrar resultados y limitaciones.

### Para Sub-GHz

1. Identificar el puerto Cat-Shell correspondiente.
2. Ejecutar `band2`.
3. Consultar `status`.
4. Confirmar `Band: 1`.
5. Abrir Cat-Bridge.
6. Seleccionar el PHY o preset Sub-GHz.
7. Identificar la frecuencia central que corresponde a sus parámetros.
8. Ejecutar la prueba.
9. Registrar los resultados.
10. No declarar frecuencia física validada sin evidencia instrumental o equivalente.

---

## 14. Hallazgos para corregir el documento 06

Se recomienda incorporar las siguientes aclaraciones a sus prerrequisitos comunes.

**Hallazgo 1 — Respuestas acumuladas**

La secuencia `band2` → `band1` imprime las respuestas de ambas selecciones. Esto no implica activación simultánea de las dos rutas RF.

**Hallazgo 2 — Consulta del estado**

El comando `status` permite consultar la banda lógica reportada sin solicitar una nueva selección.

**Hallazgo 3 — Correspondencia verificada**

Para la CatSniffer examinada con RP2040 `v3.1.0.1`:

| Campo     | Banda   |
| --------- | ------- |
| `Band: 0` | 2.4 GHz |
| `Band: 1` | Sub-GHz |

**Hallazgo 4 — Banda y frecuencia no son equivalentes**

`Band: 0` no indica por sí solo el canal IEEE 802.15.4.

`Band: 1` no identifica por sí solo una frecuencia Sub-GHz específica.

**Hallazgo 5 — Doble configuración**

La ruta RF se selecciona mediante Cat-Shell, mientras que el PHY y canal del CC1352P7 se configuran mediante FeralRF.

**Hallazgo 6 — Límite de validación**

Las respuestas del Shell y los ACK de FeralRF no constituyen mediciones físicas del RF Switch o de la frecuencia de transmisión.

---

## 15. Conclusiones

**Conclusión C1 — Confirmada experimentalmente**

Los comandos `band1` y `band2` modifican de manera consistente el estado lógico de banda reportado por el RP2040.

**Conclusión C2 — Confirmada experimentalmente**

En la versión RP2040 `v3.1.0.1` examinada:

* `Band: 0` corresponde a 2.4 GHz.
* `Band: 1` corresponde a Sub-GHz.

**Conclusión C3 — Confirmada funcionalmente**

La CatSniffer conectada mediante COM88 recibió 54 paquetes durante OP-04, utilizando el PHY IEEE 802.15.4 y canal 25.

**Conclusión C4 — Derivada de la canalización IEEE 802.15.4**

El canal 25 de IEEE 802.15.4 en 2.4 GHz corresponde a una frecuencia central nominal de 2475 MHz.

**Conclusión C5 — Limitación**

No se ha medido físicamente la frecuencia de operación ni verificado mediante instrumentación el estado eléctrico del RF Switch.

**Conclusión C6 — Pendiente**

Debe revisarse el código del RP2040 para establecer la correspondencia exacta entre `Band`, las señales GPIO y el componente RF Switch identificado en el esquemático.

---

## 16. Fuentes y procedencia

### Documentación del proyecto

* `06 - Procedimientos IEEE, TX y modos de prueba`, sección **Prerrequisitos comunes**.
* `06 - Procedimientos IEEE, TX y modos de prueba`, sección **OP-04 — Recibir IEEE 802.15.4 en canal 25**.
* `05 - Manual práctico de uso de FeralRF`, procedimiento OP-01.
* Script: `FeralRF/python/examples/smoke_phy4_ieee154.py`.

### Evidencia experimental de esta reproducción

* Ejecución `band2` + `status` mediante COM35.
* Ejecución `band1` + `status` mediante COM35.
* Consultas `status` mediante COM35 y COM87.
* Ejecución OP-04 mediante COM88.
* Salidas originales conservadas en la conversación de validación.

### Referencias técnicas pendientes de contrastar

* Código fuente de Cat-Shell que implementa `band1`, `band2` y `status`.
* Esquemático de la revisión física de CatSniffer utilizada.
* Datasheet del RF Switch correspondiente.
* Implementación FeralRF de `SET_PHY` y `SET_CHANNEL`.
* Especificación de canalización IEEE 802.15.4 aplicable al PHY de 2.4 GHz.

Estas referencias deben identificarse por versión, revisión, ruta de código o página antes de utilizarlas como evidencia de implementación.

---

**Estado final del reporte:** Verificación lógica de banda completada. Correspondencia de frecuencia nominal IEEE documentada. Verificación física y trazabilidad completa con código y esquemático pendientes.
