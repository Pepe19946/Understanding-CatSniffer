# Fase 1 — Matriz taxonómica y de cobertura corregida

## Corrección conceptual

Un nombre de preset no identifica por sí solo una PHY estándar ni una pila. Esta matriz separa: **preset**, **tecnología nominal**, **modo/perfil**, **modulación solicitada**, **soporte del comando TI usado**, **backend efectivo**, **framing efectivo** y **estado de pila**. La evidencia histórica se conserva sin promover un ACK o un intercambio raw a evidencia de modulación, frecuencia o interoperabilidad.

Los 31 presets seleccionan `PHY.PROPRIETARY_GFSK`, envían `SET_PROP_CONFIG` y terminan en `CMD_PROP_RADIO_DIV_SETUP[_PA]` + `CMD_PROP_TX/RX`. En ese setup auditado, `modType=0/1/2` significa FSK/GFSK/OOK; 4/5/6 están reservados. Que el CC1352P7 anuncie MSK y 4-(G)FSK como capacidad de silicio no vuelve válidos esos valores en este comando ni en este firmware.

`format_conf=0` se transforma en `0x00A0`: sync de 32 bits, MSB first, sin FEC ni whitening. TX/RX usan longitud variable y CRC. El host aporta el payload; el RF Core aporta preámbulo, sync, longitud y CRC. Por tanto, el framing real de las 31 filas es **TI propietario**, incluso cuando el nombre dice W‑M‑Bus, Wi‑SUN, Sidewalk o MIOTY.

La desviación interpretada es `deviation × 250 Hz`; el entero no está en Hz. `rxBw` se conserva como código de hardware y no se convierte sin tabla/exportación SmartRF trazable. “Frecuencia efectiva” es **no medida** en todas las filas: se indica la frecuencia que el software intenta programar o el fallback demostrable por código.

## Definiciones operativas

- **Protocolo:** reglas de comunicación y comportamiento entre entidades; Wi‑SUN, W‑M‑Bus, Sidewalk y MIOTY son tecnologías/protocolos, no modulaciones.
- **PHY:** mecanismo físico que transforma bits/símbolos en señal y viceversa; en FeralRF puede ser el backend TI propietario o el backend TI IEEE 802.15.4.
- **Modulación:** forma de codificar símbolos en la portadora. FSK, GFSK, OOK, MSK, 4-FSK y 4-GFSK son modulaciones, no protocolos.
- **Framing:** estructura de bits/campos alrededor del payload: preámbulo, sync, longitud, coding y CRC. Coincidir en frecuencia/rate no implica coincidir en framing.
- **Pila de protocolo:** implementación de las capas MAC/red/seguridad/aplicación requeridas. Un preset de parámetros no constituye una pila.
- **Entrega raw:** dos FeralRF intercambian un payload bajo su framing efectivo; acredita L2 de esa ruta, no el protocolo nominal.
- **Interoperabilidad:** comunicación conforme con una implementación independiente del mismo estándar/perfil; exige L4.

## Matriz técnica compuesta (31/31)

Abreviaturas: `PROP-DIV` = `CMD_PROP_RADIO_DIV_SETUP[_PA]`; `TI-var` = framing propietario común descrito arriba; `H:payload` = host entrega payload; `R:P/S/L/C` = RF Core construye preámbulo/sync/longitud/CRC; `sin pila` = no existe MAC/red/aplicación del protocolo nominal.

