# Arquitectura CatSniffer v3.1 — La placa como sistema

## En una frase

CatSniffer v3.1 reúne un [[RP2040 - El procesador de interfaz|RP2040]] que recibe a la PC, un [[CC1352P7 - El procesador de radio programable|CC1352P7]] que ejecuta otra aplicación RF y un [[SX1262 - El transceptor controlado por RP2040|SX1262]] que funciona como periférico de radio del RP2040.

## Modelo mental antes del esquema

No leas la placa como una lista de componentes. Léela como tres responsabilidades conectadas: **interfaz**, **procesamiento RF autónomo** y **radio periférico**. [[RP2040, CC1352P7 y SX1262 - Quién hace qué]] presenta primero esa comparación; esta nota demuestra después las conexiones con KiCad y Git.

Una flecha de comunicación no implica que ambos extremos hagan lo mismo. UART permite que dos procesadores intercambien bytes; SPI permite que un procesador controle un periférico; GPIO representa señales individuales como reset, boot, busy o interrupción.

## 1. Objetivo y alcance

Esta nota responde, a nivel orientado al firmware, **qué ejecuta código, qué controla cada componente y cómo se conectan el PC, los procesadores y los radios** en CatSniffer v3.1. No es una auditoría de RF, alimentación, integridad de señal o diseño PCB.

Etiquetas de evidencia usadas:

- **Documentado:** afirmación explícita en documentación.
- **Observado en repositorio/código:** dato visible en Git, KiCad o configuración de firmware.
- **Observado en hardware:** evidencia física comunicada explícitamente por el usuario; en particular, el marcaje de U8 añadido tras la Fase 1.
- **Hipótesis:** interpretación pendiente de confirmación.
- **Conclusión:** resultado respaldado por las fuentes indicadas.

## 2. Línea base exacta de v3.1

### Aclaración física posterior a la Fase 1

- **Observado en hardware — 2026-09-25:** U8 en la unidad CatSniffer v3.1 utilizada por el usuario está marcado `CC1352P74`.
- **Conclusión:** la unidad física bajo estudio usa hardware de la familia CC1352P7. Para el análisis de firmware se seguirá ese target físico.
- **Discrepancia preservada:** la fuente histórica `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch` identifica U8 como `CC1352P1F3RGZT`. La observación física no demuestra cuándo ni por qué cambió el diseño y no reescribe la procedencia del esquema publicado.

- **Observado en repositorio/código:** `v3.1` es un tag ligero que apunta directamente al commit `5b99d984933a8b8c36e789853c08d3a2a785bed4`, fechado `2024-01-10T19:07:43-05:00`, asunto `Merge pull request #62 from pwnlabmx/master`.
- Fuentes primarias leídas históricamente, sin cambiar el checkout:
  - `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch`
  - `CatSniffer:v3.1:hardware/CatSniffer.kicad_pcb`
  - `CatSniffer:v3.1:hardware/CatSniffer.kicad_pro`
  - `CatSniffer:v3.1:hardware/CatSniffer.pdf`
- El PDF exportado sí existe en el tag y su blob Git (`a5332e83f76ed42b52d29d92fe53e9e16e630518`) es idéntico al del checkout actual. Las fuentes KiCad no son idénticas entre `v3.1` y el checkout actual.
- **Advertencia de procedencia:** la serigrafía del PCB v3.1 dice CatSniffer v3.1, pero el bloque de título de `CatSniffer.kicad_pcb` dice revisión `v3.2`; el esquema plano dice revisión `2.0`. La selección de `v3.1` se basa en el tag y en la decisión de alcance, no en esos campos internos contradictorios.

## 3. Arquitectura de alto nivel

**Conclusión:** CatSniffer v3.1 tiene dos circuitos capaces de ejecutar firmware del proyecto y un transceptor controlado:

