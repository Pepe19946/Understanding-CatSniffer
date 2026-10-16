# Informe técnico de validación OTA Sub-GHz — FeralRF

**Proyecto:** CatSniffer — FeralRF
**Fecha del informe:** 16 de octubre de 2026
**Periodo experimental:** 15–16 de octubre de 2026
**Estado:** Campaña de 13 presets completada; auditoría causal histórica pendiente
**Tipo de documento:** Informe de resultados experimentales y referencia para auditoría posterior

---

# 1. Resumen ejecutivo

Se realizó una campaña de validación de comunicación *Over-the-Air* (OTA) Sub-GHz entre dos dispositivos CatSniffer ejecutando FeralRF.

El objetivo fue comprobar si ambos dispositivos podían intercambiar paquetes utilizando distintas configuraciones propietarias de modulación, frecuencia y velocidad de símbolos, estableciendo controles positivos periódicos y conservando evidencia de errores de comunicación.

La campaña incluyó:

* 13 presets nuevos.
* Cinco ejecuciones satisfactorias del control positivo.
* Una repetición adicional satisfactoria.
* Tres ejecuciones bloqueadas por errores no inducidos.
* Dos contrapruebas bloqueadas mediante RESET físico intencional.

## 1.1. Resultados generales

| Indicador                                                 |           Resultado |
| --------------------------------------------------------- | ------------------: |
| Presets nuevos evaluados                                  |                  13 |
| Presets nuevos con PASS-RF                                |           **13/13** |
| Controles positivos satisfacstorios                        | **5/5 completados** |
| Repeticiones adicionales satisfactorias                   |                   1 |
| Ejecuciones OTA satisfactorias                            |              **19** |
| Paquetes válidos observados en ejecuciones satisfactorias |         **190/190** |
| Ejecuciones BLOCKED no inducidas                          |                   3 |
| Ejecuciones BLOCKED inducidas                             |                   2 |
| Total de ejecuciones OTA registradas                      |              **24** |
| Ejecuciones FAIL-RF completas no inducidas                |               **0** |

Adicionalmente, durante una contraprueba con RESET del transmisor se recibieron cinco paquetes válidos antes de finalizar la ejecución con una excepción.

Por tanto, los registros contienen **195 eventos `RX_PACKET` con `crc_ok=True` y marcador correcto**, de los cuales 190 pertenecen a ejecuciones satisfactorias y cinco a una contraprueba interrumpida.

## 1.2. Conclusión principal

Los resultados proporcionan evidencia funcional de que los dos CatSniffer pueden intercambiar paquetes mediante FeralRF utilizando las 13 configuraciones Sub-GHz ensayadas.

Se observaron recepciones satisfactorias con frecuencias solicitadas de:

* 868.0 MHz.
* 868.3 MHz.
* 868.95 MHz.
* 902.2 MHz.
* 915.0 MHz.

Los resultados abarcan configuraciones solicitadas de FSK y GFSK con velocidades desde 32 768 hasta 300 000 símbolos/s.

No se realizó una medición independiente de la frecuencia transmitida, la potencia radiada, la desviación de frecuencia ni las características espectrales de las señales.

**El alcance de PASS-RF es, por tanto, funcional: recepción de paquetes entre los dos dispositivos mediante las interfaces y el firmware de FeralRF.**

---

# 2. Objetivos y alcance

## 2.1. Objetivo principal

Verificar la comunicación OTA entre dos CatSniffer utilizando los presets Sub-GHz disponibles en la biblioteca Python de FeralRF.

## 2.2. Objetivos específicos

1. Establecer una configuración conocida funcional antes de comenzar la campaña.
2. Verificar las interfaces de comunicación con ambos dispositivos.
3. Seleccionar explícitamente la ruta lógica Sub-GHz.
4. Ejecutar pruebas individuales con los presets seleccionados.
5. Registrar comandos aceptados, paquetes recibidos, CRC, marcadores y errores.
6. Ejecutar controles positivos periódicos.
7. Identificar incidencias de comunicación independientes de los resultados RF.
8. Realizar contrapruebas para observar el comportamiento frente a interrupciones físicas.
9. Construir evidencia que permita comparar esta campaña con experimentos históricos de recepción nula.

## 2.3. Fuera del alcance

No se validaron:

* Interoperabilidad con equipos Wi-SUN, Wireless M-Bus o Amazon Sidewalk.
* Implementaciones MAC o stacks completos de dichos protocolos.
* Conformidad con estándares RF.
* Potencia radiada o características espectrales mediante instrumentación.
* Sensibilidad del receptor, BER, PER frente a SNR o alcance máximo.
* Funcionamiento a 169 o 433 MHz.
* OOK, MIOTY, MSK, 4-FSK o 4-GFSK.
* Recuperación automática de una sesión de Python después de RESET.
* Causa raíz de las campañas históricas con cero recepciones.

---

# 3. Fuentes, procedencia y reproducibilidad

## 3.1. Evidencia primaria

**Archivo:** `Experimentos FeralRF OTA SubGHz.txt`

Este archivo contiene la transcripción cronológica proporcionada por el operador, incluyendo:

* Comandos ejecutados en PowerShell.
* Salidas de los programas Python.
* Consultas a los Shell de CatSniffer.
* Trazas de excepciones.
* Resultados de las pruebas OTA.
* Observaciones del operador.
* Análisis realizados durante la campaña.

**SHA-256 del TXT examinado:**

`1a6694430fd6f036437e7ce55501bba57e74e2e66deb1e547b0c8f68053d41d1`

Este hash identifica la copia específica utilizada para elaborar el informe.

Los registros originales permanecen en el TXT y no se sustituyen por las tablas de este documento.

## 3.2. Repositorio y entorno

| Elemento          | Valor registrado                                                    |
| ----------------- | ------------------------------------------------------------------- |
| Repositorio       | `ElectronicCats/FeralRF`                                            |
| Directorio local  | `C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF` |
| Rama              | `feature/hardware-validation`                                       |
| HEAD observado    | `ba27fbeb8c5683a31cca4339982cadeaaa406444`                          |
| Python            | `3.14.7`                                                            |
| Sistema operativo | Windows                                                             |
| Terminal          | PowerShell                                                          |
| Biblioteca        | `feralrf`, importada desde el directorio `python`                   |
| Script inicial    | `experimental_artifacts/checkpoint_3a/checkpoint_3a_single_run.py`  |
| Script de campaña | `experimental_artifacts/manual_preset_runner.py`                    |

El commit indicado fue comprobado durante la preparación del 15 de octubre.

**Limitación de procedencia:** no se demostró mediante hash que el binario instalado en el CC1352 corresponda exactamente al código de ese commit.

Asimismo, `experimental_artifacts/` apareció inicialmente como una ruta sin seguimiento de Git. Por tanto, el commit no identifica automáticamente el contenido del script experimental.

## 3.3. Criterios de interpretación