| ID | Preset | Tecnología nominal | Modo/perfil nominal | Modulación solicitada | Soporte driver/comando | Backend real | f nominal | f solicitada | f efectiva/medida | Rate | Desviación guardada → interpretada | RX BW | Sync | Framing real | Campos host / RF Core | Estado de pila |
|---|---|---|---|---|---|---|---:|---:|---|---:|---|---|---|---|---|---|
| P01 | `gfsk_433_50k` | RF genérica | — | GFSK (`1`) | Sí, condicionado por diseño RF | PROP-DIV 433 | 433.92 MHz | 433920000 | intentada; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var | H:payload / R:P/S/L/C | sin pila |
| P02 | `gfsk_433_10k` | RF genérica | — | GFSK (`1`) | Sí, condicionado por diseño RF | PROP-DIV 433 | 433.92 MHz | 433920000 | intentada; no medida | 10k | 50 → 12.5 kHz | `0x52` | `930B51DE` | TI-var | H/R | sin pila |
| P03 | `fsk_433_50k` | RF genérica | — | FSK (`0`) | Sí, condicionado por diseño RF | PROP-DIV 433 | 433.92 MHz | 433920000 | intentada; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var | H/R | sin pila |
| P04 | `ook_433_4k8` | RF genérica | — | OOK (`2`) | Sí; patch/lifecycle bloqueante | PROP-DIV + OOK | 433.92 MHz | 433920000 | intentada; no medida | 4.8k | 0 → 0 | `0x4C` | `930B51DE` | TI-var | H/R | sin pila |
| P05 | `ook_433_2k4` | RF genérica | — | OOK (`2`) | Sí; patch/lifecycle bloqueante | PROP-DIV + OOK | 433.92 MHz | 433920000 | intentada; no medida | 2.4k | 0 → 0 | `0x4C` | `930B51DE` | TI-var | H/R | sin pila |
| P06 | `msk_433_50k` | RF genérica | — | MSK pretendida (`4`) | **No en este comando**: reservado | PROP-DIV 433 | 433.92 MHz | 433920000 | no interpretable; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var si ejecuta | H/R | sin pila |
| P07 | `4fsk_433_50k` | RF genérica | — | 4-FSK pretendida (`5`) | **No en este comando**: reservado | PROP-DIV 433 | 433.92 MHz | 433920000 | no interpretable; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var si ejecuta | H/R | sin pila |
| P08 | `4gfsk_433_50k` | RF genérica | — | 4-GFSK pretendida (`6`) | **No en este comando**: reservado | PROP-DIV 433 | 433.92 MHz | 433920000 | no interpretable; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var si ejecuta | H/R | sin pila |
| P09 | `gfsk_868_50k` | RF genérica | — | GFSK (`1`) | Sí | PROP-DIV | 868 MHz | 868000000 | intentada; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var | H/R | sin pila |
| P10 | `gfsk_868_100k` | RF genérica | — | GFSK (`1`) | Sí | PROP-DIV | 868 MHz | 868000000 | intentada; no medida | 100k | 150 → 37.5 kHz | `0x56` | `930B51DE` | TI-var | H/R | sin pila |
| P11 | `ook_868_4k8` | RF genérica | — | OOK (`2`) | Sí; patch/lifecycle bloqueante | PROP-DIV + OOK | 868 MHz | 868000000 | intentada; no medida | 4.8k | 0 → 0 | `0x4C` | `930B51DE` | TI-var | H/R | sin pila |
| P12 | `msk_868_50k` | RF genérica | — | MSK pretendida (`4`) | **No en este comando** | PROP-DIV | 868 MHz | 868000000 | no interpretable; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var si ejecuta | H/R | sin pila |
| P13 | `wireless_mbus_s_868` | Wireless M‑Bus | S nominal | GFSK (`1`) | Sólo RF genérica | PROP-DIV | 868.3 MHz | 868300000 | intentada; no medida | 32.768k | 75 → 18.75 kHz | `0x52` | `543D0000` | **TI-var, no EN 13757** | H/R | W‑M‑Bus ausente |
| P14 | `wireless_mbus_t_868` | Wireless M‑Bus | T nominal | GFSK (`1`) | Sólo RF genérica | PROP-DIV | 868.95 MHz | 868950000 | intentada; no medida | 100k | 200 → 50 kHz | `0x57` | `543D0000` | TI-var, no EN 13757 | H/R | W‑M‑Bus ausente |
| P15 | `wireless_mbus_c_868` | Wireless M‑Bus | C nominal | GFSK (`1`) | Sólo RF genérica | PROP-DIV | 868.95 MHz | 868950000 | intentada; no medida | 100k | 180 → 45 kHz | `0x57` | `543D0000` | TI-var, no EN 13757 | H/R | W‑M‑Bus ausente |
| P16 | `wireless_mbus_n_169_2k4` | Wireless M‑Bus | N 2.4 nominal | GFSK (`1`) | Silicio/SDK: perfil especial; FW/placa: no demostrado | PROP-DIV; ruta 169 incoherente | 169.45 MHz | 169450000 | intentada; no medida | 2.4k | 154 → 38.5 kHz | `0x44` | `543D0000` | TI-var, no EN 13757 | H/R | W‑M‑Bus ausente |
| P17 | `wireless_mbus_n_169_4k8` | Wireless M‑Bus | N 4.8 nominal | GFSK (`1`) | Silicio/SDK: perfil especial; FW/placa: no demostrado | PROP-DIV; ruta 169 incoherente | 169.45 MHz | 169450000 | intentada; no medida | 4.8k | 154 → 38.5 kHz | `0x44` | `543D0000` | TI-var, no EN 13757 | H/R | W‑M‑Bus ausente |
| P18 | `4fsk_868_50k` | RF genérica | — | 4-FSK pretendida (`5`) | **No en este comando** | PROP-DIV | 868 MHz | 868000000 | no interpretable; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var si ejecuta | H/R | sin pila |
| P19 | `4gfsk_868_50k` | RF genérica | — | 4-GFSK pretendida (`6`) | **No en este comando** | PROP-DIV | 868 MHz | 868000000 | no interpretable; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var si ejecuta | H/R | sin pila |
| P20 | `gfsk_915_50k` | RF genérica | — | GFSK (`1`) | Sí | PROP-DIV | 915 MHz | 915000000 | intentada; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var | H/R | sin pila |
| P21 | `gfsk_902_50k` | RF genérica | — | GFSK (`1`) | Sí | PROP-DIV | 902.2 MHz | 902200000 | intentada; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var | H/R | sin pila |
| P22 | `gfsk_2440_250k` | RF propietaria 2.4 | — | GFSK (`1`) | Silicio sí; ruta actual no | **PROP sub‑GHz; fallback 868** | 2440 MHz | 2440000000 | código aplica 868; no medida | 250k | 320 → 80 kHz | `0x59` | `930B51DE` | TI-var | H/R | sin pila |
| P23 | `gfsk_2440_50k` | RF propietaria 2.4 | — | GFSK (`1`) | Silicio sí; ruta actual no | **PROP sub‑GHz; fallback 868** | 2440 MHz | 2440000000 | código aplica 868; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | TI-var | H/R | sin pila |
| P24 | `sidewalk_915_fsk_50k` | Amazon Sidewalk | FSK nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | 915 MHz | 915000000 | intentada; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | **TI-var, no Sidewalk** | H/R | Sidewalk ausente |
| P25 | `sidewalk_915_fsk_250k` | Amazon Sidewalk | FSK nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | 915 MHz | 915000000 | intentada; no medida | 250k | 200 → 50 kHz | `0x5A` | `930B51DE` | TI-var, no Sidewalk | H/R | Sidewalk ausente |
| P26 | `wisun_915_fsk_50k` | Wi‑SUN FAN | FSK 50k nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | nombre 915; dato 902.2 | 902200000 | intentada; no medida | 50k | 100 → 25 kHz | `0x52` | `930B51DE` | **TI-var, no SUN FSK** | H/R | FAN ausente |
| P27 | `mioty_868_tsunb` | MIOTY/TS‑UNB | TS‑UNB nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | 868 MHz | 868000000 | intentada; no medida | 396 | 1 → 250 Hz | `0x4A` | `930B51DE` | **TI-var; sin telegram splitting** | H/R | MIOTY ausente |
| P28 | `wisun_915_fsk_100k` | Wi‑SUN FAN | FSK 100k nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | nombre 915; dato 902.2 | 902200000 | intentada; no medida | 100k | 150 → 37.5 kHz | `0x56` | `930B51DE` | TI-var, no SUN FSK | H/R | FAN ausente |
| P29 | `wisun_915_fsk_150k` | Wi‑SUN FAN | FSK 150k nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | nombre 915; dato 902.2 | 902200000 | intentada; no medida | 150k | 200 → 50 kHz | `0x5A` | `930B51DE` | TI-var, no SUN FSK | H/R | FAN ausente |
| P30 | `wisun_915_fsk_200k` | Wi‑SUN FAN | FSK 200k nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | nombre 915; dato 902.2 | 902200000 | intentada; no medida | 200k | 200 → 50 kHz | `0x5A` | `930B51DE` | TI-var, no SUN FSK | H/R | FAN ausente |
| P31 | `wisun_915_fsk_300k` | Wi‑SUN FAN | FSK 300k nominal | FSK (`0`) | Sólo RF genérica | PROP-DIV | nombre 915; dato 902.2 | 902200000 | intentada; no medida | 300k | 200 → 50 kHz | `0x5E` | `930B51DE` | TI-var, no SUN FSK | H/R | FAN ausente |

