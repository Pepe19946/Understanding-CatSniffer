# Cat-Bridge, Cat-LoRa y Cat-Shell — Tres caminos distintos

## En una frase

Los tres CDC comparten el enlace USB y el RP2040, pero difieren en framing, intérprete y destino final.

## Modelo mental

Una **trama** (*frame*) es una forma de delimitar y estructurar bytes para que el receptor sepa dónde empieza y termina un mensaje. No todos los CDC usan la misma: Shell es texto por líneas, Bridge puede ser passthrough binario y LoRa cambia según modo. Consulta primero [[USB CDC - Cómo CatSniffer aparece ante la PC]].

## Alcance

Esta nota describe exactamente qué transporte y framing existe en cada CDC para `CatSniffer-Tools@bc80979d7348bb038c3dd5d40be8aa51b90dd653`, contrastado con `CatSniffer-Firmware@c0cd5a45e019dbd14ed11d039aacb13f300e5731`. No supone que una prueba unitaria valide hardware.

## Cat-Shell

- **Transporte:** USB CDC ACM, interfaz de comunicación USB 4 / tercer puerto CDC; `ShellConnection` usa pyserial.
- **Framing host→dispositivo:** texto ASCII, un comando por línea, terminador `\r\n`.
- **Intérprete:** RP2040, `cdc2_interrupt_handler()` → `process_command()` y handlers de `RP2040/catsniffer/src/shell_commands.c`.
- **Respuesta:** texto/eco con CRLF. Catnip acumula bytes hasta 150 ms de silencio o timeout general, no hasta prompt/longitud.
- **Semántica:** control local, información, modos del puente, boot/reset, metadatos y configuración SX1262.

Ejemplos directamente sustentados:

```text
identify\r\n
fw_version\r\n
boot\r\n
exit\r\n
lora_freq <valor>\r\n
lora_apply\r\n
lora_mode stream\r\n
```

`ShellConnection.send_command()` restablece buffers antes de enviar. La falta de respuesta, un error serial o una decodificación imposible se reduce con frecuencia a `None`/cadena vacía; el llamador aporta la semántica de error.

## Cat-Bridge

- **Transporte:** USB CDC ACM, interfaz de comunicación USB 0 / primer CDC, enlazada por RP2040 a UART del CC1352P7. Catnip abre normalmente el endpoint CDC con line coding 115200; la UART física interna es independiente: 921600 en runtime y 500000 en boot.
- **Framing RP2040:** ninguno; passthrough byte a byte en ambos sentidos.
- **Intérprete final:** la imagen activa en CC1352P7 o su bootloader ROM. El RP2040 no interpreta el tráfico normal.
- **Respuesta:** bytes CC→UART→CDC0. El consumidor del PC decide el framing.

### Sniffer TI en runtime

`protocol/sniffer_ti.py` define:

```text
40 53 | CMD | LEN_LO LEN_HI | DATA... | FCS | 40 45
 @  S                                      @  E
```

`FCS = (CMD + LEN_LO + LEN_HI + sum(DATA)) & 0xff`.

Comandos observados:

| Nombre | ID |
|---|---:|
| PING | `0x40` |
| START | `0x41` |
| STOP | `0x42` |
| PAUSE | `0x43` |
| RESUME | `0x44` |
| CFG_FREQ | `0x45` |
| CFG_PHY | `0x47` |

`run_bridge()` transmite configuración y busca límites EOF/SOF al reconstruir paquetes. `SnifferTI.Packet` interpreta categorías y contenido en el PC. La imagen normal CC no tiene fuente disponible: solo se observa el lado host del contrato.

### Sniffle/BLE y terminales

Para Sniffle, Catnip entrega el puerto a `sniffle_extcap`; el protocolo binario interno pertenece a esa dependencia y al firmware Sniffle. Para AirTag scanner, el flujo se consume como texto en un terminal. Esto confirma que Cat-Bridge es un conducto multipropósito, no una trama única.

### Bootloader CC

En programación, Catnip abre Bridge a 500000 después de `boot` por Shell. `cc2538.py` implementa el protocolo del bootloader ROM CC26xx para sincronizar, borrar, escribir y verificar. Al salir, la UART interna vuelve a 921600 para runtime.

## Cat-LoRa