Se distinguen cinco niveles:

| Categoría                | Significado                                                 |
| ------------------------ | ----------------------------------------------------------- |
| Configuración solicitada | Valores enviados mediante la API                            |
| Control confirmado       | Respuestas ACK, estado Shell o estadísticas                 |
| Comportamiento observado | Paquetes entregados por la API y errores registrados        |
| Inferencia               | Explicación compatible con los resultados                   |
| Pendiente                | Afirmación que requiere código, medición o prueba adicional |

Una respuesta ACK no constituye una medición independiente de emisión RF.

De igual manera, `crc_ok=True` representa el resultado reportado por el firmware y la biblioteca, no una comprobación de CRC realizada por instrumentación externa.

---

# 4. Plataforma experimental

## 4.1. Dispositivos utilizados

Se emplearon dos CatSniffer, identificados operacionalmente durante esta campaña como #2 y #3.

| Función    | Identificación de campaña | Cat-Bridge | Cat-Shell |
| ---------- | ------------------------- | ---------- | --------- |
| Transmisor | CatSniffer #2             | `COM33`    | `COM35`   |
| Receptor   | CatSniffer #3             | `COM88`    | `COM87`   |

Ambos se conectaron a la misma computadora mediante USB.

La numeración de los dispositivos es la utilizada durante esta campaña. Para comparaciones históricas debe utilizarse la identidad USB física, no solamente los números #2 y #3, porque la nomenclatura de otras campañas podría diferir.

### Identificadores USB observados

| Cat-Bridge | Grupo de instancia USB |
| ---------- | ---------------------- |
| `COM33`    | `1E70E7AF`             |
| `COM88`    | `106D987F`             |

La enumeración USB también identificó las interfaces Shell correspondientes.

## 4.2. Firmware reportado

Ambos Shell reportaron:

| Componente o interfaz                 | Valor                     |
| ------------------------------------- | ------------------------- |
| Firmware de control RP2040            | `v3.1.0.1`                |
| Identificador de firmware CC1352      | `feralrf_cc1352 (custom)` |
| Firmware reportado por `Radio.init()` | `1.0.0`                   |
| `capabilities`                        | `7`                       |
| Serial lógico                         | `464552414c524631`        |

El serial lógico representa `FERALRF1`.

Se observó el mismo valor en ambos dispositivos; por tanto, no permite distinguirlos físicamente.

Los valores corresponden a lo reportado por las interfaces de software. No constituyen una verificación criptográfica del firmware instalado.

## 4.3. Antenas y condiciones físicas

El operador confirmó que ambos dispositivos tenían antenas instaladas y compatibles con las frecuencias utilizadas, incluyendo 868 MHz y 902–928 MHz.

También confirmó autorización para las transmisiones previstas.

| Condición                                              | Estado                              |
| ------------------------------------------------------ | ----------------------------------- |
| Dos dispositivos conectados                            | Confirmado durante las pruebas      |
| Antenas instaladas                                     | Confirmado por el operador          |
| Compatibilidad declarada con las frecuencias ensayadas | Confirmada por el operador          |
| Potencia TX solicitada                                 | 0 dBm                               |
| Distancia exacta TX–RX                                 | No registrada en esta transcripción |
| Instrumentación RF independiente                       | No utilizada                        |
| Medición eléctrica del RF switch                       | No realizada                        |

La compatibilidad de antenas fue declarada por el operador y no se verificó mediante mediciones de adaptación o un ensayo de eficiencia.

---

# 5. Arquitectura de comunicación utilizada

## 5.1. Recorrido funcional

La arquitectura lógica de la prueba puede representarse como:

```text
                      PC — Windows
                            |
                         PowerShell
                            |
                          Python
                            |
                      feralrf.Radio
                            |
                 Interfaz serial Cat-Bridge
                            |
                 CatSniffer TX — COM33
                            |
                  Control / firmware
                            |
                       CC1352 RF
                            |
                    Enlace OTA Sub-GHz
                            |
                       CC1352 RF
                            |
                 CatSniffer RX — COM88
                            |
                 Interfaz serial Cat-Bridge
                            |
                      feralrf.Radio
                            |
                          Python
                            |
                    Eventos RX_PACKET
```

Este diagrama representa el recorrido funcional esperado y respaldado por las interfaces utilizadas. No pretende describir todas las conexiones internas de una revisión específica de PCB.

## 5.2. Control de banda

Los comandos Shell identificados fueron:

| Comando | Función declarada |
| ------- | ----------------- |
| `band1` | Selección 2.4 GHz |
| `band2` | Selección Sub-GHz |
| `band3` | Selección LoRa    |

El control de banda se realizó mediante la interfaz Shell del RP2040, separadamente de la configuración PHY aplicada al CC1352 mediante FeralRF.

Durante la preparación inicial, ambos dispositivos reportaron:

| Condición           | TX             | RX             |
| ------------------- | -------------- | -------------- |
| Antes de `band2`    | `Band: 0`      | `Band: 0`      |
| Respuesta a `band2` | `SUB-GHz Band` | `SUB-GHz Band` |
| Después de `band2`  | `Band: 1`      | `Band: 1`      |

Por tanto, se verificó la selección **lógica** Sub-GHz en ambos Shell.

Sin embargo, no se midieron directamente las señales de control del RF switch ni su posición física.

### Distinción fundamental

`configure_prop(frequency_hz=...)` solicita una frecuencia de operación al transceptor.

`band2` selecciona la ruta lógica Sub-GHz de la placa.

**Son operaciones distintas y ambas deben considerarse al investigar un resultado OTA negativo.**

## 5.3. Protocolo y modulación utilizados

La configuración común fue:

`PHY.PROPRIETARY_GFSK`

Dentro de este PHY, los presets utilizaron:

| `mod_type` | Modulación solicitada |
| ---------- | --------------------- |
| `0`        | FSK                   |
| `1`        | GFSK                  |

Por tanto, el experimento se describe como:

**Comunicación de paquetes OTA propietarios de FeralRF sobre PHY Sub-GHz FSK/GFSK.**

No se validó un protocolo de aplicación o MAC estandarizado.

Los nombres `wisun`, `sidewalk` y `wireless_mbus` solamente identificaron conjuntos de parámetros de radio utilizados para estas pruebas.

En particular:

* `wisun_915_fsk_*`: no implica interoperabilidad Wi-SUN FAN.
* `sidewalk_915_fsk_*`: no implica interoperabilidad Amazon Sidewalk.
* `wireless_mbus_*`: no implica conformidad con Wireless M-Bus Mode S, T o C.

El campo `Radio: LoRa` mostrado por el Shell tampoco permite concluir que las transmisiones de FeralRF utilizaran LoRa. La configuración aplicada mediante la API fue `PROPRIETARY_GFSK`.

---

# 6. Metodología de ejecución

## 6.1. Preparación inicial

La preparación documentada siguió esta secuencia:

