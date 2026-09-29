# Firmware CC1352P7 — La aplicación del segundo procesador

## En una frase

El firmware CC1352P7 es una imagen independiente que recibe tráfico por UART y decide cómo usar la radio integrada del [[CC1352P7 - El procesador de radio programable|segundo procesador]].

## Modelo mental

Cat-Bridge es el conducto, no la aplicación. Al otro extremo existe un procesador que arranca, mantiene estado y ejecuta lógica RF. En el baseline oficial, la imagen normal del sniffer está disponible como binario HEX, pero su implementación fuente no lo está; por eso el análisis puede llegar hasta el límite UART/protocolo sin inventar sus internos.

## 1. Aclaración del target físico

- **Observado en hardware — 2026-09-25:** U8 está marcado `CC1352P74`.
- **Fuente TI:** el datasheet CC1352P7 lista `CC1352P74T0RGZR` y el marcado `CC1352 P74`; se trata por tanto como un MCU inalámbrico CC1352P7-family.
- **Discrepancia:** la fuente histórica `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch` identifica U8 como `CC1352P1F3RGZT`.
- **Conclusión de Fase 2:** seguir el target físico P7 y conservar la contradicción histórica. No se encontró evidencia de cuándo ni por qué cambió el componente.

El CC1352P7 es un segundo computador, no un periférico SPI del RP2040. Ejecuta su propia imagen desde flash y posee radio Sub-1 GHz/2.4 GHz integrada. Se comunica con el RP por UART y recibe de éste BOOT/RESET.

## 2. Proyectos e imágenes disponibles

| Proyecto/ruta | Fuente actual | Target/build | Finalidad observable | Artefacto relevante |
| --- | --- | --- | --- | --- |
| `sniffer_fw_cc1252P_7/` | **No**, sólo metadatos CCS y HEX | `Cortex M.CC1352P7`, `DeviceFamily_CC13X2X7`, TI-RTOS; metadatos heredados de SDK CC13X2/26X2 | README/nombre lo presentan como firmware multiprotocolo CatSniffer v3.x | `sniffer_fw_Catsniffer_v3.x.hex` |
| `airtag_scanner_CC1352P_7/` | Sí | `LP_CC1352P7_1`, SimpleLink SDK 7.10.01.24, SysConfig 1.16.x, TI-RTOS7/TI Clang | BLE central que escanea anuncios AirTag y reporta UART | `Release/airtag_scanner_CC1352P_7.hex` |
| `airtag_spoofer_CC1352P_7/` | Sí | mismo target/ecosistema 7.10 | BLE peripheral/anunciante especializado | `Release/airtag_spoofer_CC1352P_7.hex` |
| `justworks_scanner_CC1352P7_1/` | Sí | device `CC1352P7RGZ`, board `LP_CC1352P7_1`, SDK 8.31.00.11, SysConfig 1.21.1, TI-RTOS7/TI Clang | BLE central/scanner con logs UART; ejemplo representativo trazable | `Release/justworks_scanner_CC1352P7_1.hex/.out` |

`CC1352P7/README.md` menciona además `Sniffle_CC1352P_7`, ausente de HEAD. Git registra su eliminación en `325fe42…` (`remove old Sniffle firmware`).

## 3. Selección relevante

**Documentado/observado:** `sniffer_fw_Catsniffer_v3.x.hex` es el candidato más claro para la operación normal multiprotocolo CatSniffer, pero es binario. En `da6ff094…` se eliminó su fuente con el asunto `erase source code since violating TI license`.

**Conclusión:** no es válido usar JustWorks/AirTag para afirmar cómo funciona internamente el sniffer normal. JustWorks sólo demuestra un patrón de aplicación P7/SimpleLink, un target P7 concreto y una ruta UART compatible con el cableado de la placa.

No se pudo establecer cuál de estas imágenes está instalada en la unidad física.

## 4. Arquitectura de build

La arquitectura común de los proyectos con fuente es:

`fuente C + .syscfg + metadatos CCS → Code Composer Studio/SysConfig + SimpleLink SDK + TI-RTOS7 → .out → utilidad hex TI → Intel HEX`.

Para JustWorks:

- `.cproject` selecciona SDK `8.31.00.11`, toolchain TI Clang y configuraciones Debug/Release.
- `justworks_scanner.syscfg` selecciona `CC1352P7RGZ`, `LP_CC1352P7_1`, TI-RTOS7 y rol BLE `CENTRAL`.
- `.project` enlaza los recursos/RTOS generados; `targetConfigs/CC1352P7.ccxml` aporta la configuración de debug CCS.
- SysConfig genera Board, drivers, BLE y configuración de radio bajo `Release/syscfg/`.