1. **U3 RP2040:** termina USB, ejecuta el firmware de control/puente, maneja el SX1262 por SPI y GPIO, se comunica con U8 por UART y dispone de GPIO para reset, arranque, depuración y selección RF.
2. **U8 CC1352:** MCU inalámbrico con radio Sub-1 GHz y 2.4 GHz integrado; ejecuta una imagen propia. La placa física está marcada `CC1352P74`, mientras las fuentes KiCad v3.1 dicen `CC1352P1F3RGZT`; el repositorio de firmware actual documenta `CC1352P7` para CatSniffer 3.x.
3. **U7 SX1262IMLTRT:** transceptor LoRa/(G)FSK controlado por U3. **No es un microcontrolador que ejecute una imagen de firmware CatSniffer independiente.** Recibe comandos y datos por SPI y devuelve estado/eventos por GPIO.

U9 `W25Q16JV` es la memoria flash QSPI externa del RP2040; almacena código/datos pero no ejecuta por sí misma firmware del proyecto. U2 `RFSW8006QTR7` selecciona una de tres rutas RF hacia la antena común J1; U6 `PE42421` conmuta la ruta TX/RX del SX1262. Son periféricos controlados, no procesadores de aplicación.

Evidencia principal: hoja única de `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch` y huellas/redes de `CatSniffer:v3.1:hardware/CatSniffer.kicad_pcb`, referencias U2, U3, U6, U7, U8 y U9.

## 4. Procesadores y componentes RF principales

| Ref. | Parte indicada en v3.1 | Tipo y función | ¿Ejecuta firmware del proyecto? | Relaciones principales |
| --- | --- | --- | --- | --- |
| U3 | `RP2040` | MCU de interfaz y control | Sí | USB con PC; UART/control con U8; SPI/GPIO con U7; QSPI con U9; GPIO con U2/U6 |
| U8 | `CC1352P1F3RGZT` en esquema y PCB | MCU inalámbrico multiprotocolo/multibanda con RF Sub-1 GHz y 2.4 GHz integrada | Sí | UART/control/debug con U3; rutas RF propias hacia U2 |
| U7 | `SX1262IMLTRT` | Transceptor LoRa y (G)FSK | No | SPI y GPIO con U3; RF mediante U6 hacia U2 |
| U9 | `W25Q16JV` | Flash QSPI externa de U3 | No | `QSPI_SS`, `QSPI_SCK`, `QSPI_SD0..3` con U3 |
| U2 | `RFSW8006QTR7` | Selector RF de tres vías hacia J1 | No | Control `CTF1..3` desde U3; selecciona LoRa, 2.4 GHz o Sub-1 GHz |
| U6 | `PE42421` | Conmutador RF TX/RX del camino SX1262 | No | Control asociado a `ANT_SW` y `DIO22` desde U3 |

**Documentado:** el README almacenado en `CatSniffer:v3.1:README.md` describe el CC1352P1 como MCU inalámbrico Sub-1 GHz/2.4 GHz y al RP2040 de V3 como puente USB-UART. El README actual de `CatSniffer-Firmware/RP2040/catsniffer/` atribuye al CC1352P7 protocolos de 2.4 GHz y Sub-GHz. Esto describe la capacidad del CC1352 como SoC con radio propia, separada del SX1262, pero no resuelve qué variante está montada físicamente en v3.1.

## 5. Interconexiones de control y datos

