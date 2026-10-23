# Fase 2 — Catálogo candidato de presets

Este documento propone una **taxonomía de catálogo**, no cambios de código ni parámetros listos para emitir. Todo candidato requiere revisión de fuente, RF design, regulación e instrumentación.

## Estados

- `KEEP-RAW`: nombre describe lo que hace el backend actual.
- `RENAME-RF`: preservar parámetros/evidencia, retirar claim de protocolo.
- `HOLD`: no exponer como activo hasta corregir backend/hardware.
- `DEPRECATE`: semántica físicamente no interpretable en el contrato actual.
- `PROTOCOL`: futuro objeto compuesto con framing/stack/peer; no simple preset.

## Disposición del catálogo actual

| IDs | Estado | Nombre/alcance candidato | Razón |
|---|---|---|---|
| P09, P10, P20, P21 | KEEP-RAW | `prop_gfsk_<freq>_<rate>` | backend GFSK real y L2 raw |
| P01–P03 | HOLD | `prop_{g}fsk_433_*_experimental` | requiere frontend/antena 433 demostrados |
| P04, P05, P11 | HOLD | `prop_ook_<freq>_*_diagnostic` | patch/lifecycle y evidencia contradictoria |
| P06–P08, P12, P18, P19 | DEPRECATE | ninguno hasta export TI | 4/5/6 reservados |
| P13–P15 | RENAME-RF | `rf_candidate_en13757_<mode>_*` | conserva trazabilidad sin afirmar W‑M‑Bus |
| P16–P17 | HOLD | `rf_candidate_169_n_*` sólo para variante 169 | placa/ruta actuales no demostradas; valores actuales sospechosos |
| P22–P23 | HOLD | `prop_2440_*` sólo con backend 2.4 dedicado | hoy caen a 868 |
| P24–P25 | RENAME-RF | `rf_candidate_sidewalk_fsk_*` | FSK raw, perfil/stack no demostrados |
| P26, P28–P31 | RENAME-RF | `rf_candidate_sun_fsk_na_<rate>` | tasas asociadas, no conformidad FAN |
| P27 | DEPRECATE | ninguno como simple preset | TS‑UNB exige generación de bursts/stack |

## Candidatos estándares que requieren fuente/exportación

| Candidato | Parámetros mínimos que deben venir de fuente primaria | Estado de entrada |
|---|---|---|
| Wi‑SUN SUN FSK 50k h=0.5 | mode ID, 2-GFSK, ±12.5 kHz, BW/setup TI, whitening/FEC | ejemplo TI disponible; falta diseño CatSniffer y stack |
| Wi‑SUN SUN FSK 50k h=1 | mode ID, 2-GFSK, ±25 kHz, BW/setup TI | ejemplo TI disponible |
| Wi‑SUN SUN FSK 100k h=0.5/1 | mode ID, ±25/±50 kHz y BW/setup | ejemplos TI disponibles |
| W‑M‑Bus N 169 4.8k | modo N exacto, 2-GFSK ±2.4 kHz según setting TI, coding/framing | setting TI citado; requiere variante 169 |
| W‑M‑Bus S/T/C | EN 13757‑4 vigente + export TI por modo | acceso normativo/peer aún requerido |
| Proprietary 2.4 GFSK | setup/overrides CC1352P7 y RF design CatSniffer | no existe export auditado |
| MSK/4-(G)FSK | setup/patch/overrides generados oficialmente | capacidad de silicio solamente |
| Sidewalk FSK | configuración del SDK Sidewalk aplicable y peer | documentación pública insuficiente para fijar números |

No se propone “MIOTY 396” como fila simple: el candidato debe ser un generador/transmisor TS‑UNB compuesto. Tampoco se propone “Zigbee preset”: Zigbee es una pila sobre la ruta IEEE 802.15.4.

## Registro autoritativo de candidatos técnicamente justificados

Se proponen **seis** candidatos de parámetros, todos pendientes de exportación/implementación. `RX bandwidth` físico es el valor objetivo publicado por TI; el encoding debe proceder del export y no copiarse por intuición.

