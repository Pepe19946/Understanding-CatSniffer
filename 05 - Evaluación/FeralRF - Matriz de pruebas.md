# FeralRF - Matriz de pruebas

> Estado inicial de nuestra validación: **NOT YET RUN** en todas las filas. Los resultados del repositorio son historia documental, no resultados propios.

| ID | Cat. | Capacidad | Origen / KI | Estado README/docs | ¿Código? | Cobertura unitaria | Script HW | Historia repo | Nuestra validación | Hardware / externo | Evidencia esperada | Resultado / notas |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| EV-00 | L0 P0 | Puertos/autodetect | baseline; KI-15 | estable implícito | sí | mocks parciales | no | no separada | NOT YET RUN | 1 placa/Windows | lista COM+VID+nombre | — |
| EV-01 | L0 P0 | init/info/stats | estable; KI-25 | Stable | sí | strict responses | smoke varios | control 18/18 | NOT YET RUN | 1 | stdout/timing | — |
| EV-02 | L0 P0 | RX start/stop IEEE | estable; KI-31 | Stable | sí | host mocks | `smoke_phy4` | PASS control/OTA | NOT YET RUN | 1 | ACK/error/packets | — |
| EV-03 | L0 P0 | reconnect | stress | no estado | sí | parcial | no | sin dato | NOT YET RUN | 1 | 5/5 ciclos | — |
| EV-04 | L0 P0 | reset/recovery | workaround; KI-15 | requerido | sí host/RP | mock adapter | baseline reset | PASS histórico | NOT YET RUN | 1, Shell | 3/3 recovery | — |
| EV-05 | L0 P1 | primera RX física | Stable; KI-22 | IEEE Stable | sí | no RF | `smoke_phy4` | 10/10 OTA | NOT YET RUN | 1 + fuente propia | trama/RSSI | — |
| EV-06 | L0 P1 | exclusión RX/TX | negativo; KI-31 | regla de estado | sí | parcial | no | sin dato | NOT YET RUN | 1 | ERROR 0x05+recovery | — |
| EV-10 | L1 P1 | PHY 0–7 control | baseline; KI-02 | Stable/Experimental | sí | enum/contract | `smoke_phase2` | PASS control | NOT YET RUN | 1 | salida por PHY | ACK≠RF |
| EV-11 | L1 P1/2 | presets control | baseline; KI-03/04/20 | mixto | sí | schema/roundtrip | `smoke_prop` | mixto | NOT YET RUN | 1, antenas | log por preset | TX breve |
| EV-12 | L1 P1 | raw/frame/burst/continuous | Stable; KI-14/29 | Stable | sí | builders | `smoke_tx_*` | PASS ambiguo | NOT YET RUN | 1 + observer ideal | ACK+captura | TX RF |
| EV-13 | L1/2 P2 | CW/PRBS/stop | Stable; KI-07 | Stable | sí | mocks | `smoke_f22` | PASS wire 2026-04-29 | NOT YET RUN | analizador/2 placas | frecuencia/potencia/stop | TX RF |
| EV-14 | L1 P0 | error RF asíncrono | KI-12/31 | documentado | sí | async errors | indirecto | troubleshooting | NOT YET RUN | 1 | timeline/error/bytes | — |
| EV-15 | L1 P2 | boundaries/invalid | KI-29/30 | límites docs | sí | amplia host | no | sin dato | NOT YET RUN | 1 | rechazo+recovery | — |
| EV-20 | L2/3 P1 | IEEE OTA | Stable; KI-22/32 | Stable raw | sí | no RF | `smoke_ota_txrx` | 10/10 2026-04-08 | NOT YET RUN | 2 placas | markers 10/10 | — |
| EV-21 | L3 P1 | BLE raw 4 PHY | Stable raw; KI-09/23 | raw, sin stack | sí | host | OTA | 8–10/10 | NOT YET RUN | 2/observer BLE | markers+2º TX | — |
| EV-22 | L3 P1 | Sub-G 868/915/4FSK | Stable; KI-06 | Stable | sí | presets | OTA | 10/10 | NOT YET RUN | 2 + antenas | ratio/roles | — |
| EV-23 | L3 P2 | 433 caracterización | KI-04/05 | marginal/experimental | sí | presets | F9/baseline | 1–10/10 | NOT YET RUN | 2/antena 433 | éxitos/100 | — |
| EV-24 | L3 P1/2 | OOK+recovery | KI-01/05 | Stable 868, fail 433 | sí | preset | OTA+auto-reset | 10/10;0/10 | NOT YET RUN | 2 | OTA+firma lock+recovery | TX RF |
| EV-25 | L4 P1/2 | W-MBus S/T/C/N | KI-20/22 | Stable S/T/C | preset raw | schema | OTA | S/T/C 10/10; N no | NOT YET RUN | 2 + W-MBus/169 | markers/decodificación | — |
| EV-26 | L4 P2 | Wi-SUN/Sidewalk FSK | KI-22 | Experimental | preset raw | schema | F29/demos | 70/70 2026-05-03 | NOT YET RUN | 2/SDR | 7 ratios/espectro | no stack |
| EV-27 | L4 P1/2 | prop 2.4 GFSK | KI-08/32 | README Experimental | sí | preset | OTA | 10/10 contradictorio | NOT YET RUN | 2/SDR | markers+2440 MHz | — |
| EV-28 | L4 P2 | emulación helpers | KI-22 | PHY-level | sí host | payloads | F17/demos | 7/7 wire | NOT YET RUN | 2/tercero | bytes observados | no stack |
| EV-29 | L4 P2 | KillerBee | KI-15/22/26 | adapter | sí | mocks; 1 skip local | sniff/runbook | RF pendiente | NOT YET RUN | Linux+2/fuente | PCAP/FCS/inject | — |
| EV-30 | L5 P1 | crypto vectors HW | KI-30 | Stable | sí | host vectors/mocks | F25 | 9/9 2026-04-30 | NOT YET RUN | 1 + cryptography | bytes vs oráculo | — |
| EV-31 | L5 P2 | crypto stress/bounds | KI-29/30 | límites | sí | host validation | F25 parcial | sin soak | NOT YET RUN | 1 | 20/20+latencia | — |
| EV-40 | L6 P1 | baseline completo | KI-27 | recomendado | sí | n/a | Bash baseline | full 2026-04-08 | NOT YET RUN | 1/2 + Git Bash | log completo | OOK último |
| EV-41 | L6 P1 | switch sin reset | KI-02/10/11 | FAIL histórico | sí | no HW | F9 parcial | deadlock 2º ciclo | NOT YET RUN | 1 | ciclo/último ACK | — |
| EV-42 | L6 P1 | switch con reset | KI-03/15 | workaround | sí | mocks reset | baseline | PASS histórico | NOT YET RUN | 1 | 10 ciclos | — |
| EV-43 | L6 P2 | RX soak/stats | KI-13 | no claim | sí | stats parser | canary | sin log | NOT YET RUN | 1 + fuente | stats monotónicas | — |
| EV-44 | L6 P2 | queue pressure | KI-13 | limitación | sí | no HIL | burst/canary | sin dato | NOT YET RUN | 2/generador | tx/rx/drop/ovf | TX RF |
| EV-45 | L6 P2 | SEQ wrap | bug histórico | corregido host | sí | `test_radio_seq` | no | timeout TX 253 pre-fix | NOT YET RUN | 1 | 300/300 stats | — |
| EV-46 | L6 P1/2 | re-init/lifecycle | KI-11 | no claim | sí | init mocks | no | sin dato | NOT YET RUN | 1 | 20/20 | — |
| EV-47 | L6 P2 | interrupted recovery | recovery | recomendado | sí | parcial | smokes | sin dato | NOT YET RUN | 1 | baseline posterior | — |
| EV-50 | L7 P3 | jamming continuo | KI-17/18 | Experimental | parcial | adapter mocks | jam smoke | ACK sólo | NOT YET RUN | recinto+observer | PER/espectro | LAB ONLY |
| EV-51 | L7 P3 | spectrum scan | KI-16 | Pending | no E2E | no | no | ninguno | BLOCKED | futura ruta | API+handler+datos | no ejecutar |
| EV-52 | L7 P3 | MIOTY | KI-19 | Planned | preset insuficiente | schema | demo/OTA genérico | FAIL 0/10 | NOT YET RUN | 2/MIOTY | OTA/espectro | — |
| EV-53 | L7 P3 | RSA/AIS/15.4g/High-PA | KI-07/21 | Planned/pending | no o incompleto | no | no | no validado | BLOCKED | futuro equipo | data path+medición | no ejecutar |
| EV-54 | L7 P3 | BLE stack retirado | KI-23 | Removed | no ruta pública | n/a | Sniffle externo | retirado 2026-07-20 | NOT APPLICABLE | Sniffle si se desea | reachability | no es bug |