| Componente A → B | Interfaz | Señales observadas | Dirección/purpose | Evidencia |
| --- | --- | --- | --- | --- |
| P1 USB-C ↔ U3 RP2040 | USB 2.0 FS | `D+`, `D-` → `USB_DP`, `USB_DM` | Comunicación/programación USB entre PC y U3 | PCB v3.1, P1 y U3 |
| U3 ↔ U8 CC1352 | UART | `TXD`, `RXD`; U3 GPIO0/1; U8 DIO12/13 en el PCB | Enlace serie bidireccional; el sentido lógico TX/RX requiere confirmar configuración | PCB v3.1, U3/U8 |
| U3 → U8 | arranque/reset GPIO | `BOOT`, `RESET_CC`; PCB: GPIO2 y GPIO3 | Entrada al cargador y reinicio del CC; comportamiento temporal requiere Fase 2 | PCB v3.1; `rpi_pico.overlay` actual |
| U3 ↔ U8 | debug | `cJTAG_TCK`, `cJTAG_TMSC`, `JTAG/TDI`, `JTAG/TDO` en GPIO11..14 | Cableado que permite control/debug del CC; uso por firmware aún no demostrado | PCB v3.1, U3/U8/J3 |
| U3 ↔ U7 SX1262 | SPI | `CIPO` GPIO16, `NSS` GPIO17, `SCK` GPIO18, `COPI` GPIO19 | U3 es controlador SPI; U7 es periférico | PCB v3.1, U3/U7 |
| U7 → U3 y U3 → U7 | GPIO de estado/control | `BUSY` GPIO4, `DIO1` GPIO5, `DIO2` GPIO23, `RESET_SX` GPIO24, `DIO3` GPIO25 | BUSY/IRQ/estado desde U7; reset desde U3; función efectiva de DIO2/DIO3 requiere firmware | PCB v3.1, U3/U7 |
| U3 → U2 | selección RF | `CTF1`, `CTF2`, `CTF3` en GPIO8..10 | Selección de una de tres rutas hacia antena | PCB v3.1, U2/U3 |
| U3 → U6 | selección TX/RX SX | `ANT_SW` GPIO20, `DIO22` GPIO21 a la red de control de U6 | Conmutación del camino RF del SX1262 | PCB v3.1, U3/U6 |
| U3 ↔ U9 | QSPI | `QSPI_SS`, `QSPI_SCK`, `QSPI_SD0..3` | Almacenamiento de firmware/datos del RP2040 | Esquema/PCB v3.1, U3/U9 |

Advertencias concretas:

- El esquema gráfico y la conectividad del PCB no están totalmente sincronizados en `BOOT`/`RESET_CC`: el PCB asigna GPIO2 a `BOOT` y GPIO3 a `RESET_CC`, mientras algunas etiquetas visibles del esquema dicen `CTS`/`RTS`; el PCB además asocia GPIO15 con `RESET_CC`. Para el pinout efectivo se prioriza aquí la conectividad del PCB, pero debe confirmarse antes de programar.
- El comentario al final de `CatSniffer-Firmware/RP2040/catsniffer/boards/rpi_pico.overlay` dice DIO1=GPIO6, mientras la propiedad activa y el PCB v3.1 usan GPIO5. La configuración activa y el PCB coinciden en GPIO5.

## 6. Topología PC/USB

**Conclusión:** v3.1 tiene un único conector de datos USB-C, P1, conectado eléctricamente al periférico USB de U3 RP2040. U8 y U7 no tienen conexión USB directa al PC.

- El RP2040 puede presentarse al host y mediar tráfico hacia ambos radios.
- U8 llega al host indirectamente por el UART interno y por el comportamiento de puente que implemente U3.
- U7 llega al host indirectamente: U3 traduce tráfico USB a operaciones SPI/GPIO.
- El firmware actual documenta tres interfaces CDC-ACM (`Cat-Bridge`, `Cat-LoRa`, `Cat-Shell`) en `RP2040/catsniffer/boards/rpi_pico.overlay`; que una imagen concreta las exponga y cómo enrute datos es materia de Fase 2, no una propiedad garantizada solo por el esquema.

## 7. Propiedad de firmware y programación

