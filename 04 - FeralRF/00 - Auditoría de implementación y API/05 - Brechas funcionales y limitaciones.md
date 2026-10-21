# Brechas funcionales y limitaciones

## Registro priorizado de hallazgos

La severidad indica impacto técnico potencial, no un plan de implementación. “Confirmado” se refiere a la instantánea fuente; la aplicabilidad al binario histórico está limitada por provenance.

| ID | Severidad | Capa | Hallazgo | Certeza | Evidencia principal |
|---|---|---|---|---|---|
| F-01 | Alta | CC/RX prop | Orden incorrecto de status/RSSI/timestamp; afecta CRC, RSSI, timestamp y filtrado | Fuente + experimento | `radio_if.c:L1631-L1642,L1706-L1754`; RF-02 |
| F-02 | Alta | contrato async | RX con secuencia 0…254 se descarta durante espera síncrona | Confirmado por fuente; síntoma observado | `data_task.c`; `radio.py:L278-L397`; RF-01/RF-03 |
| F-03 | Alta | config prop | `mod_type=4/5/6` se expone como modos válidos aunque SDK los reserva | Confirmado por fuente | `radio.py:L502-L509`; `presets.py`; `rf_prop_cmd.h` |
| F-04 | Alta | TX/API | burst/continuous carecen de completion/progreso y silencian fallo posterior | Confirmado por fuente | `control_task.c:L264-L407`; `radio.py:L752-L810` |
| F-05 | Alta | identidad | firmware/serial constantes no identifican binario/dispositivo | Fuente + limitación experimental | `control_task.c:L443-L454`; Fase I provenance |
| F-06 | Media-alta | RP/Python | descubrimiento Cat-Shell por puerto derivado puede controlar interfaz incorrecta | Fuente + experimento | `radio.py:L184-L246`; COM90/COM87 |
| F-07 | Media-alta | config/API | `SET_PROP_CONFIG` ACK sin validación semántica ni resultado RF | Confirmado por fuente | `command_processor.c`; `RadioIF_setPropConfig` |
| F-08 | Media-alta | colas | drops de output/transporte no son visibles en `GET_STATS` | Confirmado por fuente | `output_if.c`, `packet_queue.c`, `main.c:L202-L265` |
| F-09 | Media | crypto | ECDSA Python admite Curve25519, firmware sólo P-256 | Confirmado por fuente | `radio.py` ECDSA; `crypto_engine.c` |
| F-10 | Media | protocolo | Python no aplica máximo global de 255; CCM/GCM permiten payload imposible | Confirmado por fuente | `protocol.py`, métodos AES; `PROTOCOL_MAX_PAYLOAD` |
| F-11 | Media | init | `RADIO_INIT` no vuelve a ejecutar `RadioIF_init` | Confirmado por fuente | único caller en `data_task.c:L73`; handler/control |
| F-12 | Media | parsing/API | builders truncan/enmascaran entradas y RX no valida exhaustivamente longitud | Confirmado por fuente | `commands.py`; `read_packets()` |
| F-13 | Media | KillerBee | extra de dependencia/adapter no constituye integración instalable completa | Fuente + experimento | `pyproject.toml`, patch y RF-04 |
| F-14 | Media | concurrencia | `PacketQueue` cruza tareas sin sincronización visible | Sospechado por fuente | `host_if_task.c`, `data_task.c`, `packet_queue.c` |
| F-15 | Media | estado TX | `canStartTx()` no incluye jam activo aunque `isTxBusy()` sí | Sospechado por fuente | `control_task.c:L84-L96,L432-L437` |
| F-16 | Baja-media | API/docs | clasificación stable/experimental/pending inconsistente | Confirmado por fuente/docs | `radio.py:L89-L130`, `docs/PYTHON_API.md` |
| F-17 | Baja-media | potencia | potencia solicitada puede aproximarse a otra entrada de tabla sin reportar el valor efectivo | Confirmado por fuente | `radio_if.c:L209-L317` |
| F-18 | Baja | metadata | LQI propietario siempre cero; `channel` es etiqueta residual | Fuente + experimento | `radio_if.c:L1732-L1736`; RF-02 |

## Defectos confirmados

### F-01 — layout propietario

No es sólo “RSSI poco plausible”. El parser viola el layout seleccionado por sus propios flags del comando TI. La consecuencia alcanza cuatro campos y el filtro: un paquete puede aceptarse/descartarse por un bit del RSSI real. La corrección futura deberá establecer también si `pdu_len` debe exponer o retirar header/CRC; esta auditoría no la implementa.

### F-02 — correlación asíncrona

Firmware trata RX como stream con secuencia propia; Python trata principalmente secuencia cero como async. Un método síncrono concurrente puede consumir y descartar paquetes. `stop_rx()` reconoce el tipo pero no los preserva. La pérdida es determinista a nivel de política host, aunque no es causa suficiente del ACK ausente.

### F-03 — enum de modulación

La documentación y presets asignan significado a valores que el SDK 8.30.01.01 marca reservados. Esto invalida afirmar modulación física por nombre/preset. Se necesita definir una representación soportada por el comando RF concreto, no sólo renombrar números.

### F-04 — resultado TX diferido

La API devuelve después de ACK y no tiene objeto/ID de operación. Raw puede informar un error genérico asíncrono; burst/continuous no. Esta violación afecta la capacidad de evaluar count, stop y fallos, y es coherente con los límites de Fase I.

