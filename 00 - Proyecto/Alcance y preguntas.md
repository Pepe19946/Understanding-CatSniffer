# Alcance y preguntas

> [!note] Entrada para estudiar
> Esta nota conserva decisiones y preguntas del proyecto. Para aprender la arquitectura en orden, empieza en [[CatSniffer - Mapa conceptual]] y sigue [[Ruta de estudio - De CatSniffer a FeralRF]].

## Actualización de alcance — Fase 1 (2026-09-25)

- **Revisión objetivo fijada por el usuario:** CatSniffer v3.1, por ser la placa utilizada y consultada con ingeniería.
- **Enfoque:** arquitectura de hardware necesaria para comprender en la fase siguiente dónde corre cada firmware, qué controla y por qué interfaz.
- **Fuentes de revisiones posteriores:** solo comparación explícitamente identificada; ningún dato de v3.2/v3.3 se traslada a v3.1 por suposición.
- **Fuera de alcance:** auditoría exhaustiva de alimentación, RF, adaptación, impedancia, PCB, EMC, tolerancias o BOM; también quedan fuera ejecución interna, protocolos, buffers y drivers de firmware.
- **Decisión conceptual confirmada:** RP2040 y CC1352 ejecutan firmware; SX1262 es un transceptor periférico controlado por RP2040, no un tercer MCU de aplicación.
- **Bloqueante nuevo para Fase 2:** `CatSniffer:v3.1` identifica U8 como `CC1352P1F3RGZT`, pero `CatSniffer-Firmware/CC1352P7/` y su documentación actual asocian v3.x con P7. Se requiere marcaje físico o aclaración/BOM/ECO de ingeniería antes de seleccionar o interpretar como aplicable una imagen CC1352P7.
- **Incertidumbres de programación:** J2 SWD del RP2040 aparece en el esquema, no en el PCB v3.1; además hay divergencias de etiquetas `BOOT`/`RESET_CC` entre esquema y conectividad PCB. No programar basándose solo en esas etiquetas.
- Las preguntas de Fase 0 conservadas más abajo son registro histórico. La selección de revisión ya no está abierta; sí permanece abierta la variante exacta P1/P7 de U8 dentro de la documentación v3.1.

## Objetivo general

Comprender CatSniffer como plataforma embebida —hardware, firmware y herramientas de PC— y, después de establecer una línea base verificable, evaluar el estado real de FeralRF. Fuente controladora: `docs/CatSniffer-FeralRF-plan-de-trabajo-actualizado.md`.

## Alcance actual: Fase 0

- Verificar las seis rutas locales.
- Fijar rama, commit, fecha, tracking, remotos, tags y limpieza de los cuatro clones.
- Inventariar estructura, documentación, fuentes técnicas y enlaces disponibles.
- Separar revisiones y variantes explícitas sin elegir todavía la placa aplicable.
- Crear la estructura mínima del Vault y las tres notas de proyecto.
- Registrar dudas y detenerse para revisión.

## Exclusiones explícitas

En esta fase no se analiza en detalle alimentación, USB, interconexiones, buses, reset/boot, programación, RF ni antenas; tampoco flujos de ejecución, protocolos de host, capacidades reales de FeralRF, planes de prueba o propuestas de mejora. No se compila, flashea, prueba hardware ni modifica ninguno de los cuatro repositorios.

## Restricciones conocidas

- `CatSniffer/`, `CatSniffer-Firmware/`, `CatSniffer-Tools/` y `FeralRF/` son estrictamente de solo lectura durante investigación.
- Solo se escribe en `CatSniffer-Understanding/`.
- No se ejecutó red (`fetch`, descargas ni consultas externas); ramas/remotos reflejan únicamente metadatos ya clonados.
- Un README o nombre de archivo es evidencia documental, no validación física.
- Las versiones de placa, firmware, herramienta y paquete son independientes.
- `FeralRF` estaba sucio antes del inventario por un submódulo con `Board.md` modificado; se preservó.

## Hechos confirmados para orientar preguntas

- **Observado en repositorio:** el layout coincide exactamente con el indicado.
- **Documentado/observado:** existen familias v1.x/v2.x y v3.x; las fuentes asocian las primeras con SAMD21/CC1352P1 y las segundas con RP2040/CC1352P7 (`CatSniffer-Firmware/README.md`).
- **Observado en repositorio:** el checkout de hardware está en tag v3.3, pero el PCB se etiqueta v3.2 y el esquema rev 2.0 (`CatSniffer/hardware/CatSniffer.kicad_{pcb,sch}`).
- **Observado en hardware:** ninguno; no se recibieron fotos, lectura de serigrafía ni salida de dispositivo.