| Candidato | Propósito | Protocolo/tecnología | Modulación | Frecuencia/channel plan | Rate | Desviación | RX BW | Sync/framing | PHY/backend requerido | HW | Fuente | Complejidad | Validación futura | Recomendación |
|---|---|---|---|---|---:|---:|---:|---|---|---|---|---|---|---|
| `prop_gfsk_868_50k_dev25` | control raw EU | None | 2-GFSK | 868 MHz fija | 50 kbps | ±25 kHz | 100 kHz | sync/framing propietario explícito | export PROP CC1352P7 | soportado/condicional regulación | S05; P09 L2 | baja | L3 espectro + L2 reproducible | **ADD** tras export |
| `prop_gfsk_868_100k_dev25` | control raw 100k | None | 2-GFSK | 868 MHz fija | 100 kbps | ±25 kHz | 137 kHz | propietario explícito | export PROP CC1352P7 | soportado/condicional | S05; sustituye desvío P10 | baja | L3 + L2 | **ADD** tras export |
| `ieee802154g_wisun_eu_50k_h05_phy` | SUN FSK EU candidate | IEEE 802.15.4g/Wi‑SUN-inspired PHY | 2-GFSK | 866.6 MHz de referencia; channel plan explícito al implementar | 50 kbps | ±12.5 kHz | 68 kHz | SUN FSK; whitening/FEC según mode ID; no stack | setting TI/advanced command | soportado en ruta 868 | S01/S06/S07/S08 | media | vector/framing L3; peer FAN para L4 | **INVESTIGATE/ADD como PHY** |
| `ieee802154g_wisun_na_50k_h10_phy` | SUN FSK NA candidate | IEEE 802.15.4g/Wi‑SUN-inspired PHY | 2-GFSK | 918.2 MHz referencia; plan 902–928 parametrizado | 50 kbps | ±25 kHz | 98 kHz | SUN FSK según mode ID; no stack | setting TI/advanced command | soportado en ruta 902–928 | S01/S06/S07/S08 | media | L3 + peer FAN L4 | **INVESTIGATE/ADD como PHY** |
| `ieee802154g_wisun_eu_100k_h05_phy` | SUN FSK EU 100k | IEEE 802.15.4g/Wi‑SUN-inspired PHY | 2-GFSK | 866.6 MHz referencia; plan parametrizado | 100 kbps | ±25 kHz | 135 kHz | SUN FSK según mode ID; no stack | setting TI/advanced command | soportado en ruta 868 | S01/S06/S07/S08 | media | L3 + peer FAN L4 | **INVESTIGATE/ADD como PHY** |
| `ieee802154g_wisun_na_100k_h10_phy` | SUN FSK NA 100k | IEEE 802.15.4g/Wi‑SUN-inspired PHY | 2-GFSK | 920.9 MHz referencia; plan 902–928 parametrizado | 100 kbps | ±50 kHz | 196 kHz | SUN FSK según mode ID; no stack | setting TI/advanced command | soportado en ruta 902–928 | S01/S06/S07/S08 | media | L3 + peer FAN L4 | **INVESTIGATE/ADD como PHY** |

Las frecuencias de referencia de los cuatro perfiles SUN provienen de ejemplos de rendimiento TI y **no sustituyen** el channel plan Wi‑SUN objetivo. Antes de implementar deben fijarse versión FAN, región, mode ID, spacing, whitening/FEC y hopping.

## Categorías complementarias

### Generic PHY candidates

Los dos `prop_gfsk_868_*` anteriores. P20/P21 se **retienen** como controles raw existentes, pero no se declaran candidatos nuevos hasta exportar/decodificar su `rxBw` exacto para 902.2/915.

### Standards-derived PHY candidates

Los cuatro `ieee802154g_wisun_*_phy`. W‑M‑Bus T/C también tiene parámetros TI caracterizados (100 kbps, 2-GFSK, ±50/±45 kHz), pero queda `INVESTIGATE`, no dentro de los seis, hasta obtener EN 13757‑4 vigente y export completo con BW/framing.

### Protocol-capable candidates

No hay un simple preset listo. W‑M‑Bus, FAN, Sidewalk y Zigbee sólo deben adquirir nombre de protocolo cuando existan framer/MAC/stack y peer. MIOTY requiere un motor TS‑UNB, no una fila estática.

### Diagnostic-only configurations

`diag_reserved_modtype_4`, `diag_reserved_modtype_5`, `diag_reserved_modtype_6`, `diag_prop_2440_fallback`, `diag_ook_868_lifecycle` y `diag_mioty_named_raw_396` son **escenarios de diagnóstico**, no presets de producción ni propuestas de código. Sus variables deben mantenerse controladas y su resultado nunca nombrarse como validación de la modulación/protocolo pretendidos.

## Criterio de incorporación

Una fila sólo pasa a catálogo activo si tiene: fuente/version, RF design, comando/setup/patch, unidades interpretadas, framing y ownership de campos, frecuencia efectiva verificable, rango regulatorio, prueba instrumental L3 y, si usa nombre de protocolo, peer independiente L4 o sufijo explícito `candidate`.

