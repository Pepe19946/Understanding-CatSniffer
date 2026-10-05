# Auditoría técnica de las validaciones FeralRF realizadas

Fecha de auditoría: 2026-10-05

## 1. Alcance y estado de los repositorios

Esta auditoría reconstruye la procedencia y el valor probatorio de las evaluaciones que tienen evidencia de preparación o ejecución en `CatSniffer-Understanding/05 - Evaluación`. No se ejecutó hardware, Catnip, FeralRF, pruebas unitarias ni scripts de validación durante la auditoría. Tampoco se flasheó, cambió de rama o modificó ningún repositorio fuente. Los informes existentes se trataron como registros sujetos a comprobación, no como autoridad.

### 1.1 Taxonomía de evidencia usada

| Etiqueta | Significado en este informe |
| --- | --- |
| **Declarado** | Afirmación de README, guía, matriz o comentario; no implica que el comportamiento ocurra. |
| **Implementado** | Existe una ruta ejecutable en el código actual. |
| **Probado unitariamente** | Un test automatizado comprueba una propiedad de software; no prueba el hardware. |
| **Histórico** | El repositorio declara una ejecución física anterior, pero faltan en el workspace actual sus logs crudos, hashes de binario o paquete completo de evidencia. |
| **Procedimiento registrado** | El informe conserva el comando o describe lo que el usuario hizo. |
| **Observación física** | La salida sólo se explica razonablemente por interacción con la placa o la radio. |
| **Inferencia** | Interpretación compatible con los datos, pero no observada de forma directa. |
| **Conclusión** | Veredicto limitado al alcance que permiten las categorías anteriores. |

La jerarquía seguida fue: código enlazado y formato de protocolo actual → scripts ejecutables actuales → tests de contrato → documentación técnica actual → historial Git → matrices históricas → guía local → informe de laboratorio. Una fuente posterior no convierte retroactivamente una observación incompleta en evidencia literal.

### 1.2 Repositorios y versiones inspeccionados

| Repositorio | Rama | Commit inspeccionado | Estado encontrado |
| --- | --- | --- | --- |
| FeralRF | `main` | `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`, 2026-07-22 | submódulo TI SDK modificado (`m firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01`) y `python/catnip` sin seguimiento |
| CatSniffer | `master` | `6da5050fefde59138fc11d9e2ecf7c0de36f3a22`, 2025-12-09 | limpio |
| CatSniffer-Firmware | `v3.x` | `c0cd5a45e019dbd14ed11d039aacb13f300e5731`, 2026-09-03 | limpio |
| CatSniffer-Tools | `fix/CLI_control` | `126f13bc0441ad3f37fe0b029160a3c526e4d309`, 2026-09-28 | `py` sin seguimiento; el checkout estaba tres commits detrás de `origin/fix/CLI_control` según el preflight anterior |
| CatSniffer-Tools-v3.3.2.1 | rama no mostrada por `branch --show-current` | `a06b7887a20693129ffb2ed567ed73284908d0a6`, 2026-07-09 | limpio |
| Sniffle | `master` | `3a53f5ab21df7599ad0c46475d322603c1a54bb7`, 2025-09-24 | limpio |

Los estados no limpios ya existían al iniciar esta auditoría y no se limpiaron ni alteraron. En particular, no puede atribuirse el contenido no seguido a esta auditoría.

### 1.3 Plataforma y cadena realmente examinada

El DUT registrado es CatSniffer V3, con RP2040 como terminador USB/puente y CC1352P7 como procesador que ejecuta FeralRF. La ruta de las pruebas FeralRF fue:

```text
PowerShell / CPython en Windows
  → scripts o feralrf.Radio
  → tramas COBS/CRC del protocolo FeralRF
  → COM88 / Cat-Bridge a 921600
  → RP2040 con passthrough UART
  → HostIFTask / CommandProcessor en CC1352P7
  → ControlTask / DataTask
  → RadioIF / TI RF driver
  → radio integrada y front-end de CatSniffer
```

Fuentes: `FeralRF/python/feralrf/radio.py`, `protocol.py`, `commands.py`; `firmware/cc1352/src/{host_if_task,command_processor,control_task,data_task,radio_if}.c`; `CatSniffer-Firmware/RP2040/catsniffer/src/main.c`; hardware CatSniffer. El SX1262/Cat-LoRa no intervino en las EV ejecutadas.

La identificación física inicial provino de un ejecutable instalado, `C:\Program Files\Catnip\catnip.exe`, que mostró `v3.3.3.0`. El checkout local de CatSniffer-Tools no pudo ejecutarse en aquel entorno por ausencia de `rich`. Por ello, la correspondencia exacta entre el binario Catnip que produjo el mapa físico y `CatSniffer-Tools@126f13b` **no está establecida**. Coinciden la versión visible y el formato esperado, pero no existe hash/build provenance del EXE.

### 1.4 Inventario de estado real

| EV | Estado que permite la evidencia | Nota principal |
| --- | --- | --- |
| EV-00 | **PARTIAL** | Se obtuvo mapa V3/COM con `catnip devices`, pero no se preservaron `devices --debug`, `identify`, `status` ni enumeración Windows completa exigidos por la guía revisada. |
| EV-01 | **PARTIAL**, evidencia de control favorable | Hay numerosos `INIT/GET_INFO` y varios `GET_STATS`, pero no una serie dedicada y temporizada de 3/3 `init+info+stats` conforme al criterio formal. |
| EV-02 | **PASS-control** | `SET_PHY/CHANNEL`, `RX_START`, lectura y `RX_STOP` se repitieron; EV-05 además aporta RF. |
| EV-03 | **PASS-control 5/5** | Cinco procesos independientes abrieron, inicializaron y cerraron COM88. |
| EV-04 | **BLOCKED** para `Radio.reset_device()`; **PASS-recovery manual 1/1** | KI-15 reproducido: COM90 calculado frente a COM87 real. La recuperación manual fue favorable, no 3/3 ni API pública. |
| EV-05 | **PASS-RF (recepción IEEE 802.15.4)** con limitación de atribución | Tres ventanas formales recibieron 41/43/43 paquetes; no se guardaron bytes de todos los frames ni captura independiente simultánea. |
| EV-06 | **PASS-control/state-rejection** | Error síncrono `0x05` y recuperación mediante STOP/STATS. No prueba RF simultánea. |
| EV-10 | **PASS-control 8/8** | Ocho PHY completaron el smoke; no hay evidencia RF individual. |
| EV-11 | **PASS-control reportado 27/27**, auditabilidad desigual | 18 presets tienen salida individual preservada; nueve de 902/915 MHz sólo tienen resumen agregado. Ninguno tiene PASS-RF actual. |
| EV-12 | **PREPARED / NOT TESTED** | Sólo existe definición e inspección de scripts; no hay comando ni salida física. |

