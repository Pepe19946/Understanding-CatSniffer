# Fuentes herramientas PC

**Conocimiento derivado:** [[Mapa de herramientas y comunicación PC]], [[Catnip CLI]], [[Protocolos host-dispositivo]] y [[Del PC a la radio - Rutas de extremo a extremo]].

## Línea base

| Fuente | Revisión | Uso |
|---|---|---|
| `CatSniffer-Tools/` | `main`, `bc80979d7348bb038c3dd5d40be8aa51b90dd653`, 2026-09-04 | fuente primaria de Fase 3 |
| Tag de herramientas | `v3.3.2.1` → `a06b7887a20693129ffb2ed567ed73284908d0a6` | release visible más próxima; HEAD está 8 commits después |
| `CatSniffer-Firmware/` | `v3.x`, `c0cd5a45e019dbd14ed11d039aacb13f300e5731` | contraste de descriptores, handlers y framing |
| Hardware | CatSniffer v3.1 física; U8 observado `CC1352P74` | plataforma fijada por el proyecto |

## Versión y empaquetado

- `CatSniffer-Tools/catnip/VERSION`: `3.3.2.1`.
- `CatSniffer-Tools/setup.py`: lee VERSION; entry point `catnip=catnip:main_cli`.
- `CatSniffer-Tools/catnip/_version.py`: resolución local/metadata.
- `CatSniffer-Tools/catnip/README.md`: documentación de uso; contiene versiones contradictorias.
- `CatSniffer-Tools/changelog.md`: V3.3.2 aún aparece “unreleased”.

## Fuentes Catnip principales

| Ruta relativa a `CatSniffer-Tools/` | Evidencia aportada |
|---|---|
| `catnip/catnip.py` | lanzador |
| `catnip/modules/core/cli.py` | Click, comandos, operaciones representativas, extcap |
| `catnip/modules/core/usb_connection.py` | VID/PID, agrupación, roles, wrappers seriales |
| `catnip/modules/core/catnip.py` | API/compatibilidad Bridge |
| `catnip/modules/core/bridge.py` | ciclos de captura TI/SX y configuración LoRa |
| `catnip/modules/core/pipes.py` | named pipes y Wireshark |
| `catnip/protocol/sniffer_ti.py` | trama y comandos TI |
| `catnip/protocol/sniffer_sx.py` | comandos LoRa y parser de líneas RX |
| `catnip/lora_extcap.py` | extcap LoRa y contradicciones actuales |
| `catnip/modules/firmware/flasher.py` | orquestación de flasheo CC |
| `catnip/modules/firmware/cc2538.py` | protocolo ROM CC26xx derivado de cc2538-bsl |
| `catnip/modules/firmware/fw_update.py` | update UF2 RP2040 |
| `catnip/modules/firmware/fw_metadata.py` | lectura/escritura metadata |
| `catnip/modules/firmware/fw_aliases.py` | aliases e IDs oficiales host |
| `catnip/modules/firmware/board.py` | detección P1/P7 y seguridad de imagen |
| `catnip/modules/firmware/restore.py` | recuperación Free-DAP/OpenOCD/JTAG |
| `catnip/modules/protocols/cativity/` | análisis 802.15.4 actual |
| `catnip/modules/protocols/meshtastic/` | configuración/decodificación sobre SX1262 |
| `catnip/modules/protocols/sx1262/spectrum.py` | utilidad SX1262 |
| `catnip/modules/protocols/vhci/` | puente HCI/BLE en Linux |

## TUI y legado

- `catsnifferTUI/README.md`, `VERSION`, `main.py`: alcance y entrada TUI.
- `catsnifferTUI/discovery.py`: descubrimiento independiente.
- `catsnifferTUI/device.py`: endpoints/comandos seriales.
- `catsnifferTUI/testbench.py`: smoke tests y acciones de flota.
- `Legacy/`: material histórico; no se halló import directo desde Catnip actual.
- `Legacy/cc2538-bsl/`: procedencia histórica relacionada con el módulo actual copiado/adaptado.

## Pruebas consultadas como evidencia de intención

- `catnip/tests/test_catsniffer.py`: mocks para flasher, configuración LoRa, bridges, descubrimiento y robustez.
- `catnip/tests/test_fw_modules.py`: aliases y metadata.
- `catnip/tests/test_board_support.py`: P1/P7 y tamaño de imagen.
- `catnip/tests/test_sx1262.py`: espectro SX1262.
- `catnip/tests/test_pipes.py`: pipes/Wireshark.

No se ejecutaron pruebas en esta fase. Las pruebas describen expectativas de software, no validan el dispositivo físico.

## Contraste firmware

- `CatSniffer-Firmware/RP2040/catsniffer/src/USB/usbd_init.c`: VID/PID, descriptores y CDC.
- `CatSniffer-Firmware/RP2040/catsniffer/prj.conf`: configuración USB CDC.
- `CatSniffer-Firmware/RP2040/catsniffer/src/main.c`: handlers CDC, bridge UART y recepción SX.
- `CatSniffer-Firmware/RP2040/catsniffer/src/shell_commands.c`: comandos, boot/reboot y configuración LoRa.
- `CatSniffer-Firmware/RP2040/catsniffer/src/fw_metadata.c`: IDs oficiales firmware.

## Dependencias externas observadas

- pyserial: enumeración y CDC.
- Click: CLI.
- Wireshark: consumidor opcional PCAP/named pipe.
- `sniffle_extcap`: captura BLE externa por Cat-Bridge.
- PuTTY: terminal opcional AirTag en Windows.
- OpenOCD/Free-DAP: recuperación excepcional CC.

No se descargó ni ejecutó ninguna dependencia externa.

## Advertencias de procedencia

- HEAD no está etiquetado aunque VERSION conserve `3.3.2.1`.
- README y changelog son evidencia documental potencialmente obsoleta.
- La imagen normal CC1352P7 es binaria en el baseline de firmware: el protocolo host TI se observa, pero no su implementación interna.
- Las relaciones “estructuralmente compatibles” no equivalen a validación con hardware.
- No se usó FeralRF para interpretar las herramientas oficiales.