| Paso | Operación                               | Evidencia                         |
| ---- | --------------------------------------- | --------------------------------- |
| 1    | Seleccionar el repositorio              | Directorio comprobado             |
| 2    | Configurar `PYTHONPATH`                 | Biblioteca Python disponible      |
| 3    | Consultar rama, commit y versión Python | Valores registrados               |
| 4    | Enumerar interfaces USB                 | Puertos identificados             |
| 5    | Consultar Shell mediante `status`       | Ambos respondieron                |
| 6    | Ejecutar `boot` y `exit`                | Respuestas `BOOT` y `PASSTHROUGH` |
| 7    | Ejecutar `band2`                        | Respuesta `SUB-GHz Band`          |
| 8    | Consultar `status`                      | Ambos reportaron `Band: 1`        |
| 9    | Inspeccionar script inicial             | Código disponible                 |
| 10   | Ejecutar control OTA inicial            | PASS-RF 10/10                     |
| 11   | Preparar runner reutilizable            | Adaptación realizada              |
| 12   | Comprobar sintaxis                      | `SYNTAX OK`                       |
| 13   | Iniciar campaña de presets              | Ejecuciones individuales          |

La secuencia `boot` → `exit` formó parte de la preparación inicial documentada.

No se reinició el dispositivo entre cada preset exitoso.

## 6.2. Scripts utilizados

### Control inicial

```powershell
python -u '.\experimental_artifacts\checkpoint_3a\checkpoint_3a_single_run.py'
```

Este programa realizó el control conocido con `wisun_915_fsk_50k`.

### Runner de campaña

```powershell
python -u '.\experimental_artifacts\manual_preset_runner.py' `
    <preset> `
    <marker_hex> `
    <label>