| Procesador | Papel | Proyecto oficial visible | Ecosistema evidente | Programación/debug | Dispositivos controlados | Incertidumbre |
| --- | --- | --- | --- | --- | --- | --- |
| U3 RP2040 | USB, coordinación, puente al CC, control de SX1262 y switches RF | `CatSniffer-Firmware/RP2040/catsniffer/` | Zephyr, west, CMake/Kconfig; target `rpi_pico`; UF2/HEX/ELF documentados | USB P1 + BOOTSEL (SW1 sobre `QSPI_SS`) para UF2; SWD `SWDIO/SWCLK/RESET` aparece en J2 del esquema | U7, U8 (enlace/control), U2, U6, U9, LEDs | J2 no tiene huella/referencia en el PCB v3.1; acceso SWD físico no confirmado |
| U8 CC1352 | MCU inalámbrico con radio Sub-1/2.4 GHz integrada | El repositorio actual ofrece `CatSniffer-Firmware/CC1352P7/` | TI Code Composer Studio/SysConfig, TI-RTOS7/SimpleLink según proyectos; HEX precompilados | UART/ROM bootloader mediante U3 y USB, condicionado al firmware puente; J3 expone cJTAG/reset; botones `BOOT1` y `RESET_CC1` | Su propio subsistema RF; no controla U7 | El hardware v3.1 declara P1, pero el proyecto oficial actual apunta a P7; no se debe flashear hasta resolverlo |

El SX1262 no tiene una fila propia: no recibe una imagen CatSniffer. Su configuración forma parte del firmware que corre en U3.

Discrepancia documental adicional: `CatSniffer-Firmware/CC1352P7/README.md` enumera `Sniffle_CC1352P_7`, pero ese directorio no está presente en el checkout inventariado; sí están `sniffer_fw_cc1252P_7/`, `airtag_scanner_CC1352P_7/`, `airtag_spoofer_CC1352P_7/` y `justworks_scanner_CC1352P7_1/`.

### Rutas de recuperación/programación observables

- **RP2040:** SW1 lleva `QSPI_SS` a nivel de arranque para el modo BOOTSEL y P1 proporciona USB. RESET1 actúa sobre `RUN/RESET`. La ruta SWD J2 está en el esquema, pero no en el PCB versionado.
- **CC1352:** `BOOT`, `RESET_CC` y UART permiten conceptualmente el cargador serie; J3 ofrece `+3V3`, `cJTAG_TMSC`, `RESET_CC`, `cJTAG_TCK` y GND. La automatización desde U3 y compatibilidad exacta P1/P7 requieren confirmación.

## 8. Rutas conceptuales PC → RF

### Ruta por el radio integrado del CC1352

`PC → USB-C P1 → USB de U3 → UART U3↔U8 → firmware/radio integrado de U8 → ruta 2.4 GHz o Sub-1 GHz → U2 → J1/antena`

- El esquema/PCB establece P1–U3, U3–U8 y las rutas RF de U8 a U2.
- **Requiere Fase 2:** qué endpoint/comando USB alimenta el UART, qué protocolo entiende la imagen de U8, cuándo transmite y cómo se selecciona la banda.

### Ruta por SX1262

`PC → USB-C P1 → USB de U3 → SPI/GPIO U3→U7 → U7 SX1262 → U6 → ruta LoRa de U2 → J1/antena`

- El PCB establece SPI, BUSY/DIO/reset, U6 y el selector común U2.
- **Requiere Fase 2:** framing USB, comandos concretos, configuración del SX1262, tratamiento de IRQ y secuencia de conmutación U6/U2.

## 9. Rutas conceptuales RF → PC

### Recepción por CC1352

`J1/antena → U2 → camino 2.4 GHz o Sub-1 GHz → radio/firmware de U8 → UART U8↔U3 → USB de U3 → P1 → PC`

### Recepción por SX1262

`J1/antena → U2 → U6 → U7 SX1262 → DIO/BUSY + SPI hacia U3 → USB de U3 → P1 → PC`

**Requiere Fase 2** en ambos casos: confirmar que el firmware implementa cada ruta, los endpoints utilizados, el formato de los datos, las interrupciones y el control de concurrencia. El hardware solo demuestra que las conexiones existen.

## 10. Diagrama de bloques