### F-06 — frontera RP2040 no abstraída

La misma API que parece resetear “el dispositivo” infiere otra interfaz por numeración y no valida respuestas. La operación depende de enumeración de SO y firmware stock. Band selection ni siquiera está expuesta; los scripts deben abrir Cat-Shell por fuera.

### F-09/F-10/F-12 — validación host

La API realiza validación selectiva: tamaños de claves sí, pero curve/operación, máximo de frame, rangos PHY y longitudes TX no de forma uniforme. Enmascarar a `uint8` convierte un error de caller en otro valor válido. Estas condiciones deberían fallar antes del I/O, pero hoy pueden provocar error tardío, truncamiento o frame rechazado.

## Riesgos y defectos sospechados

### Cola compartida sin protección

`PacketQueue` mantiene `head`, `tail`, `count` y buffer ordinarios. La tarea RF llama push y la UART pop; no se observan primitives TI-RTOS, critical sections ni atomics. En C concurrente esto es un riesgo de data race/estado incoherente. No se ejecutó instrumentación y no se atribuye el timeout RX_STOP a este patrón.

### TX durante jam

`ControlTask_canStartTx()` excluye RX y operaciones raw/burst/continuous, pero no `s_jam_active`; en cambio `ControlTask_isTxBusy()` sí incluye jam. Algunos handlers usan una u otra condición. Es plausible iniciar TX regular con sesión jam activa y competir por RF. Debe trazarse/validarse antes de declararlo fallo reproducible.

### Transporte RP2040

El camino USB→UART no comprueba fallo de inserción al ring buffer. Esto puede perder comandos bajo presión, pero no hay contadores expuestos ni traza asociada a RF-01/RF-03. Es una hipótesis de transporte.

## Implementaciones parciales

| Área | Parte presente | Parte ausente/incompleta |
|---|---|---|
| Device lifecycle | discover/connect/init/reset/disconnect | identidad firme, estado, recovery, mapping shell seguro |
| RX | varios PHY, buffers, metadata, stats | stream multiplexado seguro, metadata prop válida, métricas end-to-end |
| TX | raw/frame/burst/continuous/stop/test | completion, counts, resultado efectivo, cancelación observable |
| Config propietario | frecuencia y parámetros TI | validación, readback, resultado del driver, enums correctos |
| KillerBee | clase adapter | instalación reproducible, propagación de errores, evidencia OTA |
| Emulación | tramas preparadas y loops | state machines/protocolos/interoperabilidad |
| Jamming | continuo BLE/IEEE | reactivo/pattern, política ejecutable, evidencia |
| OOK | modo especial | validación experimental y recovery robusto |
| BLE | PHY/raw/LL classification | pila retirada deliberadamente |

## Capacidades ausentes o no justificadas como presentes

- Spectrum scan/stop: tipos y builders no conectados no son implementación.
- Promiscuous/filter configuration: no existe contrato activo.
- Jamming reactivo/pattern: marcado pending, sin handler.
- Pilas Zigbee, Thread, Wi-SUN, Sidewalk, Wireless M-Bus y MIOTY: no aparecen state machines/capas MAC-red-seguridad-aplicación. Hay PHY/raw/presets/emulación.
- Verificación física de banda, frecuencia, potencia y radiación: no puede producirla un ACK; requiere medición/peer adecuado.

La ausencia de una pila completa sólo es brecha de requisito si el objetivo formal exige esa pila. Si FeralRF se define como herramienta raw/multiprotocolo PHY, es una frontera intencional que la documentación debe expresar.

## Deuda técnica y remanentes

- `tx_queue.c` no participa en la ruta TX actual.
- builders spectrum/promiscuous y clases de responses duplican conceptos sin callers.
- rutinas BLE internas permanecen en `radio_if.c` después de retirar la API/stack.
- comentarios de `RadioIF_init` sugieren relación con `CMD_RADIO_INIT` que el call graph actual no cumple.
- documentación de RF-open lazy contradice el handle 433 abierto al boot.
- rangos de potencia documentados difieren entre archivos y no reflejan resolución por tabla/PHY.

Estos elementos elevan el costo de auditoría y el riesgo de revivir interfaces no soportadas, aunque no todos son defectos de ejecución.

## Brechas de validación

1. No existe trazabilidad binario→commit para CC ni hash UF2 preservado.
2. No hay captura simultánea host/UART/queue/RF para RX_STOP.
3. No hay prueba contra layout conocido para metadata propietaria.
4. No hay analizador/receptor que certifique `mod_type`, frecuencia y potencia efectivas.
5. No hay telemetría emitida para burst/continuous.
6. Criptografía no fue contrastada con vectores conocidos en Fase I.
7. OOK, jamming y KillerBee no tienen evidencia hardware suficiente.
8. La tabla del RF switch no fue verificada contra esquema/revisión y medición física.

## Atribución por capa

- **Python:** validación, correlación async, port discovery, modelado de lifecycle y resultados.
- **Wire protocol:** identidad/capabilities granulares, operación IDs, completions y telemetría faltantes.
- **CC1352P7:** parser prop, validación RF, silencios de scheduler, métricas parciales y riesgos de estado/cola.
- **RP2040:** reset/band fuera del protocolo, buffers y counters no expuestos.
- **Hardware:** ruta de front-end, potencia/radiación sólo verificables físicamente.
- **Integraciones:** packaging/patch KillerBee y reducción de metadata/errors.

