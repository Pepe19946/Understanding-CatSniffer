# Fase 2 — Registro de fuentes

Consulta externa realizada el **2026-10-23**. Se priorizaron fabricante, organismos de estándar y consorcios. Cuando el estándar completo no es público, se conserva esa limitación y no se rellenan parámetros por memoria.

## Fuentes externas primarias

| ID | Fuente | Versión/alcance | Uso y límite |
|---|---|---|---|
| S01 | [TI CC1352P7 datasheet](https://www.ti.com/lit/ds/symlink/cc1352p7.pdf) | SWRS251A, rev. A | capacidad de silicio, ejemplos de sensibilidad/perfiles; no prueba la placa |
| S02 | [TI CC1352P7 product page](https://www.ti.com/product/CC1352P7) | página vigente al consultar | bandas generales, modulaciones y protocolos soportados por ecosistema; no backend FeralRF |
| S03 | [TI Proprietary RF PHY configuration](https://software-dl.ti.com/simplelink/esd/simplelink_cc13xx_cc26xx_sdk/8.32.00.07/exports/docs/proprietary-rf/proprietary-rf-users-guide/rf-core/phy-configuration.html) | SDK 8.32 guide | diferencia `CMD_PROP_RADIO_SETUP`/`DIV_SETUP`; versión cercana, contrastada con header local 8.30 |
| S04 | [TI SysConfig Proprietary RF driver](https://software-dl.ti.com/simplelink/esd/simplelink_cc13xx_cc26xx_sdk/7.40.00.77/exports/docs/proprietary-rf/proprietary-rf-users-guide/sysconfig/proprietary-rf-driver.html) | SDK 7.40 guide | RF designs/bandas, incluido `CC1352EM-XS169-XS24`; no afirma compatibilidad CatSniffer |
| S05 | [SmartRF Studio revision history](https://software-dl.ti.com/lprf/smartrftm_studio/docs/help/html/rev_hist.html) | historial hasta 2.32 | settings CC1352P7 868 y W‑M‑Bus; identifica caracterización |
| S06 | [SmartRF Studio 2.19 revision history](https://software-dl.ti.com/lprf/smartrftm_studio/2.19.0/exports/rev_hist.html) | 2.19.0 | Wi‑SUN 50/100/150/200 y W‑M‑Bus 169 N 2.4/4.8; algunos settings eran RC/preview |
| S07 | [Wi‑SUN FAN Technology Overview](https://wi-sun.org/wp-content/uploads/Wi-SUN-FAN-Tech-Overview-r0.pdf) | overview público | SUN FSK, 50–300 kbps, spacing, FEC y stack; no sustituye perfil completo |
| S08 | [Wi‑SUN PHY PICS](https://wi-sun.org/wp-content/uploads/Case-0369_PHY-PICS.pdf) | PICS público | mode IDs/tasas/índices declarables; disponibilidad/version puede limitar detalle |
| S09 | [Wi‑SUN certification tests](https://wi-sun.org/certification-tests/) | página vigente | necesidad de certificación y regiones; no parámetros de preset |
| S10 | [ETSI TS 103 357‑2 V2.1.1](https://www.etsi.org/deliver/etsi_ts/103300_103399/10335702/02.01.01_60/ts_10335702v020101p.pdf) | 2024-06 | TS‑UNB, vectors, splitting/bursts/framing; norma primaria pública |
| S11 | [OMS Volume 1 General](https://oms-group.org/wp-content/uploads/2024/05/OMS-Spec_Vol1_General_v241.pdf) | 2.4.1 | relación con EN 13757‑4 y modos S/T/R/C/F/N |
| S12 | [OMS Volume 2 Primary](https://oms-group.org/wp-content/uploads/2024/10/OMS-Spec_Vol2_Primary_v501_01.pdf) | 5.0.1 | uso de modos/primaria OMS; EN 13757‑4 completo sigue siendo referencia normativa |
| S13 | [AWS IoT Wireless — Amazon Sidewalk](https://docs.aws.amazon.com/iot-wireless/latest/developerguide/iot-sidewalk.html) | documentación vigente | BLE y enlaces 900 MHz FSK/LoRa; no se halló tabla pública suficiente para certificar P24/P25 |
| S14 | [CSA Zigbee Specification](https://csa-iot.org/wp-content/uploads/2023/04/05-3474-23-csg-zigbee-specification-compressed.pdf) | document 05-3474-23 | arquitectura IEEE PHY/MAC + Zigbee NWK/APL/APS/ZDO |
| S15 | [CSA Zigbee overview](https://csa-iot.org/all-solutions/zigbee/) | página vigente | carácter full-stack/interoperable; secundaria respecto a S14 |

## Fuentes locales primarias

| ID | Ruta | Uso |
|---|---|---|
| L01 | `FeralRF/python/feralrf/presets.py` | catálogo y valores solicitados |
| L02 | `FeralRF/python/feralrf/radio.py` y `commands.py` | API, serialización y claims host |
| L03 | `FeralRF/firmware/cc1352/src/command_processor.c` | ACK y despacho `SET_PROP_CONFIG` |
| L04 | `FeralRF/firmware/cc1352/src/radio_if.c` | backend, frecuencia, format, overrides, OOK, IEEE |
| L05 | `FeralRF/firmware/cc1352/syscfg/ti_radio_config*.c/.h` | comandos y framing generado |
| L06 | `FeralRF/firmware/sdk/.../driverlib/rf_prop_cmd.h` | contrato `modType` local exacto |
| L07 | `CatSniffer/hardware/` y `FeralRF/hardware/` | esquemático/PCB y componente frontend; revisión física no preservada |
| L08 | Fase 1 documentos 01–06 | inventario, evidencia y contradicciones consolidadas |

## Trazabilidad de conclusiones

| Conclusión | Fuentes |
|---|---|
| 4/5/6 reservados en implementación | L02/L03/L04/L06 |
| silicio admite MSK/4-(G)FSK | S01/S02 |
| P22/P23 caen a 868 | L01/L04; S03 para arquitectura correcta |
| 169 requiere tratamiento especial | S04/S05/S06 + L04/L07 |
| Wi‑SUN actual no es FAN | S07/S08 + L01/L04/L05 |
| MIOTY actual no es TS‑UNB | S10 + L01/L04/L05 |
| W‑M‑Bus names no prueban EN 13757 | S05/S06/S11/S12 + L01/L05 |
| IEEE 802.15.4 no equivale a Zigbee | S14/S15 + L04/L05 |
| 14 casos sólo alcanzan L2 raw | L08 y sus artifacts registrados |

## Secciones consultadas para claims críticos

- S01: §1 “Features” (modulaciones/PHY/protocol ecosystem) y tablas de RF/typical characteristics usadas para los ejemplos 2-GFSK.
- S03: “RF Core → PHY configuration”, distinción entre `CMD_PROP_RADIO_SETUP` y `CMD_PROP_RADIO_DIV_SETUP`.
- S04: “RF Design” y selección de banda en SysConfig, incluida la entrada `CC1352EM-XS169-XS24`.
- S05: revision history v2.20.0 y v2.24.0, settings W‑M‑Bus y CC1352P7 caracterizados.
- S06: revision history v2.19.0, entradas Wi‑SUN y W‑M‑Bus 169 N-mode.
- S07/S08: secciones PHY/SUN FSK, operating modes y PICS; el overview no sustituye el perfil normativo completo.
- S10: cláusula TS‑UNB y Annex D/test data usados para estructura de bursts y vectors.
- S11/S12: secciones que remiten los modos OMS a EN 13757‑4; no se tuvo acceso al texto completo de EN 13757‑4.
- S14: Chapter 1 “Zigbee Protocol Overview” y Chapter 2 “Application Layer Specification”.

## Limitaciones

- EN 13757‑4 y partes de Wi‑SUN/Sidewalk pueden requerir acceso/licencia; no se inventaron campos ausentes.
- Las páginas TI que enumeran “protocol support” describen el ecosistema del chip, no lo compilado en FeralRF.
- Las guías TI web no coinciden exactamente con SDK 8.30 en todos los enlaces; el header local prevalece para el contrato compilado.
- La revisión física y BOM de las unidades históricas no están vinculadas criptográficamente a los artifacts.

