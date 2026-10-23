# FeralRF — próximo plan dirigido de validación RF

Fecha de auditoría: 2026-10-19 (America/Mexico_City)  
Estado de la campaña OTA Sub-GHz 15–16 oct 2026: **CLOSED WITH LIMITATIONS**  
Objetivo: cerrar sólo brechas RF que producen evidencia nueva con dos CatSniffer v3.

## 1. Alcance y exclusiones

- Esta auditoría es documental y de código; **no se ejecutó hardware**.
- BLE queda absolutamente fuera de alcance y no forma parte del plan.
- No se propone jamming: sin instrumentación independiente no ofrece un criterio seguro ni atribuible y podría afectar la red disponible.
- No se reabre la campaña Sub-GHz ya cerrada ni se repiten sus 13 presets FSK/GFSK.
- Los nombres Wi-SUN, Sidewalk y W-MBus describen configuraciones raw; no prueban interoperabilidad de esos protocolos.
- La red Zigbee sólo se usará de forma pasiva y autorizada. IEEE 802.15.4 PHY/MAC raw no equivale a interoperabilidad Zigbee.
- No se modifica `docs/VALIDATION_MATRIX.md`, firmware ni resultados históricos.

Leyenda de evidencia usada aquí:

| Clase | Significado |
|---|---|
| **CODE VERIFIED** | La ruta existe y su ejecución actual fue trazada en fuente. |
| **HISTORICAL CLAIM** | El repositorio declara una prueba, pero no se revisó evidencia primaria suficiente. |
| **OUR OTA VERIFIED** | Los registros propios preservan recepción peer atribuible y CRC-good. |
| **CONTROL ONLY** | ACK, estado o retorno de API sin confirmación OTA suficiente. |
| **NOT VALIDATED** | No hay evidencia adecuada para la afirmación. |
| **BLOCKED** | Un requisito técnico concreto impide una prueba significativa. |

## 2. Baseline actual de repositorio y hardware

| Elemento | Baseline auditado |
|---|---|
| `FeralRF` | rama `feature/hardware-validation`, commit `fbff1b7f5077964418fb63a3fdf346a27c35a003`, árbol limpio |
| Último cambio de código RF relevante | `785a0937af26c977f7e389b09677d51959c3bad3`, 2026-07-20; después sólo cambiaron documentos revisados |
| `Understanding-CatSniffer` | rama `main`, commit `2a4b00223d91d9676e33ed0b9f28a36917e95899`, árbol limpio |
| Host/API | `feralrf` 0.3.0; Python → USB CDC Cat-Bridge 921600 → RP2040 passthrough → UART → CC1352P7 |
| Dispositivos | #2: Bridge COM33 / Shell COM35; #3: Bridge esperado COM88 / Shell COM87 |
| Estado observado | ambos RP2040 v3.1.0.1, Git `c0cd5a4`, HW v3 RP2040 CC1352P7; CC1352 reporta `feralrf_cc1352 (custom)` |
| Evidencia CC actual | COM33: CC firmware 1.0.0, capabilities 7 y `get_stats()` correcto; COM88 aún requiere preflight |
| Límite de procedencia | La identidad/hash del binario CC1352 instalado sigue sin vincularse al commit de fuente. |
| RF externo | `Band` de Shell es estado lógico del RP2040; no prueba CTF/U2, frecuencia, potencia ni ruta de antena. |

Ruta de ejecución relevante en el commit auditado:

| Procesador | Componente controlado | Interfaz | Fuente / función |
|---|---|---|---|
| PC | API y serialización | pyserial + COBS/CRC16 | `python/feralrf/radio.py`; `commands.py` |
| RP2040 | puente y selector RF externo | USB CDC / UART; Cat-Shell 115200 | passthrough; comandos `band1`/`band2` separados del CC |
| CC1352P7 | parser y planificación | UART 921600 | `command_processor.c`, `control_task.c`, `data_task.c` |
| CC1352P7 RF Core | setup, FS, RX y TX | TI RF Driver | `radio_if.c`, `smartrf_ieee_15_4_0.c`, `smartrf_prop_0.c` |

## 3. Qué ya está suficientemente validado por nuestros experimentos

