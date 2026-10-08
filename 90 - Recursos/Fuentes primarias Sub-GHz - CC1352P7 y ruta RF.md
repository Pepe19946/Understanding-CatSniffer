# Fuentes primarias Sub-GHz — CC1352P7 y ruta RF

Registro de procedencia para [[Sub-GHz en CatSniffer V3 - CC1352P7, ruta RF y control de banda]]. Fecha de consulta externa: 2026-10-08. Esta nota no convierte documentación de fabricante ni archivos de diseño en evidencia de que una unidad ensamblada transmitió.

## Versiones y árboles inspeccionados

| Fuente | Ruta local | rama / revisión | estado antes de investigar |
|---|---|---|---|
| Vault `Understanding-CatSniffer` | `Understanding-CatSniffer/` | `main`, `5b81ada31939abf0f1eaee64afd809b1b4eb4eb9` | cambios previos en `.obsidian/workspace.json` y notas de experimentos; preservados |
| FeralRF | `FeralRF/` | `main`, `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea` | submódulo SDK marcado modificado |
| SimpleLink SDK | `FeralRF/firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/` | HEAD separado `5b31d0a4903351e544546e23ef3330eaa4291ceb` | `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md` modificado previamente |
| CatSniffer-Firmware | `CatSniffer-Firmware/` | `v3.x`, `c0cd5a45e019dbd14ed11d039aacb13f300e5731` | limpio |
| CatSniffer hardware | `CatSniffer/` | `master`, HEAD `6da5050f…`; se leyó el tag `v3.1` = `5b99d984933a8b8c36e789853c08d3a2a785bed4` | limpio |
| CatSniffer-Tools | `CatSniffer-Tools/` | `fix/CLI_control`, `126f13bc…` | limpio |
| CatSniffer-Tools empaquetado | `CatSniffer-Tools-v3.3.2.1/` | HEAD separado `a06b788…` | limpio |

El HEAD de un árbol fuente no identifica el binario instalado. Las asociaciones de sesión `v3.1.0.0`–`8eaa84c` y `v3.1.0.1`–`c0cd5a4` son contexto registrado, no hashes de las imágenes leídas de las placas.

## Texas Instruments: CC1352P7

### Datasheet

- **Título:** *CC1352P7 SimpleLink™ High-Performance Multi-Band Wireless MCU With Integrated Power Amplifier datasheet (Rev. A)*.
- **Documento:** SWRS251A, mayo de 2021, revisado noviembre de 2021.
- **URL:** https://www.ti.com/lit/ds/symlink/cc1352p7.pdf
- **Puntos usados:** portada y §3 (descripción y bandas); §8.3, *Radio (RF Core)*; diagrama de bloques; *Device Information* y terminales RF.
- **Condiciones/límites relevantes:** las bandas indicadas son `287–351`, `359–527`, `861–1054`, `1076–1315` y `2360–2500 MHz`. La potencia máxima depende de banda, alimentación, temperatura, PA y diseño RF. No demuestra la ruta de una PCB concreta.
- **Afirmaciones sustentadas:** radio Sub-1 GHz y 2.4 GHz integrada; Cortex-M4F principal; RF Core con Cortex-M0 no programable por el usuario, módem/baseband y API de comandos; 2-(G)FSK, 4-(G)FSK, (G)MSK, ASK/OOK entre las capacidades declaradas; PA integrado. `169.45 MHz` queda fuera de las bandas del dispositivo.

### Technical Reference Manual

- **Título:** *CC13x2x7 and CC26x2x7 SimpleLink™ Wireless MCU Technical Reference Manual*.
- **Documento:** SWCU192, noviembre de 2021.
- **URL:** https://www.ti.com/lit/ug/swcu192/swcu192.pdf
- **Secciones:** cap. 25 *Radio*, §25.1 *RF Core* (p. 2004), §25.2 *Radio Doorbell* (p. 2006), §25.3 *RF Core HAL* (p. 2010) y §25.4 *Data Queue Usage* (p. 2141), numeración del PDF/documento.
- **Aplicabilidad:** es el TRM exacto de la familia x7. Describe la interfaz entre CPU principal y RF Core, no la adaptación externa de CatSniffer.

### SDK, RF Driver y radio proprietary

