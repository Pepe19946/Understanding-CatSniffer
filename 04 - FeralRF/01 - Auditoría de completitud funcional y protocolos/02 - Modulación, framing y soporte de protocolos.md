# Modulación, *framing* y soporte de protocolos

## Taxonomía operativa

En esta auditoría los términos tienen significados no intercambiables:

| Nivel | Qué demuestra | Qué no demuestra |
|---|---|---|
| Configuración RF | Frecuencia, potencia, tasa, desviación o BW solicitados | Que la combinación sea soportada o medida en aire |
| Modulación/PHY | Comando, patch y overrides producen una señal definida | Campos MAC, estado o interoperabilidad |
| *Framing* físico | Delimitación: preámbulo, sync, longitud, CRC/FCS, whitening | Semántica de campos de protocolo |
| Trama de protocolo | Construcción/parseo de cabeceras y campos | Procedimientos de sesión o red |
| Procedimientos | ACK, temporización, asociación, hopping, retransmisión, seguridad | Pila completa si faltan otras capas |
| Pila completa | Todas las capas necesarias para un rol expresamente definido | Certificación o interoperabilidad, si no se ensayaron |
| Interoperabilidad | Comunicación con implementación independiente y criterio verificable | Conformidad universal fuera del caso ensayado |

## Proprietary: paquete que realmente sale al aire

La configuración base de `smartrf_prop_0.c` usa preámbulo de cuatro bytes, sincronía de 32 bits, orden MSB-first, sin FEC, sin *whitening*, longitud variable y CRC habilitado. En términos funcionales:

```text
RF core: PREÁMBULO(4 B) | SYNC(32 b) | LONGITUD | HOST PAYLOAD | CRC
                                                  ^ bytes de transmit()
```

En RX, el RF core valida CRC y entrega cabecera/longitud y datos; el firmware añade/extrae metadatos RSSI, correlación/CRC y timestamp. El error F-01 está precisamente en el orden de RSSI y estado/LQI al interpretar la cola PROP.

Consecuencias:

- Un payload PT2262, Wireless M-Bus o MIOTY enviado por esta ruta no conserva automáticamente el *framing* físico exigido por esos protocolos.
- `sync_word` es configurable, pero el preámbulo permanece fijo en la API pública.
- `format_conf` expone en bruto opciones del comando (sync, orden, FEC/*whitening*), sin tipos ni validación semántica.
- La selección de CRC y su formato no forman parte de una API protocolaria.

### Contrato de modulación exacto

El `CMD_PROP_RADIO_DIV_SETUP` del SDK local 8.30.01.01 define `modulation.modType` así: `0=FSK`, `1=GFSK`, `2=OOK`; los demás valores están reservados. FeralRF copia `mod_type` directamente a ese campo. No aparece un comando alterno, patch CPE/MCE ni ruta distinta que convierta 4, 5 o 6 en MSK, 4-FSK o 4-GFSK.

Por tanto:

- Los presets 0/1/2 designan modos representables por el comando activo.
- Los nombres MSK/4-FSK/4-GFSK con 4/5/6 son aceptados por Python y firmware, pero no establecen el modo anunciado.
- Aceptación del ACK no es prueba de modulación física.
- No corresponde proponer un remapeo numérico sin seleccionar primero el comando/patch/PHY TI que implemente la modulación deseada.

## IEEE 802.15.4

### División de responsabilidades

En `smartrf_ieee_15_4_0.c`, `txOpt.bIncludePhyHdr=0` y `txOpt.bIncludeCrc=0`. Conforme a `rf_ieee_cmd.h`, cero indica que el RF core obtiene/genera el PHR y calcula el FCS. El host debe entregar el PSDU/MAC sin FCS:

```text
RF core: PREÁMBULO | SFD | PHR | MHR + MAC PAYLOAD | FCS
                                  ^ bytes del host ^
```

En recepción se incluyen PHY header y CRC en la entrada del RF core; el parser usa la longitud PHR y entrega los bytes asociados. El resultado no debe interpretarse sin considerar que la longitud PSDU de IEEE incluye FCS.

### MAC realmente disponible

- La ruta genérica transporta bytes MAC opacos; no construye FCF, secuencia o direcciones.
- `ieee154_device.py` sí construye dos cabeceras MAC fijas plausibles (baliza y datos).
- El filtro de tramas y `autoAckEn` están desactivados.
- La trama de datos fija solicita ACK en el FCF, pero el emulador no abre una ventana ACK, correlaciona secuencia ni reintenta.
- No hay asociación, escaneo MAC, supertrama, tablas de vecinos, seguridad 802.15.4 ni estado de dispositivo.

FeralRF es por tanto un **PHY/transportador de tramas MAC IEEE 802.15.4 con constructores de ejemplo**, no una implementación MAC completa.

### IEEE no implica Zigbee ni Thread

Zigbee y Thread usan IEEE 802.15.4 como base, pero requieren capas y procedimientos adicionales (red, seguridad, descubrimiento, commissioning y, para Thread, 6LoWPAN/IPv6 y funciones del perfil). Nada de ello se deduce del TX/RX IEEE actual.

## Bluetooth Low Energy

El nombre “raw” del método interno no significa que el host proporcione una trama Link Layer arbitraria. `RadioIF_transmitBleAdvRaw()` recibe datos publicitarios y configura comandos de advertising TI:

- BLE 1M: advertising no conectable mediante comando BLE4.
- BLE Coded: advertising no conectable mediante comando BLE5.
- BLE 2M: `ADV_EXT_IND` en canal primario 37 y `AUX_ADV_IND` en canal secundario fijo 9.

El RF core añade preámbulo, Access Address publicitario, cabecera/PDU pertinente, CRC de 24 bits y *whitening*. `LLManager` interpreta cabecera de dos bytes, longitud, clase advertising/data y banderas; es parseo superficial, no una máquina Link Layer.

La pila anterior con funciones de conexión fue retirada deliberadamente del firmware y Python (historial `785a093` / `d98e1ce`). En el estado actual no hay API pública de conexión, GAP, GATT, SMP, cifrado de enlace ni procedimientos LL completos. Clasificación: **PHY BLE y advertising TX/RX, con parseo LL mínimo; pila de conexión intencionalmente excluida**.

## Wireless M-Bus

Los presets S/T/C/N aproximan frecuencia, GFSK, tasa, desviación, BW y una palabra de sincronía. La ruta RF continúa siendo el paquete propietario genérico descrito arriba.

Para un rol Wireless M-Bus, las referencias OMS/EN 13757 requieren distinguir, según modo y formato, campos de enlace como L, C, M, A y CI, codificación/longitud, CRC por bloques, direcciones, seguridad y procedimientos aplicables. FeralRF:

- no construye ni valida el formato completo en la API general;
- no implementa CRC por bloques Wireless M-Bus;
- no tiene parser de L/C/M/A/CI;
- no mantiene enlace ni seguridad Wireless M-Bus;
- contiene un helper T1 fijo cuyo L-field no concuerda de forma demostrada con los bytes generados y que usa el preset reservado `msk_868_50k` en lugar del preset T actual.

Clasificación: **plantillas nominales de configuración RF; emulación T1 incompleta; interoperabilidad no demostrada**. La evaluación no declara inválidos todos los parámetros RF, porque no se obtuvo acceso normativo completo a todas las variantes de EN 13757.

## Wi-SUN

Wi-SUN FAN se apoya en PHY SUN (incluido MR-FSK), MAC IEEE 802.15.4/LLC, *frequency hopping*, incorporación a red y seguridad; el perfil FAN añade 6LoWPAN, IPv6, RPL, ICMPv6 y autenticación 802.1X/EAP-TLS, entre otras funciones según versión/rol.

Los cinco presets `wisun_915_fsk_*` ofrecen tasas nominales 50–300 kbit/s en 902.2 MHz y FSK binaria. Sin embargo:

- utilizan el framing propietario simple, no una PHR/FCS/*whitening*/FEC 802.15.4g confirmada;
- no implementan hopping, MAC FAN, joining, LLC, 6LoWPAN, IPv6, RPL ni seguridad de red;
- no existe resultado de certificación PHY ni de perfil.

Clasificación: **perfiles nominales de parámetros RF para experimentación, no soporte Wi-SUN ni PHY 802.15.4g demostrado**.

## Amazon Sidewalk

La especificación oficial pública de Sidewalk Sub-GHz describe GFSK de 50 kbit/s, desviación de 25 kHz, BT=1 y rejilla 902.2–927.8 MHz con hopping lento para la modalidad FSK de la región pertinente. También define PHR/FCS y capas de enlace, seguridad, registro y operación de red.

El preset `sidewalk_915_fsk_50k` usa 915 MHz, tasa 50 kbit/s y desviación nominal 25 kHz, pero selecciona `mod_type=0` FSK, no GFSK, y conserva sync/framing genérico. El preset de 250 kbit/s no corresponde a la modalidad GFSK pública identificada. Ninguno implementa PHR Sidewalk, hopping, enlace, seguridad, registro o red. La nota sobre “LR LoRa-like” sólo documenta una exclusión de hardware y no agrega funcionalidad.

Clasificación: **presets FSK experimentales etiquetados Sidewalk; no implementan el PHY GFSK definido ni la pila Sidewalk**.

## MIOTY / TS-UNB

ETSI TS 103 357-2 define Telegram Splitting Ultra Narrow Band: división codificada del telegrama en subpaquetes/ráfagas, distribución pseudoaleatoria en tiempo/frecuencia, modos (G)MSK y procedimientos MAC/seguridad asociados al rol.

`mioty_868_tsunb` sólo solicita FSK 396 baud, desviación mínima y framing propietario genérico. El propio comentario del código lo marca `PENDING NATIVE SUPPORT` y registra intentos 0/10. No existen división de telegrama, interleaving/codificación, scheduling de ráfagas, hopping, MAC ni seguridad.

Clasificación: **intención/configuración nominal conservada por compatibilidad de API; MIOTY no implementado**.

## OOK y “personalidades”

FeralRF sí selecciona el modo OOK del comando TI para los presets con `mod_type=2`. Los helpers PT2262, EV1527 y Hörmann sólo ofrecen secuencias fijas. Protocolos de controles OOK suelen depender de codificación por ancho de pulso, duración de símbolos, repetición y sincronización particulares. La ruta de paquete genérica añade además preámbulo, sync, longitud y CRC.

Por ello los helpers se clasifican como **generadores de estímulo canónico**, no como tramas interoperables de esos dispositivos. OOK como modo RF queda **implementado no validado experimentalmente**.

## Regla de lectura de la matriz

Un “presente” en RF o PHY no asciende automáticamente los niveles superiores. Véanse los parámetros exactos y la matriz multidimensional en [[03 - Matriz de capacidades de protocolos y presets]].

## Cadena explícita de evidencia y conclusión

| Familia | Afirmación/nombre documentado | Comportamiento fuente | Requisito externo pertinente | Observación experimental | Interpretación | Conclusión (confianza) |
|---|---|---|---|---|---|---|
| PROP genérico | Presets FSK/GFSK/OOK y modos adicionales | `presets.py` → `configure_prop()` → `RadioIF_setPropConfig()` → comando TI de paquete variable | El contrato TI limita `modType` a 0/1/2 en este comando | FSK/GFSK de un subconjunto tuvo OTA CatSniffer↔CatSniffer; OOK y 4/5/6 no quedaron demostrados | El nombre de preset no altera el contrato del comando | FSK/GFSK como PHY simple: alta; OOK físico: media; MSK/4-FSK/4-GFSK: no soportados por esta ruta, alta |
| IEEE 802.15.4 | TX raw/frame y dos personalidades | El host entrega bytes MAC; RF core añade PHR/FCS; filtro/auto-ACK desactivados | IEEE define PHY y MAC, incluidos FCF, secuencia, addressing, ACK y seguridad según rol | Raw/frame entre dos CatSniffer fue exitoso | Demuestra transporte de PSDU, no MAC completo ni Zigbee/Thread | PHY + transporte MAC, alta |
| BLE | PHY 1M/2M/Coded y advertising | Host entrega advertising data; RF core forma PDU/framing; parser clasifica superficialmente | Bluetooth LL define PDU, CRC, whitening y procedimientos de advertising/conexión | No se usó aquí evidencia de interoperabilidad externa consolidada | El RF core aporta más que bytes PHY, pero no existe pila pública conectada | Advertising limitado, alta por código; interoperabilidad no demostrada |
| Wireless M-Bus | Cinco presets y personalidad T1 | Paquete PROP genérico; helper fijo usa `msk_868_50k` reservado | OMS/EN 13757 requieren framing/campos/CRC y procedimientos según modo/rol | No hay interoperabilidad consolidada | Parámetros nominales no forman una trama W-MBus | Presets nominales; helper incompleto, alta; exactitud de todos los parámetros EN, inconclusa |
| Wi-SUN | Cinco presets FAN nominales | FSK simple sin PHY 802.15.4g confirmada ni hopping/red | FAN requiere perfil PHY/MAC y capas de red/seguridad según versión | No hay certificación ni peer Wi-SUN | Las tasas coinciden nominalmente con opciones, pero no el perfil | No implementado como Wi-SUN, alta |
| Sidewalk | Dos presets FSK 915 | FSK binaria y framing genérico | Documento oficial público: modalidad Sub-G GFSK 50 kbit/s, framing/hopping, enlace y seguridad | No hay peer Sidewalk | 50 kbit/s/frecuencia/desviación parcial no compensan modulación/framing/capas ausentes | Etiqueta experimental, no soporte Sidewalk, alta |
| MIOTY | Preset TS-UNB marcado pendiente | FSK 396 baud sin división de telegrama ni scheduler | ETSI TS 103 357-2 requiere telegram splitting, ráfagas y procedimientos del rol | Comentario fuente registra 0/10; no se promueve a evidencia histórica independiente | La tasa nominal no implementa TS-UNB | No implementado; intención preservada, alta |
| OOK “devices” | PT2262/EV1527/Hörmann | Patrones fijos enviados con framing PROP del RF core | Un rol interoperable requiere codificación/temporización del dispositivo objetivo | OOK no quedó validado en Fase I | Es estímulo reproducible, no emulación protocolaria | Generador de estímulos, alta; PHY OOK efectiva, no validada |