La matriz y `00 - Proyecto/Estado y siguientes pasos.md` todavía dicen `NOT YET RUN` o “ninguno fue ejecutado físicamente”. Son documentos de planificación que quedaron obsoletos respecto de los logs; no invalidan éstos, pero no deben utilizarse como estado actual.

## 2. Cómo se derivó el plan de validación

### 2.1 Base histórica real

El plan local no apareció de una única especificación upstream. Se reconstruyó principalmente a partir de:

- `FeralRF/docs/VALIDATION_MATRIX.md`, originado en `7bd36a6` (2026-04-07, “Add RF validation matrix and baseline smoke tooling”) y actualizado por `bee7f67`, `f327cf6`, `b089f71`, `c45114b`, `10fc47a`, `6e34ef2`, `2595276` y `7ad0a1a`;
- `FeralRF/python/examples/run_validation_baseline.sh`, que ejecuta pasos de control/OTA y llama a reset entre pasos;
- scripts `smoke_phase2.py`, `smoke_phy4_ieee154.py`, `smoke_prop_phase1.py` y la familia `smoke_tx_*`;
- API Python y handlers actuales, que definen comandos, respuestas y errores;
- resultados históricos declarados en `VALIDATION_MATRIX.md`: baseline 18/18, OTA del 2026-04-08, 433 MHz marginal, OOK 433 0/10, Wi-SUN/Sidewalk 70/70 y MIOTY 0/10;
- riesgos actuales observables en código, por ejemplo ACK previo a trabajo RF diferido, reset por `Bridge+2`, lock OOK y errores RF asíncronos;
- metodología nueva añadida en la guía local: repeticiones 3/3 o 5/5, criterios `PASS-control/PASS-RF`, tráfico Zigbee independiente y reglas de parada.

Los datos históricos son procedencia válida para formular hipótesis y repetir ensayos, pero no son resultados físicos de esta campaña. `VALIDATION_MATRIX.md` no incluye los logs crudos, hash del HEX o identificación completa de las placas de 2026-04/05.

### 2.2 Mapa de procedencia por EV ejecutada o preparada

| EV | Objetivo, comando y parámetros: fuente fuerte | Expectativa/fallo/recuperación: fuente fuerte | Naturaleza del diseño |
| --- | --- | --- | --- |
| EV-00 | `Radio.list_devices/_get_shell_port` en `radio.py`; Catnip `usb_connection.py:find_devices/_group_ports_by_device/_map_roles` y `device/cli.py` en `CatSniffer-Tools@126f13b` | Riesgo `Bridge+2` visible en `radio.py:_get_shell_port`; identificación por Catnip/Windows | **Ensamblada.** No hay EV-00 upstream; identidad inequívoca, comparación cruzada y criterio STOP son metodología local. |
| EV-01 | `Radio.init/get_stats`; `command_processor.c` casos `CMD_RADIO_INIT/GET_INFO/GET_STATS`; baseline histórico | ACK/INFO y STATS se derivan del protocolo. Tres repeticiones y tiempo son criterio local | **Ensamblada**, aunque usa operaciones upstream directas. |
| EV-02 | Directamente `smoke_phy4_ieee154.py`; PHY 4, canal 25 y 5 s también aparecen en `run_validation_baseline.sh` | `DataTask_poll` difiere el `RadioIF_startRx`; posible `ERR_RF_INIT_FAILED`; STOP en API/handler | **Directamente derivada**, con clasificación control/RF añadida localmente. |
| EV-03 | `Radio.connect/disconnect/init` y sus reintentos | No existe script upstream dedicado ni resultado histórico específico | **Diseñada parcialmente por inferencia.** Cinco procesos/5 de 5 y demora creciente son metodología local. |
| EV-04 | `radio.py:reset_device/_get_shell_port`; `run_validation_baseline.sh:_reset_one`; comandos `boot/exit` del RP2040 | `CatSniffer-Firmware/.../shell_commands.c:cmd_boot/cmd_exit` y `main.c:change_mode/reset_cc1352`; OOK/switching histórico | **Ensamblada.** El bloqueo seguro por mismatch y 3/3 son criterios locales. |
| EV-05 | `smoke_phy4_ieee154.py`; `radio_if.c:RadioIF_processIeee154Packets`; canal 25 aportado por la fuente física conocida | CRC/RSSI/LQI/timestamp vienen de RadioIF/DataTask; fuente independiente y 3×30 s son metodología local | **Script directo + diseño experimental local.** La atribución al dispositivo Zigbee no estaba especificada upstream. |
| EV-06 | `command_processor.c:CMD_TX_RAW`; `control_task.c:ControlTask_canStartTx` y `ERR_INVALID_STATE`; `Radio.transmit`/`CommandError.error_code` | Rechazo antes de encolar TX y recuperación STOP/STATS se deducen del estado y API | **Ensamblada.** El harness PowerShell fue creado para la campaña. |
| EV-10 | `smoke_phase2.py`, enum `PHY`, baseline y matriz histórica | 8 PHY y ACK de control son upstream; reset entre pasos viene del baseline/histórico | **Directamente derivada**, salvo selección `(PHY 1, canal 9)` y criterio 8/8 local. El baseline actual usa canal 37 para PHY 1. |
| EV-11 | `PROP_PRESETS`, `smoke_prop_phase1.py`, `run_validation_baseline.sh`, `Radio.configure_prop`, `RadioIF_setPropConfig` | OOK lock en API/código; MIOTY pending en preset y commit `6e34ef2`; 433 marginal y switching en matriz histórica | **Script directo + barrido exhaustivo diseñado localmente.** Upstream no define exactamente el subconjunto “31−4=27” como una sola prueba. |
| EV-12 | Familia `smoke_tx_{phase1,frame_phase1,burst_phase1,continuous_phase1}.py`; handlers de TX | ACK antes de finalización en `control_task.c:ControlTask_processTxRaw`; observador requerido | **Directamente derivada**, pero sólo preparada. |

