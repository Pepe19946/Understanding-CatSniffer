# Fase 1 — Inventario de presets existentes

Fecha de corte: 2026-10-23 (America/Mexico_City). Auditoría documental; no se abrieron puertos ni se ejecutó RF.

## Revisión auditada

| Elemento | Valor |
|---|---|
| Repositorio | `FeralRF/` |
| Rama | `feature/hardware-validation` |
| HEAD | `f2744e400f743df30499e20f03f156f08ad15bd8` |
| Fecha/asunto HEAD | 2026-10-22; `docs: Final technical documentation of Hardware Validation was added in docs.` |
| Estado | `origin/feature/hardware-validation`; no versionados: `docs/HARDWARE_VALIDATION_RESULTS/OTA-SUBGHZ-EXPERIMENT_REPORT_2026-10-12/` y `python/validation_artifacts/` |
| Paquete Python | `feralrf 0.3.0` (`python/pyproject.toml`) |
| Fuente CC1352P7 | Mismo commit FeralRF; target por defecto `DEVICE_VARIANT=CC1352P7` |
| TI SDK incorporado | submódulo `5b31d0a4903351e544546e23ef3330eaa4291ceb`, tag/descripción `lpf2-8.30.01.01` |
| RP2040 | No está versionado en FeralRF. Las campañas 15–22 oct reportan firmware instalado `v3.1.0.1`; no se preservó hash/UF2 verificable. |

El audit anterior usó `fbff1b7f5077964418fb63a3fdf346a27c35a003`. El diff documental y el informe de readiness del 22-oct indican que los presets, la API y el backend RF relevantes no cambiaron entre ese commit y HEAD; HEAD agregó documentación. Esto no identifica los binarios CC1352P7 instalados durante campañas anteriores.

## Autoridad y recuento

La autoridad del catálogo actual es `FeralRF/python/feralrf/presets.py`. Contiene **31 entradas únicas** en `PROP_PRESETS`. No hay presets BLE o IEEE 802.15.4 en ese diccionario: esas familias se seleccionan por `PHY`/canal. Todos los 31 presets usan la ruta propietaria.

Campos comunes no escritos en el diccionario:

- `format_conf=0` por default de `Radio.configure_prop()`; el firmware lo convierte en `0x00A0`: sync de 32 bits, MSB-first, sin FEC ni whitening.
- El packet path usa longitud variable, preámbulo/configuración SysConfig y CRC del RF core. El host aporta el payload; no aporta el preámbulo, sync, header/length o CRC físico.
- `deviation` es el entero almacenado. `RadioIF_setPropConfig()` lo copia al campo TI y no cambia `deviationStepSz`, que permanece 0. En SDK 8.30.01.01 el paso 0 equivale teóricamente a 250 Hz. La columna “dev teórica” sólo muestra esa interpretación; no es una medición.
- `rx_bw` se conserva como encoding de registro. No se convierte aquí a Hz.

## Inventario exacto

Todos usan PHY/API `PHY.PROPRIETARY_GFSK` → `SET_PROP_CONFIG` → backend `CMD_PROP_*`. `Fmt` es `0 → 0x00A0` para todas las filas.

