# Validación RF — Resultados de RF-01, RF-03 y recuperación

**Proyecto:** CatSniffer V3 — FeralRF
**Fecha de ejecución:** 2026-10-21
**Tipo de documento:** Reporte experimental complementario
**Estado:** Campaña detenida por incidencia recurrente de `RX_STOP`
**Procedimiento rector:** `RF_VALIDATION_RUNBOOK.md`
**Naturaleza de la evidencia:** Comunicación OTA observada mediante dos CatSniffer; sin instrumentación RF independiente.

---

## 1. Objetivo y alcance

Registrar los resultados obtenidos durante la campaña de validación del 21 de octubre de 2026, contrastándolos con las ejecuciones históricas disponibles.

La campaña se concentró en:

* **RF-01:** cambio de canal IEEE 802.15.4.
* **RF-03:** transición entre IEEE 802.15.4 y Sub-GHz.
* **Procedimientos de recuperación:** preflight y control IEEE después de un reinicio físico.

Este reporte documenta tanto las capacidades observadas como los bloqueos encontrados.

No constituye una validación completa de las capacidades Sub-GHz ni un diagnóstico definitivo de la causa de los timeouts.

## 2. Entorno de pruebas

### 2.1 Hardware y comunicación

Se utilizaron dos CatSniffer V3 con CC1352P7.

| Función | Dispositivo   | Cat-Bridge | Cat-Shell |
| ------- | ------------- | ---------- | --------- |
| TX      | CatSniffer #2 | `COM33`    | `COM35`   |
| RX      | CatSniffer #3 | `COM88`    | `COM87`   |

Identificadores USB:

* TX: `E661A89753565932`
* RX: `E661A89753656A35`

Estos identificadores permiten distinguir físicamente los dos dispositivos USB.

### 2.2 Versiones observadas

| Componente                     | Versión                |
| ------------------------------ | ---------------------- |
| Firmware FeralRF del CC1352P7  | `1.0.0`                |
| Capacidades reportadas         | `7`                    |
| Firmware Cat-Shell del RP2040  | `v3.1.0.1`             |
| Git Cat-Shell                  | `c0cd5a4 (clean)`      |
| Fecha de compilación Cat-Shell | `2026-09-04T00:40:22Z` |
| Plataforma reportada           | `v3 RP2040 CC1352P7`   |

El campo `serial='464552414c524631'` devuelto por FeralRF corresponde al marcador ASCII `FERALRF1`. No debe utilizarse como identificador único de hardware.

Las respuestas `Band: 0` y `Radio: LoRa` de Cat-Shell describen estados lógicos reportados por el RP2040. No demuestran por sí mismas la ruta física RF ni la modulación utilizada por el CC1352P7.

### 2.3 Ubicación de las evidencias

Directorio de ejecución:

```text
FeralRF/validation_runs/manual_20261021_130015/
```

Registros relevantes:

```text
00_preflight.jsonl
01_rf01_ieee_channel_switch.jsonl
02_post_recovery_preflight.jsonl
03_post_recovery_rf01_control.jsonl
04_rf03_cross_phy_switch.jsonl
05_post_rf03_recovery_preflight.jsonl
06_post_rf03_recovery_control.jsonl
```

Los archivos JSONL son la evidencia primaria de esta campaña.

---

## 3. RF-01 — Cambio de canal IEEE 802.15.4

### 3.1 Objetivo

Evaluar la recepción entre dos dispositivos al cambiar la configuración de canal IEEE 802.15.4.

Secuencia:

1. TX canal 25 / RX canal 25.
2. TX canal 25 / RX canal 20.
3. TX canal 20 / RX canal 20.
4. TX canal 25 / RX canal 25.

El criterio positivo de recepción era obtener al menos **9 de 10 marcadores con CRC válido** en cada etapa de canales coincidentes.

La etapa de canales diferentes debía registrar cero marcadores correspondientes al transmisor de prueba.

También se requería finalizar correctamente la recepción mediante `RX_STOP`.

### 3.2 Resultados observados

| Etapa                | TX/RX | Marcador   | Recepción válida | `RX_STOP` |
| -------------------- | ----- | ---------- | ---------------- | --------- |
| `CONTROL_CH25`       | 25/25 | `C1012501` | 10/10            | ACK       |
| `MISMATCH_TX25_RX20` | 25/20 | `C1012502` | 0                | ACK       |
| `MATCHED_CH20`       | 20/20 | `C1012001` | 10/10            | ACK       |
| `RETURN_CH25`        | 25/25 | `C1012503` | 9/10             | Timeout   |

