# Fase 1 — Mapeo de evidencia experimental histórica

## Método de conteo y procedencia

La consolidación previa reconcilió **193 unidades** hasta el 21-oct-2026: una unidad es una corrida, fila de preset, proceso repetido o prueba host expresamente documentada. No se cuentan otra vez copias del mismo experimento en README, matriz y resumen. A ese universo se agregan **8 operaciones OP-MO del 22-oct** (01, 02, 03, 04, 05, 06, 07a y 07b), para un total documental de **201 unidades**. Stage A fue revisión estática y el primer wrapper fallido terminó antes de abrir puertos; ninguno se cuenta como RF.

Este número no equivale a 201 presets ni a 201 enlaces OTA. Sólo cinco unidades OP-MO fueron corridas de preset OTA (03–06 y 07b). La trazabilidad de los 193 anteriores permanece en `../02 - Matriz histórica de experimentos y ejecuciones.md`; este documento detalla lo relevante a presets y añade el 22-oct.

Fuerza de evidencia: **P** artefacto primario; **R** reporte de ejecución; **S** resumen; **H** claim histórico/commit sin transcript; **C** fuente/SDK; **I** medida independiente. No existe **I** preservada para frecuencia, potencia, modulación, ancho ocupado o switch RF.

## Inventario de campañas

| Campaña | Unidades | Resultado relevante | Fuerza / límite |
|---|---:|---|---|
| Baseline 8-abr | 46 claims de matriz | Incluye claims OTA BLE/IEEE y presets propietarios; 433 marginal/fallido, 868/915/2440 y OOK declarados positivos | H/R; sin logs crudos localizados; algunas conclusiones físicas exceden la evidencia |
| F29 3-may | 7 presets | Claim 70/70 para 2 Sidewalk + 5 Wi-SUN | H; script recuperable, transcript/binarios/puertos no |
| EV-00…EV-14 | 63 | 27 presets por control EV-11; IEEE RX/TX; modos TX; sin OTA por preset en EV-11 | P/S/C desigual; 19 ejecuciones EV-11 auditables, 9 resumidas |
| OCT-IEEE 6–12 oct | 19 | RAW COM88→COM33 10/10; RAW COM33→COM88 20/20; FRAME 10/10; BURST/CONTINUOUS parciales/fallidos | P/R; marcadores repetidos, no secuencias únicas |
| OCT-PROP 8-oct | 24 | 240 retornos `transmit()`, 0 paquetes peer en 868/915/2440 | P/R; FAIL de criterio receptor, causa no establecida |
| Recuperación 15-oct | 6 | `wisun_915_fsk_50k`, 60/60 en 3A, 3A-R, 3C, 3D, 3E y 4A | P/R; marcador fijo repetido; metadata anómala |
| SUBG E02–E24, 15–16 oct | 23 | 13 presets nuevos 10/10; 18 PASS, 5 BLOCKED incluyendo controles/incidentes | P/R; exact preset execution |
| RF 20-oct | 4 intentos + RF-05 no contado | RF-02 10/10 reverso; RF-01/RF-03 bloqueados en `RX_STOP`; RF-04 bloqueado por dependencia | P/R; RF-05 NOT RUN |
| Validación manual 21-oct | incluida en las 193 | IEEE RAW 9/10 y 10/10; switching 25→20→25 observado; BURST 1/5; CONTINUOUS variable; `RX_STOP` recurrente | P/R; no añade preset Sub-GHz exacto |
| MIOTY/OOK 22-oct | 8 | GFSK controles 10/10; MIOTY-labeled 0/10; OOK 0/10 y cleanup fallido; recuperación GFSK 10/10 | P/R; payloads numerados únicos; firmware binario sin hash |

## Ejecuciones Sub-GHz positivas exactas 15–20 oct

Todas las filas siguientes invocaron el identificador exacto de `PROP_PRESETS`. TX fue CatSniffer #2/COM33 y RX #3/COM88 salvo RF-02, que invirtió roles. Los payloads de P01–P13 fueron marcadores de cuatro bytes; eran constantes por corrida, por lo que 10 paquetes no prueban 10 transmisiones únicas.