| ID | Preset | Familia nominal | Hz | mod | baud | dev almacenada | dev teórica | `rx_bw` | sync | Estado de implementación |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| P01 | `gfsk_433_50k` | GFSK raw | 433920000 | 1 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | Presente; HW/ruta 433 no establecida |
| P02 | `gfsk_433_10k` | GFSK raw | 433920000 | 1 | 10000 | 50 | 12.5 kHz | `0x52` | `0x930B51DE` | Presente; HW/ruta 433 no establecida |
| P03 | `fsk_433_50k` | FSK raw | 433920000 | 0 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | Presente; HW/ruta 433 no establecida |
| P04 | `ook_433_4k8` | OOK/ASK raw | 433920000 | 2 | 4800 | 0 | 0 | `0x4C` | `0x930B51DE` | Condicional; patch OOK terminal + HW 433 incierto |
| P05 | `ook_433_2k4` | OOK/ASK raw | 433920000 | 2 | 2400 | 0 | 0 | `0x4C` | `0x930B51DE` | Condicional; patch OOK terminal + HW 433 incierto |
| P06 | `msk_433_50k` | MSK pretendido | 433920000 | 4 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | No soportado: `modType=4` reservado |
| P07 | `4fsk_433_50k` | 4-FSK pretendido | 433920000 | 5 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | No soportado: `modType=5` reservado |
| P08 | `4gfsk_433_50k` | 4-GFSK pretendido | 433920000 | 6 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | No soportado: `modType=6` reservado |
| P09 | `gfsk_868_50k` | GFSK raw | 868000000 | 1 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | Presente |
| P10 | `gfsk_868_100k` | GFSK raw | 868000000 | 1 | 100000 | 150 | 37.5 kHz | `0x56` | `0x930B51DE` | Presente |
| P11 | `ook_868_4k8` | OOK/ASK raw | 868000000 | 2 | 4800 | 0 | 0 | `0x4C` | `0x930B51DE` | Parcial/condicional; patch no descargable con seguridad |
| P12 | `msk_868_50k` | MSK pretendido | 868000000 | 4 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | No soportado: valor reservado |
| P13 | `wireless_mbus_s_868` | W-MBus S nominal | 868300000 | 1 | 32768 | 75 | 18.75 kHz | `0x52` | `0x543D0000` | PHY raw presente; no stack W-MBus |
| P14 | `wireless_mbus_t_868` | W-MBus T nominal | 868950000 | 1 | 100000 | 200 | 50 kHz | `0x57` | `0x543D0000` | PHY raw presente; no stack W-MBus |
| P15 | `wireless_mbus_c_868` | W-MBus C nominal | 868950000 | 1 | 100000 | 180 | 45 kHz | `0x57` | `0x543D0000` | PHY raw presente; no stack W-MBus |
| P16 | `wireless_mbus_n_169_2k4` | W-MBus N nominal | 169450000 | 1 | 2400 | 154 | 38.5 kHz | `0x44` | `0x543D0000` | Bloqueado en placa/FW actual; TI ofrece perfil/diseño 169 especial |
| P17 | `wireless_mbus_n_169_4k8` | W-MBus N nominal | 169450000 | 1 | 4800 | 154 | 38.5 kHz | `0x44` | `0x543D0000` | Bloqueado en placa/FW actual; TI ofrece perfil/diseño 169 especial |
| P18 | `4fsk_868_50k` | 4-FSK pretendido | 868000000 | 5 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | No soportado: valor reservado |
| P19 | `4gfsk_868_50k` | 4-GFSK pretendido | 868000000 | 6 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | No soportado: valor reservado |
| P20 | `gfsk_915_50k` | GFSK raw | 915000000 | 1 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | Presente |
| P21 | `gfsk_902_50k` | GFSK raw | 902200000 | 1 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | Presente |
| P22 | `gfsk_2440_250k` | GFSK propietario 2.4 | 2440000000 | 1 | 250000 | 320 | 80 kHz | `0x59` | `0x930B51DE` | Defectuoso: runtime cae a 868 MHz |
| P23 | `gfsk_2440_50k` | GFSK propietario 2.4 | 2440000000 | 1 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | Defectuoso: runtime cae a 868 MHz |
| P24 | `sidewalk_915_fsk_50k` | Sidewalk FSK nominal | 915000000 | 0 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | PHY raw presente; no Sidewalk |
| P25 | `sidewalk_915_fsk_250k` | Sidewalk FSK nominal | 915000000 | 0 | 250000 | 200 | 50 kHz | `0x5A` | `0x930B51DE` | PHY raw presente; no Sidewalk |
| P26 | `wisun_915_fsk_50k` | Wi-SUN FAN nominal | 902200000 | 0 | 50000 | 100 | 25 kHz | `0x52` | `0x930B51DE` | PHY raw presente; no Wi-SUN |
| P27 | `mioty_868_tsunb` | MIOTY TS-UNB nominal | 868000000 | 0 | 396 | 1 | 250 Hz | `0x4A` | `0x930B51DE` | FSK genérico; TS-UNB ausente |
| P28 | `wisun_915_fsk_100k` | Wi-SUN FAN nominal | 902200000 | 0 | 100000 | 150 | 37.5 kHz | `0x56` | `0x930B51DE` | PHY raw presente; no Wi-SUN |
| P29 | `wisun_915_fsk_150k` | Wi-SUN FAN nominal | 902200000 | 0 | 150000 | 200 | 50 kHz | `0x5A` | `0x930B51DE` | PHY raw presente; no Wi-SUN |
| P30 | `wisun_915_fsk_200k` | Wi-SUN FAN nominal | 902200000 | 0 | 200000 | 200 | 50 kHz | `0x5A` | `0x930B51DE` | PHY raw presente; no Wi-SUN |
| P31 | `wisun_915_fsk_300k` | Wi-SUN FAN nominal | 902200000 | 0 | 300000 | 200 | 50 kHz | `0x5E` | `0x930B51DE` | PHY raw presente; no Wi-SUN |

## Semántica de nombres

- `50k`, `100k`, `150k`, `200k`, `250k`, `300k`, `10k`, `4k8` y `2k4` coinciden con `symbol_rate` de 50 000, 100 000, 150 000, 200 000, 250 000, 300 000, 10 000, 4 800 y 2 400 símbolos/s. No expresan throughput de aplicación: preámbulo, sync, header/length y CRC añaden overhead.
- `tsunb` es una etiqueta nominal TS-UNB, no una garantía. La entrada sólo configura FSK genérico a 396 baud; faltan splitting, coding/interleaving, scheduling temporal/frecuencial, reensamble y peer MIOTY.
- `wisun_915_*` es especialmente engañoso respecto de frecuencia: las cinco entradas solicitan **902.2 MHz**, no 915 MHz.
- `wireless_mbus_*`, `sidewalk_*` y `wisun_*` describen intención/familia. No construyen tramas ni pilas de esos protocolos.