### 2.3 Criterios que no se trazan a una especificación upstream

No se encontró una fuente upstream única que imponga 3/3 para EV-01/04/05, 5/5 para EV-03, 8/8 para EV-10, ni el inventario exhaustivo de 27 presets para EV-11. Son criterios razonables de diseño experimental creados al construir la guía. Deben presentarse como metodología de esta campaña, no como requisitos oficiales de FeralRF.

La distinción `PASS-control` frente a `PASS-RF` también es una mejora metodológica local. Está apoyada por la arquitectura del código —los ACK se emiten antes o sin confirmación física—, pero no es una taxonomía formal upstream.

## 3. Auditoría por EV

## 3.1 EV-00 — Enumeración e identidad

**Intención.** Identificar placa y interfaces antes de abrir FeralRF o resetear otro puerto por error.

**Procedimiento registrado.** `# Registro provisional de validación experimental...txt` conserva `catnip devices` y su fila:

```text
CatSniffer #1 | v3 (RP2040 + CC1352P7) | Bridge COM88 | LoRa COM86 | Shell COM87
```

También conserva resolución del EXE, `catnip --help`, `pip show catnip`, `find_spec('catnip')` y el fallo del checkout local por `ModuleNotFoundError: rich`. No conserva la salida de `devices --debug`, `identify --device N`, `status --device N` ni `Get-CimInstance Win32_SerialPort`.

**Qué prueba.** La salida observada prueba que el Catnip instalado agrupó tres interfaces y clasificó la placa como V3. La desigualdad `COM87 != COM88+2` queda establecida. No prueba por sí sola que CatSniffer #1 sea una identidad persistente, ni el serial/HWID/location, ni que el EXE corresponda exactamente a `126f13b`.

**Esperado frente a observado.** El resultado satisface el núcleo del preflight original, pero no el procedimiento ampliado de la guía actual. El informe inicial dice correctamente que no se debía ejecutar `reset_device()` a ciegas.

**Veredicto: PARTIAL.** Mapeo funcional suficiente para continuar con COM88 y bloquear el reset inseguro; trazabilidad física/USB incompleta según la guía actual.

## 3.2 EV-01 — Init, GET_INFO y GET_STATS

**Intención.** Confirmar comunicación host→RP2040→CC1352P7 y obtener identidad/contadores sin atribuir funcionamiento RF.

**Ruta.** `Radio.init()` abre COM, envía `RADIO_INIT`, exige ACK, envía `GET_INFO` y decodifica 12 bytes; `get_stats()` exige `RSP_STATS` de al menos 16 bytes. En firmware, `CMD_RADIO_INIT` llama `ControlTask_onRadioInit`, que detiene RX y reinicia métricas; GET_INFO/GET_STATS serializan estado.

**Procedimiento real.** No hay informe EV-01 dedicado con tres ciclos `init+info+stats`. Sí existen muchas salidas literales de:

```text
DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ...)
```

EV-03 aporta cinco `init`; EV-04, EV-06, EV-10 y EV-11 aportan respuestas posteriores, varias con STATS.

**Qué prueba.** La ruta de control y parsing funciona repetidamente en la campaña. El serial hexadecimal decodifica bytes constantes definidos por firmware (`FERALRF1`), no un identificador único de la placa. La versión `1.0.0` es el payload de firmware actual, no prueba por sí sola el commit o hash del binario flasheado.

**Veredicto: PARTIAL respecto al EV formal; evidencia funcional fuerte.** No corresponde afirmar 3/3 dedicado con timings porque no está registrado. Tampoco corresponde dejarlo como “no ejecutado”: sus operaciones fueron observadas repetidamente.

## 3.3 EV-02 — RX IEEE controlado y parada

**Intención.** Configurar IEEE 802.15.4, solicitar RX y volver a idle.

**Procedimiento real.** Las cinco corridas RX preservadas (dos preliminares y tres formales EV-05) usaron `smoke_phy4_ieee154.py`, canal 25, ventanas de 30 o 60 s, y terminaron con `RX_STOP ACK`.

**Ruta y matiz temporal.** `RX_START ACK` se envía en `command_processor.c` después de marcar el evento, antes de que `DataTask_poll` ejecute `RadioIF_startRx`. Si éste falla, llega `RSP_ERROR/ERR_RF_INIT_FAILED` asíncrono. Por eso un ACK aislado sería sólo control. En estas corridas llegaron paquetes y STOP, de modo que la evidencia supera el ACK.

**Veredicto: PASS-control.** RX start/stop fue repetible y el estado fue recuperable. La evidencia RF se asigna separadamente a EV-05.

## 3.4 EV-03 — Reconexión limpia

**Procedimiento literal.** `Registro — EV-03 reconexión limpia.txt` conserva el pipeline PowerShell que lanza cinco procesos Python independientes. Cada uno crea `Radio(port='COM88')`, ejecuta `init()` y `disconnect()`.

**Observado.** Cinco `DeviceInfo` idénticos; sin timeout, excepción ni puerto ocupado.

