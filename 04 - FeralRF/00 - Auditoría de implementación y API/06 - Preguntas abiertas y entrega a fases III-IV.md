# Preguntas abiertas y entrega a fases III-IV

## Propósito

Este documento entrega preguntas y dependencias, no una hoja de ruta definitiva. Las prioridades de desarrollo deberán fijarse después de resolver requisitos y obtener la evidencia indicada. Ningún cambio se implementó en Fase II.

## Preguntas de requisitos

1. **¿FeralRF es una API de PHY/raw o debe ofrecer pilas interoperables?** La respuesta decide si Zigbee, Thread, W-MBus, Wi-SUN, Sidewalk y MIOTY son alcance faltante o nombres que deben limitarse documentalmente.
2. **¿Qué garantiza un TX exitoso?** ¿Aceptación, scheduling, finalización RF core, contador de intentos o confirmación OTA? El contrato actual sólo cubre los primeros hitos de forma desigual.
3. **¿Qué significa `channel` en PHY propietario?** Debe definirse como índice, frecuencia derivada, canal de preset o campo no aplicable.
4. **¿La API debe controlar Cat-Shell/banda?** Si sí, ¿sobre el protocolo textual actual o mediante una interfaz RP2040 versionada?
5. **¿Qué modalidades propietarias son requisito real?** El SDK actual no soporta los valores nominales 4/5/6 tal como están codificados.
6. **¿Se requiere jamming reactivo/pattern y spectrum?** Hoy son pending/stubs; faltan criterios de uso, seguridad y aceptación.
7. **¿Qué niveles de potencia están autorizados por variante/región?** La tabla técnica llega a high-PA, pero el contrato/documentación no modela restricciones de hardware o regulación.

## Preguntas técnicas por dependencia

### Identidad y reproducibilidad

- ¿Qué fuente generó el CC firmware `1.0.0` usado en cada run?
- ¿Qué hash corresponde al UF2 RP2040 instalado?
- ¿Cuál es la variante exacta de CatSniffer y qué esquema/población de front-end aplica?
- ¿Debe el build inyectar Git SHA, SDK revision, target y feature bitmap?

Sin estas respuestas no se puede declarar regresión entre HEAD y campañas históricas.

### RX_STOP y transporte

- ¿Se pierde el ACK antes o después de `OutputIF_sendAck()`?
- ¿Cuál era ocupación de las colas RadioIF/output/RP2040 al timeout?
- ¿`PacketQueue` presenta corrupción bajo el scheduler real o sólo riesgo estático?
- ¿el ring USB→UART rechazó bytes y con qué frecuencia?
- ¿qué política debe aplicar el host a RX ya recibido al pedir stop: entregar, drenar marcado o descartar contabilizado?

La evidencia mínima futura es una traza correlacionada con command/seq, timestamps y counters por etapa. Debe realizarse sólo en una fase autorizada de instrumentación.

### Configuración y metadata propietaria

- Confirmar en la revisión SDK/binario el layout exacto con header+CRC+RSSI+timestamp+status.
- Definir si la API debe incluir CRC/header en `Packet.data`.
- Determinar unidades y dominio válidos de rate/deviation/bandwidth/frequency y sus límites de cálculo.
- Establecer mapping soportado de modulación en vez de valores reservados.
- Definir readback: parámetros solicitados frente a parámetros efectivos/limitados.

Después de una futura corrección se necesitaría un generador/receptor conocido y captura de espectro; los registros actuales no bastan para certificar MSK/4-FSK/4-GFSK.

### TX diferido

- ¿Se requiere un `operation_id` y eventos `started/progress/completed/failed/stopped`?
- ¿Qué representa count: intentos solicitados, comandos RF aceptados o paquetes OTA?
- ¿cómo debe comportarse `TX_STOP` frente a una operación RF ya bloqueante?
- ¿qué contadores/status TI deben conservarse?
- ¿puede iniciarse TX durante jam activo o debe rechazarse?

La respuesta condiciona protocolo, máquina de estado y API; no debe resolverse sólo en Python.

### RP2040 y hardware