| Capacidad | Evidencia | Alcance que sí puede afirmarse | Límite pendiente |
|---|---|---|---|
| IEEE RX | **OUR OTA VERIFIED** | COM33 y COM88 recibieron tráfico IEEE 802.15.4 en canal 25; `start_rx()`, `read_packets()` y `stop_rx()` funcionaron en sesiones normales. | El tráfico ambiental no quedó atribuido frame a frame a Zigbee. |
| `transmit()` / TX_RAW IEEE | **OUR OTA VERIFIED** | Marcadores CRC-good: #2→#3 20/20 y #3→#2 10/10 en canal 25. | No prueba otros canales ni switching en sesión. |
| `transmit_frame()` / TX_FRAME IEEE | **OUR OTA VERIFIED** | 10/10 en #2→#3. | En fuente, `ControlTask_onTxFrame()` delega al mismo camino raw; no es un constructor MAC distinto. |
| FCS IEEE | **CODE VERIFIED + OUR OTA VERIFIED** | TX entrega MPDU sin FCS a `CMD_IEEE_TX`; el RF Core genera FCS. RX usa `bIncludeCrc=1`; se observaron payload + dos bytes compatibles con FCS y CRC-good. | No hace falta otra prueba sólo para volver a observar FCS. |
| `transmit_burst()` | **OUR OTA VERIFIED, PARTIAL** | La solicitud produjo al menos una trama atribuible. | `count=5` produjo sólo un marcador; conteo e intervalo no validados. No repetir hasta análisis/corrección de scheduler. |
| `transmit_continuous(interval_us=0)` | **OUR OTA VERIFIED** | Produjo múltiples tramas atribuibles (30 observaciones en una corrida). | Temporización exacta no validada. |
| `stop_transmit()` | **CONTROL ONLY / PARTIAL OTA** | ACK repetido y ventanas posteriores sin marcador, compatibles con cese. | Cese RF inmediato no probado por buffering; RAW post-STOP 0/10 y `RX_STOP` timeout en sesión cargada. |
| Recuperación tras CONTINUOUS | **OUR OTA VERIFIED, PARTIAL** | Una sesión nueva recuperó RAW 9/10 y `RX_STOP` ACK sin reset. | Causa de la anomalía de lifecycle/colas abierta. |
| Sub-GHz FSK/GFSK raw | **OUR OTA VERIFIED** | 13 presets, 868–915 MHz, 10/10 CRC-good #2→#3; cambios 915→868→902.2 funcionales. | Sin medición espectral/modulación, sin conformidad Wi-SUN/Sidewalk/W-MBus y sin dirección inversa positiva. |

Conclusión IEEE: RAW, FRAME y CONTINUOUS rápido ya tienen evidencia OTA. BURST-count y STOP inmediato requieren primero trabajo de implementación/observabilidad, no más variaciones temporales con el mismo montaje.

Capacidades que permanecen **sólo como claims históricos del repositorio**:

| Claim | Estado tras esta auditoría |
|---|---|
| GFSK proprietary 2440 MHz 10/10 | Contradicho por la ruta actual que cae a 868 MHz; no promover. |
| OOK 868 10/10 | Plausible en código, pero evidencia primaria no revisada; candidato RF-05. |
| MSK 868 y 433 | No demuestra MSK: `mod_type=4` es reservado en el comando TI usado. |
| 4-FSK / 4-GFSK 868 10/10 | No demuestra esas modulaciones: `mod_type=5/6` es reservado. |
| 433 GFSK/FSK marginal | Implementación plausible, pero hardware/antena domina el resultado. |
| Device emulation IEEE/Sub-GHz 20/20 | Es entrega de firmas raw, no emulación de stack; valor incremental bajo. |
| Switching con reset | Describe comportamiento histórico y lifecycle anterior; no valida el hot-switch actual sin reset. |

## 4. Brechas de cobertura y discrepancias

### 4.1 IEEE 802.15.4