## Evidencia y elegibilidad independientes

`V` pregunta si la fila puede validar hoy el claim de su nombre; `D`, si puede producir evidencia diagnóstica útil y segura. `Sí-cond.` conserva todas las condiciones regulatorias, instrumentales y de hardware del alcance original.

| ID | Experimentos históricos | Nivel máximo | Fuerza | Valor diagnóstico | Elegibilidad OTA V | Elegibilidad D | Pregunta principal | Acción recomendada | Fuentes |
|---|---|---|---|---|---|---|---|---|---|
| P01 | baseline 433 6–10/10 o F10 10/10; EV‑11 | L1 | H+C | Alto: frecuencia/frontend | Excluido | Condicional, HW 433 | ¿salió a 433.92 con GFSK? | medir en plataforma 433 trazable | Phase1-02; HW audit |
| P02 | EV‑11; claim familiar no asignable | L1 | C/S | Medio | Excluido | Condicional | rate/BW/frecuencia efectivos | caracterización instrumental, no inferencia P01 | EV‑11 |
| P03 | claims 1/10 y 10/10 | L1 | H/R | Alto: contradicción | Excluido | Condicional | causa de resultados opuestos | preservar; prueba discriminante | matrices históricas |
| P04 | claims 0/10 y 10/10; EV‑11 | L1 | H+C | Muy alto: OOK/lifecycle | Excluido | Condicional, banco aislado | patch, lock o RF | instrumentar completion/espectro | Phase1-02 |
| P05 | claim familiar 0/10; EV‑11 | L1 | H+C | Alto | Excluido | Condicional, banco aislado | evidencia individual | igual P04, no extrapolar | Phase1-02 |
| P06 | ACK/claim nominal | L1 control, no MSK | C/H | Alto: valor reservado | Bloqueado | Sí, contrato; RF condicional | rechazo, stale state o RF desconocida | backend TI oficial antes de validar | SDK header |
| P07 | ACK/claim nominal | L1 control, no 4-FSK | C/H | Alto | Bloqueado | Sí, contrato; RF condicional | idem 4-FSK | export/medición independiente | SDK header |
| P08 | ACK/claim nominal | L1 control, no 4-GFSK | C/H | Alto | Bloqueado | Sí, contrato; RF condicional | idem 4-GFSK | export/medición independiente | SDK header |
| P09 | E13 10/10; OP‑MO‑03/05/07b 10/10; previo 0/10 | L2 raw | P+R | Alto: control | Condicional, raw | Sí | frecuencia/modulación física | reutilizar; medir si cambia el claim | Phase1-02 |
| P10 | E14 10/10; E15 blocked; E16 10/10 | L2 raw | P+R | Alto | Condicional, raw | Sí | causa de bloqueo transitorio | conservar incidente | matriz 261016 |
| P11 | claim 10/10; RF‑05 NOT RUN; OP‑MO‑06 1/10 ACK, 0/10 peer, cleanup error | L1 | P+R+H | Muy alto | Bloqueado | Sí, banco aislado | TX, patch, RX o cleanup | diagnosticar lifecycle antes de repetir OTA | OP‑MO‑06 |
| P12 | claim 10/10 nominal | L1 control, no MSK | H+C | Alto | Bloqueado | Sí, contrato; RF condicional | backend MSK real | deprecar hasta export TI | SDK header |
| P13 | E18 blocked; E19 10/10 | L2 raw, no W‑M‑Bus | P+R | Alto | Condicional, sólo raw | Sí | framing S real | framer/decoder/peer EN 13757 | matriz 261016 |
| P14 | E20 10/10 | L2 raw, no W‑M‑Bus | P+R | Alto | Condicional, sólo raw | Sí | framing T real | framer/decoder/peer | matriz 261016 |
| P15 | E21 10/10; E22/E23 reset; E24 recuperación | L2 raw, no W‑M‑Bus | P+R | Muy alto | Condicional, sólo raw | Sí | causa reset y framing C | separar diagnóstico de interoperabilidad | matriz 261016 |
| P16 | EV‑11/control | L1 control | C | Alto: ruta 169 | Excluido HW actual | Condicional a variante 169 | firmware/frontend/antena | export TI + HW 169 + medida | S04–S06; HW audit |
| P17 | EV‑11/control | L1 control | C | Alto: ruta 169 | Excluido HW actual | Condicional a variante 169 | idem 4.8k | igual P16 | S04–S06; HW audit |
| P18 | claim/control 10/10 nominal | L1 control, no 4-FSK | H+C | Alto | Bloqueado | Sí, contrato; RF condicional | backend 4-FSK | deprecar hasta export TI | SDK header |
| P19 | claim/control 10/10 nominal | L1 control, no 4-GFSK | H+C | Alto | Bloqueado | Sí, contrato; RF condicional | backend 4-GFSK | deprecar hasta export TI | SDK header |
| P20 | E08 10/10; previo 0/10 | L2 raw | P+R | Alto | Condicional, raw | Sí | causa discrepancia | no repetir sin hipótesis | E08; OCT report |
| P21 | E02 10/10 | L2 raw | P+R | Alto: control | Condicional, raw | Sí | frecuencia física | reutilizar y medir para L3 | E02 |
| P22 | claim 10/10; OCT 0/10; fallback por fuente | sin L2 a 2.4 | H+R+C | Muy alto | Bloqueado | Sí, con analizador | ¿emite a 868/stale/ninguna? | backend 2.4 antes de validación | GAP‑14 |
| P23 | claim 10/10; OCT 0/10; fallback por fuente | sin L2 a 2.4 | H+R+C | Muy alto | Bloqueado | Sí, con analizador | idem | igual P22 | GAP‑14 |
| P24 | F29; E11 10/10; previo 0/10 | L2 raw, no Sidewalk | P+R+H | Alto | Condicional raw; protocolo bloqueado | Sí | perfil FSK/stack reales | renombrar o usar SDK/peer | E11; AWS |
| P25 | F29; E12 10/10; previo 0/10 | L2 raw, no Sidewalk | P+R+H | Alto | Condicional raw; protocolo bloqueado | Sí | idem 250k | igual P24 | E12; AWS |
| P26 | F29; previo 0/10; recovery 60/60; RF‑02 10/10 | L2 raw, no Wi‑SUN | P+R+H | Muy alto | Condicional raw; FAN bloqueado | Sí | mode ID/framing/hopping | conservar control y separar FAN | recovery; Wi‑SUN |
| P27 | F29.b 0/10; OP‑MO‑04 10 ACK, 0/10 peer | L1, no MIOTY | P+R+H | Muy alto | Bloqueado | Sí, negativo controlado | arquitectura TS‑UNB | no repetir como MIOTY raw | OP‑MO‑04; ETSI |
| P28 | F29; E03 10/10 | L2 raw, no Wi‑SUN | P+R+H | Alto | Condicional raw; FAN bloqueado | Sí | desviación/mode ID | comparar export TI | E03; Wi‑SUN |
| P29 | F29; E04 10/10 | L2 raw, no Wi‑SUN | P+R+H | Alto | Condicional raw; FAN bloqueado | Sí | mode ID 150k | comparar export TI | E04; Wi‑SUN |
| P30 | F29; E06 10/10 | L2 raw, no Wi‑SUN | P+R+H | Alto | Condicional raw; FAN bloqueado | Sí | mode ID 200k | comparar export TI | E06; Wi‑SUN |
| P31 | F29; E07 10/10 | L2 raw, no Wi‑SUN | P+R+H | Alto | Condicional raw; FAN bloqueado | Sí | perfil 300k exacto | fuente/export antes de claim | E07; Wi‑SUN |