| Experimento | Fecha | Preset exacto | Hz solicitados | TX/RX | ACK | Peer | Payload/método | Resultado / nivel |
|---|---|---|---:|---|---:|---:|---|---|
| REC-3A…4A | 15-oct | `wisun_915_fsk_50k` | 902200000 | #2→#3 | 60/60 | 60/60 | `DEADBEEF`, contains + `crc_ok` reportado | PASS raw, L2 |
| E02/P01 | 15-oct | `gfsk_902_50k` | 902200000 | #2→#3 | 10/10 | 10/10 | `A5010001` | PASS raw, L2 |
| E03/P02 | 15-oct | `wisun_915_fsk_100k` | 902200000 | #2→#3 | 10/10 | 10/10 | `A5020001` | PASS raw, L2 |
| E04/P03 | 15-oct | `wisun_915_fsk_150k` | 902200000 | #2→#3 | 10/10 | 10/10 | `A5030001` | PASS raw, L2 |
| E06/P04 | 15-oct | `wisun_915_fsk_200k` | 902200000 | #2→#3 | 10/10 | 10/10 | `A5040001` | PASS raw, L2 |
| E07/P05 | 15-oct | `wisun_915_fsk_300k` | 902200000 | #2→#3 | 10/10 | 10/10 | `A5050001` | PASS raw, L2 |
| E08/P06 | 15-oct | `gfsk_915_50k` | 915000000 | #2→#3 | 10/10 | 10/10 | `A5060001` | PASS raw, L2 |
| E11/P07 | 15-oct | `sidewalk_915_fsk_50k` | 915000000 | #2→#3 | 10/10 | 10/10 | `A5070001` | PASS raw, L2 |
| E12/P08 | 15-oct | `sidewalk_915_fsk_250k` | 915000000 | #2→#3 | 10/10 | 10/10 | `A5080001` | PASS raw, L2 |
| E13/P09 | 15-oct | `gfsk_868_50k` | 868000000 | #2→#3 | 10/10 | 10/10 | `A5090001` | PASS raw, L2 |
| E14/P10 | 15-oct | `gfsk_868_100k` | 868000000 | #2→#3 | 10/10 | 10/10 | `A50A0001` | PASS raw, L2 |
| E19/P11 | 15-oct | `wireless_mbus_s_868` | 868300000 | #2→#3 | 10/10 | 10/10 | `A50B0001` | PASS raw, L2 |
| E20/P12 | 16-oct | `wireless_mbus_t_868` | 868950000 | #2→#3 | 10/10 | 10/10 | `A50C0001` | PASS raw, L2 |
| E21/P13 | 16-oct | `wireless_mbus_c_868` | 868950000 | #2→#3 | 10/10 | 10/10 | `A50D0001` | PASS raw, L2 |
| RF-02 | 20-oct | `wisun_915_fsk_50k` | 902200000 | #3→#2 | 10/10 | 10/10 | `C2028802` | PASS raw reverso, L2 |

Los incidentes E09, E15, E18, E22 y E23 se conservan como BLOCKED: timeout de stats, puerto ocupado o reset inducido. Cero observado en esas filas no significa cero radiación. E16/E19/E24 documentan recuperación posterior, no causa raíz.

## Campaña OCT-PROP negativa del 8-oct

Fueron 24 corridas exactas sobre 19 presets únicos. Cada corrida solicitó diez TX y usó un marcador fijo por corrida; total 240 ACK y 0 hits peer. La tabla evita esconder repeticiones dentro del agregado.