| Tema | Estado | Decisión |
|---|---|---|
| Selección inicial canal 25 | **OUR OTA VERIFIED** | Sólo control positivo. |
| `set_channel()` cambiando 25→20→25 en una sesión | **NOT VALIDATED** | Probar con aislamiento de canal y marcadores únicos (RF-01). |
| RX KillerBee / PCAP sobre tráfico real | **CODE VERIFIED**, **NOT VALIDATED** | Prueba pasiva RF-04; no inyectar en la red Zigbee. |
| KillerBee `inject()` | **CODE VERIFIED**, pero reutiliza `transmit_frame()` | Diferir: aporta poca evidencia RF nueva y la red no autoriza inyección. |
| Device emulation IEEE | **HISTORICAL CLAIM** 20/20 | No priorizar: son frames raw preconstruidos, no asociación ni stack. |
| BURST `count>1` | **NOT VALIDATED** | **BLOCKED** por anomalía reproducida; necesita diagnóstico/corrección. |
| STOP inmediato y RAW post-STOP | **NOT VALIDATED** | **BLOCKED** por buffering/lifecycle; no repetir OTA sin nueva telemetría. |

### 4.2 Proprietary 2.4 GHz: ambos presets están bloqueados por código

Traza actual de `gfsk_2440_50k` y `gfsk_2440_250k`:

1. `PROP_PRESETS` entrega 2440000000 Hz y `mod_type=1` (`presets.py:188,196`).
2. `Radio.configure_prop()` llama `CommandBuilder.set_prop_config()` y serializa 18 bytes (`radio.py:490`; `commands.py:99`).
3. El CC1352 deserializa y llama `RadioIF_setPropConfig()` antes del ACK (`command_processor.c:168-184`).
4. `RadioIF_setPropConfig()` escribe inicialmente 2440 MHz y `loDivider=0` (`radio_if.c:2020-2169`).
5. **Defecto:** antes de RX y TX, `RadioIF_applySub1ghzChannelConfig()` acepta `frequency_hz` sólo si es `<1000 MHz`; para PHY proprietary cae al default 868 MHz (`radio_if.c:1578-1609`, llamadas en 1629 y 1765).
6. Por ello el ACK de configuración no demuestra, ni la ejecución actual conserva, 2440 MHz.

| Evidencia en conflicto | Evaluación |
|---|---|
| `VALIDATION_MATRIX.md` declara GFSK 2440 10/10 y PHY 7 PASS | **HISTORICAL CLAIM**; evidencia primaria no revisada y no compatible con la ruta actual. |
| La misma matriz llama proprietary 2.4 “parcial / GFSK pendiente” | Discrepancia interna documental. |
| Campaña propia del 8-oct: ambos presets, ACK, 0 paquetes | **CONTROL ONLY**; consistente con el defecto, pero no prueba por sí sola la causa RF. |

Clasificación de ambos presets: **NEEDS CODE FIX**. No ejecutar OTA hasta corregir el manejo ≥1 GHz, revisar overrides/setup de 2.4 GHz y obtener diagnóstico RF de frecuencia/configuración efectiva.

### 4.3 Switching y recuperación

| Transición | Fuente actual | Evidencia | Decisión |
|---|---|---|---|
| IEEE ↔ Sub-GHz FSK/GFSK no-OOK | `RF_MODE_MULTIPLE`, handle no-433 persistente y `RF_flush` + nuevo RadioSetup (`radio_if.c:1194-1251`) | **CODE VERIFIED**; claim histórico usa resets; nuestra secuencia cruzada no está validada | Elegible como RF-03, con `band1`/`band2` explícitos y sin reset durante el ciclo. |
| Cambios de preset 868/902/915 | Reconfiguración normal no-OOK | **OUR OTA VERIFIED** | No repetir. |
| IEEE ↔ proprietary 2.4 | Compartiría handle no-433 | **BLOCKED** por fallback a 868 MHz | No probar. |
| Sub-GHz ↔ proprietary 2.4 | Igual | **BLOCKED** por fallback a 868 MHz | No probar. |
| OOK → cualquier otro PHY | Patches MCE/RFE no descargables de forma segura | **CODE VERIFIED**, claim histórico de recovery | Power-cycle total obligatorio; OOK debe ser la última prueba. |
| 433 ↔ no-433 | Handle 433 separado y persistente | Claims históricos; hardware actual marginal | No incluir sin antena/ruta 433 adecuada. |

## 5. Revaluación de presets excluidos

El manual original los excluyó por lockup OOK, valores `mod_type` sin auditar, ruta/antena 433/169 incierta y patch MIOTY ausente. La fuente actual confirma o endurece esas exclusiones:

