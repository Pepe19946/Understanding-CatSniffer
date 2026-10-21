# Brechas verificadas y limitaciones de implementación

## Revalidación de F-01 a F-18

La tabla conserva los identificadores de [[../00 - Auditoría de implementación y API/05 - Brechas funcionales y limitaciones|Fase II-A]]. “Consecuencia confirmada” se limita a lo deducible del código o demostrado; no convierte una posibilidad en causa de un experimento.

| ID | Contrato y condición | Evidencia exacta | Consecuencia confirmada / hipotética | Relación experimental | Dictamen II-B |
|---|---|---|---|---|---|
| F-01 | RX PROP con metadatos anexos activos | `radio_if.c`, parser PROP frente a opciones de `smartrf_prop_0.c`/SDK | Conf.: status, RSSI, timestamp, CRC filtering y LQI se leen con layout incompatible. Hip.: magnitud exacta de pérdidas | RSSI anómalo/ausente y recepción PROP histórica son coherentes, no miden todos los campos | **Confirmado, alta** |
| F-02 | Evento RX con secuencia 0…254 mientras el host espera una respuesta síncrona | `data_task.c`; correlador/event loop de `radio.py` | Conf.: el lector síncrono puede consumir/descartar RX no reconocido como async. Hip.: proporción y contribución a cada timeout | Síntomas de stream/stop son compatibles; no explica por sí solo un ACK nunca emitido | **Confirmado; causalidad experimental parcial** |
| F-03 | `configure_prop` con 4/5/6 | `presets.py`; `radio.py`; `rf_prop_cmd.h` 8.30.01.01 (`0/1/2`, resto reservado) | Conf.: el nombre no selecciona MSK/4-FSK/4-GFSK con el comando actual. Hip.: señal concreta producida por valor reservado | La aceptación/10 paquetes, si existiera, no certifica modulación | **Confirmado inequívocamente para el SDK actual** |
| F-04 | TX burst/continuous diferido, o fallo después del ACK | `control_task.c`; `radio.py` | Conf.: no hay completion/progreso correlacionado; host no distingue fin/fallo. Hip.: pérdida RF concreta | Explica la imposibilidad de probar garantía de conteo/fin, no cada fallo OTA | **Confirmado, alta** |
| F-05 | Identificación de binario/dispositivo | respuestas fijas en `control_task.c` | Conf.: versión/serial constantes no trazan artefacto ni unidad. | Limita atribución de campañas históricas | **Confirmado** |
| F-06 | Reset/descubrimiento automático Cat-Shell bajo enumeración no contigua/cambiante | lógica de puerto en `radio.py` | Conf.: deriva un puerto sin identidad fuerte. Hip.: cuál interfaz controló cada intento | COM90/COM87 evidencia el riesgo operativo | **Confirmado para el contrato; efecto por ejecución depende del SO** |
| F-07 | `SET_PROP_CONFIG` bien formado pero semánticamente inválido/no soportado | handler `command_processor.c` y setter `RadioIF_setPropConfig()` | Conf.: ACK certifica recepción/almacenamiento, no éxito RF. | Aceptación de presets no prueba la PHY | **Confirmado** |
| F-08 | Saturación/fallo de cola de salida | `output_if.c`, `packet_queue.c`, stats expuestos | Conf.: drops no aparecen de extremo a extremo en `GET_STATS`. Hip.: número durante runs | Puede contribuir a pérdida observable; no hay contador para atribuirla | **Confirmado; magnitud desconocida** |
| F-09 | ECDSA solicitada con Curve25519 | validación en `radio.py`; ECDSA P-256 en `crypto_engine.c` | Conf.: contrato host acepta una combinación que firmware no implementa como ECDSA Curve25519 | Sin vector hardware consolidado | **Confirmado** |
| F-10 | Operación crypto que excede límite real, especialmente CCM/GCM | límites de `protocol.h`, API y buffers/response por operación | Conf.: validación no es uniforme y algunos payloads aceptados no caben en la ruta. Hip.: efecto exacto según tamaño | No probada en Fase I | **Confirmado con matiz:** no todas las operaciones carecen de límite |
| F-11 | `CMD_RADIO_INIT` después del arranque | único `RadioIF_init()` en inicialización de tarea frente al handler | Conf.: el comando no reinicializa íntegramente RadioIF. | No establece por sí solo causa de recuperación RF | **Confirmado** |
| F-12 | Valores fuera de rango/frames truncados o respuestas cortas | masks en `commands.py`; parseo `radio.py` | Conf.: algunos errores de caller se convierten en otro valor; validación RX no es exhaustiva. | Sin caso histórico aislado | **Confirmado** |
| F-13 | Instalación estándar de integración KillerBee | `pyproject.toml`, módulo/patch y evidencia RF-04 | Conf.: presencia de adapter/extra no entrega integración reproducible completa. | Intento histórico no cerró OTA reproducible | **Confirmado** |
| F-14 | Push/pop concurrentes sobre `PacketQueue` | `packet_queue.c` y callers de tareas distintas, sin sincronización visible | Conf.: existe riesgo de data race en el modelo C. Hip.: corrupción efectiva bajo TI-RTOS y relación con RX_STOP | No instrumentado | **Sospecha fundada; no elevar a defecto reproducido** |
| F-15 | Inicio de TX normal con jam activo | diferencia entre `canStartTx()` e `isTxBusy()` | Conf.: guardas no modelan jam de forma uniforme. Hip.: competencia RF real y resultado | Sin ejecución dirigida | **Sospecha fundada** |
| F-16 | Etiquetas stable/experimental/pending frente a realidad | `radio.py`, documentación y superficies residuales | Conf.: clasificación no sigue un criterio único de completitud/validación | Puede inducir inferencias sobre presets/funciones | **Confirmado como problema contractual/documental** |
| F-17 | Potencia solicitada sin entrada exacta | selección de tabla en `radio_if.c` | Conf.: puede usarse una entrada cercana y no se reporta. | ACK no prueba dBm radiados | **Confirmado** |
| F-18 | RX PROP y semántica de metadatos | asignaciones en `radio_if.c` | Conf.: LQI se entrega como cero y `channel` es etiqueta residual, no medición/config efectiva. | Observado parcialmente en RF-02 | **Confirmado** |

