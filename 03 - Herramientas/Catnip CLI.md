# Catnip — La herramienta que coordina desde la PC

## En una frase

Catnip es una aplicación de línea de comandos que corre en la PC, descubre CatSniffer y coordina control, captura y actualización mediante sus distintas interfaces USB.

## Modelo mental

Una **CLI** es una interfaz de línea de comandos: transforma opciones escritas por el usuario en llamadas de software. Catnip está por encima de [[USB CDC - Cómo CatSniffer aparece ante la PC|los tres CDC]]; no es uno de ellos y no corre en RP2040. Su trabajo es elegir el camino, formar el mensaje apropiado y presentar la respuesta.

> [!important] Límite de responsabilidad
> Catnip puede construir bytes destinados al CC1352P7, pero eso no significa que RP2040 interprete su semántica. En Cat-Bridge, el intérprete normal está al otro extremo del UART.

## 1. Versión y baseline

- Repositorio: `CatSniffer-Tools/`
- Rama/HEAD: `main` / `bc80979d7348bb038c3dd5d40be8aa51b90dd653`
- Fecha del commit: `2026-09-04T13:33:26-06:00`
- Descripción Git: `v3.3.2.1-8-gbc80979`; HEAD no tiene tag.
- Tag visible más cercano: `v3.3.2.1` → `a06b7887a20693129ffb2ed567ed73284908d0a6`.
- Versión consumida por el paquete: `catnip/VERSION` = `3.3.2.1`; `setup.py` la lee y `_version.py` prefiere ese archivo.

**Contradicción documental:** `catnip/README.md` aún anuncia “Current Version: v3.0.0” en una sección y 3.3.2.1 en otra; `changelog.md` llama V3.3.2 “unreleased”. La evidencia Git indica desarrollo ocho commits posterior al tag, con el archivo VERSION aún en 3.3.2.1. No se reconstruyó una historia de releases más allá de esta clasificación.

## 2. Entrada y estructura

`setup.py` registra `catnip=catnip:main_cli`. `catnip/catnip.py` es un lanzador delgado; la aplicación Click vive en `modules/core/cli.py:main_cli()`/`cli()`.

| Área | Ruta | Responsabilidad |
|---|---|---|
| CLI | `modules/core/cli.py` | comandos, validación y despacho |
| USB/serial | `modules/core/usb_connection.py` | descubrir, agrupar y abrir Bridge/LoRa/Shell |
| Captura | `modules/core/bridge.py`, `modules/core/pipes.py` | ciclos de captura, PCAP, pipes y Wireshark |
| Protocolos radio | `protocol/sniffer_ti.py`, `protocol/sniffer_sx.py` | trama TI y líneas/PCAP SX1262 |
| Firmware | `modules/firmware/` | selección, metadatos, flasheo CC, UF2 RP y restore |
| Protocolos de nivel superior | `modules/protocols/` | Cativity, Meshtastic, espectro SX1262 y VHCI |
| Pruebas | `tests/` | expectativas unitarias/mock; no validación física |

`modules/core/catnip.py:Catnip` hereda de `BridgeConnection`; conserva una API de compatibilidad y no representa todo el dispositivo de tres puertos.

## 3. Descubrimiento

`usb_connection.py:find_devices()` enumera pyserial, filtra `0x1209:0xBABB`, agrupa por serie/ubicación y llama `_map_roles()`.

- Prioriza palabras de rol en `description` e `interface`.
- Interpreta interfaces USB de comunicación `0`, `2`, `4` como Bridge, LoRa y Shell.
- Usa orden del nombre de puerto solo como fallback.
- Exige los tres roles antes de crear `CatSnifferDevice`.
- `find_device(None)` devuelve el primer equipo; un entero selecciona su ID efímero del inventario ordenado.

El regex de serie del HWID acepta solo dígitos hexadecimales. Una serie con otros caracteres dependería de `serial_number` o ubicación.

## 4. Abstracciones de puertos

### `BridgeConnection`

Abre pyserial como flujo binario y desactiva DTR/RTS. No añade framing. El valor de línea CDC predeterminado en los wrappers es 115200; no debe confundirse con la UART física RP2040↔CC, que el firmware fija en 921600 durante runtime y 500000 durante boot. En runtime, cualquier trama superior pertenece a la herramienta de host y a la imagen del CC.

### `LoRaConnection`

Abre Cat-LoRa, activa DTR y ofrece lectura/escritura de flujo. En modo stream, las líneas RP→host son resultados RX y los bytes host→RP son datos a transmitir por RF. No debe usarse un keepalive arbitrario.

### `ShellConnection`

`send_command()` limpia buffers, escribe ASCII `comando + "\r\n"` y acumula la respuesta hasta 150 ms de silencio dentro del timeout global. Decodifica ASCII ignorando bytes inválidos y hace `strip()`. No espera un prompt ni longitud explícita.