| Familia / presets | Clasificación | Justificación concisa |
|---|---|---|
| `ook_868_4k8` | **ELIGIBLE WITH PRECONDITIONS** | `mod_type=2` sí es válido; firmware carga patches genook. Sólo prueba aislada, última, con ruta 868 y power-cycle completo antes/después. Claim histórico 10/10 no revisado. |
| `ook_433_4k8`, `ook_433_2k4` | **NEEDS ADDITIONAL HARDWARE** | Además del lock OOK, la antena/matching 433 actual tiene margen insuficiente; histórico 0/10. |
| `gfsk_433_*`, `fsk_433_50k` | **NEEDS ADDITIONAL HARDWARE** | FSK/GFSK son valores válidos y existe setup 433, pero el enlace actual es marginal (claims 1–10/10) y no aislaría software de antena/ruta. |
| `msk_868_50k`, `msk_433_50k` | **NEEDS CODE FIX** | Se copia `mod_type=4`, pero `CMD_PROP_RADIO_DIV_SETUP` sólo define 0=FSK, 1=GFSK, 2=OOK; los demás valores están reservados en el SDK. |
| `4fsk_868_50k`, `4gfsk_868_50k` y variantes 433 | **NEEDS CODE FIX** | `mod_type=5/6` también es reservado; `format_conf` no convierte ese comando en 4-FSK/4-GFSK. Claims 10/10 no prueban la modulación física solicitada. |
| `wireless_mbus_n_169_*` | **NEEDS CODE FIX** | Existen presets/overrides, pero `<861 MHz` selecciona el setup/handle 433 mientras los overrides 169 se escriben en el setup compartido no usado por esa ruta. Después requerirá front-end/antena 169 demostrados. |
| `mioty_868_tsunb` | **DEFERRED** | El preset raw 396 baud no implementa TS-UNB; fuente e historial requieren patch CPE/RF Core y stack/framing adicional. No probar antes de implementarlos. |
| 13 presets FSK/GFSK 868–915 de octubre | **ALREADY VALIDATED** | Entrega raw #2→#3 10/10 por preset; no repetir salvo control explícito. |

## 6. Campaña mínima priorizada

Se proponen **cinco** pruebas. RF-05 es opcional por riesgo de recuperación; las cuatro primeras forman el mínimo de bajo riesgo.

### RF-01 — Switching de canal IEEE y aislamiento

| Campo | Plan |
|---|---|
| Funcionalidad / brecha | `set_channel()` real 25→20→25 dentro de la misma sesión; hasta ahora sólo se demostró canal 25. |
| Hardware / roles | #2 COM33 TX; #3 COM88 RX; ruta `band1`/2.4 GHz en ambos. |
| Script/API | API existente; helper mínimo nuevo basado en `ota_rx_probe.py` + `smoke_tx_phase1.py` para conservar objetos y cambiar canal. No implementar aún. |
| Configuración | PHY 4; 0 dBm; 10 TX por etapa; marcadores hex `C1012501` (control 25), `C1012502` (mismatch), `C1012001` (canal 20), `C1012503` (retorno 25). |
| Preparación | Verificar COM88; control 25; RX activo y lectura concurrente para evitar cola tardía. |
| PASS | ≥9/10 CRC-good en 25; 0 del marcador 25 mientras RX está en 20; ≥9/10 tras mover TX a 20; ≥9/10 al regresar ambos a 25; `RX_STOP` ACK. |
| BLOCKED | COM88 no enumera/inicializa, error RF asíncrono o tráfico previo con un marcador reutilizado. |
| Cleanup / complejidad / riesgo | `stop_rx()`, desconectar, dejar `band1`; **LOW–MEDIUM**. Riesgo: buffering, mitigado por marcadores y ventanas drenadas. |

### RF-02 — Sub-GHz en dirección inversa