- **SDK local de referencia:** `simplelink_cc13xx_cc26xx_sdk_8_30_01_01`, commit documentado `5b31d0a…`.
- **Guía TI 8.30:** *SimpleLink CC13xx CC26xx SDK 8.30 Proprietary RF User's Guide*, secciones *Examples* y *PHY Configuration*; https://software-dl.ti.com/simplelink/esd/simplelink_cc13xx_cc26xx_sdk/8.30.00.121/exports/docs/proprietary-rf/proprietary-rf-users-guide/proprietary-rf-guide/examples-cc13xx_cc26xx.html. La publicación disponible es 8.30.00.121; se contrastó con los headers del SDK local 8.30.01.01.
- **RF Driver API:** documentación `RF.h`/`RFCC26X2.h`; https://software-dl.ti.com/simplelink/esd/simplelink_cc13xx_cc26xx_sdk/latest/exports/docs/rflib/html/_r_f_c_c26_x2_8h.html. `RF_open()` crea el cliente; `RF_postCmd()` encola una operación; `RF_runCmd()` espera su terminación; el driver administra cola, callbacks y dominio de potencia RF.
- **Definición exacta usada:** `source/ti/devices/cc13x2x7_cc26x2x7/driverlib/rf_prop_cmd.h`, estructura `rfc_CMD_PROP_RADIO_DIV_SETUP_s`. `modType`: 0 FSK, 1 GFSK, 2 OOK; otros valores reservados en esa estructura. `deviation` usa pasos elegidos por `deviationStepSz`; con paso 0 son 250 Hz. `rxBw` es un código de configuración, no Hz directos.
- **Consecuencia:** en la configuración FeralRF examinada `deviation=100` y `deviationStepSz=0` equivalen a 25 kHz. No se decodificó `rxBw=0x52` a un ancho en Hz porque no se localizó en las fuentes examinadas una tabla primaria exacta que justificara el valor.

## Componentes RF externos

### Johanson 0900PC15A0036

- **Título:** *High Frequency Ceramic Solutions — 868/915 MHz and 2.4 GHz Dual Band Integrated Passive Component for TI CC1352R/P*.
- **Parte:** `0900PC15A0036`; revisión 1.1, 2021-05-12.
- **URL:** https://www.johansontechnology.com/datasheets/0900PC15A0036/0900PC15A0036.pdf
- **Páginas:** p. 1, especificaciones eléctricas; p. 2, configuración de terminales y aplicación.
- **Condiciones:** rutas especificadas `862–928 MHz` y `2400–2500 MHz`, con layout/red recomendados. La huella llamada `0900FM15D0039E` en KiCad no cambia el valor del componente U4 escrito por el esquema/PCB.
- **Uso:** sustenta que U4 es un componente pasivo integrado de adaptación/balun/filtro y que el frente publicado no cubre 433 ni 169 MHz.

### Qorvo RFSW8006Q

- **Título:** *RFSW8006Q 100 MHz to 4000 MHz, SP3T Switch*.
- **Parte en U2:** `RFSW8006QTR7`.
- **URL de producto/datasheet:** https://www.qorvo.com/products/p/RFSW8006Q
- **Datos usados:** SP3T, 100–4000 MHz, control GPIO; pinout y tabla lógica del datasheet enlazado por Qorvo.
- **Límite:** el switch selecciona continuidad RF; no crea portadora, bits ni modulación.

### Johanson 0900FM15D0039001E

- **Título:** *Low Pass Filter 0900FM15D0039001E*.
- **URL:** https://www.johansontechnology.com/products/chipset-specific-rf-front-end/semtech/0900fm15d0039001e/ y https://www.johansontechnology.com/docs/4855/IPD-0900FM15D0039001E.pdf
- **Datos usados:** 902–928 MHz en la ficha actual; configuración de terminales del filtro/balun para SX126x.
- **Límite:** el esquema antiguo incluye también `hardware/Datasheet/0900FM15D0039.pdf`; la respuesta exacta del componente montado y su idoneidad a 868 MHz no se midieron.

### pSemi PE42421

- **Título:** *PE42421 UltraCMOS® SPDT RF Switch*.
- **Revisión consultada:** DOC-33214-1.02, septiembre de 2025.
- **URL:** https://www.psemi.com/pdf/datasheets/pe42421ds.pdf
- **Páginas/secciones:** portada/especificaciones (10–3000 MHz), pinout y tabla de control.
- **Uso:** U6 conmuta las ramas RFI/RFO del SX1262; no modula.

### Semtech SX1262