## Arquitectura separada: IEEE 802.15.4 y Zigbee

IEEE 802.15.4 **no es un preset de `PROP_PRESETS`**. FeralRF selecciona `PHY_MANAGER_PHY_IEEE_802_15_4`, usa `Ieee154_0_mode`, `CMD_IEEE_RX/TX` y canales 11–26; el RF Core maneja el formato 802.15.4 y el host entrega/recibe la PSDU. Esa ruta tiene evidencia histórica propia, pero no valida ninguna fila P01–P31.

Zigbee añade NWK, APS, ZDO, perfiles/aplicación y seguridad sobre IEEE 802.15.4 MAC/PHY. No hay pila Zigbee en la implementación auditada. Son defendibles “radio/frames IEEE 802.15.4” y los resultados de esa campaña; no “Zigbee interoperable”.

En concreto, canal 25 se convierte a 2475 MHz mediante `2405 + 5 × (canal−11)` y se programa en `Ieee154_0_cmdIeeeRx`/`CMD_FS`. RX entrega al host la PSDU recibida y metadata añadida por la cola/comando (status con CRC/correlación, RSSI y timestamp según la configuración); TX recibe del host la PSDU y el comando IEEE genera la PHY correspondiente. El firmware filtra/transporta frames, pero no clasifica NWK/APS/ZCL como Zigbee.