| Campo | Plan |
|---|---|
| Funcionalidad / brecha | TX físico de #3 y RX físico de #2; octubre sólo cerró positivamente #2→#3. |
| Hardware / roles | #3 COM88 TX; #2 COM33 RX; Shells explícitos COM87/COM35. |
| Script/API | `manual_preset_runner.py`; cambio mínimo: parámetros `--tx-port/--rx-port` en lugar de constantes. |
| Configuración | `wisun_915_fsk_50k` (real 902.2 MHz), PHY 7, 0 dBm, 10 TX, marcador único `C20288003302`. |
| Preparación | `band2` lógico ambos; fresh `Radio`; conocido #2→#3 sólo como control si el inverso falla. |
| PASS | 10/10 ACK y 10/10 paquetes peer CRC-good con marcador, sin `RxStreamError`, `RX_STOP` ACK. |
| BLOCKED | COM88 no pasa `init()/get_stats()`, o no puede establecerse ruta/antena Sub-GHz. |
| Cleanup / complejidad / riesgo | `stop_rx()`, estadísticas, desconectar; **LOW**. No demuestra potencia ni protocolo Wi-SUN. |

### RF-03 — Recuperación cruzada IEEE → Sub-GHz → IEEE

| Campo | Plan |
|---|---|
| Funcionalidad / brecha | Hot-switch no-OOK y recuperación sin reset; la matriz histórica con reset describe un lifecycle anterior. |
| Hardware / roles | #2 TX / #3 RX, mismos objetos durante tres etapas; Shells explícitos. |
| Script/API | Adaptación mínima de `smoke_f9_phy_matrix_ota.py`: eliminar toda etapa excluida, añadir preset conocido y puertos Shell explícitos. |
| Configuración | IEEE ch25 / `band1`, marcador `C3032501`; luego `wisun_915_fsk_50k` / `band2`, marcador `C3039002`; regreso IEEE ch25 / `band1`, marcador `C3032503`; 10 TX por etapa. |
| Preparación | RF-01 y RF-02 PASS; sin OOK previo; `stop_rx()` antes de cada cambio; no reset dentro del ciclo. |
| PASS | ≥9/10 CRC-good en cada etapa, sin timeout/error asíncrono, y `RX_STOP` ACK en las tres. |
| BLOCKED | Cualquier handle/RF-init error, pérdida persistente tras una transición o Shell no verificable. |
| Cleanup / recuperación | Si falla: cerrar sesiones, reset CC por COM35/COM87 explícitos, `band1`, control IEEE fresco. **MEDIUM**. |

### RF-04 — Recepción pasiva Zigbee mediante KillerBee

| Campo | Plan |
|---|---|
| Funcionalidad / brecha | Adapter `KillerBeeFeralRF`, `pnext()` y escritura PCAP con RF real; aún sólo **CODE VERIFIED**. |
| Hardware / roles | Un CatSniffer RX; red Zigbee autorizada como fuente pasiva; el segundo board no es necesario. |
| Script/API | `python/examples/killerbee_sniff.py`; sin cambios si la dependencia opcional ya está instalada. |
| Configuración | Canal real de la red (históricamente 25), `--count 10 --pcap <archivo>`. No inyección. |
| Preparación | Confirmar canal desde la administración de la red; provocar una acción legítima sólo si el propietario lo autoriza. |
| PASS | ≥10 frames con `validcrc=True`, bytes no vacíos, PCAP abre como IEEE 802.15.4 y permite observar campos MAC coherentes. |
| BLOCKED | Canal desconocido sin forma autorizada de consultarlo, dependencia KillerBee ausente, o red sin tráfico durante la ventana. |
| Cleanup / complejidad / riesgo | `sniffer_off()` y cierre; **LOW**. Evidencia de captura IEEE/MAC, no asociación, descifrado ni interoperabilidad Zigbee. |

### RF-05 — OOK 868 aislado (opcional y último)

| Campo | Plan |
|---|---|
| Funcionalidad / brecha | Validar OTA `mod_type=2`; hoy sólo hay claim histórico 10/10. |
| Hardware / roles | #2 TX / #3 RX, antenas/ruta 868; capacidad de retirar toda alimentación de ambos. |
| Script/API | API `Radio`; helper mínimo derivado del runner manual, sin `reset_device()` aritmético y con puertos explícitos. |
| Configuración | `ook_868_4k8`, PHY 7, 0 dBm, 10 TX, marcador único `C5050001`; `band2` antes de abrir Bridge. |
| Preparación | Power-cycle total inicial; ejecutar después de todas las demás pruebas; no cambiar preset/PHY tras cargar OOK. |
| PASS | 10/10 paquetes peer con marcador y cierre documentado; luego power-cycle total y control IEEE fresco. |
| BLOCKED | No puede garantizarse retirada total de alimentación, antena/ruta 868 no confirmada, o aparece hang/error RF. No reintentar en caliente. |
| Cleanup / complejidad / riesgo | Cerrar host, retirar toda alimentación ≥15 s, reconectar/verificar puertos, `band1`, control IEEE; **HIGH** por lock de patches. |