```

El runner fue obtenido como una copia del programa inicial y posteriormente adaptado para recibir argumentos.

La salida `MANUAL RUNNER UPDATED` confirmó la modificación, seguida de una comprobación `SYNTAX OK`.

La adaptación conservó los puertos, el PHY, la potencia solicitada y el procedimiento general de transmisión-recepción.

## 6.3. Secuencia de una ejecución OTA

El procedimiento operativo del runner fue:

| Orden | Operación Python            | Finalidad                    |
| ----- | --------------------------- | ---------------------------- |
| 1     | `Radio.init()` en TX        | Inicializar sesión           |
| 2     | `Radio.init()` en RX        | Inicializar sesión           |
| 3     | `set_phy()` en TX/RX        | Seleccionar PHY              |
| 4     | `configure_prop()` en TX/RX | Aplicar preset solicitado    |
| 5     | `set_power(0)` en TX        | Solicitar potencia           |
| 6     | `get_stats()` en ambos      | Obtener línea base           |
| 7     | `start_rx()`                | Activar recepción            |
| 8     | Espera de 0.3 s             | Separación previa            |
| 9     | Diez llamadas `transmit()`  | Solicitar diez transmisiones |
| 10    | Espaciado de 0.1 s          | Separar solicitudes TX       |
| 11    | Espera posterior de 1 s     | Permitir procesamiento       |
| 12    | `read_packets(timeout=2)`   | Recuperar eventos RX         |
| 13    | Observación TX de 0.5 s     | Detectar errores asíncronos  |
| 14    | `stop_rx()`                 | Detener recepción            |
| 15    | `get_stats()`               | Obtener contadores finales   |
| 16    | `disconnect()`              | Cerrar ambas sesiones        |

No se realizó un barrido automático de presets.

Cada configuración fue ejecutada individualmente y evaluada antes de continuar.

### Consideración de temporización

El programa recupera los eventos de recepción después de realizar las diez solicitudes TX.

Por tanto, la validez de la observación depende de la conservación de los eventos durante ese intervalo en las capas correspondientes.

El funcionamiento satisfactorio de esta secuencia demuestra que fue viable en las condiciones ensayadas, pero no permite concluir que otros helpers históricos implementaran exactamente la misma gestión de eventos.

## 6.4. Criterios de clasificación

| Estado         | Criterio                                                                                                                |
| -------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `PASS-RF`      | 10 TX ACK, 10 paquetes CRC-good con marcador, sin errores asíncronos ni excepciones fatales y con finalización correcta |
| `PARTIAL`      | Recepción válida incompleta en una ejecución evaluable                                                                  |
| `FAIL-RF`      | Prueba completa sin cumplir el criterio de recepción                                                                    |
| `BLOCKED`      | Ejecución interrumpida por una excepción o condición que impide evaluar normalmente el enlace                           |
| `INCONCLUSIVE` | Evidencia insuficiente para atribuir un resultado                                                                       |

Una ejecución que termina antes de transmitir no se clasifica como fallo RF.

Una ejecución interrumpida mediante RESET intencional se conserva como `BLOCKED` inducido, aunque se hayan observado paquetes antes de la interrupción.

---

# 7. Resultados de los presets

## 7.1. Configuraciones verificadas

La siguiente tabla contiene los valores reportados por `PROP_PRESETS` y por los eventos `TEST_BEGIN`.

**Unidades:** `symbol_rate` se expresa en símbolos/s. Los valores `deviation` y `rx_bw` se conservan como argumentos numéricos de la API; sus unidades físicas y su interpretación dentro del firmware todavía requieren auditoría.

| N.º | Preset                  | Frecuencia solicitada (MHz) | Mod. | Symbol rate | `deviation` | `rx_bw` | `sync_word`  |
| --- | ----------------------- | --------------------------: | ---- | ----------: | ----------: | ------: | ------------ |
| 1   | `gfsk_902_50k`          |                       902.2 | GFSK |      50 000 |         100 |      82 | `0x930B51DE` |
| 2   | `wisun_915_fsk_100k`    |                       902.2 | FSK  |     100 000 |         150 |      86 | `0x930B51DE` |
| 3   | `wisun_915_fsk_150k`    |                       902.2 | FSK  |     150 000 |         200 |      90 | `0x930B51DE` |
| 4   | `wisun_915_fsk_200k`    |                       902.2 | FSK  |     200 000 |         200 |      90 | `0x930B51DE` |
| 5   | `wisun_915_fsk_300k`    |                       902.2 | FSK  |     300 000 |         200 |      94 | `0x930B51DE` |
| 6   | `gfsk_915_50k`          |                       915.0 | GFSK |      50 000 |         100 |      82 | `0x930B51DE` |
| 7   | `sidewalk_915_fsk_50k`  |                       915.0 | FSK  |      50 000 |         100 |      82 | `0x930B51DE` |
| 8   | `sidewalk_915_fsk_250k` |                       915.0 | FSK  |     250 000 |         200 |      90 | `0x930B51DE` |
| 9   | `gfsk_868_50k`          |                       868.0 | GFSK |      50 000 |         100 |      82 | `0x930B51DE` |
| 10  | `gfsk_868_100k`         |                       868.0 | GFSK |     100 000 |         150 |      86 | `0x930B51DE` |
| 11  | `wireless_mbus_s_868`   |                       868.3 | GFSK |      32 768 |          75 |      82 | `0x543D0000` |
| 12  | `wireless_mbus_t_868`   |                      868.95 | GFSK |     100 000 |         200 |      87 | `0x543D0000` |
| 13  | `wireless_mbus_c_868`   |                      868.95 | GFSK |     100 000 |         180 |      87 | `0x543D0000` |

Todos los presets tuvieron al menos una ejecución satisfactoria con 10/10 paquetes.

## 7.2. Interpretación de los resultados

La campaña demuestra que las distintas configuraciones pudieron utilizarse para intercambiar paquetes entre los dos dispositivos.

Sin embargo, no permite aislar experimentalmente el efecto de cada parámetro.

Por ejemplo, entre algunos presets cambian simultáneamente `symbol_rate`, `deviation` y `rx_bw`. Un resultado positivo demuestra que la combinación funcionó, pero no cuánto contribuyó cada parámetro.

Tampoco se utilizó un patrón de prueba diseñado para medir BER ni se ensayaron distintas condiciones de atenuación.

Por tanto, **no es válido construir una clasificación de calidad RF entre presets a partir de estos resultados**.

---

# 8. Controles positivos

El preset de referencia fue:

`wisun_915_fsk_50k`

Sus parámetros reportados fueron:

| Parámetro           | Valor         |
| ------------------- | ------------- |
| Frecuencia          | 902.2 MHz     |
| `mod_type`          | 0 — FSK       |
| `symbol_rate`       | 50 000        |
| `deviation`         | 100           |
| `rx_bw`             | 82            |
| `sync_word`         | `0x930B51DE`  |
| Potencia solicitada | 0 dBm         |
| Marcador            | `DE AD BE EF` |

## 8.1. Resultados de controles

| Control                    | Momento                   | TX ACK |    RX válido | Estado  |
| -------------------------- | ------------------------- | -----: | -----------: | ------- |
| Inicial                    | Antes de los 13 presets   |  10/10 |        10/10 | PASS-RF |
| Después de tres presets    | Tras #3                   |  10/10 |        10/10 | PASS-RF |
| Después de seis, intento 1 | Tras #6                   |   0/10 | 0 observados | BLOCKED |
| Después de seis, intento 2 | Recuperación              |  10/10 |        10/10 | PASS-RF |
| Después de diez presets    | Tras #10 y reconexión USB |  10/10 |        10/10 | PASS-RF |
| Después de trece           | Tras contrapruebas RESET  |  10/10 |        10/10 | PASS-RF |

Los cinco controles completos fueron satisfactorios.

El intento bloqueado después del sexto preset falló antes de comenzar las transmisiones.

## 8.2. Valor experimental de los controles

Los controles permiten comprobar que una configuración previamente funcional continuó operando después de distintas configuraciones y eventos.

En particular, el control posterior al décimo preset verificó nuevamente la comunicación utilizando 902.2 MHz después de realizar pruebas a 868 MHz y de una reconexión USB.

El control posterior a las contrapruebas RESET mostró que ambos dispositivos pudieron participar satisfactoriamente en una nueva sesión de FeralRF.

**Limitación:** estos resultados no prueban que la recuperación haya sido automática ni que el RF switch haya cambiado físicamente según lo esperado.

## 8.3. Desviación del procedimiento

El manual contemplaba un control positivo después de cada tres presets nuevos satisfactorios.

En el bloque posterior al sexto preset se ejecutaron cuatro presets (#7–#10) antes del siguiente control.

Esta desviación debe conservarse para la auditoría metodológica. El control posterior fue satisfactorio, pero no sustituye retrospectivamente un control que debió realizarse antes.

---

# 9. Incidencias y ejecuciones bloqueadas

Se registraron cinco ejecuciones `BLOCKED`.

Es importante separar las tres incidencias no inducidas de las dos interrupciones intencionales.

## 9.1. Incidencias no inducidas

| ID  | Ejecución                                  | Punto de fallo      | Excepción                               | Impacto                |
| --- | ------------------------------------------ | ------------------- | --------------------------------------- | ---------------------- |
| B01 | Control después de seis presets, intento 1 | `tx.get_stats()`    | `TimeoutError`                          | No inició RX ni TX     |
| B02 | Repetición `gfsk_868_100k`                 | Apertura de `COM33` | `ConnectionError` / `FileNotFoundError` | No configuró el enlace |
| B03 | `wireless_mbus_s_868`, intento 1           | `rx.get_stats()`    | `TimeoutError`                          | No inició RX ni TX     |

### B01 — Timeout del transmisor

Se observó un timeout durante la consulta inicial de estadísticas de TX.

Las configuraciones anteriores habían sido satisfactorias y posteriormente ambos dispositivos respondieron a consultas aisladas de `get_stats()`.

Una nueva ejecución del control consiguió PASS-RF.

**Conclusión:** timeout transitorio observado; causa raíz no determinada.

### B02 — Puerto COM33 no disponible

Después de una intervención de desconexión y reconexión USB reportada por el operador, un intento falló al abrir `COM33`.

La siguiente ejecución consiguió inicializar ambos dispositivos y recibió 10/10 paquetes.

**Conclusión:** indisponibilidad del puerto durante ese intento; no constituye evidencia de fallo RF.

La posible relación con la reenumeración USB es una hipótesis compatible con el contexto, no una causa confirmada.

### B03 — Timeout del receptor

En el primer intento con `wireless_mbus_s_868`, la aplicación completó la inicialización y aceptó la configuración solicitada, pero `rx.get_stats()` produjo un timeout.

Una consulta aislada posterior de `get_stats()` fue satisfactoria y el segundo intento OTA obtuvo PASS-RF.

**Conclusión:** fallo transitorio de consulta de estadísticas; no se identificó su causa.

## 9.2. Contrapruebas inducidas mediante RESET

Las contrapruebas se realizaron con `wireless_mbus_c_868`.

El operador reportó haber presionado físicamente RESET en un dispositivo diferente durante cada ejecución.

| Condición            | TX ACK |  Paquetes RX observados | Resultado        |
| -------------------- | -----: | ----------------------: | ---------------- |
| Sin intervención     |  10/10 |                   10/10 | PASS-RF          |
| RESET del receptor   |  10/10 | 0 antes de la excepción | BLOCKED inducido |
| RESET del transmisor |   5/10 |                    5/10 | BLOCKED inducido |
| Control posterior    |  10/10 |                   10/10 | PASS-RF          |

### RESET del receptor

Durante `rx.read_packets()` apareció una `SerialException` de Windows.

El registro mostró cero paquetes entregados antes de la excepción.

Esto **no demuestra que el receptor no haya recibido paquetes físicamente antes del RESET**, porque el proceso de lectura fue interrumpido.

### RESET del transmisor

Las primeras cinco llamadas `transmit()` fueron aceptadas.

A partir de la sexta llamada aparecieron errores `SerialException` asociados a escritura sobre el puerto serial.

El receptor entregó cinco paquetes válidos con el marcador esperado.

La correspondencia entre los cinco comandos aceptados y los cinco paquetes observados respalda la dependencia funcional del procedimiento respecto del transmisor.

### Interpretación conjunta

Las contrapruebas muestran que RESET puede interrumpir las sesiones de comunicación serial y afectar el procedimiento experimental.

**No aíslan una interrupción exclusivamente RF**, porque RESET también puede modificar el estado del microcontrolador, sus interfaces y la conexión USB.

El control posterior confirmó recuperación funcional en una sesión nueva, no recuperación automática de la sesión interrumpida.

---

# 10. Anomalías y discrepancias identificadas

## 10.1. Metadatos de recepción

En los 195 eventos `RX_PACKET` registrados en la transcripción se observaron:

| Campo                           | Observación                                                      |
| ------------------------------- | ---------------------------------------------------------------- |
| `channel`                       | Reportado como `37`                                              |
| Canal solicitado en `set_phy()` | `0`                                                              |
| `lqi`                           | Reportado como `0`                                               |
| `rssi_dbm`                      | Valores negativos y positivos muy elevados                       |
| `device_timestamp_us`           | Incrementos cuya correspondencia temporal requiere investigación |

Se contabilizaron 98 valores de RSSI estrictamente positivos dentro de los 195 eventos.

La presencia de valores positivos muy elevados y la discrepancia del canal sugieren que estos campos no deben emplearse todavía como indicadores confiables de calidad RF.

### Hipótesis abiertas

* Interpretación incorrecta de campos de metadatos.
* Datos asociados a estructuras compartidas con otros PHY.
* Conversión incorrecta de valores con signo.
* Escalas o unidades de timestamp no documentadas.
* Error en el firmware o en la decodificación Python.

No se asigna causa raíz sin revisar las estructuras y funciones correspondientes.

## 10.2. Discrepancia del `sync_word` Wireless M-Bus

Los registros muestran:

`sync_word=1413283840`

Conversión correcta:

`1413283840 = 0x543D0000`

Sin embargo, durante los análisis anteriores se presentó repetidamente:

`0x543C0000`

Ese valor corresponde realmente a:

`1413218304`

Por tanto, existió un **error de conversión decimal–hexadecimal en los análisis previos**.

### Consecuencia

Los resultados positivos siguen demostrando recepción entre dispositivos configurados con el mismo valor reportado.

Lo que no podemos afirmar es que ese valor coincida con el sync word requerido por un modo Wireless M-Bus específico.

La auditoría posterior debe verificar la definición real de `PROP_PRESETS`, el tratamiento del sync word en el firmware y su relación con los formatos de trama correspondientes.

## 10.3. Nombres de presets y frecuencia efectiva solicitada

Los presets `wisun_915_fsk_50k`, `_100k`, `_150k`, `_200k` y `_300k` utilizaron:

`frequency_hz=902200000`

Por tanto, aunque sus nombres contienen `915`, durante esta campaña solicitaron **902.2 MHz**.

Los resultados deben clasificarse por la frecuencia configurada y no únicamente por el nombre del preset.

## 10.4. Identificadores de ejecución repetidos

El mismo `label` se reutilizó en:

* Tres ejecuciones de `gfsk_868_100k`.
* Tres ejecuciones de `wireless_mbus_c_868`.

Por ello, `label` no es una clave primaria única para la campaña.

Para identificar una ejecución debe utilizarse, como mínimo:

`fecha UTC + timestamp TEST_BEGIN + label + preset`

En esta matriz se asignan identificadores documentales independientes `E01`–`E24`.

Estos identificadores no fueron generados por el runner; se incorporan únicamente para facilitar la auditoría.

## 10.5. Trazabilidad de la selección de banda

La transcripción confirma explícitamente `band2` y `Band: 1` durante la preparación inicial.

Sin embargo, después de reabrir PowerShell el 16 de octubre, la transcripción conserva la instrucción para volver a enviar `band2`, pero no contiene su salida literal.

Tampoco contiene una lectura explícita del Shell posterior a las contrapruebas RESET y anterior al control final.

Por tanto, no debe declararse que esas selecciones posteriores quedaron verificadas documentalmente.

Las pruebas OTA positivas muestran funcionamiento en las sesiones posteriores, pero no sustituyen una lectura histórica inexistente de `status`.

---

# 11. Interpretación de los experimentos históricos con cero recepciones

## 11.1. Problema pendiente

El operador reporta que campañas realizadas semanas antes produjeron cero recepciones con configuraciones proprietary Sub-GHz.

La campaña del 15–16 de octubre ofrece un nuevo punto de referencia: **los mismos tipos generales de configuración pueden producir recepciones satisfactorias bajo el procedimiento actual**.

Esto justifica una auditoría comparativa, pero no permite determinar automáticamente la causa de los resultados históricos.

El presente documento no reinterpreta ni reclasifica las ejecuciones antiguas. Para ello deberán utilizarse sus registros originales.

## 11.2. Hipótesis que deben investigarse

| Hipótesis                              | Relación con los resultados actuales                                 | Estado                                                            |
| -------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Selección incorrecta de banda          | La campaña actual verificó inicialmente `band2` y `Band: 1`          | No descartada                                                     |
| Comandos o argumentos incorrectos      | El procedimiento actual utilizó comandos y parámetros inspeccionados | No descartada                                                     |
| Diferencias entre scripts              | Se utilizó un runner distinto de algunos procedimientos históricos   | No descartada                                                     |
| RESET como requisito previo            | Hubo resultados positivos sin reiniciar entre cada preset            | Necesidad universal debilitada; causa histórica no descartada     |
| Diferencias de firmware                | No existe correspondencia binaria completa documentada               | No descartada                                                     |
| Problemas de captura/buffering         | El runner actual recuperó paquetes después del ciclo TX              | No descartada para herramientas históricas                        |
| Estado del RF switch                   | No se midió directamente durante la campaña                          | No descartada                                                     |
| Problemas de antena o ruta RF          | No hay caracterización instrumental comparable                       | No descartada                                                     |
| Errores de comunicación PC–dispositivo | Se observaron timeouts y fallos de apertura COM                      | Demostrados como incidencias, no como causa de cero OTA histórico |

## 11.3. Implicaciones de la hipótesis RESET

Los resultados actuales demuestran que:

1. Una preparación inicial con reinicio puede preceder a una campaña satisfactoria.
2. Se pueden ejecutar varios presets consecutivos satisfactoriamente sin reiniciar entre cada uno.
3. Un RESET físico puede interrumpir una ejecución activa.
4. Los dispositivos pueden volver a funcionar en una sesión posterior.

No demuestran que RESET haya sido la causa de los resultados históricos negativos ni que pueda descartarse cualquier problema de estado persistente.

## 11.4. Ruta recomendada para Codex

La auditoría posterior debe comparar la campaña histórica con la actual siguiendo un orden reproducible.

| Prioridad | Investigación                               | Evidencia requerida                                                                |
| --------- | ------------------------------------------- | ---------------------------------------------------------------------------------- |
| P0        | Reconstruir comandos históricos             | Comando literal, argumentos, salida, exit code                                     |
| P0        | Comparar selección de banda                 | `band1`/`band2`, respuestas Shell, estado reportado                                |
| P0        | Comparar scripts y secuencia de operaciones | Código del helper, funciones llamadas y tiempos                                    |
| P0        | Comparar TX/RX y criterio de recepción      | ACK, eventos RX, marcadores, estadísticas                                          |
| P1        | Revisar firmware de control RP2040          | Ruta Shell → control de banda                                                      |
| P1        | Revisar firmware CC1352                     | Configuración RF, inicio RX, transmisión y entrega de eventos                      |
| P1        | Revisar la biblioteca Python                | `configure_prop()`, `transmit()`, `read_packets()`, `get_stats()`                  |
| P1        | Auditar versiones instaladas                | Hash de binarios, commits y variantes de hardware                                  |
| P2        | Investigar RF switch físico                 | Señales de control y esquemático de la revisión real                               |
| P2        | Caracterizar recepción y metadatos          | RSSI, LQI, canal, timestamps y CRC                                                 |
| P2        | Ejecutar nuevas pruebas discriminantes      | Únicamente cuando la comparación documental identifique una incertidumbre concreta |

### Requisito metodológico

Codex deberá reconstruir los dos procedimientos antes de atribuir una causa.

La comparación debe identificar:

**Qué cambió → en qué capa cambió → qué efecto puede producir → qué evidencia lo respalda → cómo falsar la hipótesis.**

No se deberá modificar firmware o scripts únicamente porque una hipótesis parezca plausible.

---

# 12. Conclusiones

**C1 — Comunicación OTA funcional.**
Los dos CatSniffer entregaron paquetes a través de FeralRF en las 13 configuraciones Sub-GHz evaluadas, con 10/10 paquetes válidos en al menos una ejecución de cada preset.

**C2 — Control positivo reproducido.**
La configuración de referencia obtuvo cinco ejecuciones completas satisfactorias en distintos momentos de la campaña.

**C3 — Amplia cobertura de configuraciones solicitadas.**
Se verificaron combinaciones de FSK y GFSK, distintas frecuencias y velocidades de símbolos. No se caracterizó independientemente la señal RF.

**C4 — Errores de comunicación diferenciados.**
Se identificaron dos timeouts de `get_stats()` y una indisponibilidad de `COM33`, sin confundirlos con fallos de recepción RF.

**C5 — Contrapruebas RESET documentadas.**
El RESET físico interrumpió las sesiones de prueba. La recuperación funcional posterior quedó demostrada mediante un nuevo control satisfactorio.

**C6 — Metadatos pendientes de validar.**
Los valores reportados de canal, RSSI, LQI y timestamps requieren revisión antes de utilizarse como métricas físicas.

**C7 — Corrección documental necesaria.**
Se identificó una conversión incorrecta del sync word utilizado en los presets Wireless M-Bus. El valor reportado fue `0x543D0000`.

**C8 — Causa de los resultados históricos negativos no determinada.**
La campaña actual proporciona una referencia funcional sólida, pero no identifica por sí sola por qué experimentos anteriores produjeron cero recepciones.

**Resultado global:** campaña OTA Sub-GHz de 13 presets completada satisfactoriamente dentro de su alcance funcional.

**Siguiente fase:** auditoría comparativa del procedimiento histórico y del actual, utilizando los registros originales y las implementaciones correspondientes.

---

# 13. VALIDATION_MATRIX — Campaña OTA Sub-GHz, 15–16 de octubre de 2026

> Matriz complementaria de la campaña actual. Sigue la organización conceptual del archivo `docs/VALIDATION_MATRIX.md` del repositorio ElectronicCats/FeralRF.
>
> No sustituye la matriz general del repositorio ni modifica sus resultados históricos. Los estados aquí registrados se refieren exclusivamente a las ejecuciones documentadas en `Experimentos FeralRF OTA SubGHz.txt`.

## 13.1. Leyenda

| Estado          | Significado                                             |
| --------------- | ------------------------------------------------------- |
| `PASS-RF`       | Intercambio funcional de paquetes cumplido              |
| `PASS-CTRL`     | Control o comando confirmado sin validar por sí solo RF |
| `PARTIAL`       | Resultado incompleto en una ejecución evaluable         |
| `FAIL-RF`       | Prueba RF completa con criterio incumplido              |
| `BLOCKED`       | Ejecución impedida o interrumpida                       |
| `NOT VALIDATED` | Fuera del alcance o sin evidencia suficiente            |
| `DEFERRED`      | Evaluación aplazada                                     |

### Niveles de evidencia

| Nivel | Evidencia                                            |
| ----- | ---------------------------------------------------- |
| A     | Medición RF física independiente                     |
| B     | Comportamiento observado mediante dispositivos y API |
| C     | Control/API, ACK o estado Shell                      |
| D     | Código o configuración inspeccionada                 |
| E     | Inferencia o hipótesis                               |
| F     | No evaluado                                          |

La campaña actual proporciona principalmente evidencia **B/C/D**.

No se dispone de evidencia instrumental de nivel A para frecuencia, potencia, modulación o selección eléctrica del RF switch.

## 13.2. Baseline de comandos

| Comando / API            | Procesador o interfaz de control | Componente / función             | Estado                             |
| ------------------------ | -------------------------------- | -------------------------------- | ---------------------------------- |
| `status`                 | Shell RP2040                     | Consultar estado lógico          | PASS-CTRL                          |
| `boot`                   | Shell RP2040                     | Preparar reinicio/control CC1352 | PASS-CTRL                          |
| `exit`                   | Shell RP2040                     | Regresar a passthrough           | PASS-CTRL                          |
| `band2`                  | Shell RP2040                     | Solicitar ruta Sub-GHz           | PASS-CTRL                          |
| `Radio.init()`           | Python → Cat-Bridge              | Inicialización FeralRF           | PASS-CTRL                          |
| `Radio.set_phy()`        | Python → CC1352                  | Selección PHY                    | PASS-CTRL                          |
| `Radio.configure_prop()` | Python → CC1352                  | Parámetros propietarios          | PASS-CTRL                          |
| `Radio.set_power()`      | Python → CC1352                  | Potencia solicitada              | PASS-CTRL                          |
| `Radio.get_stats()`      | Python → CC1352                  | Estadísticas                     | PASS-CTRL con incidencias          |
| `Radio.start_rx()`       | Python → CC1352                  | Activación RX                    | PASS-CTRL                          |
| `Radio.transmit()`       | Python → CC1352                  | Solicitud TX                     | PASS-CTRL                          |
| `Radio.read_packets()`   | Python → Cat-Bridge              | Obtención de paquetes            | PASS-RF funcional                  |
| `Radio.stop_rx()`        | Python → CC1352                  | Detención RX                     | PASS-CTRL en ejecuciones completas |
| `Radio.disconnect()`     | Python → USB serial              | Cierre de sesión                 | PASS-CTRL en ejecuciones completas |

Los ACK de configuración no constituyen validación de parámetros físicos.

## 13.3. Matriz OTA por preset

| Test ID | Preset                  | Frecuencia MHz | Modulación solicitada | Marcador   | TX ACK | RX CRC-good + marcador | Resultado   |
| ------- | ----------------------- | -------------: | --------------------- | ---------- | -----: | ---------------------: | ----------- |
| P01     | `gfsk_902_50k`          |          902.2 | GFSK                  | `A5010001` |  10/10 |                  10/10 | **PASS-RF** |
| P02     | `wisun_915_fsk_100k`    |          902.2 | FSK                   | `A5020001` |  10/10 |                  10/10 | **PASS-RF** |
| P03     | `wisun_915_fsk_150k`    |          902.2 | FSK                   | `A5030001` |  10/10 |                  10/10 | **PASS-RF** |
| P04     | `wisun_915_fsk_200k`    |          902.2 | FSK                   | `A5040001` |  10/10 |                  10/10 | **PASS-RF** |
| P05     | `wisun_915_fsk_300k`    |          902.2 | FSK                   | `A5050001` |  10/10 |                  10/10 | **PASS-RF** |
| P06     | `gfsk_915_50k`          |          915.0 | GFSK                  | `A5060001` |  10/10 |                  10/10 | **PASS-RF** |
| P07     | `sidewalk_915_fsk_50k`  |          915.0 | FSK                   | `A5070001` |  10/10 |                  10/10 | **PASS-RF** |
| P08     | `sidewalk_915_fsk_250k` |          915.0 | FSK                   | `A5080001` |  10/10 |                  10/10 | **PASS-RF** |
| P09     | `gfsk_868_50k`          |          868.0 | GFSK                  | `A5090001` |  10/10 |                  10/10 | **PASS-RF** |
| P10     | `gfsk_868_100k`         |          868.0 | GFSK                  | `A50A0001` |  10/10 |                  10/10 | **PASS-RF** |
| P11     | `wireless_mbus_s_868`   |          868.3 | GFSK                  | `A50B0001` |  10/10 |                  10/10 | **PASS-RF** |
| P12     | `wireless_mbus_t_868`   |         868.95 | GFSK                  | `A50C0001` |  10/10 |                  10/10 | **PASS-RF** |
| P13     | `wireless_mbus_c_868`   |         868.95 | GFSK                  | `A50D0001` |  10/10 |                  10/10 | **PASS-RF** |

**Alcance:** recepción de paquetes propietarios FeralRF, no conformidad de protocolo.

## 13.4. Registro cronológico completo de ejecuciones

Todos los timestamps corresponden a UTC.

| ID  | Inicio UTC      | Tipo            | Preset / identificador abreviado                 | TX ACK |    RX válido | Estado           |
| --- | --------------- | --------------- | ------------------------------------------------ | -----: | -----------: | ---------------- |
| E01 | 15-oct 21:08:08 | Control inicial | `wisun_915_fsk_50k` / Checkpoint 3A              |     10 |           10 | PASS-RF          |
| E02 | 15-oct 22:11:27 | P01             | `gfsk_902_50k` / run-01                          |     10 |           10 | PASS-RF          |
| E03 | 15-oct 22:14:21 | P02             | `wisun_915_fsk_100k` / run-01                    |     10 |           10 | PASS-RF          |
| E04 | 15-oct 22:16:36 | P03             | `wisun_915_fsk_150k` / run-01                    |     10 |           10 | PASS-RF          |
| E05 | 15-oct 22:21:16 | Control         | `control-after-3-presets-run-01`                 |     10 |           10 | PASS-RF          |
| E06 | 15-oct 22:23:13 | P04             | `wisun_915_fsk_200k` / run-01                    |     10 |           10 | PASS-RF          |
| E07 | 15-oct 22:24:23 | P05             | `wisun_915_fsk_300k` / run-01                    |     10 |           10 | PASS-RF          |
| E08 | 15-oct 22:35:46 | P06             | `gfsk_915_50k` / run-01                          |     10 |           10 | PASS-RF          |
| E09 | 15-oct 22:36:43 | Control         | `control-after-6-presets-run-01`                 |      0 |            0 | BLOCKED          |
| E10 | 15-oct 22:41:11 | Control         | `control-after-6-presets-run-02`                 |     10 |           10 | PASS-RF          |
| E11 | 15-oct 22:43:01 | P07             | `sidewalk_915_fsk_50k` / run-01                  |     10 |           10 | PASS-RF          |
| E12 | 15-oct 22:46:04 | P08             | `sidewalk_915_fsk_250k` / run-01                 |     10 |           10 | PASS-RF          |
| E13 | 15-oct 22:49:50 | P09             | `gfsk_868_50k` / run-01                          |     10 |           10 | PASS-RF          |
| E14 | 15-oct 22:53:00 | P10             | `gfsk_868_100k` / run-01                         |     10 |           10 | PASS-RF          |
| E15 | 15-oct 22:54:05 | Repetición      | `gfsk_868_100k` / run-01 reutilizado             |      0 |            0 | BLOCKED          |
| E16 | 15-oct 22:54:12 | Repetición      | `gfsk_868_100k` / run-01 reutilizado             |     10 |           10 | PASS-RF          |
| E17 | 15-oct 22:56:29 | Control         | `control-after-10-presets-post-reconnect-run-01` |     10 |           10 | PASS-RF          |
| E18 | 15-oct 22:58:27 | P11, intento 1  | `wireless_mbus_s_868` / run-01                   |      0 |            0 | BLOCKED          |
| E19 | 15-oct 23:00:14 | P11, intento 2  | `wireless_mbus_s_868` / run-02                   |     10 |           10 | PASS-RF          |
| E20 | 16-oct 14:52:01 | P12             | `wireless_mbus_t_868` / run-01                   |     10 |           10 | PASS-RF          |
| E21 | 16-oct 14:54:19 | P13             | `wireless_mbus_c_868` / run-01                   |     10 |           10 | PASS-RF          |
| E22 | 16-oct 14:54:52 | RESET RX        | `wireless_mbus_c_868` / run-01 reutilizado       |     10 | 0 observados | BLOCKED inducido |
| E23 | 16-oct 14:55:05 | RESET TX        | `wireless_mbus_c_868` / run-01 reutilizado       |      5 |            5 | BLOCKED inducido |
| E24 | 16-oct 15:01:00 | Control final   | `control-after-13-presets-post-reset-run-01`     |     10 |           10 | PASS-RF          |

**Nota:** los valores cero de E09, E15 y E18 corresponden a ejecuciones que no llegaron a transmitir. En E22, la lectura del RX fue interrumpida; cero eventos entregados no significa cero paquetes recibidos físicamente.

## 13.5. Matriz de incidencias

| Incidente                | Test ID | Interfaz afectada | Error observado   | Recuperación posterior |
| ------------------------ | ------- | ----------------- | ----------------- | ---------------------- |
| Timeout `get_stats()` TX | E09     | `COM33`           | `TimeoutError`    | E10 PASS-RF            |
| Puerto TX no disponible  | E15     | `COM33`           | `ConnectionError` | E16 PASS-RF            |
| Timeout `get_stats()` RX | E18     | `COM88`           | `TimeoutError`    | E19 PASS-RF            |
| RESET físico RX          | E22     | `COM88`           | `SerialException` | E24 PASS-RF            |
| RESET físico TX          | E23     | `COM33`           | `SerialException` | E24 PASS-RF            |

Las recuperaciones posteriores no identifican las causas raíz de los incidentes originales.

## 13.6. Matriz por PHY

| PHY / categoría             | Control                     | OTA                        | Estado            |
| --------------------------- | --------------------------- | -------------------------- | ----------------- |
| `PROPRIETARY_GFSK` con FSK  | PASS-CTRL                   | 10/10 en presets ensayados | PASS-RF funcional |
| `PROPRIETARY_GFSK` con GFSK | PASS-CTRL                   | 10/10 en presets ensayados | PASS-RF funcional |
| IEEE 802.15.4               | No evaluado en esta campaña | No evaluado                | NOT VALIDATED     |
| BLE PHY                     | No evaluado en esta campaña | No evaluado                | NOT VALIDATED     |
| LoRa/SX1262                 | No evaluado                 | No evaluado                | NOT VALIDATED     |

## 13.7. Matriz por modulación propietaria

| Modulación solicitada | Frecuencias probadas                | Presets nuevos | Resultado |
| --------------------- | ----------------------------------- | -------------: | --------- |
| FSK                   | 902.2 y 915 MHz                     |              6 | PASS-RF   |
| GFSK                  | 868, 868.3, 868.95, 902.2 y 915 MHz |              7 | PASS-RF   |
| OOK                   | No probado                          |              0 | DEFERRED  |
| MSK                   | No probado                          |              0 | DEFERRED  |
| 4-FSK                 | No probado                          |              0 | DEFERRED  |
| 4-GFSK                | No probado                          |              0 | DEFERRED  |

La agrupación corresponde al `mod_type` solicitado por los presets; la modulación física no fue medida independientemente.

## 13.8. Matriz de switching y recuperación

| Secuencia                                                | Evidencia                                       | Estado                       |
| -------------------------------------------------------- | ----------------------------------------------- | ---------------------------- |
| `boot` → `exit`                                          | `BOOT` y `PASSTHROUGH`                          | PASS-CTRL                    |
| `band2` → `status`                                       | `SUB-GHz Band`, `Band: 1`                       | PASS-CTRL                    |
| Cambios de preset dentro de Sub-GHz sin reset por prueba | 13 presets con recepción                        | PASS-RF funcional            |
| 915 → 868 MHz mediante `configure_prop()`                | Recepciones posteriores                         | PASS-RF funcional            |
| 868 → 902.2 MHz                                          | Control posterior 10/10                         | PASS-RF funcional            |
| Desconexión/reconexión USB                               | Un BLOCKED seguido de PASS-RF                   | Recuperación en nueva sesión |
| RESET RX durante ejecución                               | Excepción serial                                | BLOCKED inducido             |
| RESET TX durante ejecución                               | Cinco ACK y cinco paquetes antes de excepciones | BLOCKED inducido             |
| Control posterior a RESET                                | 10/10                                           | PASS-RF funcional            |
| Estado eléctrico CTF1/CTF2/CTF3                          | Sin medición                                    | NOT VALIDATED                |
| Posición física del RF switch                            | Sin medición                                    | NOT VALIDATED                |

## 13.9. Matriz de protocolo frente a validación

| Contexto del preset  | PHY/modulación solicitada | Paquetes FeralRF entre CatSniffer | Interoperabilidad de protocolo  |
| -------------------- | ------------------------- | --------------------------------- | ------------------------------- |
| Wi-SUN               | Sub-GHz FSK               | PASS-RF                           | NOT VALIDATED                   |
| Amazon Sidewalk      | Sub-GHz FSK               | PASS-RF                           | NOT VALIDATED                   |
| Wireless M-Bus S/T/C | Sub-GHz GFSK              | PASS-RF                           | NOT VALIDATED                   |
| GFSK genérico        | Sub-GHz GFSK              | PASS-RF                           | No aplica como stack específico |

## 13.10. Validaciones pendientes

| ID  | Tema                                                  | Estado        | Evidencia requerida                          |
| --- | ----------------------------------------------------- | ------------- | -------------------------------------------- |
| V01 | Causa histórica de cero recepciones                   | OPEN          | Comparación completa de comandos y ejecución |
| V02 | Estado físico del RF switch                           | OPEN          | Revisión hardware y medición                 |
| V03 | Correspondencia firmware–commit                       | OPEN          | Hash y procedencia de binarios               |
| V04 | Tratamiento de `sync_word`                            | OPEN          | Python, firmware y especificación aplicable  |
| V05 | Unidades de `deviation` y `rx_bw`                     | OPEN          | Definiciones y conversión del firmware       |
| V06 | `channel=37`                                          | OPEN          | Estructuras de paquetes y decodificación     |
| V07 | RSSI y LQI                                            | OPEN          | Ruta de generación y escalado                |
| V08 | `device_timestamp_us`                                 | OPEN          | Origen, reloj y unidades                     |
| V09 | Timeouts intermitentes de `get_stats()`               | OPEN          | Secuencia de solicitudes y respuestas        |
| V10 | Reconexión USB y recuperación de sesión               | OPEN          | Gestión de puertos y lifecycle               |
| V11 | Interoperabilidad Wi-SUN                              | NOT VALIDATED | Implementación y equipo compatible           |
| V12 | Interoperabilidad Wireless M-Bus                      | NOT VALIDATED | Tramas conformes y equipo compatible         |
| V13 | Interoperabilidad Sidewalk                            | NOT VALIDATED | Stack y entorno compatible                   |
| V14 | Medición espectral                                    | NOT VALIDATED | Instrumentación RF                           |
| V15 | Reproducibilidad bajo distintas condiciones de enlace | NOT VALIDATED | Ensayos controlados adicionales              |

**Estado final de VALIDATION_MATRIX:** 13 presets nuevos con PASS-RF funcional, cinco controles positivos completos, una repetición satisfactoria y cinco ejecuciones bloqueadas documentadas. La auditoría causal de los experimentos históricos permanece abierta.