## 5. Cat-Shell

Comandos relevantes usados por Catnip:

| Función host | Texto | Handler/efecto RP2040 |
|---|---|---|
| `send_identify_command()` | `identify\r\n` | `cmd_identify()`: texto y LEDs |
| `get_device_fw_version()` | `fw_version\r\n` | versión, Git y placa |
| actualización RP | `reboot\r\n` | ROM BOOTSEL RP2040 |
| flasheo CC | `boot\r\n`, `exit\r\n` | boot/reset CC y UART 500000/921600 |
| metadatos CC | `cc1352_fw_id get/set ...\r\n` | metadata local RP2040 |
| `_configure_lora()` | `lora_freq`, `lora_bw`, `lora_sf`, `lora_cr`, `lora_power`, `lora_syncword`, `lora_apply`, `lora_mode` | configura la API LoRa/SX1262 |

`parse_fw_version_response()` interpreta líneas `Clave: Valor`. Una respuesta ausente/malformada produce `None`; no hay framing binario adicional.

## 6. Cat-Bridge

### Runtime TI

`bridge.py:run_bridge()` usa `SnifferTI` para emitir:

- SOF `40 53` (`@S`);
- command ID;
- longitud de datos de 16 bits little-endian;
- datos;
- FCS de un byte, suma de command + bytes de longitud + datos, módulo 256;
- EOF `40 45` (`@E`).

Los IDs observados incluyen PING `0x40`, START `0x41`, STOP `0x42`, PAUSE `0x43`, RESUME `0x44`, CFG_FREQ `0x45` y CFG_PHY `0x47`. La secuencia de captura hace ping/stop, configura PHY/frecuencia e inicia.

El RP2040 mueve los bytes CDC0↔UART. `SnifferTI.Packet` interpreta en host las respuestas/capturas. La imagen normal CC1352P7 es binaria; su parser interno no puede trazarse desde la fuente disponible.

### Otros consumidores

- `sniff_ble()` verifica/instala Sniffle y `run_extcap_directly()` lanza el paquete externo `sniffle_extcap` contra el puerto Bridge.
- `sniff_airtag_scanner()` usa una imagen especializada y un terminal/PuTTY para texto en Bridge.
- Cativity consume la captura 802.15.4 actual, no una interfaz USB adicional.

## 7. Cat-LoRa

`bridge.py:run_sx_bridge()` abre Shell y LoRa. `_configure_lora()` manda comandos por Shell y después cambia a `lora_mode stream`. La recepción llega por LoRa como:

`LORA RX: <hex> | RSSI: <entero> | SNR: <entero>`

`SnifferSx.Packet` también reconoce la variante FSK, quita una posible marca `...`, convierte el hex y crea PCAP linktype 148. En firmware, `lora_rx_cb()` imprime un máximo de 40 bytes; la captura no conserva payloads mayores completos.

En modo comando, Cat-LoRa acepta órdenes RP locales usadas por verificación, espectro o la TUI. En modo stream, escribir bytes equivale a solicitar transmisión RF. `run_sx_bridge()` evita deliberadamente keepalive.

## 8. Captura y salida

`PacketLogWriter` y `pipes.py` separan el parser de la salida. La captura TI puede ir a PCAP, log ASCII/raw o pipe de Wireshark. La captura SX genera PCAP 148, terminal/log o Wireshark. BLE delega a `sniffle_extcap`; por ello Catnip coordina firmware/puerto/proceso, pero no interpreta el protocolo interno Sniffle.

No todas las radios comparten arquitectura: CC entrega datos por Bridge; SX1262 requiere configuración por Shell y datos por LoRa.

## 9. Actualización de firmware

### RP2040 (`modules/firmware/fw_update.py`)

1. `check_and_update_rp2040()` obtiene versiones y selecciona UF2.
2. `enter_boot_mode()` manda `reboot` por Shell.
3. `find_rp2040_mount_point()` espera `RPI-RP2` hasta aproximadamente 15 s.
4. `flash_rp2040_uf2()` copia el archivo con `shutil.copy2`.
5. Si no hay CDC, presenta recuperación BOOTSEL manual.

No se observó validación posterior obligatoria de la versión ya arrancada.

### CC1352P7 (`modules/firmware/flasher.py`, `cc2538.py`)

1. `flash()` y `Flasher.find_flash_firmware()` seleccionan/validan imagen y placa.
2. `CCLoader` abre Bridge a 500000 y manda `boot` por Shell.
3. RP2040 pone CC en bootloader y cambia UART de 921600 a 500000.
4. La implementación derivada de cc2538-bsl sincroniza ROM, consulta chip, protege variante/capacidad, borra, escribe y verifica CRC.
5. `exit` vuelve al runtime y Catnip registra metadata.

Los ficheros bajo `Legacy/cc2538-bsl/` no se importan; la derivación activa está copiada/adaptada en el paquete actual.