**Qué prueba.** Apertura, vaciado de buffers, intercambio init/info, cierre y nueva apertura entre procesos 5/5. No prueba una sesión larga, reconexión en una misma instancia, reconexión tras USB loss ni ausencia de fuga interna en el RP2040/CC.

**Corrección procedimental.** Coincide exactamente con el comando que la guía terminó definiendo. Como ese comando fue diseñado localmente, la coincidencia demuestra disciplina respecto al plan, no reproducción de un test upstream.

**Veredicto: PASS-control 5/5.** El adverbio “provisionalmente” del informe es conservador; el escenario concreto quedó cerrado.

## 3.5 EV-04 — Reset y reinicialización / KI-15

**Intención.** Validar recuperación antes de modos frágiles sin abrir un COM ajeno.

**Código actual.** `Radio._get_shell_port()` extrae el sufijo numérico de COM88 y suma dos, por lo que retorna COM90. `reset_device()` cerraría Bridge, abriría ese puerto, escribiría `boot\r\n`, después `exit\r\n`, esperaría, reabriría Bridge y ejecutaría `init()`.

**Procedimiento real.** Se verificó `_get_shell_port()` y se obtuvo COM90. Correctamente no se invocó la API contra ese puerto. Sobre el Shell mapeado por Catnip, COM87, se ejecutó:

1. escritura exacta de `boot\r\n`;
2. intento `init/get_stats` en COM88, que agotó timeout;
3. escritura exacta de `exit\r\n`;
4. `init/get_stats` exitosos por COM88 sin reconectar USB.

**Qué prueban `OPEN/SENT/CLOSED`.** Sólo que pySerial abrió, escribió y cerró. No son respuestas del RP2040. El cambio posterior de disponibilidad en COM88 y la recuperación constituyen la observación física fuerte.

**Correspondencia con firmware oficial.** En `CatSniffer-Firmware@c0cd5a4`, `shell_commands.c:cmd_boot` llama `change_mode(BOOT)` y `cmd_exit` llama `change_mode(PASSTHROUGH)`. `main.c:change_mode` acciona boot/reset del CC, cambia el UART a 500000 en BOOT y a 921600 al salir. Por tanto, la secuencia manual es técnicamente coherente con el firmware RP2040 actual, aunque el log no leyó las respuestas textuales `BOOT`/`PASSTHROUGH`.

**Contradicción descubierta.** La guía describe la ruta como “GPIO15→RESET_N”, siguiendo `FeralRF/hardware/PINOUT.md`. Sin embargo, el firmware RP2040 ejecutable actual obtiene `pin-reset` del overlay `rpi_pico.overlay`, donde el alias apunta a `gpio0 3`; `pin-boot` apunta a GPIO2. El esquema contiene una red `RESET_CC`, pero la afirmación concreta GPIO15 no coincide con este target ejecutable. Para describir el procedimiento actual debe citarse el alias/overlay, no GPIO15, hasta reconciliar revisiones de hardware/documentación.

**Veredicto dual.**

- `Radio.reset_device()`: **BLOCKED**, KI-15 reproducido (`COM90` calculado frente a `COM87` real).
- Ruta manual por Shell explícito: **PASS-recovery 1/1**, no PASS formal 3/3 y no prueba “watchdog reset”.

Los informes posteriores que llaman “reset correcto” a sólo `OPEN/SENT/CLOSED` son demasiado fuertes si no incluyen verificación posterior. Cuando una prueba siguiente obtiene INFO/STATS, existe evidencia indirecta de recuperación, pero no de cada detalle eléctrico interno.

## 3.6 EV-05 — Primera recepción RF IEEE 802.15.4

**Intención.** Pasar de aceptación de comandos a recepción física con `PROTOCOL-DEVICE-ZIGBEE-CH25` como fuente independiente.

**Procedimiento formal preservado.** Tres ejecuciones literales de:

```powershell
python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
```

Resultados: 41, 43 y 43 paquetes. La primera trama mostrada en cada corrida tuvo `crc_ok=True`, canal 25, RSSI −85/−79/−83 dBm, LQI 52/57/56 y longitudes 52/5/52 bytes. Dos corridas preliminares añadieron 48 paquetes/30 s y 100/60 s. Existe además un control negativo descrito en otro canal, pero no conserva canal, comando ni stdout y sólo vale como observación auxiliar no reproducible.

**Ruta de datos.** `RadioIF_processIeee154Packets` extrae RSSI, correlación/LQI, bit de error CRC y timestamp del entry TI; `DataTask_emitRxPacket` los serializa; `Radio.read_packets` los convierte en `Packet`. Esto respalda que `crc_ok` no fue inventado por el script.

**Cumplimiento del plan.** Se cumplieron tres ventanas de 30 s y STOP. No se cumplió la instrucción de registrar los bytes raw y metadatos de **cada** paquete: el script sólo imprime conteo y primera trama, y ni siquiera imprime sus bytes. Tampoco hubo observador independiente simultáneo que correlacionara frames o confirmara la actividad de la fuente durante las mismas ventanas.

**Qué prueba.** Prueba recepción física reproducible de frames compatibles con IEEE 802.15.4 en canal 25 a través del CC1352P7 y la ruta host. La atribución a la fuente Zigbee conocida es plausible y fuerte por contexto, repetición y canal, pero sigue siendo inferencia sin correlación simultánea. No prueba asociación, parsing Zigbee, stack Zigbee, sensibilidad, PER ni exactitud RSSI.

**Veredicto: PASS-RF limitado a recepción IEEE 802.15.4.** Mantener `PASS-RF` es defendible si su etiqueta se lee con ese alcance. La afirmación más fuerte “procedente de la fuente concreta” debe llevar la limitación anterior. El registro sería académicamente más sólido con PCAP/bytes y observador concurrente.

## 3.7 EV-06 — Exclusión RX/TX