## Resumen de recursos

- **Disponible ahora:** `DUT-V3-FERAL` ejecuta EV-00–06, 10–12 (control), 14–15, 30–31, 41–43, 45–47; `PROTOCOL-DEVICE-ZIGBEE-CH25` convierte EV-05 y EV-43 en recepción RF real. Los V2 permanecen stock.
- **AUX-V3-FERAL:** habilita OTA simétrica EV-20–28, EV-40 y EV-44; no sustituye una implementación independiente.
- **AUX-V3-STOCK:** fortalece IEEE/sniffing y tooling oficial; no se presume que cubra OOK, 4FSK o propietario 2.4 GHz.
- **Equipo RF independiente:** EV-13, 23–27 y 50; SDR sirve para energía/frecuencia/decodificación compatible, analizador/power meter para potencia/espectro.
- **Dispositivo/software tercero:** EV-25 (W-MBus), EV-26 (Wi-SUN/Sidewalk), EV-29 (KillerBee/Wireshark), EV-30 (`cryptography`). Un PC necesita radio compatible para producir RF.

## Matriz compacta de roles y preparación

Leyenda: `D`=`DUT-V3-FERAL`; `AF`=`AUX-V3-FERAL`; `AS`=`AUX-V3-STOCK`; `V2`=`OBS-V2-A/B-STOCK`; `Z25`=`PROTOCOL-DEVICE-ZIGBEE-CH25`; `I`=`RF-OBSERVER`; `P`=`PROTOCOL-DEVICE`; `H`=`HOST-TOOL`. `Sí*` significa sólo la parte indicada. `DISC`, `ZSN` y `FSK` remiten a [[FeralRF - Guía de validación experimental#Procedimiento operativo de Catnip CLI|Procedimiento operativo de Catnip CLI]] §§3–9; allí están los comandos exactos y sus efectos. `—`=sin acción Catnip adicional después del preflight. La tabla complementa, no reemplaza, los estados históricos de arriba.

| EV | Mínimo hardware/roles | Ahora D+V2 | Con AF | Con AS | ¿Independiente preferible? | Preparación Catnip | Firmware peer requerido |
|---|---|---|---|---|---|---|---|
| EV-00 | D+H | Sí | Sí | Sí | No | DISC | ninguno |
| EV-01 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-02 | D | Sí control | igual | igual | No | DISC | ninguno |
| EV-03 | D | Sí | igual | igual | No | DISC tras reconexión | ninguno |
| EV-04 | D, Shell verificado | Sí | por placa | por placa | No | DISC; comparar Shell con +2 | ninguno |
| EV-05 | D+Z25 | Sí, RF real | comparación simétrica | observación fuerte | Sí | DISC; ZSN sólo AS/V2 ya compatible | AS `ti_sniffer`; V2 sólo preinstalado |
| EV-06 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-10 | D | Sí control | igual | igual | Para RF | DISC | ninguno |
| EV-11 | D; observer para RF | Sí control; V2 FSK/GFSK parcial | OTA presets | parcial según firmware | Sí | DISC; FSK opcional | AF FeralRF; stock según PHY |
| EV-12 | D; receptor/I para PASS-RF | Sí control; V2 FSK parcial | Sí completo simétrico | IEEE/BLE/FSK parcial | Sí | DISC; ZSN/FSK según PHY | AF FeralRF o receptor compatible |
| EV-13 | D+I | No instrumentado | smoke parcial | no sustituye I | Sí, obligatorio | DISC | ninguno; instrumento |
| EV-14 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-15 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-20 | D+AF para script; D+Z25 para RX | Sí* RX independiente | Sí baseline | Sí observer IEEE | Sí para compliance | DISC; ZSN AS | AF FeralRF / AS TI sniffer |
| EV-21 | D+AF o sniffer BLE | No con V2 preservado | Sí baseline | parcial con Sniffle | Sí | DISC; BLE sniff puede auto-flash | AF FeralRF / stock Sniffle |
| EV-22 | D+AF | V2 FSK/GFSK parcial | Sí | FSK/GFSK parcial SX1262 | Sí | DISC; FSK | AF FeralRF |
| EV-23 | D+AF+antena 433 | V2 FSK/GFSK parcial | Sí | FSK/GFSK parcial | Sí, SDR/analizador | DISC; FSK | AF FeralRF |
| EV-24 | D+AF; I preferido | No OTA OOK | Sí | No workflow OOK confirmado | Sí | DISC | AF FeralRF |
| EV-25 | D+AF para marker; P para interop | V2 PHY FSK parcial | Sí PHY | PHY FSK parcial | Sí, P W-MBus | DISC; FSK | AF FeralRF / dispositivo W-MBus |
| EV-26 | D+AF para marker; P/I para interop | V2 PHY FSK parcial | Sí PHY | PHY FSK parcial | Sí | DISC; FSK | AF FeralRF / nodo Wi-SUN/Sidewalk |
| EV-27 | D+AF; I 2.4 | No RF independiente | Sí | no confirmado | Sí | DISC | AF FeralRF |
| EV-28 | D+AF o receptor conforme | No completo | Sí firmas | subconjunto según PHY | Sí para interop | DISC; ZSN/FSK condicional | AF FeralRF / receptor conforme |
| EV-29 | D+H+Z25; peer para inject | Sí* sniff si KillerBee instalado | Sí inject/simétrico | Sí captura independiente | Sí | DISC; ZSN para AS | AS TI sniffer; V2 sólo preinstalado |
| EV-30 | D+H (`cryptography`) | Sí | igual | igual | Oráculo host | DISC | ninguno |
| EV-31 | D+H | Sí | igual | igual | Oráculo host | DISC | ninguno |
| EV-40 | D; D+AF para full OTA | Sí* control | Sí completo | sólo observación separada | Sí para claims RF | DISC ambas; verificar Shell | AF FeralRF |
| EV-41 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-42 | D | Sí | igual | igual | No | DISC/EV-04 | ninguno |
| EV-43 | D+Z25 | Sí | también controlable | observación paralela | Útil, no obligatorio | DISC; ZSN condicional | ninguno |
| EV-44 | D+generador controlado | No; Z25 no controlado | Sí | no generador confirmado | Sí, opcional | DISC | AF FeralRF o generador RF |
| EV-45 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-46 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-47 | D | Sí | igual | igual | No | DISC | ninguno |
| EV-50 | D+víctima+I, recinto | No | víctima simétrica | víctima independiente | Sí, obligatorio | DISC; no `verify` | víctima compatible |
| EV-51 | ruta futura | BLOCKED | BLOCKED | BLOCKED | Sí cuando exista | — | no implementado |
| EV-52 | D+AF/P MIOTY+I | No | limitación reproducible | no confirmado | Sí | DISC | AF FeralRF / dispositivo MIOTY |
| EV-53 | implementación/equipo futuro | BLOCKED | BLOCKED | BLOCKED | Sí | — | no implementado |
| EV-54 | análisis; Sniffle externo opcional | N/A | N/A | caso externo | Sí para BLE stack | DISC; BLE puede auto-flash | stock Sniffle, no FeralRF |

**Anterior:** [[FeralRF - Guía de validación experimental]]
