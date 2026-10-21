# Investigación pendiente para Codex — Limitaciones de TX_BURST, TX_CONTINUOUS y RX_STOP

**Proyecto:** CatSniffer — FeralRF
**Origen:** Evaluación experimental OP-08 y OP-09
**Prioridad:** Alta
**Estado:** Pendiente de auditoría de código
**Componentes involucrados:** Firmware CC1352P7, API Python de FeralRF y protocolo de comunicación entre ambos.

## 1. Objetivo

Realizar una auditoría técnica de las implementaciones actuales de `TX_BURST`, `TX_CONTINUOUS`, `TX_STOP` y `RX_STOP` para explicar las limitaciones observadas durante las pruebas OTA con IEEE 802.15.4.

La investigación debe determinar:

1. Cómo se implementan las transmisiones repetitivas.
2. Cómo se interpretan y aplican los intervalos de transmisión.
3. Qué significa realmente la confirmación de una transmisión programada.
4. Cómo se administran los estados de transmisión y recepción.
5. Por qué `RX_STOP` puede presentar un timeout después de recibir múltiples paquetes.
6. Si los comportamientos observados corresponden a defectos de implementación, restricciones de diseño o limitaciones de los procedimientos experimentales.

**No modificar el código inicialmente.** Primero se requiere un diagnóstico sustentado en el código y los resultados experimentales.

---

## 2. Evidencia experimental que motiva la investigación

Las pruebas se realizaron utilizando dos CatSniffer V3:

* **Transmisor:** CatSniffer #2, Cat-Bridge COM33.
* **Receptor:** CatSniffer #3, Cat-Bridge COM88.
* **PHY:** IEEE 802.15.4.
* **Canal:** 25.
* **Frecuencia nominal:** 2475 MHz.
* **Potencia TX solicitada:** 0 dBm.
* **Firmware reportado por GET_INFO:** `1.0.0`.

### 2.1. OP-08 — TX_BURST

Parámetros:

* Cantidad solicitada: 5 paquetes.
* Intervalo solicitado: 250 000 µs.
* Marcador: `A30133008803`.
* Ventana RX: 12 segundos.

Resultados:

```text
TX BURST PASS scheduled=5

packets_total=18
crc_ok=18
marker_hits=1

RX PROBE FAIL hits=1 min_hits=5
```

**Observación:** el transmisor reportó cinco transmisiones programadas, pero el receptor independiente identificó solamente un paquete con el marcador esperado.

Este comportamiento coincide con el antecedente histórico de OP-08.

### 2.2. OP-09A — TX_CONTINUOUS con intervalo positivo

Parámetros:

* `interval_us=250000`.
* `run_seconds=3`.
* Marcador: `A40133008804`.
* Ventana RX: 10 segundos.

Resultados:

```text
TX_CONTINUOUS ACK
TX_STOP ACK

packets_total=17
crc_ok=17
marker_hits=1

RX PROBE FAIL hits=1 min_hits=5
```

**Observación:** la transmisión continua fue aceptada y posteriormente detenida, pero solamente se identificó una recepción con el marcador esperado.

Un control posterior de cinco segundos registró cero coincidencias.

También existió una ejecución intermedia de diez segundos con una coincidencia, cuya relación temporal con una segunda ejecución TX no quedó confirmada. No debe utilizarse como prueba de transmisión posterior a `TX_STOP`.

### 2.3. OP-09B — TX_CONTINUOUS con intervalo cero

Parámetros:

* `interval_us=0`.
* `run_seconds=1`.
* Marcador: `A40233008804`.
* Ventana RX: 7 segundos.

Resultados:

```text
TX_CONTINUOUS ACK
TX_STOP ACK

packets_total=37
crc_ok=37
marker_hits=37
```

Los timestamps reportados muestran una separación media aproximada de **1118,7 µs** entre las 37 recepciones registradas.

Sin embargo, el receptor terminó con:

```text
[STEP] RX_STOP
[FAIL] Timeout waiting for response:
Response timeout (ignored 8 unexpected response(s), last=0x90)
```

Una ejecución posterior confirmó que el receptor podía inicializarse nuevamente, iniciar RX y detenerse correctamente.

El control posterior registró:

```text
packets_total=8
crc_ok=8
marker_hits=0
RX_STOP ACK
```

**Observación:** con intervalo cero se demostraron múltiples recepciones OTA, pero la ejecución principal presentó un fallo de confirmación de `RX_STOP`.

---

# 3. Investigación A — Temporización de TX_CONTINUOUS

**Prioridad: Alta**

## Problema observado