Los ejemplos Airtag usan SDK `7.10.01.24`. Esta mezcla de SDKs significa que el directorio P7 no es un único producto compilable con una sola versión.

El sniffer binario conserva metadatos de target P7 y una dependencia TI-RTOS, pero sin fuente/SysConfig actual no puede reconstruirse ni auditarse desde HEAD.

## 5. Entrada y startup representativo con fuente

En `justworks_scanner_CC1352P7_1/Startup/main.c`, `main()`:

1. registra `RegisterAssertCback()`;
2. ejecuta `Board_initGeneral()`;
3. habilita/configura cache VIMS y restricciones de energía opcionales;
4. configura el temporizador del stack BLE;
5. llama `ICall_init()`;
6. crea tareas remotas del stack BLE con `ICall_createRemoteTasks()` (prioridad 5);
7. crea la tarea de aplicación con `JustWorksScanner_createTask()` (prioridad 1);
8. entrega ejecución a `BIOS_start()`.

**Límite:** es el startup de JustWorks, no evidencia del contenido del HEX de sniffer.

## 6. Modelo TI-RTOS/SimpleLink representativo

`Application/justworks_scanner.c` construye una tarea TI-RTOS. En su inicialización registra la aplicación con ICall, crea cola/evento, configura GATT/GAP, servicios y bond manager, inicia GAP Central y abre `Display` sobre UART.

La tarea espera indefinidamente con `Event_pend()`. Al despertar:

- obtiene mensajes del stack BLE mediante ICall;
- despacha eventos GAP/GATT/HCI;
- drena su cola de mensajes de aplicación;
- procesa anuncios/conexiones y reinicio periódico del escaneo.

El stack BLE corre en tareas remotas separadas creadas antes. Éste es el patrón de concurrencia demostrable para el ejemplo fuente.

## 7. UART con RP2040

En JustWorks, la configuración generada usa UART a `921600`, TX DIO13 y RX DIO12. Esto coincide con el enlace cruzado de la PCB: RP TX GPIO0 llega a CC RX DIO12 y CC TX DIO13 llega a RP RX GPIO1.

La aplicación abre `Display` sobre UART y emite logs/resultados de escaneo. No se encontró un parser de comandos UART en JustWorks; por tanto:

- el retorno `CC → RP → USB` sí queda ejemplificado por sus logs;
- la entrada `RP → CC` no se convierte aquí en comandos de radio;
- no debe extrapolarse el protocolo del HEX multiprotocolo.

El spoofer contiene rutas NPI/PTM opcionales según defines de build, pero no se analizaron como protocolo CatSniffer normal.

## 8. Operación de radio

JustWorks configura rol BLE Central a través de GAP/ICall; el stack SimpleLink/TI maneja el radio integrado y entrega eventos a la tarea. Los ejemplos Airtag también son aplicaciones BLE especializadas.

**Documentado por TI:** CC1352P7 integra MCU Cortex-M4F y radios Sub-1 GHz/2.4 GHz. Las variantes de LaunchPad `LP-CC1352P7-1` y `-4` tienen redes/rangos RF diferentes. Los proyectos fuente seleccionan explícitamente `LP_CC1352P7_1`; esto no prueba que toda la red RF de la CatSniffer custom sea idéntica.

No es posible observar desde el HEX normal qué protocolos inicializa, qué comandos acepta, cómo programa el RF core ni cómo devuelve capturas.

## 9. Buffers, concurrencia y errores

Para JustWorks:

- tarea de aplicación prioridad 1;
- tareas BLE remotas prioridad 5;
- `Event_pend()` + cola ICall + cola de aplicación;
- buffer Display de 128 bytes según SysConfig;
- rings generados UART2 de 32 bytes RX y 32 bytes TX;
- callbacks/eventos GAP/GATT/HCI, más temporizadores/watchdog de escaneo.

`AssertHandler()` muestra el error y se detiene; `smallErrorHook()` entra en un bucle infinito. Las rutas normales comprueban varios status de stack y publican/descartan eventos según tipo. No se generaliza este manejo al sniffer binario.

## 10. Programación y control de arranque

