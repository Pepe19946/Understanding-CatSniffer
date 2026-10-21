# FeralRF — Fase II-B: alcance, método y cobertura

## Resultado ejecutivo

Esta auditoría verifica FeralRF en el *snapshot* `feature/hardware-validation` / `fbff1b7f5077964418fb63a3fdf346a27c35a003`. Su conclusión principal es que FeralRF sí implementa un instrumento RF multipropósito con control de PHY, transporte de paquetes, funciones de prueba y primitivas criptográficas; no implementa, por el solo hecho de contener presets con nombres de protocolos, las pilas completas de Wireless M-Bus, Wi-SUN, Amazon Sidewalk o MIOTY.

La Fase II-A es sólida para arquitectura, transporte, API pública, RX/TX y hallazgos F-01–F-18. Esta Fase II-B amplía lo que allí quedó inventariado: delimitación entre bytes del host y *framing* del RF core, significado real de los presets, procedimientos de protocolo, alcance de los emuladores y requisitos de interoperabilidad.

> [!important]
> “Configuración RF compatible”, “PHY”, “trama física”, “trama de protocolo”, “procedimiento de protocolo”, “pila completa” e “interoperabilidad demostrada” son niveles diferentes. Ningún nivel superior se infiere únicamente de que el inferior exista.

## Alcance y restricciones observadas

- Repositorio primario: `FeralRF/`.
- Rama y revisión: `feature/hardware-validation`, `fbff1b7f5077964418fb63a3fdf346a27c35a003`.
- SDK examinado para contratos numéricos: TI SimpleLink CC13xx/CC26xx SDK `8.30.01.01`, incluido en `firmware/sdk/`.
- Evidencia experimental histórica: [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/00 - Índice, alcance y método|Fase I]].
- Auditoría antecedente: [[../00 - Auditoría de implementación y API/00 - Índice, alcance y método|Fase II-A]].
- No se ejecutaron transmisiones, puertos serie, flasheo ni pruebas de hardware.
- No se usaron como evidencia consolidada los registros manuales en curso ni artefactos no versionados.
- No se modificó código, firmware, API, presets, manuales, pruebas, evidencia histórica ni documentos de Fase II-A.

## Jerarquía de evidencia

1. Código actual de FeralRF y contratos del SDK exacto incluido en el repositorio.
2. Historial Git cuando explica intención, incorporación o exclusión deliberada.
3. Evidencia experimental consolidada de Fase I.
4. Normas y documentación oficial del organismo o proveedor.
5. Inferencias explícitas, nunca presentadas como medición.

Las afirmaciones se califican como:

- **Confirmada por código:** el camino y el contrato son directamente observables.
- **Confirmada por especificación:** la obligación pertenece al rol de protocolo indicado.
- **Demostrada experimentalmente:** existe ejecución histórica trazable.
- **No demostrada:** puede estar implementada, pero no hay evidencia OTA/hardware consolidada aplicable.
- **Inconclusa:** faltan datos para establecer comportamiento o causa.

## Modelo de completitud funcional

Para cada capacidad se consideran solamente las dimensiones pertinentes:

| Dimensión | Pregunta auditada |
|---|---|
| API | ¿Existe una operación pública y estable? |
| Validación | ¿Se rechazan parámetros imposibles o incompatibles? |
| Configuración efectiva | ¿El valor aceptado llega al comando/driver que puede realizarlo? |
| Ejecución | ¿La operación se inicia realmente? |
| *Framing* | ¿Quién aporta preámbulo, sincronía, longitud y CRC/FCS? |
| Construcción/parseo | ¿Se crean o interpretan campos del protocolo? |
| Estado/procedimientos | ¿Hay temporización, respuesta, reintento, asociación o sesión? |
| Errores/finalización | ¿Se informa aceptación, fallo y fin real? |
| Seguridad | ¿Se aplican los mecanismos exigidos al protocolo, no sólo primitivas genéricas? |
| Interoperabilidad | ¿Se ha probado frente a una implementación independiente? |
| Validación experimental | ¿Existe evidencia reproducible para esta configuración concreta? |