## Preguntas bloqueantes antes de Fase 1

1. ¿Cuál es el texto exacto de modelo y revisión en la serigrafía de la placa física, por ambas caras?
2. ¿Puedes proporcionar fotografías nítidas de ambas caras, incluyendo conectores, chips y etiquetas, sin ocultar la marca de revisión?
3. Si la placa dice v3.3 (u otra revisión que no coincida con los títulos internos), ¿qué esquema/fuente entregada por Electronic Cats debe tratarse como canónica? Si no se sabe, habrá que mantener variantes separadas en Fase 1.
4. ¿Se autoriza usar una variante documental como objeto provisional de Fase 1 si la revisión física no puede confirmarse, o debe detenerse el análisis hasta tener evidencia física?

## Requiere inspección física

1. ¿La placa usa RP2040 o SAMD21 y qué marcaje exacto tiene el CC1352 (P1/P7)?
2. ¿Qué firmware y versión reporta actualmente cada procesador relevante? Conservar la salida exacta de cualquier comando de versión, sin asumir que coincide con el checkout.
3. ¿La placa arranca, enumera por USB y expone qué puertos/dispositivos? Esto pertenece a una fase posterior; por ahora solo se necesita saber si hay acceso.
4. ¿Hay modificaciones, reparaciones, jumpers o componentes DNP/poblados distintos del diseño publicado?

## Útiles pero no bloqueantes

1. ¿Qué programadores/debuggers están disponibles (por ejemplo, sonda cJTAG/JTAG compatible, cables y adaptadores)?
2. ¿Qué sistemas operativos y versiones se usarán para las herramientas de host?
3. ¿Qué acceso de laboratorio existe: segunda placa, analizador lógico, osciloscopio, SDR/analizador de espectro, atenuadores, cargas, antenas y entorno RF autorizado?
4. ¿Hay restricciones de laboratorio, transmisión RF, horarios, seguridad o fechas límite?
5. ¿Qué espera exactamente el asesor del informe, demostraciones y mejoras, y cuáles son los criterios de aceptación?
6. ¿Existen documentos privados, BOM, Gerbers, esquema v3.3 o notas de fabricación que deban añadirse como fuente sin copiarlos íntegros al Vault?

## Requiere aclaración del estado local

1. ¿Es intencional la marca `M` en `FeralRF/firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md`? Git no mostró diff textual y avisó LF→CRLF.
2. ¿Debe preservarse indefinidamente ese checkout sucio como parte de la línea base experimental, o se definirá posteriormente una línea base limpia mediante una acción autorizada?

## Requiere análisis posterior de código o documentación

1. ¿Cómo se relacionan exactamente el tag de hardware v3.3, el PCB v3.2 y el esquema rev 2.0? Fase 1, conservando fuentes separadas.
2. ¿Qué significado tienen las discrepancias `catnip/VERSION` 3.3.2.1, README 3.0.0 y changelog V3.3.2 “unreleased”? Fase 3.
3. ¿Qué parte de las funciones declaradas por FeralRF está implementada, cubierta por tests y validada físicamente? Fases 4–5.
4. ¿La copia `FeralRF/hardware/` deriva de una revisión concreta de `CatSniffer` y qué cambios propios contiene? Fase 1 o 4, sin mezclar sus hechos.

## Incertidumbres que no deben resolverse por especulación

- Revisión exacta de la placa disponible.
- Correspondencia entre el PDF idéntico de ambos repositorios y sus fuentes KiCad divergentes.
- Firmware realmente instalado en el dispositivo.
- Actualidad de ramas/remotos respecto de GitHub, al no haberse hecho `fetch`.
- Intención de la modificación del submódulo TI.
- Aplicabilidad de documentación histórica/talleres a la placa de prueba.

## Decisión de alcance

La investigación se centrará estrictamente en **CatSniffer v3.1**, por ser la revisión utilizada actualmente por el usuario y la consultada con ingeniería. Otras revisiones podrán utilizarse únicamente como referencia comparativa cuando sea necesario y deberán identificarse explícitamente como tales.

## Profundidad de hardware

Fase 1 estudiará hardware al nivel necesario para comprender la ejecución, construcción, programación e interacción del firmware. No se realizará inicialmente un análisis electrónico exhaustivo de RF, alimentación o PCB salvo cuando sea necesario para explicar el comportamiento del firmware.