| Preset exacto | Corridas/dirección | ACK | Peer | Frecuencia solicitada |
|---|---|---:|---:|---:|
| `gfsk_868_50k` | 3: #2→#3 una; #3→#2 dos | 30 | 0 | 868000000 |
| `gfsk_915_50k` | 1: #2→#3 | 10 | 0 | 915000000 |
| `4fsk_868_50k` | 2: ambas direcciones | 20 | 0 | 868000000 |
| `gfsk_2440_50k` | 3: helper #2→#3, proceso separado #2→#3, reverso | 30 | 0 | 2440000000 |
| `gfsk_2440_250k` | 1: #2→#3 | 10 | 0 | 2440000000 |
| `gfsk_868_100k` | 1: #2→#3 | 10 | 0 | 868000000 |
| `msk_868_50k` | 1: #2→#3 | 10 | 0 | 868000000 |
| `4gfsk_868_50k` | 1: #2→#3 | 10 | 0 | 868000000 |
| `wireless_mbus_s_868` | 1: #2→#3 | 10 | 0 | 868300000 |
| `wireless_mbus_t_868` | 1: #2→#3 | 10 | 0 | 868950000 |
| `wireless_mbus_c_868` | 1: #2→#3 | 10 | 0 | 868950000 |
| `gfsk_902_50k` | 1: #2→#3 | 10 | 0 | 902200000 |
| `sidewalk_915_fsk_50k` | 1: #2→#3 | 10 | 0 | 915000000 |
| `sidewalk_915_fsk_250k` | 1: #2→#3 | 10 | 0 | 915000000 |
| `wisun_915_fsk_50k` | 1: #2→#3 | 10 | 0 | 902200000 |
| `wisun_915_fsk_100k` | 1: #2→#3 | 10 | 0 | 902200000 |
| `wisun_915_fsk_150k` | 1: #2→#3 | 10 | 0 | 902200000 |
| `wisun_915_fsk_200k` | 1: #2→#3 | 10 | 0 | 902200000 |
| `wisun_915_fsk_300k` | 1: #2→#3 | 10 | 0 | 902200000 |

El estado final es FAIL del criterio receptor y `CAUSE NOT ESTABLISHED`. Para mod 4/5 y 2440, además, el test no acredita la modulación/banda nominada por los defectos de implementación actuales.

## Ejecuciones exactas del 22-oct

Fuente primaria: `FeralRF/python/validation_artifacts/mioty_ook_20261022_133716/`. TX Unidad A/COM33; RX Unidad B/COM88; potencia solicitada 0 dBm; RP2040 reportado `v3.1.0.1`; CC reportado `1.0.0`, capabilities `0x07`, serial lógico `FERALRF1`. Los payloads incluyeron prefijo de etapa y secuencia 1–10, por lo que sí son atribuibles individualmente.

| ID | Preset exacto | TX payload | ACK | RX/bytes comparados | Cleanup | Resultado |
|---|---|---|---:|---|---|---|
| OP-MO-03 | `gfsk_868_50k` | `MO03C00` + seq 1–10 | 10/10 | 10/10 prefijo+secuencia, `crc_ok=True` reportado | RX_STOP ACK | PASS raw L2 |
| OP-MO-04 | `mioty_868_tsunb` | `MO04M00` + seq 1–10 | 10/10 | 0/10; `rx_ok=0` | RX_STOP ACK | FAIL raw; L1 solamente |
| OP-MO-05 | `gfsk_868_50k` | `MO05C00` + seq 1–10 | 10/10 | 10/10 únicas | RX_STOP ACK | PASS raw L2; control posterior |
| OP-MO-06 | `ook_868_4k8` | `MO06O00` + seq 1–10 | 1/10; timeout en 2, errores 3–10 | 0/10 | RX_STOP error | BLOCKED/FAIL cleanup; L1 parcial |
| OP-MO-07b | `gfsk_868_50k` | control numerado 1–10 | 10/10 | 10/10 únicas | RX_STOP ACK; stats TX timeout | PASS raw L2; recuperación global PARTIAL |

OP-MO-01/02 identificaron dispositivos y plano de control; OP-MO-07a documentó power-cycle, reidentificación y preflight. No son pruebas OTA de preset.

## Evidencia negativa y contradictoria

| Configuración | Observaciones preservadas | Conclusión |
|---|---|---|
| Siete F29 Sidewalk/Wi-SUN | claim 70/70 (mayo); subconjunto posterior 0/70; recuperación/actual 10/10 por preset | `CAUSE NOT ESTABLISHED`; el claim mayo no sustituye primarios, los resultados recientes sí prueban entrega raw estrecha |
| Campaña propietaria amplia | 24 corridas/240 ACK/0 peer (8-oct); 14 presets posteriores positivos | Falló el criterio receptor en octubre temprano; no demuestra regresión ni ausencia de radiación |
| `ook_868_4k8` | claim abril 10/10; RF-05 20-oct NOT RUN; OP-MO-06 22-oct 1/10 ACK, 0/10 peer y cleanup fallido | Resultado contradictorio; `CAUSE NOT ESTABLISHED`; no asignar L2 consolidado |
| `mioty_868_tsunb` | claim F29.b 0/10; OP-MO-04 0/10 con secuencias únicas | Fallo raw reproducido en dirección general, pero no prueba TS-UNB ni incompatibilidad del chip |
| Proprietary 2440 | claim abril 10/10; octubre 0/10; source actual cae a 868 | La banda efectiva histórica no está establecida; no es evidencia válida de 2.4 GHz |
| 433 | claims 0/10, 1/10, 6–10/10 y algunos 10/10 posteriores | Sin primarios/hardware comparable; `CAUSE NOT ESTABLISHED`; además fuera del scope OTA actual |
| mod 4/5/6 | claims de entrega 10/10 en algunas filas | No acreditan MSK/4-FSK/4-GFSK porque el contrato TI reserva esos valores |