Los cuatro escenarios alcanzaron sus respectivos criterios de recepción.

En particular:

* La configuración 25/25 permitió recibir los marcadores esperados.
* La configuración 25/20 no produjo coincidencias del marcador de prueba.
* La configuración 20/20 permitió recibir los marcadores esperados.
* El regreso a 25/25 volvió a producir recepción válida.

Las diez transmisiones de cada etapa recibieron ACK de la interfaz de control.

Sin embargo, la última etapa terminó con:

```text
RX_STOP_CLEANUP_ERROR
TimeoutError(
  'Response timeout (ignored 4 unexpected response(s), last=0x90)'
)
```

El script registró un evento `FATAL` y no consideró completada la última etapa.

**Clasificación global: `BLOCKED`.**

### 3.3 Comparación con la evidencia histórica

| Aspecto                     | RF-01 histórico | RF-01 del 21/10/2026    |
| --------------------------- | --------------- | ----------------------- |
| Recepción inicial, canal 25 | 10/10           | 10/10                   |
| `RX_STOP` inicial           | Timeout         | ACK                     |
| Etapa de canales diferentes | No alcanzada    | Ejecutada: 0 marcadores |
| Recepción en canal 20       | No alcanzada    | 10/10                   |
| Regreso al canal 25         | No alcanzado    | 9/10                    |
| Bloqueo por `RX_STOP`       | Etapa inicial   | Etapa final             |
| Resultado global            | Bloqueado       | `BLOCKED`               |

**Diferencia principal:** la ejecución actual avanzó mucho más que la histórica.

La evidencia nueva respalda el funcionamiento del cambio de canal IEEE 802.15.4 dentro de una misma sesión de pruebas.

No obstante, no permite declarar RF-01 como `PASS`, porque falló el procedimiento de finalización.

### 3.4 Conclusión específica

**Capacidad observada:** cambio funcional entre canales IEEE 25 y 20, incluyendo retorno al canal inicial, con recepción OTA y control negativo coherentes.

**Limitación observada:** `RX_STOP` no termina de manera fiable en todas las etapas.

**Pendiente:** determinar la causa del timeout y repetir el procedimiento completo bajo condiciones controladas.

---

## 4. Primera recuperación — Posterior a RF-01

Tras el bloqueo de RF-01 se realizó una desconexión física de ambos dispositivos durante al menos 15 segundos, seguida de reconexión.

### 4.1 Preflight

Archivo:

```text
02_post_recovery_preflight.jsonl
```

Resultados:

* Los cuatro puertos COM conservaron su asignación.
* Ambos Cat-Shell respondieron.
* Ambas interfaces FeralRF respondieron.
* `uart_overrun=0` en ambos dispositivos.
* `ring_dropped=0` en ambos dispositivos.
* Los contadores de recepción reportados por FeralRF estaban en cero.
* Las desconexiones de la API finalizaron correctamente.

**Resultado: `PASS`.**

### 4.2 Control IEEE de recuperación

Archivo:

```text
03_post_recovery_rf01_control.jsonl
```

Configuración:

* PHY: `IEEE_802_15_4`.
* Canal TX/RX: 25.
* Marcador: `C10125F0`.
* Intentos TX: 10.
* Umbral: ≥9 marcadores con CRC válido.

Resultados:

| Indicador                 | Resultado |
| ------------------------- | --------- |
| TX ACK                    | 10/10     |
| Marcadores con CRC válido | 10/10     |
| `RX_START`                | ACK       |
| `RX_STOP`                 | ACK       |
| Desconexión TX/RX         | Correcta  |
| Resultado global          | `PASS`    |
| Código de salida          | `0`       |

### 4.3 Interpretación

El reinicio físico permitió recuperar una ejecución completa y satisfactoria del control IEEE.

Esto demuestra recuperación operativa en esa ejecución, pero **no demuestra que el power-cycle corrija permanentemente la causa del problema**.

Tampoco permite concluir que el origen esté necesariamente en el firmware del CC1352P7.

---

## 5. RF-03 — Transición IEEE 802.15.4 ↔ Sub-GHz

### 5.1 Objetivo