## 7. Preparación y reglas de recuperación

1. No asumir COM88: enumerar por PNP/MI y confirmar #3 Bridge COM88 ↔ Shell COM87.
2. Registrar commit, import real de `feralrf`, `DeviceInfo`, Shell `status` y contadores antes de RF-01.
3. Usar siempre puertos Shell explícitos. `Radio.reset_device()` deriva Bridge+2 y produciría COM90 para COM88; no usarlo aquí.
4. Fresh `Radio` por prueba, salvo RF-01/RF-03 donde conservar sesión es la variable bajo prueba.
5. RX primero, lectura concurrente, marcadores únicos, 10 TX cortos a 0 dBm y sin reintento oculto.
6. ACK = control, nunca PASS-RF. PASS exige paquete peer, marcador y `crc_ok=True`.
7. Drenar ventanas antes/después; no usar tiempo de impresión host como tiempo RF.
8. Ante timeout: detener el test, guardar salida literal y no encadenar otra configuración.
9. Recuperación normal: cerrar procesos/puertos, `boot`→`exit` por COM35 y COM87 explícitos, espera ≥3.5 s, seleccionar banda y repetir sólo un control etiquetado.
10. Después de OOK: no intentar `set_phy()` ni `configure_prop()`; power-cycle total obligatorio.

## 8. Secuencia mínima de ejecución

1. Preflight de identidad/puertos, incluyendo COM88, sin transmitir.
2. RF-01: IEEE channel switching; deja un control IEEE conocido.
3. RF-02: Sub-GHz inverso; cierra simetría básica de las dos placas.
4. RF-03: IEEE→Sub-GHz→IEEE; prueba lifecycle y recuperación no-OOK.
5. RF-04: KillerBee pasivo sobre Zigbee; puede omitirse si canal/tráfico no son verificables.
6. Decidir go/no-go para RF-05. Si se autoriza, ejecutarlo último y terminar con power-cycle + control IEEE.

Detener la campaña si RF-01 no establece un control IEEE estable o si COM88 no pasa preflight. No usar RF-05 como diagnóstico de fallos anteriores.

## 9. Bloqueos y preguntas abiertas

- **Bloqueo de código:** proprietary 2.4 GHz cae a 868 MHz en ejecución; corregir y añadir readback/diagnóstico antes de OTA.
- **Bloqueo de código:** MSK/4-FSK/4-GFSK usan valores reservados del comando TI; requieren una implementación RF válida, no otro preset.
- **Bloqueo código + hardware:** 169 MHz usa incoherentemente la ruta/setup 433 y carece de front-end/antena demostrados.
- **Hardware adicional:** 433 MHz necesita antena/matching adecuados para separar un fallo de enlace de un fallo de firmware.
- **Implementación ausente:** MIOTY/TS-UNB requiere patch RF Core/CPE y framing; queda diferido.
- **Instrumentación adicional:** CW/PRBS, potencia, frecuencia, espectro y modulación no son verificables de forma concluyente con un segundo receptor de paquetes.
- **Procedencia:** sigue pendiente vincular el binario CC1352 instalado con un hash/commit reproducible.
- **Observabilidad:** BURST-count y STOP inmediato necesitan telemetría/corrección de scheduler/colas antes de otra prueba OTA.
- **Pregunta de operación:** confirmar que ambos boards tienen antena útil a 868/915 MHz y que un power-cycle completo es posible antes de autorizar RF-05.

Resultado recomendado: ejecutar primero sólo RF-01 a RF-04. RF-05 es la única ampliación de modulación técnicamente justificable sin cambiar firmware, pero su valor no compensa el riesgo si no se garantiza recuperación eléctrica completa.