## Correcciones y ampliaciones II-B

| ID II-B | Tipo | Hallazgo | Certeza | Clasificación de requisito |
|---|---|---|---|---|
| B-01 | Corrección documental | II-A redujo `SET_PHY`; hoy soporta `<BHI>` con canal `u16` y frecuencia `u32`, además de payload heredado de 1 byte | Código confirmado | Corrección de auditoría, no defecto de producto |
| B-02 | Precisión de alcance | BLE TX recibe advertising data, no PDU LL arbitraria; RF core arma framing, y 2M crea advertising extendido con canal auxiliar fijo | Código + SDK | Documentar alcance actual |
| B-03 | Defecto de helper | El helper Wireless M-Bus usa `msk_868_50k` reservado aunque existe `wireless_mbus_t_868`; su docstring quedó obsoleto | Código confirmado | Corrección necesaria si se conserva la promesa de emulación T1 |
| B-04 | Incompletitud de helper | Bytes W-MBus fijos no implementan de forma verificable L/CI/CRC por bloques ni procedimientos | Código + OMS; EN completa con acceso limitado | Requerido sólo para afirmar trama/emulación W-MBus |
| B-05 | Contrato de catálogo | Nombres Wi-SUN/Sidewalk/W-MBus/MIOTY exceden la funcionalidad: son parámetros nominales sobre framing PROP | Código + fuentes oficiales | Documentación obligatoria para evitar sobreafirmación; pila superior es decisión de producto |
| B-06 | Límite de API | Preamble y formato CRC PROP no tienen API tipada; `format_conf` bruto puede alterar parte del formato sin validación | Código/SDK | Mejora de API si se requiere PHY arbitraria reproducible |
| B-07 | Incompletitud de emulación IEEE | La trama de datos solicita ACK, pero filtro/auto-ACK están desactivados y el helper no espera/reintenta | Código/SDK | Requerido sólo para rol MAC/emulador de dispositivo |
| B-08 | Brecha de validación | El catálogo creció a 31; la evidencia de Fase I no cubre automáticamente las entradas nuevas | Código + corpus Fase I | Validación, no defecto de implementación |
| B-09 | Prueba estática engañosa | `test_props.py` acepta 4/5/6 por el contrato Python y no comprueba el RF command ni hardware | Código | Corregir criterio si la prueba pretende soporte físico |
| B-10 | Alcance protocolario | Primitivas crypto genéricas no implementan seguridad de Wi-SUN, Sidewalk, BLE o W-MBus | Código + especificaciones | Capacidad superior opcional según rol elegido |

## Brechas por naturaleza

### Clasificación por tipo de mejora