**Procedimiento.** El informe conserva dos errores de harness sin interacción útil (quoting de PowerShell e import `Phy` en vez de `PHY`), una primera ejecución válida que consultó el atributo incorrecto `code`, inspección local de `Radio.transmit`/`CommandError`, y una ejecución definitiva. Ésta inició RX en PHY 4/canal 25, llamó `transmit(b'\x01', power_dbm=-20)`, obtuvo `CommandError`, `error_code=5/0x05`, detuvo RX y obtuvo STATS.

**Ruta exacta.** `start_rx()` hace que `ControlTask_onRxStart` establezca `s_rx_enabled=true`. Al llegar `CMD_TX_RAW`, `ControlTask_onTxRaw` llama `ControlTask_canStartTx`, que exige `!s_rx_enabled`; retorna falso y `command_processor.c` envía `ERR_INVALID_STATE`. La solicitud se rechaza antes de copiar/encolar el payload para `RadioIF_transmitRaw`.

**Qué prueba.** Regla lógica de exclusión y recuperación del protocolo/estado. En esta ruta concreta, el código respalda que no se solicitó TX al backend RF. No prueba simultaneidad real de RX/TX ni que el receptor RF hubiera terminado de abrir: basta el flag lógico activado por RX_START.

**Corrección procedimental.** La corrección de `e.code` a `e.error_code` fue necesaria y está bien documentada. Los intentos inválidos no deben contarse como repeticiones del DUT.

**Veredicto: PASS-control/state-rejection.** El informe final `PASS` es válido si se conserva este alcance; sería mejor nombrarlo explícitamente `PASS-control`.

## 3.8 EV-10 — Matriz de PHY por control

**Procedimiento.** Ocho invocaciones literales de `smoke_phase2.py` para PHY 0–7, potencia 0 dBm y canales 37, 9, 37, 37, 25, 0, 0, 0. Todas muestran INFO, ACK de SET_PHY/CHANNEL/POWER, ACK de RX_START/STOP y `SMOKE TEST PASS`. Entre filas se registró `boot`/`exit` por COM87 y verificación INFO/STATS; hubo además reset final.

**Qué prueba.** La API, enum, protocolo y handlers aceptan las ocho configuraciones y la sesión responde después. No prueba frecuencia, potencia, modulación, apertura efectiva duradera, recepción ni transmisión RF. En particular, PHY 7 sin `configure_prop` no representa un preset concreto.

**Matiz de procedencia.** El script es upstream directo, pero la matriz exacta de la guía no es idéntica al baseline actual: `run_validation_baseline.sh` usa canal 37 para PHY 1, mientras esta campaña usó 9. Canal 9 es plausible para BLE 2M raw y aparece en el antecedente de Extended ADV, pero no debe describirse como copia exacta del baseline de control.

**Reset.** El reset entre cada step reproduce el workaround del baseline y `VALIDATION_MATRIX.md` §10, donde el cambio sin reset tuvo `RF_close deadlock`. No es una necesidad demostrada por EV-10 ni una propiedad universal del chip; es un workaround conservador derivado de fallos históricos. Los resets manuales fueron más numerosos que lo estrictamente necesario para demostrar ACK, pero coherentes con el plan de aislamiento.

**Veredicto: PASS-control 8/8.** El informe limita correctamente el alcance y no asigna PASS-RF.

## 3.9 EV-11 — Presets propietarios, sólo control

### Conteo independiente

`FeralRF/python/feralrf/presets.py@0178721` contiene 31 entradas. Las exclusiones fueron exactamente:

```text
ook_433_4k8
ook_433_2k4
ook_868_4k8
mioty_868_tsunb
```

Quedan 27 objetivos: 6 de 433 MHz, 8 de 868 MHz, 2 de 169 MHz, 9 de 902/915 MHz y 2 de 2.44 GHz. El conteo de la conclusión es correcto.

### Duplicados y evidencia preservada

`gfsk_868_50k` fue ejecutado dos veces y correctamente no se contó como un preset adicional. Los tres informes EV-11 preservan salida individual completa para:

- 6/6 de 433 MHz;
- 8/8 de 868 MHz, más el duplicado;
- 2/2 de 169 MHz;
- 2/2 de 2.44 GHz.

Para los nueve presets 902/915 MHz sólo se conserva una tabla y un patrón de salida agregado común. No aparecen los nueve comandos ni sus nueve stdout individuales. Por tanto, “se ejecutaron nueve y todos pasaron” es un **registro narrativo del procedimiento**, no evidencia terminal individual auditable desde el Vault. No hay base para afirmar que no se ejecutaron, pero tampoco puede reconstruirse cada corrida sólo desde el material conservado.

### Qué hace realmente el smoke

`smoke_prop_phase1.py` ejecuta INIT, PHY 7, `SET_PROP_CONFIG`, SET_POWER, RX_START, lee tres segundos con `min_packets=0`, RX_STOP, TX_RAW y GET_STATS.

Hay tres límites de observabilidad importantes:

1. `CMD_SET_PROP_CONFIG` llama a la función `void RadioIF_setPropConfig(...)` y envía ACK sin recibir un resultado de éxito/fallo del backend. “Backend aceptado” es una formulación demasiado fuerte; lo demostrado es que el handler aceptó el payload y llamó a la configuración sin error síncrono visible.
2. RX_START recibe ACK antes de que `DataTask_poll` ejecute `RadioIF_startRx`. El script excluye de la lista todo `RxStreamError`, de modo que un error RF asíncrono durante la ventana no queda impreso ni hace fallar por sí solo la prueba.
3. TX_RAW recibe ACK al encolarse. `ControlTask_processTxRaw` ejecuta después `RadioIF_transmitRaw`; si falla, emite un error asíncrono. El script no espera TX_DONE ni consume explícitamente ese evento después del ACK.

Por ello `PROP PRESET SMOKE PASS` prueba que la cadena de comandos síncronos terminó y que el dispositivo siguió contestando, pero no prueba que RX o TX RF se hayan completado físicamente.

### Valor de STATS