Evaluar la siguiente secuencia de PHY:

```text
IEEE_802_15_4_CH25
        ↓
wisun_915_fsk_50k
        ↓
IEEE_802_15_4_CH25
```

El objetivo era comprobar recepción antes y después de los cambios, además de verificar la correcta finalización de las etapas.

### 5.2 Resultado observado

Archivo:

```text
04_rf03_cross_phy_switch.jsonl
```

La ejecución comenzó correctamente:

* `band1` fue aceptado por ambos Cat-Shell.
* Ambas interfaces FeralRF se inicializaron.
* `RX_START` recibió ACK.
* Se registraron 10/10 ACK de transmisión.

Durante la etapa `IEEE_INITIAL` se observaron:

| Indicador                 | Resultado  |
| ------------------------- | ---------- |
| Canal IEEE                | 25         |
| Marcador                  | `C3032501` |
| Intentos TX               | 10         |
| TX ACK                    | 10/10      |
| Paquetes observados       | 12         |
| Marcadores con CRC válido | **7/10**   |
| Paquetes sin marcador     | 5          |
| Umbral exigido            | ≥9/10      |
| `RX_STOP`                 | Timeout    |

El script registró:

```text
RX_STOP_CLEANUP_ERROR
TimeoutError(
  'Response timeout (ignored 3 unexpected response(s), last=0x90)'
)
```

Posteriormente:

```text
FATAL
TimeoutError(
  'Response timeout (ignored 5 unexpected response(s), last=0x90)'
)
```

Y finalmente:

```json
{
  "completed_stages": [],
  "event": "RESULT",
  "result": "BLOCKED",
  "recovery_required": true
}
```

Código de salida:

```text
3
```

### 5.3 Interpretación de los paquetes recibidos

Los siete paquetes identificados mediante el marcador `C3032501` presentaron:

```text
crc_ok = true
```

Por tanto, se observó recepción IEEE del transmisor de prueba.

Los otros cinco paquetes tenían contenido diferente y niveles RSSI distintos de los marcadores de prueba.

Esto evidencia actividad adicional en el canal 25, pero no permite establecer que dicha actividad haya causado:

* La ausencia de tres marcadores.
* El timeout de `RX_STOP`.
* El bloqueo global de RF-03.

No se dispone de evidencia suficiente para atribuir causalidad.

### 5.4 Comparación histórica

| Aspecto                | RF-03 histórico | RF-03 actual |
| ---------------------- | --------------- | ------------ |
| Recepción IEEE inicial | 10/10           | 7/10         |
| Umbral ≥9/10           | Cumplido        | No cumplido  |
| `RX_STOP` inicial      | Timeout         | Timeout      |
| Etapa Sub-GHz          | No alcanzada    | No alcanzada |
| Regreso a IEEE         | No alcanzado    | No alcanzado |
| Resultado              | Bloqueado       | `BLOCKED`    |

**Diferencia principal:** la ejecución actual recibió menos marcadores que la histórica.

**Coincidencia principal:** ambas ejecuciones se bloquearon durante la finalización de la primera etapa IEEE.

### 5.5 Conclusión específica

RF-03 **no aporta evidencia experimental sobre el cambio entre IEEE 802.15.4 y Sub-GHz**.

El bloqueo ocurrió antes de configurar y evaluar la etapa Sub-GHz.

Por ello, sería incorrecto afirmar que el switching entre PHY falló.

La conclusión respaldada es más limitada: **el procedimiento RF-03 sigue sin poder completar de forma fiable su etapa inicial IEEE**.

---

## 6. Segunda recuperación — Posterior a RF-03

Después del bloqueo de RF-03 se realizó otro power-cycle de ambos dispositivos.

### 6.1 Preflight posterior a RF-03

Archivo:

```text
05_post_rf03_recovery_preflight.jsonl
```

Resultados:

* Los cuatro puertos COM se identificaron correctamente.
* Las versiones de firmware coincidieron con las anteriores.
* Ambos Cat-Shell respondieron.
* Ambas interfaces FeralRF respondieron.
* `uart_overrun=0`.
* `ring_dropped=0`.
* Contadores de recepción reportados en cero.
* Desconexión correcta de ambas interfaces.

Resultado:

```text
RESULT: PASS
LASTEXITCODE: 0
```

**Interpretación:** la accesibilidad de las interfaces quedó restablecida.