## Traza de implementación

1. PC/CPython ejecuta `Radio.configure_prop()` en `python/feralrf/radio.py`.
2. `CommandBuilder.set_prop_config()` en `python/feralrf/commands.py` serializa 18 bytes `<IBIHBIH>`.
3. USB Cat-Bridge/RP2040 transporta el frame al UART del CC1352P7. Su fuente no está en este repositorio; la revisión instalada sólo se conoce por `v3.1.0.1` reportado.
4. Cortex-M4F CC1352P7 ejecuta `CMD_SET_PROP_CONFIG` en `firmware/cc1352/src/command_processor.c`, deserializa y llama `RadioIF_setPropConfig()`; luego envía ACK incondicional sin readback del RF core.
5. `firmware/cc1352/src/radio_if.c` muta `CMD_PROP_RADIO_DIV_SETUP`, `CMD_FS`, `CMD_PROP_RX/TX` y overrides TI. La aplicación efectiva ocurre después, al iniciar RX o procesar TX.
6. El RF core ejecuta los comandos/patches del SDK 8.30.01.01.

`SET_PHY` sigue `Radio.set_phy()` → `CommandBuilder.set_phy()` → `CMD_SET_PHY` → `ControlTask_onSetPhy()` → `RadioIF_setPhy()`. `SET_CHANNEL` sigue `Radio.set_channel()` → `CMD_SET_CHANNEL` → `ControlTask_onSetChannel()` → `RadioIF_setChannel()`. Para estos presets, los scripts seleccionan `PHY.PROPRIETARY_GFSK`; no existe un backend diferente por nombre de protocolo.

## Restricciones confirmadas

- SDK `rf_prop_cmd.h`, estructura `CMD_PROP_RADIO_DIV_SETUP`: `modType` válido 0=FSK, 1=GFSK, 2=OOK; “Others: Reserved”. Por tanto 4/5/6 no acreditan MSK/4-FSK/4-GFSK aunque el host y parser los acepten.
- Proprietary 2.4: `RadioIF_setPropConfig()` escribe inicialmente 2440 MHz, pero `RadioIF_applySub1ghzChannelConfig()` sólo respeta `frequency_hz < 1000 MHz`; con PHY propietario cae al fallback 868 MHz antes de RX/TX. P22/P23 quedan bloqueados.
- 169 MHz: selecciona el modo/handle 433; los overrides 169 se escriben en una estructura distinta de la usada por la ruta 433. TI publica un setting y RF design 169 especiales para CC1352P7, pero el frontend CatSniffer y esta ruta no los demuestran. P16/P17 quedan bloqueados en la implementación/placa actuales, no universalmente en el silicio.
- OOK carga patches `genook`; `RFCCpePatchReset` no permite descargarlos de forma segura. Cambiar modo/frecuencia requiere retirar alimentación. El ACK no prueba OOK física.
- 433 MHz se inventaría pero se excluye de la campaña OTA actual. La revisión de placa/front-end/antena disponible no está establecida de forma suficiente.
- Los 14 FSK/GFSK de 868–915/902.2 se consideran implementaciones raw presentes. Las autorizaciones regulatorias, potencia y antena siguen siendo prerrequisitos experimentales, no propiedades demostradas por source.

## Nivel de certeza

El inventario y los defectos de contrato son hallazgos de fuente estática. La frecuencia, modulación, desviación, ancho ocupado, potencia y posición eléctrica del switch no se han medido independientemente. Un ACK de `SET_PROP_CONFIG` demuestra aceptación/serialización, no aplicación física.

## Corrección taxonómica 2026-10-23

La columna histórica “Familia” de este inventario sólo describe la **intención nominal del nombre**. La autoridad para tecnología, perfil, modulación solicitada, soporte del comando, backend, framing, pila y elegibilidad separada es la matriz corregida `03 - Preset Validation Coverage Matrix.md`.

Para las 31 entradas, el backend real es propietario TI y el framing efectivo común es longitud variable + CRC con sync de 32 bits; el host entrega payload y el RF Core construye preámbulo/sync/longitud/CRC. Los nombres W‑M‑Bus, Wi‑SUN, Sidewalk y MIOTY no seleccionan otra ruta ni una pila.

La frase anterior “169 MHz no soportado por el CC1352P7” queda **superseded**: TI documenta una configuración 169 MHz N‑mode y un RF design especial para CC1352P7. Lo no demostrado es esa ruta en este firmware y en el frontend CatSniffer auditado. P16/P17 continúan no elegibles para validación en la placa/firmware actuales, pero no por una supuesta imposibilidad universal del silicio.

## Changelog

- 2026-10-23: se separó la intención del nombre del backend/framing efectivos y se corrigió el alcance de 169 MHz.