```mermaid
flowchart LR
    PC[PC] <-->|USB D+/D-| P1[USB-C P1]
    P1 <--> RP[U3 RP2040\nfirmware de control]
    RP <-->|UART TXD/RXD| CC[U8 CC1352\nfirmware propio + radio 2.4/Sub-1]
    RP -->|BOOT, RESET, cJTAG/JTAG| CC
    RP <-->|SPI CIPO/COPI/SCK/NSS\nBUSY, DIO, RESET| SX[U7 SX1262\ntransceptor LoRa/(G)FSK]
    RP -->|CTF1..3| RFSW[U2 selector RF]
    RP -->|ANT_SW, DIO22| TRSW[U6 switch TX/RX]
    CC -->|RF 2.4 GHz / Sub-1 GHz| RFSW
    SX <--> TRSW
    TRSW <-->|ruta LoRa| RFSW
    RFSW <--> ANT[J1 antena SMA]
    DBG1[J3 cJTAG externo] <--> CC
    DBG2[J2 SWD en esquema\nno hallado en PCB] -.-> RP
```

## 11. Lo que debe confirmar la Fase 2

1. Compatibilidad real de la imagen `CC1352P7` con U8: resolver primero P1 frente a P7.
2. Sentido/configuración UART y protocolo entre U3 y U8.
3. Secuencias reales de `BOOT` y `RESET_CC`, incluida la discrepancia GPIO3/GPIO15.
4. Uso efectivo de GPIO11..14 para recuperación cJTAG/JTAG.
5. Inicialización SPI, IRQ `DIO1`, `BUSY`, reset y uso —si existe— de DIO2/DIO3 del SX1262.
6. Lógica y estados válidos de U2/U6 para cada banda y dirección.
7. Mapeo de endpoints USB a puente CC, radio SX1262 y consola.
8. Qué binarios oficiales corresponden exactamente a la placa v3.1 utilizada.

## 12. Preguntas abiertas

### Procedencia histórica aún no resuelta

- ¿Ingeniería dispone de una corrección/BOM/ECO de v3.1 que explique el cambio P1↔P7 y las etiquetas de revisión internas?

### No bloqueante para comprender la topología

- ¿Está accesible físicamente un punto SWD del RP2040 aunque J2 no aparezca en el PCB versionado?
- ¿Qué imagen/versiones están instaladas actualmente en U3 y U8? Esto afecta la Fase 2, no altera el cableado documentado.

## 13. Referencias

El registro conciso y las advertencias de procedencia están en [[Fuentes hardware CatSniffer v3.1]]. Las fuentes técnicas principales son `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch`, `CatSniffer:v3.1:hardware/CatSniffer.kicad_pcb`, `CatSniffer:v3.1:README.md`, `CatSniffer-Firmware/RP2040/catsniffer/boards/rpi_pico.overlay` y `CatSniffer-Firmware/CC1352P7/README.md`.

## Por qué esto importa después

El cableado fija los límites del software: la PC no llega directamente al CC o al SX; toda entrada USB pasa primero por RP2040. También fija qué puede cambiar FeralRF: la aplicación del CC, no el enlace SPI del SX1262 ni la terminación USB.

## Comprueba tu comprensión

1. ¿Por qué UART entre RP2040 y CC1352P7 indica dos dominios de firmware?
2. ¿Qué evidencia diferencia a SX1262 de un MCU programable del proyecto?
3. ¿Qué componente decide a cuál subsistema se entrega cada CDC?
4. ¿Por qué la discrepancia P1/P7 debe conservarse aunque la unidad física diga P74?

**Anterior:** [[CatSniffer - Mapa conceptual]]  
**Siguiente:** [[RP2040, CC1352P7 y SX1262 - Quién hace qué]]

**Profundización RF:** [[Sub-GHz en CatSniffer V3 - CC1352P7, ruta RF y control de banda]] traza U8→U4→U2→J1, separa la callback DIO28/29/30 del selector CTF gobernado por RP2040 y conserva la incertidumbre de revisión P1/P7.