Este resultado, por sí solo, no validaba la recuperación OTA.

### 6.2 Control IEEE posterior a RF-03

Archivo:

```text
06_post_rf03_recovery_control.jsonl
```

Configuración:

* RF-01 con `--only-control`.
* Canal IEEE 25.
* Marcador `C10125F0`.
* Diez transmisiones.
* Umbral de nueve marcadores válidos.

Resultados:

| Indicador                 | Resultado     |
| ------------------------- | ------------- |
| `RX_START`                | ACK           |
| TX ACK                    | 10/10         |
| Marcadores con CRC válido | **9/10**      |
| Paquetes observados       | 13            |
| Paquetes sin marcador     | 4             |
| Umbral de recepción       | Cumplido      |
| `RX_STOP` normal          | No confirmado |
| `RX_STOP_CLEANUP_ACK`     | Recibido      |
| `FATAL`                   | Timeout       |
| Resultado global          | **`BLOCKED`** |

La secuencia registrada fue:

```text
OBSERVATION_END
crc_good_marker_count = 9

RX_STOP_CLEANUP_BEGIN

RX_STOP_CLEANUP_ACK

FATAL
TimeoutError(
  'Response timeout (ignored 4 unexpected response(s), last=0x90)'
)

DISCONNECT_OK — TX
DISCONNECT_OK — RX

RESULT: BLOCKED
```

El ACK obtenido durante la limpieza no elimina el fallo previo que provocó la excepción.

Por ello, la recuperación no puede declararse satisfactoria conforme al procedimiento.

### 6.3 Comparación de los controles de recuperación

| Indicador                 | Después de RF-01 | Después de RF-03 |
| ------------------------- | ---------------- | ---------------- |
| Preflight                 | PASS             | PASS             |
| Marcadores IEEE válidos   | 10/10            | 9/10             |
| Umbral de recepción       | Cumplido         | Cumplido         |
| Finalización normal de RX | Correcta         | Fallida          |
| ACK durante limpieza      | No requerido     | Recibido         |
| Resultado del control     | PASS             | **BLOCKED**      |

**Diferencia principal:** aunque ambos controles alcanzaron el umbral de recepción, solamente el primero finalizó correctamente.

Esto refuerza la necesidad de evaluar por separado la recepción RF y el manejo del ciclo de vida de RX.

---

## 7. Consolidación de resultados de la campaña

| Ejecución                   | Recepción o verificación principal     | Resultado   |
| --------------------------- | -------------------------------------- | ----------- |
| Preflight inicial           | Interfaces disponibles                 | PASS        |
| RF-01 completo              | Cuatro escenarios de canales evaluados | **BLOCKED** |
| Preflight posterior a RF-01 | Interfaces recuperadas                 | PASS        |
| Control posterior a RF-01   | IEEE 10/10; `RX_STOP` correcto         | PASS        |
| RF-03                       | IEEE inicial 7/10; timeout             | **BLOCKED** |
| Preflight posterior a RF-03 | Interfaces recuperadas                 | PASS        |
| Control posterior a RF-03   | IEEE 9/10; timeout                     | **BLOCKED** |

### 7.1 Capacidades demostradas

La evidencia permite sostener:

1. Comunicación IEEE 802.15.4 entre los dos CatSniffer.
2. Recepción de marcadores identificables con CRC válido.
3. Cambio funcional de canal IEEE 25 → 20 → 25 durante RF-01.
4. Ausencia del marcador esperado cuando TX y RX utilizaron canales diferentes en el control negativo.
5. Capacidad de recuperar temporalmente una ejecución completa después de un power-cycle.

Estas conclusiones se limitan a las condiciones ensayadas y a la recepción observada mediante los propios dispositivos.

### 7.2 Capacidades no demostradas

La campaña no permite afirmar:

1. Finalización fiable de `RX_STOP` en todas las ejecuciones.
2. Switching funcional IEEE ↔ Sub-GHz.
3. Recepción Sub-GHz durante RF-03.
4. Recuperación satisfactoria después del último bloqueo.
5. Ausencia de pérdida de paquetes en condiciones generales.
6. Validación independiente de potencia, frecuencia, espectro o modulación RF.

---

## 8. Incidencia recurrente: timeout durante `RX_STOP`

### 8.1 Hecho observado