- **Título:** *SX1262 LoRa Connect™ Long Range Low Power LoRa® Transceiver*.
- **URL:** https://www.semtech.com/products/wireless-rf/lora-connect/sx1262
- **Datos usados:** transceptor 150–960 MHz, LoRa/LR-FHSS y (G)FSK, PA de hasta +22 dBm; control por interfaz digital.
- **Límite:** esas son capacidades del SX1262. No implican que un preset proprietary enviado al CC1352 use el SX1262.

### Raspberry Pi RP2040

- **Título:** *RP2040 Datasheet*.
- **URL:** https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf
- **Secciones relevantes:** §1.1 (doble Cortex-M0+ y periféricos), §4.1 (UART), §4.2 (USB) y §4.4 (SSI/SPI).
- **Uso:** respalda la naturaleza de MCU/controlador del RP2040. Su función concreta en CatSniffer se prueba mejor por el esquema y `CatSniffer-Firmware`, no por la ficha genérica.

## Diseño CatSniffer versionado

- **Objeto Git leído sin checkout:** `CatSniffer:v3.1`, commit `5b99d984933a8b8c36e789853c08d3a2a785bed4`.
- **Esquema:** `hardware/CatSniffer.kicad_sch`, una hoja, bloque de título rev. `2.0`.
- **PCB:** `hardware/CatSniffer.kicad_pcb`, serigrafía CatSniffer v3.1 y bloque de título rev. `v3.2`.
- **PDF:** `hardware/CatSniffer.pdf`.
- **BOM/Gerbers:** no se halló un BOM independiente ni un conjunto de fabricación Gerber en el tag; los valores KiCad no son una verificación de población.
- **Identidad física registrada:** el usuario informó el marcaje `CC1352P74` en U8 el 2026-09-25. El esquema versionado dice `CC1352P1F3RGZT`. No existe en las fuentes suministradas una foto legible, BOM/ECO o esquema exacto de ensamblaje que cierre la discrepancia.

## Código fuente determinante

### FeralRF `0178721c…`

- Host: `python/feralrf/presets.py`, `radio.py`, `commands.py`, `enums.py`.
- Parser/control: `firmware/cc1352/src/command_processor.c`, `control_task.c`, `data_task.c`.
- Radio: `firmware/cc1352/src/radio_if.c`, `smartrf_prop_0.c`, `phy_manager.c` y headers.
- Configuración generada: `firmware/cc1352/syscfg/ti_drivers_config.c`, `ti_radio_config*.c`; la configuración declara `LP_CC1352P7-1`/`LP_CC1352P7_4` y DIO28/29/30.
- Build TI-RTOS: `firmware/cc1352/CMakeLists.txt` incluye `syscfg/ti_drivers_config.c`; el archivo alterno `src/ti_rf_config_min.c` no es el que sustenta este análisis del build documentado.
- Colas: tres entradas RF de 270 bytes, cola interna RX de ocho paquetes, cola de salida de 32 tramas y vaciado máximo de cuatro por pasada. Son límites de código, no prueba de que se hayan llenado durante las corridas.

### CatSniffer-Firmware `8eaa84c…` y `c0cd5a4…`

- Archivos: `RP2040/catsniffer/src/main.c`, `shell_commands.c`, `include/catsniffer.h`, `boards/rpi_pico.overlay`.
- La comparación de esos cuatro archivos entre ambas revisiones solo mostró, en este ámbito, la adición de la línea `Board:` a `fw_version`; enum, GPIO y `change_band()` son iguales.
- `CTF1/2/3` son GPIO 8/9/10. `band1` escribe `0,1,0`; `band2`, `0,0,1`; `band3`, `1,0,0`.
- `Band` es el valor almacenado en `catsniffer.band`; no existe lectura del estado físico de U2. La instancia global se inicializa a cero y `change_band()` retorna si el valor solicitado coincide, detalle relevante para el primer `change_band(GIG)` de arranque.

## Límites de esta investigación

- No se leyó un hash desde las placas ni se identificó el binario instalado.
- No se midieron GPIO, potencia, espectro, continuidad, VSWR ni paquetes OTA.
- No se verificaron modelo, respuesta o conexión real de las antenas informadas como multibanda.
- No se asumió que nombres como Wi-SUN, W-MBus, Sidewalk o MIOTY equivalgan a sus stacks o interoperabilidad.
- No se tomó el Vault antiguo `CatSniffer-Understanding` como autoridad.