- **Transporte:** USB CDC ACM, interfaz de comunicación USB 2 / segundo CDC.
- **Intérprete:** RP2040; este controla SX1262 por API LoRa de Zephyr/SPI/GPIO.
- **Modos:** comando y stream.

### Modo comando

Acepta líneas de control interpretadas por `cdc1_interrupt_handler()`/lógica RP2040. La TUI, verificación y utilidades SX1262 lo usan para pruebas/operaciones específicas. No llega directamente un comando serial al SX1262.

### Modo stream, host→radio

Los bytes escritos por el host se convierten en payload de transmisión RF a través de RP2040/SX1262. No hay keepalive inocuo definido; por eso el bridge actual no escribe bytes periódicos.

### Modo stream, radio→host

Formato LoRa observado:

```text
LORA RX: <payload hexadecimal> | RSSI: <entero> | SNR: <entero>\r\n
```

Hay una variante FSK equivalente reconocida por `SnifferSx.Packet`. El host parsea hex y métricas, acepta `...` y empaqueta PCAP linktype 148.

**Limitación:** `lora_rx_cb()` en firmware imprime un máximo de 40 bytes y agrega `...`; el PC no puede reconstruir lo omitido.

### Configuración LoRa: canal de control separado

Aunque los datos de captura usan Cat-LoRa, `run_sx_bridge()` configura SX1262 mediante comandos `lora_*` enviados por Cat-Shell. Por tanto, la ruta completa tiene dos CDC:

```text
Cat-Shell: configuración → RP2040 → SX1262
Cat-LoRa:  recepción SX1262 → RP2040 → PC
```

## Propiedad semántica resumida

| CDC | Transporte | Framing visible | Quién interpreta | Respuesta |
|---|---|---|---|---|
| Cat-Shell | USB CDC | ASCII + CRLF | RP2040 | texto hasta silencio |
| Cat-Bridge | USB CDC ↔ UART | ninguno en RP; depende de imagen (p. ej. TI `@S…@E`) | CC1352P7/bootloader o herramienta externa | bytes transparentes |
| Cat-LoRa | USB CDC | líneas ASCII RX; bytes TX; comandos de línea en modo comando | RP2040; SX1262 es periférico | líneas/estado generado por RP2040 |

## Cruces y contradicciones

- Host y firmware concuerdan en VID/PID, tres CDC, etiquetas, CRLF Shell y velocidades CC 921600/500000.
- `lora_extcap.py` espera `LoRaShellCommands.apply()` aunque solo existe `apply_config()`.
- Su keepalive `00` entra en conflicto con la semántica TX de CDC1 en stream.
- La metadata oficial discrepa (`catnip_v3` host / `catsniffer_v3` firmware; `justworks_scanner...` solo host).
- No se puede confirmar la aceptación real de tramas TI sin hardware ni fuente del sniffer CC normal.

## Evidencia

- `CatSniffer-Tools/catnip/modules/core/usb_connection.py`: conexiones y Shell framing.
- `CatSniffer-Tools/catnip/modules/core/bridge.py`: configuración y ciclos TI/SX.
- `CatSniffer-Tools/catnip/protocol/sniffer_ti.py`: trama/IDs TI.
- `CatSniffer-Tools/catnip/protocol/sniffer_sx.py`: comandos Shell y parser RX.
- `CatSniffer-Tools/catnip/lora_extcap.py`: ruta extcap contradictoria.
- `CatSniffer-Tools/catnip/modules/firmware/{flasher.py,cc2538.py}`: bootloader CC.
- `CatSniffer-Firmware/RP2040/catsniffer/src/{main.c,shell_commands.c}`: CDC handlers, UART y callbacks LoRa.

## Por qué esto importa después

“USB serie” solo identifica el transporte visible por la PC; no identifica quién comprende el contenido. Distinguir transporte, framing e intérprete permite seguir una orden sin saltar directamente de Catnip al radio.

## Comprueba tu comprensión

1. ¿Por qué Cat-Bridge puede transportar tráfico TI, bootloader o FeralRF?
2. ¿Quién interpreta CRLF en Cat-Shell?
3. ¿Qué diferencia existe entre configurar SX1262 y recibir su stream?
4. ¿Por qué una ausencia de respuesta puede pertenecer a capas distintas?

**Anterior:** [[USB CDC - Cómo CatSniffer aparece ante la PC]]  
**Siguiente:** [[Mapa de herramientas y comunicación PC]]