Se produjeron timeouts al finalizar recepción en diferentes circunstancias:

* Última etapa de RF-01 completo.
* Etapa IEEE inicial de RF-03.
* Control IEEE posterior a la recuperación de RF-03.

En los mensajes de error aparece repetidamente:

```text
Response timeout
ignored N unexpected response(s)
last=0x90
```

Las respuestas `0x90` corresponden al tipo de respuesta asíncrona `RSP_RX_PACKET` según la interpretación actual del protocolo.

La aparición de estas respuestas mientras se espera otra respuesta no demuestra, por sí sola, cuál componente ocasionó el timeout.

### 8.2 Comportamiento que merece investigación

En el último control de recuperación se observó una diferencia importante:

```text
RX_STOP_CLEANUP_ACK
```

Esto indica que una operación de detención realizada durante la limpieza obtuvo ACK, aunque el procedimiento terminó en `BLOCKED`.

La observación es compatible con un problema dependiente de la secuencia temporal, del estado de RX o del manejo de respuestas pendientes.

**No demuestra todavía que el defecto se encuentre exclusivamente en la API Python.**

### 8.3 Hipótesis abiertas

| Hipótesis                                                                                            | Fundamento                                                 | Estado                   |
| ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- | ------------------------ |
| La API Python no correlaciona correctamente algunas respuestas cuando existen paquetes RX pendientes | Timeouts acompañados de respuestas `0x90`                  | Sin verificar            |
| El firmware CC1352P7 retrasa u omite determinadas respuestas de `RX_STOP`                            | Ausencia del ACK esperado en algunas ejecuciones           | Sin verificar            |
| La cola o el procesamiento de eventos asíncronos interfiere con la finalización                      | Variación entre etapas y ejecuciones                       | Sin verificar            |
| Existe un problema de sincronización entre comando, respuesta y limpieza                             | Un intento de limpieza obtuvo ACK después del fallo previo | Sin verificar            |
| La actividad IEEE ajena al experimento contribuye al problema                                        | Paquetes adicionales observados en canal 25                | Causalidad no demostrada |

Estas hipótesis son líneas de investigación, no conclusiones.

---

## 9. Implicaciones para una futura revisión del código

La evidencia justifica una revisión focalizada antes de modificar el firmware.

### 9.1 Primera prioridad: API Python

Identificar:

* Cómo se envía `RX_STOP`.
* Qué respuesta espera la API.
* Cómo se correlacionan respuestas y comandos.
* Cómo se procesan las respuestas asíncronas `RSP_RX_PACKET (0x90)`.
* Qué ocurre cuando hay paquetes pendientes.
* Cómo se aplican los timeouts.
* Cómo interactúan la detención normal y la limpieza.

Debe determinarse si una respuesta válida puede quedar pendiente, ser descartada o no reconocerse.

### 9.2 Segunda prioridad: firmware FeralRF del CC1352P7

Reconstruir el recorrido de `RX_STOP`:

```text
Comando recibido
       ↓
Decodificación
       ↓
Detención de recepción RF
       ↓
Actualización de estado
       ↓
Generación de respuesta
       ↓
Transmisión hacia la PC
```

Se deberá verificar:

* En qué estados se acepta el comando.
* Qué ocurre cuando existen eventos RX pendientes.
* Cuándo se genera el ACK.
* Si existen operaciones bloqueantes o condiciones de carrera.
* Qué sucede cuando la detención se solicita más de una vez.

### 9.3 Tercera prioridad: interacción con Cat-Bridge y Cat-Shell

Confirmar el recorrido físico y lógico de las respuestas entre CC1352P7 y PC.

La revisión debe distinguir las responsabilidades del CC1352P7, del RP2040 y de la API Python.

Los contadores `uart_overrun=0` y `ring_dropped=0` observados en los preflight no bastan para descartar todos los problemas posibles de comunicación durante una ejecución.

### 9.4 Condiciones antes de proponer una corrección

Cualquier modificación futura deberá documentar:

1. Comportamiento actual demostrado.
2. Causa identificada mediante código o prueba.
3. Cambio propuesto.
4. Efectos sobre los comandos y respuestas existentes.
5. Riesgos de regresión.
6. Procedimiento reproducible para comprobar la corrección.

No se recomienda aumentar arbitrariamente los timeouts ni descartar respuestas `0x90` sin conocer primero el contrato del protocolo.

