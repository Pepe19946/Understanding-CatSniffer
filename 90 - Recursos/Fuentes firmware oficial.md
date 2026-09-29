# Fuentes firmware oficial

**Conocimiento derivado:** [[Mapa de firmware oficial]], [[Firmware RP2040]], [[Firmware CC1352P7]] y [[USB CDC - Cómo CatSniffer aparece ante la PC]].

## Línea base

- Repositorio: `CatSniffer-Firmware/`
- Rama: `v3.x` (tracking `origin/v3.x` local)
- HEAD: `c0cd5a45e019dbd14ed11d039aacb13f300e5731`
- Tags en HEAD: `v2.1.0.0`, `v3.1.0.1`
- Estado al análisis: limpio
- Método: lectura directa y Git read-only; sin build, checkout, fetch ni generación.

## RP2040

| Ruta | Tipo/uso |
| --- | --- |
| `RP2040/README.md` | Build/carga documentados y uso de `cc2538-bsl` |
| `RP2040/catsniffer/README.md` | Descripción de aplicación y hardware |
| `RP2040/catsniffer/CMakeLists.txt` | Fuentes compiladas, integración driver/versionado |
| `RP2040/catsniffer/prj.conf` | Kconfig: USB, UART, SPI, LoRa, NVS/settings |
| `RP2040/catsniffer/west.yml` | Zephyr fork `wero1414/zephyr`, rama `fsk-native-driver` |
| `RP2040/catsniffer/VERSION` | `0.2.0.0-dual-mode` |
| `RP2040/catsniffer/boards/rpi_pico.overlay` | UART, SPI/SX1262, GPIO, 3 CDC y storage |
| `RP2040/catsniffer/src/main.c` | Runtime: init, USB callbacks, UART, SX1262, hilo, buffers |
| `RP2040/catsniffer/src/USB/usbd_init.c` | Pila/dispositivo/descriptores USB |
| `RP2040/catsniffer/src/shell_commands.c` | Consola local y dispatch |
| `RP2040/catsniffer/src/fw_metadata.c` | Etiquetas NVS de firmware CC; no detecta físicamente la imagen |

Advertencia: el driver SX1262 se espera en `${ZEPHYR_BASE}/drivers/lora/native/sx126x`; no está vendorizado y el manifest no fija SHA. Los detalles internos del driver no son evidencia local reproducible de este commit.

## CC1352P7

### Registro de proyectos

| Ruta | Fuente/configuración | Artefactos versionados |
| --- | --- | --- |
| `CC1352P7/sniffer_fw_cc1252P_7/` | `.project`, `.cproject`; sin fuente actual | `sniffer_fw_Catsniffer_v3.x.hex`, SHA-256 `7C9FC088A946374E35113A82B32125CF825CA16ADAC1C2635E29A1A7109BA753` |
| `CC1352P7/airtag_scanner_CC1352P_7/` | fuente, `.project`, `.cproject`, `simple_central.syscfg`, `.ccxml` | `Release/airtag_scanner_CC1352P_7.hex`, SHA-256 `2A423E8265664EA4437A2D59936713AA697D0D7B065714B6CD42888CE35F4AEF` |
| `CC1352P7/airtag_spoofer_CC1352P_7/` | fuente, `.project`, `.cproject`, `simple_peripheral.syscfg`, `.ccxml` | `Release/airtag_spoofer_CC1352P_7.hex`, SHA-256 `A868580BEF8001196AB32A1C38667E6EA2C495FFF1C659B2AA2D05B005F2C917` |
| `CC1352P7/justworks_scanner_CC1352P7_1/` | fuente, `.project`, `.cproject`, `justworks_scanner.syscfg`, `.ccxml`, generated syscfg | `Release/justworks_scanner_CC1352P7_1.hex` SHA-256 `E6E4288A20C3000107739E2BF0E3A26A8D376D0362D60F337671120E30329042`; `.out` SHA-256 `FE83F8A2C040E57427B0CBC434A3B426315675CCC2ED63F9551FB3F4DE9CDB89` |

Los hashes identifican los archivos versionados; no prueban que estén instalados.

### Fuentes de ejecución CC usadas

- `CC1352P7/justworks_scanner_CC1352P7_1/Startup/main.c`
- `CC1352P7/justworks_scanner_CC1352P7_1/Application/justworks_scanner.c`
- `CC1352P7/justworks_scanner_CC1352P7_1/Release/syscfg/ti_drivers_config.{c,h}`
- `CC1352P7/justworks_scanner_CC1352P7_1/Release/syscfg/ti_ble_config.{c,h}`
- Fuentes `Application/`/`Startup/` equivalentes de Airtag, consultadas sólo para clasificación.

### Historia/procedencia

- `da6ff094…`, `2024-01-13`, `erase source code since violating TI license`: elimina la fuente del sniffer. La historia anterior contenía rutas P1; no se toma como implementación actual.
- `325fe42…`, `2024-10-15`, `remove old Sniffle firmware`: explica la ausencia del proyecto que aún menciona el README.
- El directorio `sniffer_fw_cc1252P_7` contiene un aparente typo `1252`; el target técnico en `.cproject` es `CC1352P7`.

## Fuentes externas oficiales TI

- `https://www.ti.com/product/CC1352P7` — clasificación P7 como MCU inalámbrico Cortex-M4F con Sub-1 GHz/2.4 GHz.
- `https://www.ti.com/lit/ds/symlink/cc1352p7.pdf` — datasheet; part number/marking `CC1352P74T0RGZR` / `CC1352 P74`.
- `https://www.ti.com/tool/LP-CC1352P7` — variantes LaunchPad `-1` y `-4`; contexto para la selección `LP_CC1352P7_1`.

Se usaron sólo para identidad/clasificación y contexto de target, no como sustituto del código CatSniffer.

## Fuentes hardware cruzadas

- `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch` y `.kicad_pcb`: cableado físico histórico; U8 allí dice P1.
- `CatSniffer-Firmware/RP2040/catsniffer/boards/rpi_pico.overlay`: asignaciones activas del firmware.
- [[Fuentes hardware CatSniffer v3.1]]: procedencia detallada.
- **Observado en hardware — 2026-09-25:** U8 de la unidad en uso marcado `CC1352P74`.

## Advertencias

1. La fuente actual no permite analizar internos de `sniffer_fw_Catsniffer_v3.x.hex`.
2. Los proyectos fuente son especializados; no se extrapolan al binario normal.
3. `LP_CC1352P7_1` es una configuración de LaunchPad, no evidencia de identidad completa con el front-end custom.
4. No se confirmó qué artefactos están instalados.
5. El repositorio contiene proyectos con SDK 7.10.01.24 y 8.31.00.11; no existe una única versión de SDK para todo el directorio.
6. No se descargó ninguna dependencia ni se compiló/flasheó.