El RP2040 puede forzar BOOT bajo, pulsar RESET y cambiar UART a 500000; después `Cat-Bridge` transporta los bytes de un cargador host. En modo normal libera BOOT mediante pull-up, resetea y usa 921600. El README documenta `cc2538-bsl`, pero la lógica del host corresponde a Fase 3.

La ruta independiente es J3/cJTAG con CCS y los `.ccxml`. SysConfig del ejemplo JustWorks habilita bootloader/backdoor en DIO15 activo bajo, mientras la placa/firmware RP usa su red `BOOT`; la equivalencia eléctrica exacta debe verificarse antes de programar y no se probó aquí.

## 11. P1/P7 y compatibilidad

1. **Target actual explícito:** sí; `.cproject`, `.syscfg` y `.ccxml` identifican P7, `CC1352P7RGZ` y/o `LP_CC1352P7_1`.
2. **Camino P1 histórico:** sí; el esquema v3.1 dice P1 y el árbol borrado del sniffer incluía configuración `smartrf_settings/cc1352p1lp`. Es procedencia, no prueba de que el HEX actual sea P1.
3. **Explicación de transición:** no hallada.
4. **Suposiciones relevantes:** los ejemplos fuente están configurados para LaunchPad `P7_1`; la CatSniffer es una placa custom. Pines UART coinciden, pero RF/boot y artefactos deben validarse por proyecto.
5. **Asociación física:** el marcaje P74 y los targets actuales permiten asociar con confianza la familia de CPU. No prueban la imagen instalada ni toda su configuración.

Resultado: **procedencia histórica no resuelta; target físico confirmado como familia P7.**

## 12. Limitación fuente frente a binario

| Afirmación | Categoría |
| --- | --- |
| El directorio normal contiene un HEX P7 y metadatos CCS | Observado en repositorio |
| El README lo presenta como sniffer multiprotocolo CatSniffer | Documentado |
| JustWorks arranca TI-RTOS/ICall/GAP y escribe por UART | Observado en código de JustWorks |
| El HEX normal usa el mismo startup/parser/buffers | **No establecido** |
| El HEX normal está instalado en la placa del usuario | **No establecido** |

## 13. Preguntas abiertas

- ¿Qué imagen y versión están instaladas realmente en U8? Requiere leer/probar hardware.
- ¿Qué protocolo acepta `sniffer_fw_Catsniffer_v3.x.hex` por UART? Requiere documentación/Catnip (Fase 3) o prueba; no se puede inferir de JustWorks.
- ¿Cuál fue la transición de P1 a P7 y qué configuración fuente produjo el HEX actual? La historia local no lo explica.
- ¿La ruta BOOT/DIO15 usada por SysConfig coincide de forma segura con la red física y secuencia RP? Requiere validación antes de flashear.

## 14. Evidencia principal

- `CC1352P7/README.md`.
- `CC1352P7/sniffer_fw_cc1252P_7/{.project,.cproject,sniffer_fw_Catsniffer_v3.x.hex}`.
- `CC1352P7/justworks_scanner_CC1352P7_1/{.project,.cproject,justworks_scanner.syscfg,Startup/main.c,Application/justworks_scanner.c,targetConfigs/CC1352P7.ccxml}` y `Release/syscfg/`.
- `.cproject`, `.syscfg` y source de los proyectos Airtag.
- Commits históricos `da6ff094…` y `325fe42…`.
- Baseline `c0cd5a45e019dbd14ed11d039aacb13f300e5731`.
- TI: `https://www.ti.com/product/CC1352P7`, `https://www.ti.com/lit/ds/symlink/cc1352p7.pdf`, `https://www.ti.com/tool/LP-CC1352P7`.

## Por qué esto importa después

Este límite de visibilidad explica por qué [[Arquitectura FeralRF|FeralRF]] es arquitectónicamente relevante: sustituye la aplicación binaria normal por una alternativa fuente-visible sin eliminar el RP2040 ni Cat-Bridge.

## Comprueba tu comprensión

1. ¿Qué puede conocerse de un HEX sin disponer de su fuente?
2. ¿Por qué un ejemplo TI con fuente no demuestra el flujo interno del sniffer normal?
3. ¿Quién controla BOOT/RESET del CC y quién ejecuta el bootloader?
4. ¿Qué evidencia permite seleccionar P7 para la unidad física?

**Anterior:** [[Firmware RP2040]]  
**Siguiente:** [[USB CDC - Cómo CatSniffer aparece ante la PC]]  
**Relacionado:** [[CC1352P7 - El procesador de radio programable]]