- ¿Cómo se vinculan Cat-Bridge y Cat-Shell de una misma unidad en Windows, Linux y macOS?
- ¿El shell puede reportar versión, modo, GPIO/banda y counters de buffer?
- ¿Cuál es la tabla verificada CTF1/2/3→ruta RF para cada revisión?
- ¿Por qué material de FeralRF menciona GPIO15 mientras el overlay usa GPIO3 para reset?
- ¿el hardware necesita secuencia/settling adicional entre switch, PHY y TX?

### Criptografía e integraciones

- ¿Curve25519 debe rechazarse en ECDSA host o existe intención de implementarla?
- ¿Cuál es el límite de datos esperado para AES/SHA: un frame o streaming/chunking?
- ¿Qué distribución exacta de KillerBee se soportará: fork, patch o upstream?
- ¿`inject()` recibe FCS siempre, nunca o mediante parámetro explícito?
- ¿los errores asíncronos deben aparecer como eventos generales además de RX stream?

## Evidencia necesaria antes de implementar

| Tema | Evidencia faltante | Decisión que habilita |
|---|---|---|
| Binarios | manifest/hash por firmware y SDK | comparabilidad y regresiones |
| RX_STOP | traza end-to-end y ocupación de colas | capa responsable y contrato de drain |
| Metadata prop | paquetes controlados + layout del driver | corrección de parser y modelo Packet |
| Modulación | medición/esclavo decodificador por modo | enum/presets soportados |
| TX burst/cont | counters internos + captura independiente | semántica de completion/count |
| RF switch | esquema + GPIO + medición por ruta | API de banda segura |
| Potencia | medición por PHY/banda/PA | rangos y readback efectivos |
| Crypto | vectores conocidos y límites | estabilidad de API |
| KillerBee | entorno reproducible + OTA | soporte oficial de integración |

## Dependencias entre hallazgos

- La trazabilidad de binario precede a cualquier afirmación de regresión.
- El contrato de eventos/secuencias precede a una solución robusta de RX_STOP y a KillerBee.
- La definición de metadata propietaria precede a presets y análisis RSSI confiables.
- La definición de “TX completado” precede al diseño de burst/continuous y sus métricas.
- La decisión PHY/raw versus stacks precede a clasificar protocolos como trabajo faltante.
- La identidad/mapping RP2040 precede a automatizar reset y band selection.
- Los requisitos regulatorios/hardware preceden a exponer high-PA de forma general.

## Áreas candidatas para fases futuras

Sin ordenar una implementación, las áreas justificadas por evidencia son:

- contrato asíncrono y resultados de operaciones;
- exactitud de metadata/configuración propietaria;
- identidad, observabilidad y métricas end-to-end;
- abstracción explícita del RP2040/Cat-Shell;
- validación uniforme de parámetros y límites de trama;
- limpieza de superficie pública, stubs y documentación de nivel de protocolo;
- reproducibilidad de KillerBee y validación de crypto/OOK/jamming.

Estas son áreas de investigación/diseño, no compromisos de feature.

## Criterios de salida sugeridos para la siguiente fase de análisis

Antes de aprobar cambios conviene disponer de:

1. requisitos formales para alcance PHY/protocolos y semántica de ACK/completion;
2. manifest de los binarios bajo prueba;
3. especificación versionada del wire protocol y de eventos;
4. definición de ownership de estado entre Python, RP2040, ControlTask y RadioIF;
5. casos de aceptación por capa, separando aceptación, ejecución RF y observación OTA;
6. matriz hardware/SDK/board revision y límites regulatorios;
7. decisión de compatibilidad para comandos y métodos existentes.

## Conclusión de entrega

La base funcional de FeralRF es real para control PHY y paquetes raw, pero la API no ofrece observabilidad suficiente para convertir todos los ACK en resultados técnicos verificables. Los defectos deterministas más urgentes para cualquier fase posterior son el layout de metadata propietaria, el contrato de secuencias asíncronas y los valores de modulación reservados. RX_STOP, concurrencia de colas, jam/TX y transporte requieren evidencia dirigida antes de elegir una solución. El alcance de protocolos completos debe resolverse como requisito, no inferirse de nombres de presets.