Para identificar Zigbee harían falta, como mínimo: parseo completo del FCF y direccionamiento IEEE 802.15.4; reconocimiento/decodificación NWK; APS/ZDO/ZCL; manejo de seguridad, claves y contadores cuando proceda; y asociación de perfiles/clusters. La mera presencia de una trama 802.15.4 en canal 25 no prueba Zigbee porque otras pilas comparten esa PHY/MAC.

## Resultado

Se conservan los 14 presets con L2 raw exacto (P09, P10, P13–P15, P20, P21, P24–P26 y P28–P31), limitados al backend/framing propietario real. P11 sigue siendo el caso operativo prioritario. Los demás no son elegibles para validación positiva actual; algunos sí para diagnóstico controlado. Ninguna fila alcanza L3 ni L4.

## Changelog

- 2026-10-23: se separaron tecnología, perfil, modulación solicitada, soporte del comando, backend, framing y pila.
- 2026-10-23: se documentaron desviación, framing común y división host/RF Core.
- 2026-10-23: se separaron elegibilidad de validación y de diagnóstico.
- 2026-10-23: 169 MHz pasó de “silicio no soportado” a “perfil TI especial posible, pero firmware/frontend no demostrados”.
- 2026-10-23: se añadió la arquitectura independiente IEEE 802.15.4/Zigbee.