Con `interval_us=250000`, el receptor identificó una sola coincidencia.

Con `interval_us=0`, identificó 37 coincidencias.

Ambos modos fueron aceptados por el transmisor.

## Preguntas para Codex

1. ¿Cómo se interpreta `interval_us` en la API Python?
2. ¿Cómo se serializa este parámetro y cómo llega al CC1352P7?
3. ¿Qué componente implementa realmente el intervalo entre transmisiones?
4. ¿Se utilizan temporizadores, comandos RF programados, callbacks o un bucle de software?
5. ¿El intervalo se mide desde el inicio o desde la finalización de una transmisión?
6. ¿Qué ocurre después de completar el primer paquete?
7. ¿Existe algún estado que impida programar correctamente la siguiente transmisión?
8. ¿Qué diferencias de implementación existen entre `interval_us=0` e `interval_us>0`?
9. ¿Se comprueba el resultado de cada operación RF?
10. ¿Existen restricciones del driver RF o del dispositivo que puedan explicar el comportamiento?

## Evidencia requerida

Codex debe identificar:

* Archivos y funciones que procesan `TX_CONTINUOUS`.
* Recorrido del parámetro `interval_us`.
* Operaciones del driver RF utilizadas.
* Estados y transiciones relevantes.
* Tratamiento de errores y condiciones de terminación.

**Resultado esperado:** explicar por qué las dos configuraciones producen comportamientos OTA tan diferentes, o identificar con precisión qué información falta para determinarlo.

---

# 4. Investigación B — Relación entre TX_BURST y TX_CONTINUOUS

**Prioridad: Alta**

## Problema observado

Dos modalidades diferentes produjeron resultados similares con intervalos positivos:

| Operación       |  Intervalo |    Resultado OTA |
| --------------- | ---------: | ---------------: |
| `TX_BURST`      | 250 000 µs |              1/5 |
| `TX_CONTINUOUS` | 250 000 µs |              1/5 |
| `TX_CONTINUOUS` |       0 µs | 37 coincidencias |

La coincidencia sugiere que podría existir una limitación compartida.

No obstante, esto debe verificarse mediante el código.

## Preguntas para Codex

1. ¿`TX_BURST` y `TX_CONTINUOUS` utilizan la misma función de transmisión?
2. ¿Comparten un mecanismo de temporización?
3. ¿Comparten variables de estado, buffers, callbacks o estructuras RF?
4. ¿Cómo se administra la finalización de cada paquete?
5. ¿Existe una transición incorrecta entre transmisión activa, transmisión completada y siguiente transmisión?
6. ¿La primera transmisión utiliza una ruta diferente de las posteriores?
7. ¿Qué ocurre cuando el intervalo solicitado es mayor que la duración del paquete?
8. ¿Se liberan o reutilizan correctamente los recursos asociados a cada transmisión?

## Resultado esperado

Determinar si existe una causa común o si los resultados similares corresponden a problemas independientes.

**No asumir que ambas operaciones comparten una implementación defectuosa únicamente porque producen una coincidencia OTA.**

---

# 5. Investigación C — Significado de scheduled y los ACK

**Prioridad: Alta**

## Problema observado

Durante OP-08, el transmisor reportó:

```text
TX BURST PASS scheduled=5
```

Sin embargo, solamente se observó una recepción OTA.

## Preguntas para Codex

1. ¿Qué condición produce `scheduled=5`?
2. ¿Representa paquetes solicitados, aceptados, encolados, iniciados o completados?
3. ¿El firmware confirma cada transmisión individual?
4. ¿Se inspeccionan los resultados del driver RF?
5. ¿Puede devolverse un ACK aunque alguna transmisión posterior falle?
6. ¿Se distinguen errores de programación, ejecución y finalización?
7. ¿La API Python interpreta correctamente la respuesta del firmware?

## Resultado esperado

Establecer la semántica exacta de las confirmaciones de transmisión.

Si `scheduled` representa únicamente operaciones solicitadas o programadas, documentarlo explícitamente.

No modificar su significado sin analizar previamente la compatibilidad con la API existente.

---

# 6. Investigación D — Timeout de RX_STOP

**Prioridad: Alta**

## Problema observado

Durante OP-09B, después de registrar 37 paquetes, el receptor presentó:

```text
Response timeout (ignored 8 unexpected response(s), last=0x90)
```

Posteriormente, una nueva sesión RX funcionó correctamente.

## Preguntas para Codex

1. ¿Qué representa `0x90` en el protocolo FeralRF?
2. ¿Corresponde a un evento asíncrono de recepción?
3. ¿Cómo distingue la API Python entre respuestas a
