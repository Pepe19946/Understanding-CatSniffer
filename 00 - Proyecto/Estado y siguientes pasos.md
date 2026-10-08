# Estado y siguientes pasos

## Actualización OTA proprietary — 2026-10-08

- [[Reporte OTA Sub-GHz - GFSK 868 y 915 MHz]] y [[Segunda campaña OTA proprietary FeralRF — Ampliación de presets y verificación del procedimiento]] acumulan 24 corridas, 240 retornos exitosos de `transmit()`, 19 presets únicos y cero paquetes/hits proprietary entregados; no equivalen a 240 emisiones.
- El control concurrente confirma que la lectura RX estuvo activa durante los ACK TX y continuó `24.690 s` tras el último. La lectura tardía del helper pierde fuerza como explicación única, sin validar RF.
- Los casos GFSK/FSK válidos abarcan 868/902/915/2440 MHz. Los nombres MSK/4FSK/4GFSK no demuestran esas modulaciones porque el setup auditado reserva `modType` 4/5/6.
- Los ocho presets 433 y W-MBus N169 no se ejecutarán en esta campaña; OOK y MIOTY quedan diferidos. La expansión ciega de presets está pausada.
- Próximo paso: una observación física discriminante con `gfsk_868_50k`, niveles CTF1/2/3, selección U2 por tabla de verdad y energía RF en J1. Véase [[Seguimiento documental OTA Sub-GHz - ruta RF y control de banda#Siguiente observación física discriminante]].

## Foco inmediato corregido — 2026-10-07

> **Immediate validation focus: physical TX/RX of known RF frames/bytes using two CatSniffer V3 boards running FeralRF. ACK/control-only results do not count as RF validation. Higher-layer stacks and unrelated capabilities are deferred unless directly useful to this objective.**

- La campaña inmediata usa dos CatSniffer V3: `COM33` (TX inicial) → RF → `COM88` (RX inicial), y repite la línea base en sentido inverso. Los Shell son explícitamente `COM35` para `COM33` y `COM87` para `COM88`; no se permite inferir Shell como `Bridge+2`.
- La fuente autoritativa de conocimiento e historial es este Vault, `Understanding-CatSniffer`, junto con el HEAD actual de `FeralRF`. La guía ejecutable es [[Guía enfocada OTA FeralRF - TX RX con dos CatSniffer]] y la hoja de banco es [[Checklist OTA FeralRF - dos CatSniffer]].
- Una ejecución anterior de Codex usó por error el Vault obsoleto `CatSniffer-Understanding`. La guía OTA enfocada y el checklist generados allí **no son autoritativos** para el estado actual; no deben usarse ni fusionarse sin revalidación independiente.
- Se conserva el plan amplio y todo el historial que sigue. Este cambio reduce únicamente la prioridad inmediata: demostrar bytes/tramas conocidos sobre el aire, separar ACK de evidencia RF y registrar cada dirección sin promediar asimetrías.

## Preparación de evaluación física — 2026-10-01

- Se crearon [[FeralRF - Guía de validación experimental]] y [[FeralRF - Matriz de pruebas]] en `05 - Evaluación`.
- Baseline host aportado: Windows, Python 3.14.7, pytest 9.1.1, `python -m pytest -rs` → 422 passed y 1 skipped por dependencia opcional KillerBee; el fallo de `pytest` simple se conserva como observación de reproducibilidad.
- Se conservan 38 casos (`EV-00`…`EV-54`, con numeración por nivel) y ahora 35 registros `KI`; **ninguno fue ejecutado físicamente**. KI-33–35 documentan identidad Catnip no persistente/fallback COM, auto-flash y ausencia de TI sniffer V2 provisionable, y compatibilidad FeralRF-V2 no establecida.
- Hardware actual: `DUT-V3-FERAL` (V3/P7/RP2040) más `OBS-V2-A-STOCK` y `OBS-V2-B-STOCK` (V2/P1/SAMD21), que se preservan como referencias conocidas-buenas. Fuente disponible: `PROTOCOL-DEVICE-ZIGBEE-CH25`, tráfico Zigbee continuo en canal 25.
- CatSniffer-Tools se revalidó en la rama local solicitada `fix/CLI_control`, HEAD `126f13bc0441ad3f37fe0b029160a3c526e4d309` (2026-09-28), tres commits detrás del upstream y con el archivo no rastreado preexistente `py`. La implementación agrupa interfaces USB y mapea Bridge/LoRa/Shell antes de recurrir al orden COM; `devices --debug`, `identify` y `status` sustituyen la inspección ciega, aunque FeralRF aún calcula Shell=`Bridge+2` internamente.
- Prioridad inmediata: Catnip+Windows para identidad/puertos, init/info/stats, RX start/stop, reconnect, reset sólo tras verificar Shell, y EV-05 contra la fuente Zigbee CH25. Después se separan OTA simétrica (`AUX-V3-FERAL`) y referencia independiente (`AUX-V3-STOCK`/instrumento/protocol device).
- **FeralRF on CatSniffer V2: compatibility not established; do not flash as part of the current baseline.** No existe target explícito P1/SAMD21; ambos V2 permanecen stock.
- Bloqueadores principales: estado U2/CTF, hashes de firmware, llegada/rol del segundo V3, equipo RF y autorización de bandas. La ausencia de segunda V3 ya no bloquea EV-05 ni EV-43 por existir tráfico Zigbee CH25.
- Repositorios fuente permanecen fuera del alcance de escritura; FeralRF conserva el submódulo TI sucio preexistente.

## Trabajo transversal — refactor pedagógico

- **Fecha:** `2026-09-28`.
- **Naturaleza:** reorganización teórica del Vault; no constituye una nueva fase y no cambia la validación técnica de Fases 0–3 ni el estado pendiente de Fase 4.
- **Entrada principal creada:** [[CatSniffer - Mapa conceptual]].
- **Secuencia de aprendizaje:** [[Ruta de estudio - De CatSniffer a FeralRF]].
- **Resultado:** las notas de hardware, firmware, herramientas y FeralRF conservan evidencia y detalle, pero ahora están conectadas mediante explicaciones introductorias, modelos mentales, diagramas, errores comunes y preguntas de comprensión.
- **Restricciones respetadas:** sin cambios en los cuatro repositorios, builds, flashing, hardware, benchmarks o experimentos.

## Estado actual — Fase 4

- **Fase actual:** `Fase 4 — FeralRF`
- **Estado de revisión:** `pendiente`
- **Fecha:** `2026-09-28`
- **Fases previas:** Fases 0, 1, 2 y 3 validadas por el usuario para el alcance actual.
- **Baseline FeralRF:** `FeralRF/`, rama `main`, upstream local `origin/main`, HEAD `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`, fecha `2026-07-22T12:34:03-06:00`, asunto `docs: fix remaining 'RF_open at boot' folklore in architecture layer rules`; sin tags visibles; Python `0.3.0`.
- **SDK:** submódulo `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/` en `5b31d0a4903351e544546e23ef3330eaa4291ceb` (`lpf2-8.30.01.01`). Se preservó su estado sucio preexistente.
- **Baseline hardware:** CatSniffer v3.1 física; U8 observado `CC1352P74`/familia P7; RP2040 oficial como endpoint USB/puente; SX1262 como periférico del RP2040 en la arquitectura oficial. Sigue documentada la discrepancia histórica P1/P7.
- **Notas completadas/verificadas:** `04 - FeralRF/Arquitectura FeralRF.md`, `04 - FeralRF/Protocolo y API Python.md`, `04 - FeralRF/Matriz de capacidades.md`, `04 - FeralRF/Pruebas y evidencia existente.md`, `90 - Recursos/Fuentes FeralRF.md`; esta nota fue actualizada. Los borradores de la ejecución interrumpida se revalidaron contra HEAD y se corrigieron en lugar de duplicarlos.
- **Arquitectura establecida:** FeralRF sustituye la imagen oficial binaria del CC1352P7 por firmware fuente-visible para ese SoC. Reutiliza el firmware oficial/compatible del RP2040 como Cat-Bridge USB CDC↔UART a 921600 y usa Cat-Shell solo para reset/boot o flasheo externo; no sustituye el RP2040, no usa Cat-LoRa y no controla SX1262.
- **Runtime/protocolo:** target predeterminado CC1352P7 con ARM GCC, TI SDK 8.30.01.01 y TI-RTOS7/SYSBIOS; dos tasks principales (RF y UART), callbacks TI RF y estructuras estáticas. Host y firmware usan frames COBS delimitados por `00`, CRC16-CCITT y request/response/eventos propios.
- **Capacidades confirmadas por código:** sesión/info/stats, configuración PHY/propietaria, RX continuo, TX raw/frame/burst/continuous, modos CW/PRBS, IEEE 802.15.4 raw, BLE PHY raw, Sub-1/2.4 proprietary, crypto TI, presets/emulación PHY y adapter KillerBee. No son stacks completos Zigbee/Thread/BLE/W-MBus/Wi-SUN/Sidewalk.
- **Huecos principales:** spectrum no tiene data path; BLE active scan/conexión/GATT, RSA, jamming reactivo/pattern y SX1262 no están implementados end-to-end; FeralRF no incorpora updater propio.
- **Evidencia física:** `docs/VALIDATION_MATRIX.md` reporta corridas fechadas y conteos, pero no adjunta logs crudos, hashes de binarios/commits por corrida, estado CTF ni configuración completa. No se reprodujo ninguna prueba en esta fase y varias afirmaciones requieren revalidación contra HEAD.
- **Contradicciones/riesgos prioritarios:** el selector U2/`CTF1..3` de CatSniffer está controlado por RP2040, mientras FeralRF configura DIO28/29/30 como switch tipo LaunchPad y su API no manda `band1/2/3`; la ruta externa de RF queda sin establecer. `docs/ARCHITECTURE.md` declara `RF_open` lazy, pero `main_rtos.c` abre 433 MHz en boot. El ACK de TX confirma aceptación, no terminación RF; descubrimiento de puertos y `Bridge+2` para Shell son supuestos frágiles.
- **Preguntas para Fase 5:** fijar estado/control de U2/CTF por banda; congelar hashes de RP2040, HEX FeralRF, Python y SDK; repetir control/OTA con la placa v3.1/P74; comprobar enumeración/reset; observar errores asíncronos, cambios de PHY, throughput y drops; validar KillerBee real y modos físicamente reclamados.
- **Limitaciones:** análisis estático; sin build, tests, instalación, flasheo, radio ni hardware. CI solo prueba Python y configura CMake best-effort; no demuestra que el firmware compile o funcione físicamente.
- **Verificación de repositorios:** al inicio y al final, `CatSniffer`, `CatSniffer-Firmware` y `CatSniffer-Tools` estaban limpios. `FeralRF` conservó únicamente ` m firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01`; dentro, solo `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md` mostró `M`, mientras `Board.html` no apareció modificado. `git diff --name-only`, `--raw` y `--numstat` no produjeron un cambio textual y sí la advertencia LF→CRLF para `Board.md`. No se alteró ningún repositorio ni submódulo.

## Siguiente acción permitida tras Fase 4

**Esperar la revisión del usuario y ChatGPT. No comenzar la Fase 5 hasta que la Fase 4 sea validada explícitamente.**

## Registro histórico de Fase 3

### Estado de Fase 3

- **Fase actual:** `Fase 3 — Herramientas y comunicación con la PC`
- **Estado de revisión:** `validada por el usuario para el alcance actual`
- **Fecha:** `2026-09-25`
- **Fases previas:** Fases 0, 1 y 2 validadas por el usuario para el alcance actual.
- **Línea base de herramientas:** `CatSniffer-Tools/`, rama `main`, HEAD `bc80979d7348bb038c3dd5d40be8aa51b90dd653`, fecha `2026-09-04T13:33:26-06:00`; árbol limpio. HEAD es `v3.3.2.1-8-gbc80979`, mientras `catnip/VERSION` conserva `3.3.2.1`.
- **Línea base firmware para contraste:** `CatSniffer-Firmware/`, rama `v3.x`, HEAD `c0cd5a45e019dbd14ed11d039aacb13f300e5731`; árbol limpio.
- **Notas creadas:** `03 - Herramientas/Mapa de herramientas y comunicación PC.md`, `03 - Herramientas/Catnip CLI.md`, `03 - Herramientas/Protocolos host-dispositivo.md`, `90 - Recursos/Fuentes herramientas PC.md`.
- **Nota actualizada:** esta nota.
- **Relaciones establecidas:** Cat-Shell es control ASCII interpretado por RP2040; Cat-Bridge es passthrough CDC↔UART hacia CC1352P7; Cat-LoRa transporta captura/datos SX1262, mientras su configuración normal se manda por Cat-Shell.
- **Captura establecida:** TI 802.15.4 usa trama binaria por Bridge y PCAP/Wireshark; Sniffle/BLE delega a `sniffle_extcap`; LoRa/FSK usa configuración Shell, líneas RX por LoRa y PCAP 148.
- **Programación establecida:** RP2040 entra a ROM BOOTSEL y recibe UF2 por volumen `RPI-RP2`; CC1352P7 entra al bootloader ROM por control Shell y se programa por Bridge a 500000, regresando a runtime 921600.
- **Contradicciones:** documentación de versión obsoleta/inconsistente; `lora_extcap.py` llama `apply()` inexistente y manda un keepalive incompatible con stream; IDs oficiales `catnip_v3`/`catsniffer_v3` no coinciden; captura SX limita a 40 bytes impresos.
- **Limitaciones:** análisis estático, sin hardware, RF, ejecución de tests, instalación de dependencias o comprobación de reconexión; la imagen normal CC sigue siendo binaria y su implementación interna no es visible.
- **Preguntas abiertas:** no hay una pregunta bloqueante para el análisis estático de Fase 4; son útiles el SO/Wireshark objetivo, si habrá validación física y qué imágenes están instaladas.
- **Verificación final de repositorios:** `CatSniffer`, `CatSniffer-Firmware` y `CatSniffer-Tools` permanecen limpios. `FeralRF` conserva exactamente ` m firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01`; dentro solo `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md` figura `M`, `Board.html` no figura modificado, el diff textual/numstat está vacío y Git advierte conversión LF→CRLF. No se alteró ningún repositorio.

## Siguiente acción permitida tras Fase 3

**Esperar la revisión del usuario y ChatGPT. No comenzar la Fase 4 hasta que la Fase 3 sea validada explícitamente.**

## Registro histórico de Fase 2

### Estado de Fase 2

- **Fase actual:** `Fase 2 — Firmware oficial`
- **Estado de revisión:** `validada por el usuario para el alcance actual`
- **Fecha:** `2026-09-25`
- **Línea base firmware:** `CatSniffer-Firmware/`, rama `v3.x`, HEAD `c0cd5a45e019dbd14ed11d039aacb13f300e5731`, tags en HEAD `v2.1.0.0` y `v3.1.0.1`; árbol limpio.
- **Target físico:** **observado en hardware** U8=`CC1352P74`; se sigue CC1352P7-family. Se preserva la contradicción con `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch`, que dice `CC1352P1F3RGZT`.
- **Notas creadas:** `02 - Firmware/Mapa de firmware oficial.md`, `02 - Firmware/Firmware RP2040.md`, `02 - Firmware/Firmware CC1352P7.md`, `90 - Recursos/Fuentes firmware oficial.md`.
- **Notas actualizadas:** `01 - Hardware/Arquitectura CatSniffer v3.1.md`, `90 - Recursos/Fuentes hardware CatSniffer v3.1.md`, esta nota.
- **Relaciones confirmadas:** RP2040 ejecuta Zephyr, termina tres CDC USB, sirve de puente UART transparente al CC1352P7 y controla el SX1262 mediante API Zephyr→SPI/GPIO; CC1352P7 ejecuta una imagen independiente; SX1262 no recibe firmware de proyecto independiente.
- **Build confirmado:** RP2040 usa west/CMake/Kconfig y target `rpi_pico`; CC usa CCS/SysConfig/SimpleLink/TI-RTOS(7), con targets P7 explícitos y proyectos que dependen de distintas versiones de SDK.
- **Limitación principal:** el sniffer multiprotocolo normal está presente como `sniffer_fw_Catsniffer_v3.x.hex`, pero su fuente fue eliminada; sus internos no pueden establecerse desde HEAD. Los proyectos P7 con fuente son aplicaciones BLE especializadas y no sustituyen esa evidencia.
- **Preguntas no resueltas:** imágenes realmente instaladas; protocolo interno del HEX normal; revisión exacta del fork Zephyr referida sólo por rama; historia P1→P7; validación de boot/programación en hardware.
- **Potenciales problemas observados, no evaluados:** habilitación USB antes de inicializar rings; callback de shell con operaciones potencialmente bloqueantes; selección inicial `GIG` que puede no escribir `CTF1..3`; retornos/drops ignorados; ausencia de control visible para `ANT_SW`/`DIO22`.
- **Verificación final de repositorios:** `CatSniffer`, `CatSniffer-Firmware` y `CatSniffer-Tools` limpios. `FeralRF` conserva exactamente el estado preexistente ` m firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01`; dentro, sólo `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md` figura `M`, sin diff textual y con advertencia LF→CRLF. No se restauró, generó ni modificó nada en los repositorios.

## Siguiente acción permitida tras Fase 2

**Esperar la revisión del usuario y ChatGPT. No comenzar la Fase 3 hasta que la Fase 2 sea validada explícitamente.**

## Registro histórico de Fase 1

### Estado de Fase 1

- **Fase actual:** `Fase 1 — Hardware`
- **Estado de revisión:** `validada por el usuario para el alcance orientado a firmware`
- **Fecha:** `2026-09-25`
- **Línea base CatSniffer v3.1:** tag ligero `v3.1` → `5b99d984933a8b8c36e789853c08d3a2a785bed4`, fecha `2024-01-10T19:07:43-05:00`.
- **Notas creadas:** `01 - Hardware/Arquitectura CatSniffer v3.1.md`; `90 - Recursos/Fuentes hardware CatSniffer v3.1.md`.
- **Notas actualizadas:** `00 - Proyecto/Alcance y preguntas.md`; esta nota.
- **Confirmado:** P1 USB-C conecta directamente con U3 RP2040; U3 se conecta por UART/control/debug con U8 CC1352 y por SPI/GPIO con U7 SX1262; U3 controla los switches RF U2/U6; U8 y U3 ejecutan firmware, U7 no.
- **Incertidumbre prioritaria:** fuentes KiCad v3.1 indican U8=`CC1352P1F3RGZT`, mientras el firmware actual para 3.x está organizado bajo `CC1352P7/`.
- **Otras limitaciones:** sin inspección física; esquema y PCB contienen etiquetas/revisiones internas divergentes; J2 SWD está en esquema pero no en PCB; no se verificó conducta de firmware ni se compiló/flasheó.
- **Preguntas sin resolver:** marcaje físico y variante real de U8; existencia de una corrección/BOM/ECO de ingeniería; acceso SWD real del RP2040; imágenes instaladas actualmente.
- **Verificación de repositorios:** `CatSniffer`, `CatSniffer-Firmware` y `CatSniffer-Tools` limpios; `FeralRF` conserva solo el estado preexistente ` m firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01`. Dentro del submódulo, Git marca `Board.md`; `Board.html` no está modificado. Ninguno fue alterado durante esta fase.

## Siguiente acción permitida tras Fase 1

**Esperar la revisión del usuario y ChatGPT. No comenzar la Fase 2 hasta que la Fase 1 sea validada explícitamente.** Esta instrucción reemplaza la siguiente acción histórica de Fase 0 conservada más abajo.

## Registro histórico de Fase 0

## Estado auditable

- **Fase actual:** `Fase 0 — Inventario y preparación`
- **Estado de revisión:** `pendiente`
- **Fecha del inventario:** `2026-09-25`
- **Documento controlador:** `docs/CatSniffer-FeralRF-plan-de-trabajo-actualizado.md`
- **Alcance ejecutado:** inventario local y preparación del Vault; sin análisis de Fase 1.

## Línea base congelada

| Repositorio            | Rama     | Tracking        | HEAD                                       | Estado                                                 |
| ---------------------- | -------- | --------------- | ------------------------------------------ | ------------------------------------------------------ |
| `CatSniffer/`          | `master` | `origin/master` | `6da5050fefde59138fc11d9e2ecf7c0de36f3a22` | limpio                                                 |
| `CatSniffer-Firmware/` | `v3.x`   | `origin/v3.x`   | `c0cd5a45e019dbd14ed11d039aacb13f300e5731` | limpio                                                 |
| `CatSniffer-Tools/`    | `main`   | `origin/main`   | `bc80979d7348bb038c3dd5d40be8aa51b90dd653` | limpio                                                 |
| `FeralRF/`             | `main`   | `origin/main`   | `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea` | sucio antes y después: submódulo TI marcado modificado |

El submódulo de FeralRF permanece en `5b31d0a4903351e544546e23ef3330eaa4291ceb` (`lpf2-8.30.01.01`, detached HEAD). Su ruta modificada es `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md`; no se observó diff textual y hubo advertencia de finales de línea.

## Notas creadas o actualizadas

- `00 - Proyecto/Inventario de fuentes y versiones.md`
- `00 - Proyecto/Alcance y preguntas.md`
- `00 - Proyecto/Estado y siguientes pasos.md`

Carpetas mínimas verificadas/creadas: `00 - Proyecto`, `01 - Hardware`, `02 - Firmware`, `03 - Herramientas`, `04 - FeralRF`, `05 - Evaluación`, `06 - Experimentos`, `90 - Recursos`.

## Hechos confirmados

- Las seis rutas solicitadas existen y coinciden con el layout indicado.
- Los cuatro clones tienen remoto `origin`, rama activa y upstream localmente visible; ninguna está adelantada o atrasada respecto de su referencia remota local.
- `CatSniffer`, `CatSniffer-Firmware` y `CatSniffer-Tools` estaban y quedaron limpios.
- `FeralRF` estaba y quedó sucio únicamente a nivel del repositorio padre por el submódulo TI modificado; dentro del submódulo se reporta un `M` en `Board.md`.
- Hay fuentes KiCad, PDF de esquema, STEP y cuatro PDFs de referencia en `CatSniffer/hardware/`; no se encontró BOM o conjunto de Gerbers claramente presente.
- Las revisiones/versiones visibles incluyen v1.0, v1.2, v1.3, v2.0, v2.1, v3.0, v3.1, v3.2 y v3.3, con distinta fuerza de evidencia (tags frente a README).
- El hardware actual del checkout presenta etiquetas contradictorias entre tag v3.3, PCB v3.2 y esquema rev 2.0. La copia en FeralRF añade otra variante documental con silkscreen v3.1.
- No hay evidencia física que identifique la placa del usuario.

## Preguntas abiertas prioritarias

1. Revisión objetivo acordada para la investigación: CatSniffer v3.1.
2. Fuente canónica aplicable si la serigrafía no coincide con los archivos versionados.
3. Firmware instalado actualmente en cada procesador relevante.
4. Disponibilidad de programadores/debuggers y acceso de laboratorio.
5. Expectativas, criterios de aceptación y fechas del asesor.
6. Intención de la modificación local del submódulo TI.

El detalle y la categorización están en `00 - Proyecto/Alcance y preguntas.md`.

## Limitaciones del inventario

- No hubo acceso de red ni actualización de referencias remotas.
- No se inspeccionó una placa física ni se ejecutó tooling sobre ella.
- No se validaron afirmaciones de README, matrices de prueba, binarios ni capacidades anunciadas.
- No se realizó análisis de arquitectura o trazado de señales.
- No se abrieron ramas/tags alternativos; solo se inventariaron referencias visibles.
- Los metadatos de revisión internos son inconsistentes y no permiten seleccionar hardware con seguridad.

## Siguiente acción permitida

**Esperar la revisión del usuario y ChatGPT de la Fase 0. No comenzar la Fase 1 hasta que la Fase 0 sea validada explícitamente.**
