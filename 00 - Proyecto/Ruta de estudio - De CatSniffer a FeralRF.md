# Ruta de estudio — De CatSniffer a FeralRF

## Propósito

Esta ruta convierte el Vault en un curso progresivo. Lee primero las explicaciones conceptuales; usa las notas detalladas cuando quieras comprobar implementación y termina en `90 - Recursos` para verificar procedencia.

## Etapa 1 — Qué sistema estamos estudiando

1. [[CatSniffer - Mapa conceptual]] — visión completa y vocabulario inicial.
2. [[Arquitectura CatSniffer v3.1]] — placa objetivo, revisión y conexiones verificadas.
3. [[RP2040, CC1352P7 y SX1262 - Quién hace qué]] — comparación de responsabilidades.

Al terminar debes poder explicar por qué CatSniffer no es “un microcontrolador con dos radios”.

## Etapa 2 — Cada componente importante

4. [[RP2040 - El procesador de interfaz]] — USB, bridge, shell y control del SX1262.
5. [[CC1352P7 - El procesador de radio programable]] — segundo procesador y radio integrada.
6. [[SX1262 - El transceptor controlado por RP2040]] — periférico SPI/GPIO sin imagen propia.

Aquí aparecen tres buses importantes:

- **UART:** enlace serie punto a punto usado entre RP2040 y CC1352P7.
- **SPI:** bus síncrono usado por RP2040 para enviar órdenes/registros a SX1262.
- **GPIO:** pines digitales individuales usados para reset, boot, selección, interrupciones o estado.

## Etapa 3 — Qué software corre dónde

7. [[Mapa de firmware oficial]] — mapa de propiedad de las imágenes.
8. [[Firmware RP2040]] — implementación Zephyr de la coordinación.
9. [[Firmware CC1352P7]] — ecosistema TI, imágenes oficiales y límite binario.

Un **bootloader** es código de arranque que permite cargar otra imagen; no es la aplicación normal. Esta distinción será importante al separar operación diaria de programación.

## Etapa 4 — Cómo aparece ante la PC

10. [[USB CDC - Cómo CatSniffer aparece ante la PC]] — un USB, tres puertos serie lógicos.
11. [[Protocolos host-dispositivo]] — qué formato viaja por cada ruta.
12. [[Mapa de herramientas y comunicación PC]] — arquitectura del software host.
13. [[Catnip CLI]] — cómo Catnip descubre, abre y usa esas rutas.

**CLI** significa interfaz de línea de comandos. Catnip es una CLI de la PC; Cat-Shell es una consola dentro del RP2040. Sus nombres parecidos no significan que ejecuten en el mismo lugar.

## Etapa 5 — Seguir una operación completa

14. [[Del PC a la radio - Rutas de extremo a extremo]] — comandos, respuestas y capturas de principio a fin.

Al terminar debes poder narrar por separado:

- PC → Cat-Bridge → RP2040 → UART → CC1352P7;
- PC → Cat-Shell → lógica local RP2040;
- PC → control Shell + datos Cat-LoRa → RP2040 → SPI/GPIO → SX1262.

## Etapa 6 — Entender FeralRF por contraste

15. [[Arquitectura FeralRF]] — qué sustituye y qué reutiliza.
16. [[Protocolo y API Python]] — protocolo alternativo y API host.
17. [[Matriz de capacidades]] — diferencia entre declaración, implementación, pruebas y evidencia física.
18. [[Pruebas y evidencia existente]] — qué se sabe sin confundir tests con validación de hardware.

## Etapa 7 — Preparación, no ejecución, de trabajo futuro

19. [[Alcance y preguntas]] — límites y preguntas abiertas.
20. [[Estado y siguientes pasos]] — baseline auditable y autorización de fase.

No realices pruebas, flashing ni modificaciones siguiendo esta ruta. Esas acciones requieren una fase experimental explícitamente validada.

## Cómo leer la evidencia

- **Documentado:** una fuente lo afirma.
- **Observado en código/repositorio:** se ve directamente en archivos o Git.
- **Observado en hardware:** existe evidencia física suministrada.
- **Hipótesis:** explicación plausible aún no demostrada.
- **Conclusión:** síntesis respaldada por evidencia revisada.

## Comprueba tu comprensión

1. ¿En qué etapa se aprende quién interpreta cada mensaje USB?
2. ¿Por qué conviene estudiar la arquitectura oficial antes de FeralRF?
3. ¿Qué diferencia hay entre una nota conceptual y una nota de `90 - Recursos`?
4. ¿Cuándo una prueba Python permite afirmar algo sobre el radio físico?

**Anterior:** [[CatSniffer - Mapa conceptual]]  
**Siguiente:** [[Arquitectura CatSniffer v3.1]]