## Qué se transmitió y qué se recibió

- Las campañas de presets transmitieron bytes raw arbitrarios: `DEADBEEF`, marcadores `A5...`/`C2...`, o prefijos `MO...` con contador. No construyeron PDUs Wi-SUN, Sidewalk, Wireless M-Bus o MIOTY.
- El host construyó únicamente el payload. El RF core añadió preámbulo, sync, longitud/header propietario y CRC según la configuración interna.
- P01–P13 y REC usaron un marcador idéntico en los diez envíos. El receptor contó paquetes que contenían ese marcador y cuyo parser reportaba `crc_ok`; no existe correspondencia uno-a-uno por secuencia. Se demuestra entrega identificable, no PDR exacto.
- OP-MO-03/04/05/06/07b sí incorporaron secuencias 1–10. Sus éxitos GFSK son atribuibles a intentos únicos; los ceros MIOTY/OOK son resultados negativos claros dentro de la ventana definida.
- El parser propietario actual tiene un problema de layout/metadata: se observaron canales 25/37 y RSSI positivos o incoherentes, con LQI 0. RSSI, canal, timestamp y `crc_ok` reportado no son mediciones independientes. La coincidencia de payload permanece evidencia válida aunque esa metadata no lo sea.
- No se preservó captura IQ, analizador de espectro, contador de potencia ni decoder externo. Ninguna fila alcanza L3 de framing físico ni L4 de interoperabilidad.

## Manuales 05, 06 y 07

- Manual 05 cubre identificación, reconexión, reset manual y reglas de interpretación. Usa `radio.py`; describe resultados control históricos y procedimientos reconstruidos.
- Manual 06 cubre IEEE RX, exclusión TX/RX, RAW/FRAME/BURST/CONTINUOUS, CW/PRBS y RF-01/RF-03. Referencia scripts bajo `python/examples/` y `experimental_artifacts/next_rf_validation/`. Mezcla comandos históricos exactos con secuencias reconstruidas, claramente marcadas.
- Manual 07 cubre control `gfsk_868_50k`, flujo OTA Sub-GHz, catálogo de 31 presets, RF-02/KillerBee/OOK. Declara explícitamente que su revisión no fue ejecutada y que OP-13 es reconstruido. **No es evidencia de ejecución.** OP-MO del 22-oct siguió otro runbook preservado, no valida manual 07 en conjunto.

## Niveles máximos

- L1: aceptación por control para al menos las entradas cubiertas por EV-11 y todas las corridas configuradas.
- L2: **14 presets exactos** con payload identificable recibido por un segundo CatSniffer (tabla positiva). Se limita a la frecuencia/modulación solicitada, no medida.
- L3: ninguno. El CRC/framing del RF core se inspecciona estáticamente y `crc_ok` no fue validado contra captura independiente.
- L4: ninguno. Los dos extremos usaron FeralRF; no hubo implementación independiente del protocolo nominal.

## Aclaración taxonómica y changelog

Los niveles anteriores se aplican a la **ruta realmente ejecutada**. Los 14 casos L2 demuestran payload raw identificable bajo framing propietario TI; no heredan L2 para W‑M‑Bus, Wi‑SUN, Sidewalk o MIOTY por el nombre. La evidencia IEEE 802.15.4 usa un backend dedicado distinto y tampoco se transfiere a `PROP_PRESETS`. 2026-10-23: se añadió esta aclaración sin alterar ningún conteo ni resultado histórico.