## 10. Manejo de errores y ciclo de vida

- Dispositivo ausente: la CLI informa y termina; `devices --debug` muestra enumeración cruda.
- Puerto incompleto: `find_devices()` lo descarta antes de construir el dispositivo; diagnóstico limitado.
- Apertura serial: wrappers devuelven `False`/`None` y Shell absorbe excepciones.
- Timeout Shell: silencio de 150 ms termina una respuesta; timeout total impide espera infinita.
- Respuesta malformada: parsers de versión/metadata retornan fallo; no siempre distinguen timeout de formato.
- Desconexión durante captura: no existe una política uniforme de reconexión; una excepción serial puede abandonar el ciclo.
- Varios dispositivos: IDs derivados del inventario; el primero es default y el ID puede cambiar con la enumeración.
- Update: se verifican descargas/checksums cuando aplica, pero la copia UF2 no confirma una nueva versión después del reboot.

Estas son observaciones arquitectónicas, no una auditoría de confiabilidad.

## 11. Funciones y clases clave

- `modules/core/cli.py`: `main_cli`, `get_device_or_exit`, `sniff_zigbee`, `sniff_thread`, `sniff_lora`, `sniff_ble`, `identify`, `flash`.
- `modules/core/usb_connection.py`: `find_devices`, `_group_ports_by_device`, `_map_roles`, `CatSnifferDevice`, `BridgeConnection`, `LoRaConnection`, `ShellConnection`.
- `modules/core/bridge.py`: `run_bridge`, `run_sx_bridge`, `_configure_lora`, `_stop_lora_capture`.
- `protocol/sniffer_ti.py`: `TIBaseCommand`, `SnifferTI`.
- `protocol/sniffer_sx.py`: `LoRaShellCommands`, `SnifferSx`.
- `modules/firmware/flasher.py`: `CCLoader`, `Flasher`.
- `modules/firmware/fw_update.py`: `check_and_update_rp2040`, `enter_boot_mode`, `flash_rp2040_uf2`.

## 12. Contradicciones y potenciales problemas

1. `lora_extcap.py` llama `LoRaShellCommands.apply()`, pero la clase expone `apply_config()`: la ruta extcap no es ejecutable tal como está, por inspección estática.
2. El keepalive `00` de `lora_extcap.py` contradice el modo stream actual, donde todo byte entrante por CDC1 se transmite por RF.
3. IDs “oficiales”: host `catnip_v3` frente a firmware `catsniffer_v3`; host añade `justworks_scanner_cc1352p7` que el firmware no lista como oficial.
4. La captura SX trunca payloads mayores de 40 bytes desde firmware.
5. README/changelog/VERSION/Git no expresan una única versión de desarrollo coherente.
6. La compatibilidad TI interna y la conducta física siguen sin prueba.

## 13. Preguntas abiertas

- **No bloqueante para Fase 4:** ¿qué SO y versión de Wireshark serán el entorno final? Afecta pipes, extcap y nombres de puertos.
- **Requiere hardware:** ¿los tres puertos conservan etiquetas/serie estables en los SO objetivo y con varios CatSniffer conectados?
- **Requiere hardware:** ¿update y flash recuperan/re-enumeran correctamente la placa v3.1 concreta?
- **Requiere hardware:** ¿qué imágenes RP2040/CC están instaladas actualmente?

## 14. Evidencia

Toda ruta de esta nota se leyó en `CatSniffer-Tools@bc80979d7348bb038c3dd5d40be8aa51b90dd653`. El contraste firmware corresponde a `CatSniffer-Firmware@c0cd5a45e019dbd14ed11d039aacb13f300e5731`, principalmente `RP2040/catsniffer/src/main.c`, `shell_commands.c`, `fw_metadata.c` y `src/USB/usbd_init.c`.

## Concepciones erróneas comunes

- **“Catnip es el firmware de CatSniffer.”** No: es software host.
- **“Un comando Catnip siempre lo ejecuta RP2040.”** No: Catnip puede usar RP2040 como puente hacia CC1352P7.
- **“Catnip y Cat-Shell son lo mismo.”** No: Catnip corre en PC; Cat-Shell interpreta texto dentro del RP2040.
- **“Catnip controla directamente SX1262.”** Lo hace indirectamente mediante firmware RP2040 y sus rutas Shell/LoRa.

## Comprueba tu comprensión

1. ¿Qué datos usa Catnip para agrupar los tres CDC de una placa?
2. ¿Qué parte de una captura TI interpreta Catnip y qué parte queda dentro del CC?
3. ¿Por qué Catnip usa más de una interfaz para operaciones SX1262?
4. ¿Qué cambia cuando Catnip pasa de runtime a programación del CC?

**Anterior:** [[Mapa de herramientas y comunicación PC]]  
**Siguiente:** [[Del PC a la radio - Rutas de extremo a extremo]]