Clasificaciones usadas: **completa para el alcance actual**, **implementada con limitaciones**, **incompleta frente al contrato documentado**, **implementada no validada**, **no implementada**, **fuera del alcance**, **mejora opcional sujeta a decisión** e **inconclusa**.

## Cobertura real de la Fase II-A

| Área | Profundidad previa | Verificación II-B |
|---|---|---|
| Transporte Python, COBS, CRC y correlación | Trazado exhaustivo | Confirmado; se conserva F-02 sobre secuencias asíncronas. |
| Serialización y *command builders* | Parcial | Se corrige el detalle de `SET_PHY`: `<BHI>`, 7 bytes, además del formato heredado de un byte. |
| API pública y ciclo de vida | Trazado exhaustivo | Confirmado. |
| Configuración RF general | Parcial | Ampliada hasta driver, RF core, parámetros efectivos y ausencia de *readback*. |
| RX, TX y modos BURST/CONTINUOUS | Trazado exhaustivo | Confirmado; se añade responsabilidad de *framing* por PHY. |
| Proprietary y valores de modulación | Parcial | Ampliado con `CMD_PROP_RADIO_DIV_SETUP` del SDK exacto: 0 FSK, 1 GFSK, 2 OOK; restantes reservados. |
| Presets | Inventariado, no analizado en profundidad | Ampliado a los 31 presets actuales y a sus límites protocolarios. |
| CW/PRBS | Trazado exhaustivo | Sin cambio material. |
| Criptografía | Inventariada, con hallazgos puntuales | Ampliada en límites, curvas y naturaleza de la evidencia. |
| Emulación | Inventariada, no analizada en profundidad | Ampliada: constructores fijos/estímulos, no emulación completa. |
| Jamming | Trazado parcial | Confirmado como continuo y limitado; no hay reactivo/patrón. |
| Espectro | Trazado exhaustivo como ausencia funcional | Confirmado: hay identificadores/builders residuales, no camino firmware funcional equivalente. |
| KillerBee | Trazado exhaustivo | Confirmados traducción, ciclo de vida y reproducibilidad incompleta. |
| Errores, estado y colas | Trazado exhaustivo | Confirmados F-04, F-07, F-08, F-14 y F-16, preservando causalidad no demostrada. |
| RP2040/Cat-Shell | Trazado suficiente para el contrato | Confirmado F-06; no se amplió a código ajeno al contrato auditado. |
| Protocolos y pilas | No analizado sistemáticamente | Cubierto en [[02 - Modulación, framing y soporte de protocolos]] y [[03 - Matriz de capacidades de protocolos y presets]]. |

## Correcciones y matices respecto de II-A

Estas notas no alteran los documentos originales; delimitan su lectura correcta.

1. **`SET_PHY` actual no es sólo PHY + canal de 8 bits.** `CommandBuilder.set_phy()` serializa `<BHI>`: PHY `u8`, canal `u16` y frecuencia `u32`. El firmware acepta un byte heredado o siete bytes. `frequency_hz`, cuando no es cero, llega a `RadioIF_setPhy()`.
2. **BLE no es únicamente “PHY/raw” en sentido literal.** El host entrega datos publicitarios; los comandos BLE4/BLE5 del RF core construyen acceso, cabecera LL, CRC y *whitening*. Para 2 Mbit/s el firmware crea un `ADV_EXT_IND` primario y un `AUX_ADV_IND` secundario fijo. Sigue sin haber pila pública de conexión/GAP/GATT/seguridad.
3. **El catálogo actual contiene 31 presets**, mientras la evidencia consolidada de Fase I valida sólo el subconjunto histórico documentado. Existencia y aceptación estática no equivalen a comportamiento RF ni interoperabilidad.
4. **El emulador Wireless M-Bus conserva una descripción obsoleta:** afirma que no existe un preset T1 y usa `msk_868_50k`, aunque hoy existe `wireless_mbus_t_868`. Además, los bytes construidos no satisfacen por sí solos la trama y CRC por bloques de Wireless M-Bus.
5. **F-10 debe leerse por operación.** CTR, CBC y SHA tienen límites host explícitos; la discrepancia útil permanece especialmente en CCM/GCM y en el límite genérico de transporte frente a los búferes de cada operación.