Los ceros `rx_crc_err/rx_drop/rx_overflow` son datos reales devueltos por GET_STATS, pero tienen valor probatorio bajo aquí: `RADIO_INIT` reinicia métricas, las ventanas duraron tres segundos, `min_packets=0` y se observaron cero paquetes. Muestran ausencia de contadores de error durante una ventana sin tráfico recibido; no caracterizan capacidad de cola, robustez ni calidad RF. La frase “no se registró overflow” es literalmente cierta; usarla como argumento fuerte de estabilidad sería engañoso.

### Reset entre bandas

Hay evidencia literal de envío `boot/exit` para 433→868, 169→902/915 y 902/915→2.4 GHz. El propio cierre reconoce que no aparece la salida de 868→169. Esto es **evidencia faltante**, no evidencia de que el reset no ocurrió.

Además, en algunos cambios sólo se imprimió `OPEN/SENT/CLOSED`; eso confirma escritura host, no por sí solo reset/recuperación. El éxito del primer preset posterior aporta evidencia indirecta de que el DUT estaba funcional. La transición 433→868 no conserva un INFO/STATS inmediatamente intermedio, aunque las pruebas 868 posteriores sí respondieron.

El reset entre bandas proviene de `VALIDATION_MATRIX.md` §10 y `PYTHON_API.md`, que documentan deadlock/inconsistencia histórica sin reset. Es un workaround conservador. No hay en `smoke_prop_phase1.py` una obligación intrínseca de reset entre todos los presets de la misma banda.

### 433 MHz, nombres de protocolo y exclusiones

`PASS-control 6/6` en 433 MHz no contradice el historial OTA: GFSK 6–10/10, FSK/MSK 1/10 y OOK 0/10. EV-23 sigue siendo la caracterización RF pendiente.

Los nombres Wireless M-Bus, Wi-SUN y Sidewalk seleccionan parámetros PHY. EV-11 no ejecutó framing, MAC, asociación, seguridad ni interoperabilidad de esos protocolos. “W-MBus N preset control validado” es correcto; “Wireless M-Bus N validado” sin calificador no lo sería.

Excluir OOK fue correcto: `Radio.configure_prop`, `presets.py` y `RadioIF_setPropConfig` advierten que cargar los patches OOK deja el radio bloqueado hasta reset/power-cycle. Debe revisitarse en **EV-24**. Excluir MIOTY también fue correcto: el preset se conserva como API pero está marcado pending; commit `6e34ef2` y la matriz histórica registran 0/10 por falta de soporte TS-UNB/CPE. Debe revisitarse en **EV-52**, no contarse como un cuarto fallo de EV-11.

### Veredicto

**PASS-control reportado 27/27, con nivel de evidencia desigual.** Hay evidencia literal sólida para 18 presets y una afirmación agregada para nueve. No hay PASS-RF. La conclusión defendible es: “los 27 objetivos fueron registrados como terminados por el script de control; el Vault permite auditar individualmente 18 y sólo resumidamente los nueve de 902/915”.

## 3.10 EV-12 — Estado de preparación

La guía identifica y describe los cuatro scripts TX y sus riesgos. En los informes no existe comando ejecutado, stdout, observación RF ni resultado de EV-12. La inspección de `smoke_tx_phase1.py` o del código no constituye ejecución.

**Veredicto: PREPARED / NOT TESTED.** No existe PASS parcial. No se continuó EV-12 durante esta auditoría.

## 4. Hallazgos transversales

### 4.1 ACK no equivale a RF

La campaña respetó conceptualmente esta distinción en EV-10 y EV-11, pero algunos verbos de los informes (“backend aceptado”, “TX completado”) deben acotarse. En el firmware actual:

- SET_PROP_CONFIG ACK no recibe confirmación del backend;
- RX_START ACK precede a `RadioIF_startRx`;
- TX_RAW ACK precede a `RadioIF_transmitRaw` y no es TX_DONE;
- el fallo posterior puede viajar como error asíncrono.

Sólo EV-05 contiene evidencia RF propia actual. EV-11 incluye TX breve real solicitado, pero sin observador ni TX_DONE sigue siendo **RF no establecida**.

### 4.2 Preset no equivale a protocolo

Las conclusiones finales de EV-11 suelen conservar esta distinción. Debe mantenerse en cualquier síntesis: nombres Wi-SUN, Sidewalk, W-MBus o MIOTY describen parámetros/intención, no implementación de sus stacks. El repositorio incluso califica Sidewalk como sólo capa FSK y excluye Sidewalk LR del CC1352.

### 4.3 Reset y KI-15

La decisión de bloquear la API fue correcta y evitó abrir COM90. El procedimiento manual es coherente con el firmware oficial y produjo una transición observable. La evidencia no autoriza a decir que `Radio.reset_device()` funciona en esta enumeración; precisamente se demostró lo contrario respecto a selección de puerto.

Los resets posteriores deben registrarse en dos niveles: “bytes enviados al Shell” y “recuperación comprobada por INFO/STATS o siguiente prueba”. Mezclarlos oculta la diferencia.

### 4.4 Switching PHY/banda

La fuente histórica declara fallo sin reset y éxito con reset. El código actual contiene gestión de handles, frecuencia y configuraciones separadas, pero eso no demuestra que el fallo histórico persista en HEAD. En esta campaña los resets fueron un aislamiento conservador y coherente con el baseline, no una nueva reproducción de EV-41 ni una prueba de necesidad causal.

### 4.5 RF marginal histórica

No se midió 433 MHz por aire en la campaña. EV-11 no puede confirmar ni refutar marginalidad, link budget o limitación de antena. Esas afirmaciones permanecen históricas hasta EV-23/instrumentación.

### 4.6 Versiones e identidad

`GET_INFO 1.0.0`, paquete Python `0.3.0`, documentación con otros números y Catnip `3.3.3.0` pertenecen a componentes/capas diferentes. Ninguno identifica por sí solo el hash del HEX ejecutado. La campaña conoce el commit fuente bajo evaluación, pero **no está establecido a partir de la evidencia disponible que el binario físico sea reproduciblemente trazable a ese commit mediante hash/build manifest**.