---

## 10. Estado final y continuación

**Estado de la campaña:** detenida.

**RF-01:** `BLOCKED`, con evidencia positiva de cambio de canal IEEE.

**RF-03:** `BLOCKED`, sin evidencia sobre la transición a Sub-GHz.

**Última recuperación:** preflight `PASS`, control IEEE `BLOCKED`.

### Condición para reanudar la validación

Antes de continuar con nuevos experimentos, debe resolverse o acotarse suficientemente el comportamiento de `RX_STOP`.

El procedimiento de recuperación pendiente no debe darse por completado únicamente porque los dispositivos respondan al preflight.

### Orden de validación que debe conservarse

Conforme al manual actual:

```text
Preflight
   ↓
RF-01 completo — PASS
   ↓
RF-02 — PASS
   ↓
RF-03
   ↓
RF-04
   ↓
RF-05 (opcional)
```

**Desviación registrada:** durante esta campaña se ejecutó RF-03 después de un control de recuperación satisfactorio, aunque RF-01 completo seguía `BLOCKED` y RF-02 no había sido aprobado. El control de recuperación no sustituye la aprobación de RF-01 completo.

Esta desviación debe conservarse en la trazabilidad y no repetirse sin una justificación explícita.

---

## 11. Conclusión general

La campaña del 21 de octubre de 2026 amplió significativamente la evidencia disponible sobre IEEE 802.15.4.

A diferencia de la ejecución histórica, **RF-01 consiguió recorrer los cuatro escenarios de canales y obtener los resultados de recepción esperados**, incluido el cambio de canal 25 a 20 y el regreso al canal 25.

Sin embargo, el procedimiento terminó bloqueado por un timeout durante `RX_STOP`.

RF-03 reprodujo el bloqueo histórico durante su etapa inicial IEEE, antes de alcanzar Sub-GHz. Además, su recepción inicial fue inferior al umbral exigido: 7/10 frente a los 10/10 históricos.

Los procedimientos posteriores demostraron que un power-cycle puede permitir recuperar la comunicación, pero también que esa recuperación no es consistentemente satisfactoria: el último control alcanzó 9/10 marcadores válidos y aun así terminó en `BLOCKED`.

**Conclusión técnica principal:**

> La evidencia actual respalda el funcionamiento del cambio de canal IEEE 802.15.4, pero identifica una limitación recurrente en la finalización de RX que impide completar de forma fiable los procedimientos de validación. La transición IEEE ↔ Sub-GHz permanece sin evaluar experimentalmente.

La prioridad futura no es asumir un defecto de Sub-GHz, sino **identificar el origen del comportamiento de `RX_STOP` mediante el análisis del contrato entre firmware y API Python**.

---

## 12. Referencias y procedencia

### Procedimiento rector

* `RF_VALIDATION_RUNBOOK.md` — procedimiento de ejecución, recuperación y orden de experimentos.
* `NEXT_RF_VALIDATION_PLAN.md` — planificación de validación RF.

### Evidencia experimental actual

* `00_preflight.jsonl`
* `01_rf01_ieee_channel_switch.jsonl`
* `02_post_recovery_preflight.jsonl`
* `03_post_recovery_rf01_control.jsonl`
* `04_rf03_cross_phy_switch.jsonl`
* `05_post_rf03_recovery_preflight.jsonl`
* `06_post_rf03_recovery_control.jsonl`

Directorio:

```text
FeralRF/validation_runs/manual_20261021_130015/
```

### Antecedentes históricos utilizados para comparación

* `validation_matrix_20261012_113056.md`
* `SUBGHZ_RECOVERY_AUDIT.md`
* `SUBGHZ_RECOVERY_RUN.md`
* `SUBGHZ_PROVENANCE_AUDIT.md`
* `Auditoría causal Codex - comparación histórica OTA SubGHz.md`
* `Auditoría metodológica Codex - campaña OTA SubGHz octubre 2026(2).md`

Los antecedentes históricos aportan contexto comparativo. Los resultados de la campaña actual se fundamentan en los registros JSONL identificados anteriormente.

**Nota de trazabilidad:** no se registró en este reporte el commit exacto del código FeralRF ni de los scripts Python ejecutados. Esos identificadores deberán recuperarse del repositorio antes de realizar comparaciones de implementación o atribuir regresiones a cambios concretos.