## Referencias externas principales

| Fuente oficial | Uso | Limitación |
|---|---|---|
| [IEEE 802.15.4-2020](https://standards.ieee.org/ieee/802.15.4/7029/) | Alcance PHY/MAC de IEEE 802.15.4 | Página pública; el texto normativo completo puede requerir acceso. La revisión 2020 fue sustituida por 2024. |
| [TI IEEE 802.15.4 packet format](https://dev.ti.com/tirex/content/simplelink_cc13xx_cc26xx_sdk_8_32_00_07/docs/proprietary-rf/proprietary-rf-users-guide/ieee-rf/packet-format.html) | División entre MHR, payload y FCS en el RF core | SDK 8.32 como explicación; para bits/comandos se usó el 8.30.01.01 local. |
| [Bluetooth Core Specification 5.4](https://www.bluetooth.com/specifications/specs/core-specification-amended-5-4/) y [Link Layer](https://www.bluetooth.com/wp-content/uploads/Files/Specification/HTML/Core-54/out/en/low-energy-controller/link-layer-specification.html) | Paquete, PDU, CRC, *whitening* y procedimientos LL | Referencia de protocolo, no prueba de conformidad FeralRF. |
| [Wi-SUN FAN Technical Overview](https://wi-sun.org/wp-content/uploads/Wi-SUN-FAN-Tech-Overview-r0.pdf) | Capas FAN, PHY/MAC, hopping, IPv6 y seguridad | Resumen técnico oficial; no sustituye las especificaciones certificables completas. |
| [Wi-SUN certification tests](https://wi-sun.org/certification-tests/) | Separación entre ensayo PHY y perfil/interoperabilidad | No aporta resultados para FeralRF. |
| [Amazon Sidewalk: Sub-GHz PHY modulations](https://docs.sidewalk.amazon/specifications/sidewalk-protocol-spec/sub-ghz-protocol-stack/sub-ghz-physical-layer-modulations.html) | GFSK 50 kbit/s, desviación, rejilla y hopping | Documento oficial público. |
| [Amazon Sidewalk: protocol layers and structure](https://docs.sidewalk.amazon/specifications/sidewalk-protocol-spec/sub-ghz-protocol-stack/protocol-stack-layers-and-structure.html) | PHR/FCS, enlace, seguridad y registro | Documento oficial público. |
| [ETSI TS 103 357-2 V2.1.1](https://www.etsi.org/deliver/etsi_ts/103300_103399/10335702/02.01.01_60/ts_10335702v020101p.pdf) | TS-UNB/MIOTY: división en ráfagas, acceso y codificación | Especificación oficial pública. |
| [OMS Specification](https://oms-group.org/en/specification/) y [Annex N](https://oms-group.org/wp-content/uploads/2024/10/Open-Metering-System-Specification_Annex-N.pdf) | Campos L/C/M/A/CI, ejemplos y seguridad Wireless M-Bus | OMS perfila EN 13757; no sustituye íntegramente la norma EN, cuyo acceso normativo es limitado. |
| [TI CC1352P7 datasheet](https://www.ti.com/lit/ds/symlink/cc1352p7.pdf) | Arquitectura del RF core | Documento de proveedor, no estándar de protocolo. |
| [TI PHY configuration](https://dev.ti.com/tirex/content/simplelink_cc13xx_cc26xx_sdk_8_32_00_07/docs/proprietary-rf/proprietary-rf-users-guide/rf-core/phy-configuration.html) | Relación setup/patch/overrides | Versión 8.32 usada sólo para contexto; se verificó el SDK local para el contrato exacto. |

## Índice de trazabilidad al código

Todos los rangos siguientes corresponden a la revisión `fbff1b7f5077964418fb63a3fdf346a27c35a003`; son rangos prácticos, no identificadores estables entre commits.

| Tema | Ruta, símbolo y rango relevante |
|---|---|
| `SET_PHY` host | `FeralRF/python/feralrf/commands.py`, `CommandBuilder.set_phy()`, L32–34; `radio.py`, `Radio.set_phy()`, L455–463 |
| `SET_PHY` firmware | `FeralRF/firmware/cc1352/src/command_processor.c`, `CMD_SET_PHY`, L128–166; `control_task.c`, `ControlTask_onSetPhy()`, L225–247; `radio_if.c`, `RadioIF_setPhy()`, L1912–1976 |
| Configuración PROP | `radio.py`, `configure_prop()`, L490–532; `commands.py`, `set_prop_config()`, L100–121; `command_processor.c`, `CMD_SET_PROP_CONFIG`, L168–208; `radio_if.c`, `RadioIF_setPropConfig()`, L2020 en adelante |
| Modulación TI exacta | `FeralRF/firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/source/ti/devices/cc13x2x7_cc26x2x7/driverlib/rf_prop_cmd.h`, `modType`, L475–478 y L581–584 |
| Framing PROP base | `firmware/cc1352/src/smartrf_prop_0.c`, `Prop0_cmdPropRadioDivSetup`/RX/TX, L207–213 y L261–283 |
| TX IEEE | `FeralRF/firmware/cc1352/src/smartrf_ieee_15_4_0.c`, `Ieee154_0_cmdIeeeTx`, L146–163; contrato `FeralRF/firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/source/ti/devices/cc13x2x7_cc26x2x7/driverlib/rf_ieee_cmd.h`, L290–299 |
| RX IEEE | `radio_if.c`, `RadioIF_startIeee154RfBackend()` y `RadioIF_processIeee154Packets()`, L1335–1369 y L1497–1568 |
| Filtro/ACK IEEE | `smartrf_ieee_15_4_0.c`, `frameFiltOpt`, L103–122 |
| BLE TX/parsing | `radio_if.c`, `RadioIF_transmitBle2mExtAdv()`, L494–584; `RadioIF_transmitBleAdvRaw()`, L586–656; `RadioIF_processBlePackets()`, L1400–1495 |
| RX PROP/metadatos | `radio_if.c`, `RadioIF_processPropPackets()`, L1706–1754 |
| TX público/diferido | `radio.py`, `transmit*()`, L725–810; `control_task.c`, `ControlTask_onTxRaw()` y schedulers, L249–407 |
| Emulación IEEE | `python/feralrf/emulation/ieee154_device.py`, `_BEACON_FRAME`/`_DATA_FRAME`, L38–83 |
| Emulación W-MBus | `python/feralrf/emulation/sub1ghz_device.py`, `WIRELESS_MBUS_T1_METER`, L57–70 |
| Emulación OOK | `python/feralrf/emulation/ook_device.py`, personalidades, L35–59 |
| Catálogo de presets | `python/feralrf/presets.py`, `PROP_PRESETS`, L11–279 |
| ECDSA | `radio.py`, `ecdsa_sign()`/`ecdsa_verify()`, L1042–1082; `firmware/cc1352/src/crypto_engine.c`, operaciones ECDSA, L445–524 |
| Límites por PHY | `radio_if.c`, `BLE_ADV_TX_MAX_PAYLOAD_LEN`, `IEEE_15_4_TX_MAX_PAYLOAD_LEN`, `PROP_TX_MAX_PAYLOAD_LEN`, L54–80; `control_task.c`, `CONTROL_TASK_TX_RAW_MAX_LEN`, L34–45 |

Los hallazgos transversales F-01–F-18 conservan además su índice de evidencia en [[../00 - Auditoría de implementación y API/05 - Brechas funcionales y limitaciones|Fase II-A]] y su revalidación condicionada en [[04 - Brechas verificadas y limitaciones de implementación]].

## Navegación de esta fase

- [[01 - Completitud funcional por subsistema]]
- [[02 - Modulación, framing y soporte de protocolos]]
- [[03 - Matriz de capacidades de protocolos y presets]]
- [[04 - Brechas verificadas y limitaciones de implementación]]
- [[05 - Requisitos y preguntas técnicas para desarrollo futuro]]