| Tipo requerido por la auditoría | Hallazgos incluidos |
|---|---|
| Corrección de defecto | F-01, F-02, F-03, F-05, F-06, F-09, F-10, F-12, B-03 |
| Completar implementación prometida | F-04, F-08, F-13, B-04; F-11 si `RADIO_INIT` promete reinicialización completa |
| Mejora de API | F-07, F-17, F-18, B-06; semántica de completion/readback |
| Extensión de soporte protocolario | B-07 y cualquier MAC/pila W-MBus, Wi-SUN, Sidewalk, MIOTY, Zigbee/Thread o BLE conectada elegida explícitamente |
| Mejora de validación | F-14, F-15, B-08, B-09; validación física de OOK/potencia/modulación |
| Corrección documental | F-16, B-01, B-02, B-05 y el alcance de los emuladores |
| Mejora opcional | Spectrum, jamming reactivo/patrón, control tipado de framing y pilas no seleccionadas como requisito |

La asignación es funcional, no una prioridad. Un hallazgo condicionado puede cambiar de tipo cuando se defina su contrato; por ejemplo, spectrum es opcional mientras no se anuncie como función soportada.

### Defectos respecto del contrato ya expuesto

- F-01: metadatos PROP.
- F-02: política de eventos/secuencia asíncrona.
- F-03: valores reservados anunciados como modulaciones.
- F-04: insuficiente semántica de resultado para operaciones TX diferidas.
- F-05/F-06: identidad y asociación de interfaces para diagnóstico/automatización.
- F-07/F-09/F-10/F-12: validación/resultado incompatibles con la superficie pública.
- F-13: integración KillerBee no reproducible como paquete soportado.
- B-03/B-04: el helper se presenta como personalidad W-MBus sin realizar el rol.

### Implementación completa para un alcance limitado

- FSK/GFSK proprietary como transporte de paquetes TI simples, sujeto a metadatos/observabilidad.
- IEEE como PHY y transporte de bytes MAC con PHR/FCS del RF core.
- BLE advertising limitado con framing del RF core.
- CW/PRBS como modos de prueba.
- OOK como selección de modo disponible, aunque no validada experimentalmente.

### Funciones requeridas sólo para roles superiores

- MAC IEEE con ACK/CSMA/asociación/seguridad.
- Zigbee o Thread sobre IEEE.
- Wireless M-Bus conforme a modo/formato y rol.
- Wi-SUN FAN con hopping/red/seguridad.
- Sidewalk con PHY/framing/enlace/registro/seguridad.
- MIOTY con telegram splitting y scheduler TS-UNB.
- BLE de conexión/GAP/GATT/SMP.

Su ausencia no es un defecto si FeralRF se define formalmente como herramienta PHY/raw y los nombres/documentación no prometen esas funciones.

### Inconclusiones que deben preservarse

- La causa única de los timeouts `RX_STOP`.
- Si la cola sin sincronización se corrompe bajo el scheduler real.
- El resultado físico de valores de modulación reservados.
- Potencia, frecuencia, BW y desviación efectivamente radiadas para todas las combinaciones.
- Conformidad completa de los parámetros W-MBus con cada revisión/modo de EN 13757.
- Interoperabilidad de cualquier preset protocolario con implementación independiente.

## Resumen de riesgo de afirmaciones

| Afirmación | Estado defendible |
|---|---|
| “FeralRF transmite y recibe paquetes GFSK proprietary” | Sí, para configuraciones históricamente probadas y con salvedad de metadatos RX |
| “FeralRF transmite tramas IEEE 802.15.4” | Sí, como bytes MAC/PSDU y framing físico del RF core |
| “FeralRF implementa IEEE 802.15.4 MAC completo” | No |
| “FeralRF soporta BLE” | Sólo PHY/advertising limitado; la frase sin calificador es ambigua |
| “FeralRF soporta Wireless M-Bus/Wi-SUN/Sidewalk/MIOTY” | No como protocolo; sólo existen parámetros nominales o helpers incompletos |
| “MSK/4-FSK/4-GFSK están soportados por presets” | No con el comando/campo actual |
| “ACK de configuración prueba el modo RF” | No |
| “10/10 entre dos CatSniffer prueba interoperabilidad externa” | No |

## Relación con la Fase I

La evidencia histórica sigue siendo válida dentro de su procedencia. II-B no reinterpreta entrega raw como pila completa y no incorpora borradores manuales en curso. Para trazabilidad véanse [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/02 - Matriz histórica de experimentos y ejecuciones|matriz histórica]] y [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/04 - Evidencia, procedencia y preguntas abiertas|procedencia]].