## 5. Auditoría de los informes de evaluación

### 5.1 Conservar literalmente

- comandos realmente copiados de terminal y directorio de trabajo;
- mapa `COM88/COM86/COM87`, ruta y versión del Catnip instalado;
- stdout/stderr que discrimina fallo de harness de fallo del DUT;
- `DeviceInfo`, códigos de error y excepciones relevantes;
- conteos RX, canal, duración, RSSI, LQI, CRC, longitud y, en el futuro, bytes/PCAP;
- comandos `boot/exit`, puerto explícito y verificación posterior;
- una salida completa por clase de comportamiento y toda salida inesperada;
- lista exacta de presets, parámetros que cambian y duplicados;
- hashes de fuente/binario y estado Git cuando existan.

Estos elementos permiten reproducir condiciones y revisar interpretaciones sin confiar en la narrativa.

### 5.2 Conservar, pero resumir con referencia al log crudo

- ocho salidas idénticas de EV-10: tabla por PHY con comando, retorno y anomalía basta, conservando el log original;
- secuencias repetidas de EV-11: tabla por preset con frecuencia/modulación/rate, resultado y enlace al bloque crudo;
- `OPEN/SENT/CLOSED` repetidos: conservar una plantilla y una tabla de transiciones, salvo errores o tiempos distintos;
- largas explicaciones repetidas de la misma cadena PC→RP2040→CC1352;
- la reiteración de que `packets=0` es permitido y ACK≠RF;
- análisis paso a paso de cada línea que no cambia el veredicto.

El resumen no debe fingir que existe stdout individual cuando sólo existe una afirmación agregada, como ocurre en 902/915 MHz.

### 5.3 Omitir de un futuro informe consolidado

- recomendaciones conversacionales del “siguiente candidato” al final de cada log;
- teoría repetida que ya pertenece a la guía o a una sección metodológica única;
- conclusiones duplicadas en cinco formatos dentro del mismo informe;
- diagramas repetidos sin nueva información;
- especulación sobre el tipo de frame Zigbee sin bytes/decodificación;
- inferencias causales formuladas como hechos (“backend configurado correctamente”, “reset eléctrico demostrado”) cuando sólo hubo ACK o escritura serial;
- errores de quoting/import una vez resumidos como incidencias de harness, salvo que expliquen por qué una corrida no cuenta;
- listas reiteradas de lo que no se probó si pueden centralizarse en una tabla de alcance.

No se recomienda borrar los logs actuales: su verbosidad preserva contexto de ejecución. La recomendación aplica a un futuro reporte consolidado, que debe enlazar a esos originales.

### 5.4 Inexactitudes o sobreafirmaciones concretas

1. EV-05 dice que el procedimiento registró paquetes conforme a la guía; en realidad sólo se guardó conteo y primera metadata, no bytes/metadatos de todos.
2. EV-11 usa “backend aceptado”; el handler ACK no recibe retorno de `RadioIF_setPropConfig`.
3. EV-11 afirma ausencia de error asíncrono visible, pero el script filtra `RxStreamError` durante RX y no espera conclusión RF de TX.
4. Algunos resets posteriores se llaman “ejecutados correctamente” sólo por `OPEN/SENT/CLOSED`; eso prueba envío, no recuperación completa.
5. La ruta GPIO15 de reset en la guía/FeralRF PINOUT contradice el overlay ejecutable RP2040 actual (GPIO3 para reset, GPIO2 para boot).
6. EV-10 se presenta como la matriz prescrita por el baseline, pero PHY 1/canal 9 no coincide con `run_validation_baseline.sh` actual, que usa canal 37.
7. La matriz local y el estado del proyecto aún marcan las EV como no ejecutadas; son estados desactualizados, no evidencia negativa.

No se encontró un caso donde un comando reconstruido se presentara inequívocamente como copia literal contra evidencia contraria. Sí existen descripciones sin transcripción —control negativo EV-05 y nueve presets 902/915— que deben rotularse como tales, como en general ya hacen los informes.

## 6. Asuntos abiertos / pruebas aún requeridas

Sin rediseñar el roadmap existente:

- **EV-00:** completar identidad debug/status/HWID/location y vincular físicamente DUT con el ID temporal.
- **EV-01:** si se desea cierre formal, conservar la serie dedicada 3/3 con INFO, STATS y tiempos.
- **EV-04:** permanece bloqueada para API hasta resolver o parametrizar KI-15; la repetición manual no sustituye el criterio de la API.
- **EV-05/EV-20/EV-29:** captura independiente simultánea y bytes/PCAP reforzarían atribución, FCS y decodificación, sin convertir FeralRF en stack Zigbee.
- **EV-11/EV-22/EV-23/EV-25/EV-26/EV-27:** falta RF/OTA/instrumentación de presets; 433 sigue abierto.
- **EV-24:** OOK diferido, con recuperación segura disponible.
- **EV-41/EV-42:** determinar si switching sin reset sigue fallando en el binario actual; EV-10/11 no contestan eso.
- **EV-52:** MIOTY continúa como limitación histórica/pending.
- **EV-12:** sigue no ejecutada.
- **EV-40:** reproducir el baseline histórico contra HEAD con paquete de evidencia y hash de firmware.

## 7. Conclusión técnica general

Los procedimientos ejecutados fueron, en su mayoría, técnicamente sensatos y coherentes con la guía. Las mejores evidencias son EV-03, EV-05, EV-06 y EV-10: conservan comandos, salida y un criterio claro. EV-04 manejó correctamente el riesgo al no abrir COM90 y reprodujo KI-15. EV-11 aplicó bien las exclusiones y la separación control/RF, pero su veredicto debe leerse con una semántica más estrecha de la que sugieren algunas frases: finalización del smoke síncrono, no confirmación del backend físico.

Las conclusiones no deben agregarse en un único “FeralRF funciona”. Lo actualmente defendible es:

- la comunicación host y los comandos básicos responden repetidamente;
- la máquina de estados rechaza TX durante RX con `0x05` y se recupera;
- ocho IDs PHY y 27 presets fueron recorridos a nivel de control, con evidencia individual incompleta para nueve presets;
- el DUT recibió físicamente frames IEEE 802.15.4 en canal 25 de forma repetible;
- la API de reset no es segura con el mapa COM observado;
- no se han validado físicamente los TX, la exactitud de frecuencia/potencia/modulación, la mayoría de PHY, los stacks nombrados por presets ni el baseline OTA actual.

La procedencia del plan es mayoritariamente sólida, pero híbrida: combina scripts y código upstream con criterios experimentales diseñados localmente. Eso es válido, siempre que las repeticiones y umbrales locales no se presenten como requisitos originales del proyecto.

## 8. Síntesis para asesor académico

La campaña ha pasado de una revisión estática a evidencia física limitada pero significativa. El resultado más fuerte es la recepción IEEE 802.15.4: en tres ejecuciones formales de 30 segundos sobre canal 25, el CatSniffer V3 con FeralRF entregó 41, 43 y 43 paquetes, con al menos la primera trama de cada ventana marcada como CRC válida y con RSSI/LQI plausibles. Otras dos ventanas produjeron 48 y 100 paquetes. Esto demuestra que la cadena antena/radio CC1352P7→firmware→UART→RP2040→USB→Python produjo datos RF reales y repetibles. No demuestra un stack Zigbee: FeralRF recibió frames raw. La atribución al dispositivo Zigbee conocido es razonable, pero falta correlación simultánea con un segundo receptor y no se conservaron los bytes de todos los paquetes.

También hay evidencia robusta del plano de control. Cinco procesos independientes inicializaron y cerraron el dispositivo sin dejar el puerto ocupado. Ocho valores de PHY recorrieron configuración y start/stop. Una prueba negativa demostró que TX solicitado mientras el estado lógico RX está activo es rechazado por el firmware con `ERR_INVALID_STATE=0x05`, y que STOP/STATS siguen funcionando después. Este último resultado puede trazarse exactamente desde la excepción Python hasta `ControlTask_canStartTx`, que impide encolar la transmisión.

EV-11 amplió la cobertura de configuración propietaria. El código contiene 31 presets; se excluyeron correctamente tres OOK por su lock conocido y MIOTY por soporte nativo pendiente, dejando 27. Los informes registran los 27 como PASS-control y no les asignan PASS-RF. La aritmética y las exclusiones son correctas, y sólo hubo un duplicado (`gfsk_868_50k`), no contado dos veces. Sin embargo, la evidencia no tiene la misma calidad para todos: hay stdout individual para 18 presets, mientras los nueve de 902/915 MHz aparecen sólo en un resumen. Además, el script recibe ACK antes de que RX/TX RF necesariamente terminen y filtra ciertos errores asíncronos. Por eso EV-11 acredita que el flujo síncrono de control terminó y el equipo siguió respondiendo; no acredita ondas, frecuencia, modulación ni interoperabilidad Wi-SUN, Sidewalk o Wireless M-Bus.

El problema concreto ya reproducido es KI-15. Catnip agrupó la placa como V3 con Bridge COM88, LoRa COM86 y Shell COM87. FeralRF calcula el Shell como Bridge+2 y elegiría COM90. La campaña hizo lo correcto: no llamó la API pública contra el puerto equivocado. Al enviar manualmente `boot` a COM87, FeralRF dejó de responder por COM88; tras `exit`, INIT y STATS regresaron sin desconectar USB. Eso apoya el mecanismo subyacente de recuperación una vez, pero la API permanece bloqueada. También surgió una contradicción documental: FeralRF PINOUT/guía hablan de GPIO15, mientras el target RP2040 oficial actual usa el alias de reset en GPIO3 y boot en GPIO2.

Los problemas históricos de RF continúan abiertos. En particular, un PASS-control a 433 MHz no elimina el antecedente de OTA marginal ni el fallo OOK 433. Los resets entre bandas usados en EV-10/11 fueron un workaround prudente derivado del baseline histórico; no demuestran que HEAD aún necesite reset ni caracterizan el fallo sin reset. OOK debe tratarse en EV-24, 433 en EV-23 y MIOTY en EV-52. EV-12 no fue ejecutada: estudiar sus scripts no equivale a validar TX.

La confianza apropiada en el baseline actual es moderada para transporte, sesión, rechazo de estado y recepción IEEE 802.15.4 en la condición probada; baja o nula para transmisión física, exactitud espectral, otras PHY por aire y protocolos completos. La metodología es creíble porque conserva comandos y stdout, separa ACK de RF, usa una fuente independiente y documenta fallos del harness. Debe fortalecerse con hashes del binario, captura raw/PCAP, observadores independientes, logs individuales completos y consumo explícito de errores RF asíncronos. Hasta entonces, la formulación académicamente defendible es “baseline de control ampliamente recorrido y una ruta RX IEEE validada físicamente”, no “FeralRF validado en todas sus capacidades”.

## Questions / evidence gaps that should be resolved before continuing validation

1. ¿Qué hash exacto del `.hex` está flasheado en DUT-V3 y qué build reproducible lo vincula con `FeralRF@0178721`?
2. ¿Puede recuperarse la salida individual original de los nueve presets 902/915 MHz y la evidencia del reset 868→169, o debe conservarse explícitamente como pérdida de evidencia?
3. ¿Qué canal, duración, comando y stdout correspondieron al control negativo informal de EV-05?
4. ¿Puede una captura simultánea independiente correlacionar bytes/tiempos de canal 25 con el DUT para cerrar la atribución al `PROTOCOL-DEVICE-ZIGBEE-CH25`?
5. ¿Qué revisión/esquema explica la discrepancia GPIO15 frente al overlay RP2040 GPIO3 para `RESET_CC`?
6. ¿Existe hash o manifest del `C:\Program Files\Catnip\catnip.exe` que lo vincule con un commit concreto, además del encabezado `v3.3.3.0`?
