# Inventario de fuentes y versiones

> [!info] Papel de esta nota
> Este es un registro de procedencia, no la introducción conceptual. La interpretación pedagógica empieza en [[CatSniffer - Mapa conceptual]]; las fuentes especializadas están en [[Fuentes hardware CatSniffer v3.1]], [[Fuentes firmware oficial]], [[Fuentes herramientas PC]] y [[Fuentes FeralRF]].

Inventario realizado el **2026-09-25**. Esta nota fija la línea base local de la Fase 0; no identifica por sí sola la revisión de la placa física ni el firmware instalado.

## Convención de evidencia

- **Documentado:** afirmación explícita en documentación o metadatos de la fuente.
- **Observado en repositorio:** dato leído directamente de Git, nombres/rutas, archivos de versión o configuración.
- **Observado en hardware:** no hay evidencia física suministrada en esta fase.
- **Hipótesis:** interpretación pendiente de comprobación; no se usa para seleccionar una revisión.
- **Conclusión:** resultado respaldado por la evidencia enumerada.

## Rutas locales verificadas

Todas existen bajo `C:/Users/Support/Documents/ec-projects/catsniffer-feralrf/`; no se encontró discrepancia con el layout indicado.

| Función | Ruta |
| --- | --- |
| Hardware CatSniffer (solo lectura) | `CatSniffer/` |
| Firmware oficial (solo lectura) | `CatSniffer-Firmware/` |
| Herramientas de host (solo lectura) | `CatSniffer-Tools/` |
| Proyecto evaluado (solo lectura) | `FeralRF/` |
| Plan y documentación inicial | `docs/` |
| Vault de Obsidian, único destino de escritura | `CatSniffer-Understanding/` |

Documento controlador leído: `docs/CatSniffer-FeralRF-plan-de-trabajo-actualizado.md`, versión declarada **25 de septiembre de 2026**.

## Línea base de los repositorios

Los SHA siguientes son la referencia exacta para fases posteriores. Todas las ramas locales siguen a su rama remota visible y están `0` commits adelante / `0` atrás según las referencias locales; no se hizo `fetch`, por lo que no se confirma el estado actual del servidor.

| Repositorio | Remoto | Rama y tracking | HEAD | Fecha autor | Asunto | Estado inicial |
| --- | --- | --- | --- | --- | --- | --- |
| `CatSniffer/` | `origin`: `https://github.com/ElectronicCats/CatSniffer.git` | `master` → `origin/master` | `6da5050fefde59138fc11d9e2ecf7c0de36f3a22` (`6da5050`) | `2025-12-09T10:23:00-06:00` | `Merge pull request #91 from ElectronicCats/catsniffer` | limpio; sin archivos no rastreados |
| `CatSniffer-Firmware/` | `origin`: `https://github.com/ElectronicCats/CatSniffer-Firmware.git` | `v3.x` → `origin/v3.x` | `c0cd5a45e019dbd14ed11d039aacb13f300e5731` (`c0cd5a4`) | `2026-09-03T18:34:26-06:00` | `Merge pull request #20 from ElectronicCats/feat/samd21-catsniffer-v2` | limpio; sin archivos no rastreados |
| `CatSniffer-Tools/` | `origin`: `https://github.com/ElectronicCats/CatSniffer-Tools.git` | `main` → `origin/main` | `bc80979d7348bb038c3dd5d40be8aa51b90dd653` (`bc80979`) | `2026-09-04T13:33:26-06:00` | `Merge pull request #59 from ElectronicCats/ci/precommit-detached-checkout` | limpio; sin archivos no rastreados |
| `FeralRF/` | `origin`: `https://github.com/ElectronicCats/FeralRF.git` | `main` → `origin/main` | `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea` (`0178721`) | `2026-07-22T12:34:03-06:00` | `docs: fix remaining 'RF_open at boot' folklore in architecture layer rules` | **sucio antes del trabajo**: submódulo modificado; sin archivos no rastreados reportados en el repositorio padre |

### Ramas visibles localmente

- `CatSniffer`: rama local `master`. Ramas remotas visibles: `origin/master`, `origin/Docs`, `origin/Enlarged`, `origin/Internsupport-patch-1`, `origin/M.2`, `origin/Marcelol52-patch-3`, `origin/Marcelol52-patch-4`, `origin/Syscfg_template`, `origin/Update_Bom`, `origin/Update_DNP_Parts_Temp`, `origin/catsniffer`, `origin/docs`, `origin/new-case`, `origin/test_notebooks`, `origin/v3.2label`.
- `CatSniffer-Firmware`: rama local `v3.x`. Ramas remotas visibles: `origin/v3.x`, `origin/v1.x/v2.x`, `origin/Marcelo_advertiser`, `origin/Raul360-features-1`, `origin/cc1352_rawRepeater`, `origin/ci-test`, `origin/feat/samd21-catsniffer-v2`, `origin/freq_error`, `origin/gatts_poc`, `origin/pre_release`, `origin/rp2040_cjtag`, `origin/switcher`, `origin/sx1262-rssi`, `origin/uart-bridge-sdk`, `origin/update_github_actions`.
- `CatSniffer-Tools`: rama local `main`. Ramas remotas visibles: `origin/main`, `origin/dev`, `origin/experimental`, `origin/FSK`, `origin/cat_dissector`, `origin/feat/catsniffer-v2-support`, `origin/feature/restore-cc1352`, `origin/feature/vhci-bridge`, `origin/fix/CLI_Control`, `origin/lora_extcap`, `origin/pypip`, `origin/refactor`, `origin/remove-md`, `origin/ti_sniffer_full`, `origin/uf2`, `origin/update-programingTool`.
- `FeralRF`: rama local `main`. Ramas remotas visibles: `origin/main`, `origin/feature/f12-ble-scanner-active`, `origin/feature/f20a1-peripheral-read`, `origin/feature/f25-crypto-hw`, `origin/feature/f26-prop-24ghz`, `origin/feature/f29-stack-presets`, `origin/feature/f8b-track-a`, `origin/feature/killerbee-integration`, `origin/feature/macos-compat`, `origin/feature/remove-ble-protocol`, `origin/feature/smp-phase-a`, `origin/feature/ti-rtos-migration`.

### Tags y versiones relevantes

- `CatSniffer`: `v1.0`, `v1.2`, `v2.0`, `v3.0`, `v3.1`, `v3.2`, `v3.3`. `HEAD` coincide con el tag ligero `v3.3`.
- `CatSniffer-Firmware`: `HEAD` es el commit apuntado por los tags anotados `v3.1.0.1` (release para CatSniffer v3) y `v2.1.0.0` (release para CatSniffer v1/v2). También existen `v3.1.0.0`, `board-v3.x-v1.0.0`, `board-v3.x-v1.1.0`, `board-v3.x-v1.2.0`, `board-v3.x-v1.2.1`, `board-v3.x-v1.2.2`, `board-v3.x-v2.0.0` y `board-v2.x-v1.0.0`.
- Versiones internas observadas: `CatSniffer-Firmware/RP2040/catsniffer/VERSION` = `0.2.0.0-dual-mode`; `RP2040/blink/VERSION` = `1.0.0.0`; `SAMD21/catsniffer/VERSION` = `0.1.0.0samd21` según los campos del archivo.
- `CatSniffer-Tools`: último tag visible `v3.3.2.1` en `a06b7887a20693129ffb2ed567ed73284908d0a6`; `HEAD` está después del tag y no tiene tag. `catnip/VERSION` declara `3.3.2.1`. `catnip/README.md` conserva además texto desactualizado que dice “Current Version: v3.0.0”; `changelog.md` llama a V3.3.2 “unreleased”.
- `FeralRF`: no hay tags visibles. `python/pyproject.toml` declara el paquete `feralrf` versión `0.3.0`.

### Estado del submódulo de FeralRF

**Observado en repositorio:** `.gitmodules` fija `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01` al remoto `https://github.com/TexasInstruments/simplelink-lowpower-f2-sdk.git`. El gitlink y el checkout están en `5b31d0a4903351e544546e23ef3330eaa4291ceb` (`lpf2-8.30.01.01`), pero el checkout está sucio: `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md` aparece como modificado. `git diff --stat` no mostró diferencias textuales y Git avisó de conversión LF→CRLF; no se atribuye causa sin más evidencia. Esta condición era preexistente y no se corrigió.

## Estructura de alto nivel

### `CatSniffer`

- `hardware/`: fuentes KiCad, PDF de esquema, modelo STEP, librerías y documentos de referencia.
- `cases/`: modelos de cajas.
- `pcaps/`: capturas BLE, Thread y Zigbee.
- `poc/`: prueba de concepto Zigbee.
- `workshops/`: material, ejemplos, imágenes y binarios de talleres.
- `README.md`: descripción general, lista histórica de versiones y enlaces al Wiki/repositorios relacionados.

### `CatSniffer-Firmware`

- `RP2040/catsniffer/`, `RP2040/blink/`: proyectos Zephyr, configuraciones, boards, código y archivos `VERSION`.
- `RP2040/LEGACY/`: proyectos Arduino/Zephyr heredados y ejemplos LoRa/SX126x/passthrough.
- `SAMD21/catsniffer/`: firmware Zephyr para placas v1.x/v2.x, board definition, scripts y archivo `VERSION`.
- `SAMD21/zephyr-patches/`: parche local requerido por la documentación SAMD21.
- `CC1352P7/`: proyectos CCS/SysConfig y archivos HEX de sniffer, scanners y ejemplos.
- `.github/workflows/firmware-ci.yml`, `.github/workflows/firmware-release.yml`: configuración de build/release.
- `docs/superpowers/{plans,specs}/`: documentos de diseño/plan del port SAMD21.

### `CatSniffer-Tools`

- `catnip/`: CLI actual, módulos de firmware/protocolos/utilidades, instaladores, empaquetado y tests.
- `catsnifferTUI/`: interfaz de terminal y utilidades de descubrimiento/dispositivo.
- `Legacy/`: `cativity`, cargador Catnip anterior, `cc2538-bsl`, `pycatsniffer_bv3`, herramientas SX1262 y scripts históricos.
- `.github/workflows/`: builds para Arch, Debian, macOS y Windows, más tests.
- `README.md`, `changelog.md`, `catnip/README.md`, `catnip/InstallationGuide.md`: documentación local de herramientas.

### `FeralRF`

- `firmware/cc1352/`: build CMake, código, headers, linker scripts y configuración generada; `CMakeLists.txt` admite `CC1352P` y `CC1352P7`, con `CC1352P7` por defecto.
- `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/`: submódulo TI SimpleLink SDK.
- `python/feralrf/`: paquete host; `python/examples/` contiene smoke/release-gate/lab; `python/tests/` contiene tests unitarios/contratos.
- `docs/`: arquitectura, API Python, protocolo, matriz de validación, KillerBee y ejecución en Linux.
- `hardware/`: copia local de fuentes/dokumentación CatSniffer y `PINOUT.md`.
- `docker/Dockerfile`, `.github/workflows/build.yml`, `.github/workflows/release.yml`: entorno y automatización de build/release.

## Fuentes técnicas disponibles

La clasificación “primaria” significa diseño/código/configuración original o documentación del fabricante; no significa que aplique a la placa física del usuario.

| Fuente | Ruta o URL | Repositorio | Tipo / versión aparente | Clase | Uso futuro / ambigüedad |
| --- | --- | --- | --- | --- | --- |
| Plan de investigación | `docs/CatSniffer-FeralRF-plan-de-trabajo-actualizado.md` | `docs` | Plan v. 2026-09-25 | primaria para alcance | Controla fases y puerta de revisión. |
| Fuentes de placa CatSniffer | `hardware/CatSniffer.kicad_sch`, `hardware/CatSniffer.kicad_pcb`, `hardware/CatSniffer.kicad_pro` | CatSniffer | etiquetas conflictivas: esquema rev 2.0; PCB rev/silkscreen v3.2 | primaria de diseño | Fase 1, solo tras seleccionar revisión. |
| Esquema exportado | `hardware/CatSniffer.pdf` | CatSniffer | revisión no confirmada en Fase 0 | primaria de diseño | Fase 1; no se asume correspondencia física. |
| Modelo mecánico | `hardware/CatSniffer.step` | CatSniffer | sin revisión explícita inventariada | primaria/metadata insuficiente | Distinción física posterior. |
| Referencias locales | `hardware/Datasheet/0900FM15D0039.pdf`, `jti-an100.pdf`, `LAUNCHXL-CC1352P-2_SCHEMATIC.pdf`, `SX1262MB2xAS_e499v01b_sch_layout.pdf` | CatSniffer | filtro/app note, esquema TI LaunchPad y esquema/layout Semtech (según nombres) | fabricante o unclear hasta revisar portada | Fase 1; son referencias de componente/evaluation board, no esquema de la placa por sí solas. |
| Documentación general/Wiki | `README.md`; `https://github.com/ElectronicCats/CatSniffer/wiki` | CatSniffer | lista de versiones y enlaces | secundaria/documentada | Orientación; no valida hardware físico. |
| Firmware y builds | `RP2040/catsniffer/`, `SAMD21/catsniffer/`, `CC1352P7/`, `.github/workflows/firmware-*.yml` | CatSniffer-Firmware | Zephyr, CCS/SysConfig y HEX; tags indicados arriba | primaria de código/configuración | Fase 2. |
| Documentación de firmware | `README.md`, `RP2040/README.md`, `RP2040/catsniffer/README.md`, `SAMD21/catsniffer/README.md`, `CC1352P7/README.md` | CatSniffer-Firmware | variantes v1/v2 y v3 | secundaria junto a código | Afirmaciones de pruebas no fueron revalidadas en Fase 0. |
| Guía Zephyr 4.1 | `https://docs.zephyrproject.org/4.1.0/develop/getting_started/index.html` | enlace desde firmware | documentación de build | primaria externa | Fase 2/tooling. |
| Herramienta Catnip | `catnip/`, `catnip/VERSION`, `catnip/setup.py`, `catnip/README.md`, `catnip/InstallationGuide.md` | CatSniffer-Tools | paquete/CLI 3.3.2.1; README contiene texto 3.0.0 | primaria de código + documentación secundaria | Fase 3; resolver inconsistencia documental. |
| Herramientas heredadas | `Legacy/cc2538-bsl/`, `Legacy/pycatsniffer_bv3/`, `Legacy/catnip_uploader/`, `Legacy/sx1262Tools/` | CatSniffer-Tools | legado | primaria de código | Fase 3; no asumir vigencia. |
| Firmware FeralRF | `firmware/cc1352/`, `firmware/cc1352/CMakeLists.txt` | FeralRF | CMake; TI SDK 8.30.01.01; targets P/P7 | primaria de código/configuración | Fase 4. |
| API host FeralRF | `python/feralrf/`, `python/pyproject.toml` | FeralRF | paquete 0.3.0, Python ≥3.9 | primaria de código/configuración | Fase 4. |
| Docs y pruebas FeralRF | `docs/ARCHITECTURE.md`, `docs/PYTHON_API.md`, `docs/protocol.md`, `docs/VALIDATION_MATRIX.md`, `docs/TESTING-ON-LINUX.md`, `docs/killerbee-catsniffer.md`, `python/tests/`, `python/examples/` | FeralRF | docs, tests y ejemplos | mixto | Inventariados, no evaluados ni validados en esta fase. |
| Pinout FeralRF | `hardware/PINOUT.md` | FeralRF | referencia CatSniffer sin revisión de placa explícita | secundaria | No usar como prueba para otra revisión. |
| Copia de hardware en FeralRF | `hardware/CatSniffer.*`, `hardware/Datasheet/` | FeralRF | PCB rev v3.2 con silkscreen v3.1; esquema rev 2.0 | primaria/derivada, procedencia exacta no documentada | No mezclar con la fuente `CatSniffer/`; los archivos de diseño difieren. |
| TI SimpleLink SDK | `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/`; `https://github.com/TexasInstruments/simplelink-lowpower-f2-sdk.git` | FeralRF | `lpf2-8.30.01.01`, commit `5b31d0a...` | primaria de fabricante | Submódulo local sucio; debe preservarse hasta aclarar. |

No se encontró un BOM claramente nombrado ni un conjunto de Gerbers presente como salida en el checkout actual de `CatSniffer`; `CatSniffer.kicad_pro` sí contiene nombres de directorio de salida Gerber. Esta ausencia es un resultado de inventario, no prueba de que esos artefactos no existan en otras ramas/tags o externamente.

## Revisiones y variantes identificadas

| Identificador | Evidencia concreta | Estado |
| --- | --- | --- |
| v1.0 | tag Git `CatSniffer:v1.0` | observado en repositorio; revisión física no asociada |
| v1.2 | tag `CatSniffer:v1.2`; `CatSniffer/README.md` | observado/documentado |
| v1.3 | `CatSniffer/README.md` | documentado; no hay tag homónimo visible |
| v2.0 | tag `CatSniffer:v2.0`; `CatSniffer/README.md` | observado/documentado |
| v2.1 | `CatSniffer/README.md` | documentado; no hay tag de hardware homónimo visible; no confundir con firmware `v2.1.0.0` |
| v3.0 | tag `CatSniffer:v3.0` | observado en repositorio |
| v3.1 | tag `CatSniffer:v3.1`; `CatSniffer/README.md` | observado/documentado |
| v3.2 | tag `CatSniffer:v3.2`; `CatSniffer/hardware/CatSniffer.kicad_pcb` rev y silkscreen `v3.2` | observado; no prueba placa física |
| v3.3 | tag `CatSniffer:v3.3` en HEAD | observado; los archivos de diseño actuales no llevan una etiqueta coherente v3.3 |
| familia v1.x/v2.x | `CatSniffer-Firmware/README.md`: SAMD21 + CC1352P1; `SAMD21/catsniffer/` | documentado/observado en repositorio |
| familia v3.x+ | `CatSniffer-Firmware/README.md`: RP2040 + CC1352P7; `RP2040/catsniffer/`, `CC1352P7/` | documentado/observado en repositorio |
| targets FeralRF P/P7 | `FeralRF/firmware/cc1352/CMakeLists.txt` | observado en configuración; no demuestra compatibilidad física ni funcionamiento |

### Ambigüedades de revisión

- El commit/tag `CatSniffer:v3.3` contiene `hardware/CatSniffer.kicad_pcb` con `rev "v3.2"` y silkscreen v3.2, mientras `hardware/CatSniffer.kicad_sch` tiene `rev "2.0"`. No se infiere equivalencia entre esos números.
- En `FeralRF/hardware/`, el PCB declara rev v3.2 pero muestra texto de silkscreen v3.1; su esquema declara rev 2.0. Sus fuentes KiCad no son idénticas a las de `CatSniffer/`. El PDF `CatSniffer.pdf` sí es idéntico por SHA-256 (`1829805519F0D830A5707B390575F82BF329E67781A09D7CB704E2AEC0C29431`), lo que tampoco establece qué revisión física representa.
- El README general enumera v1.3 y v2.1, pero las referencias Git locales de `CatSniffer` no incluyen tags homónimos.
- Los tags de firmware, las versiones internas de proyectos y la revisión de placa son espacios de versión distintos y no deben equipararse.
- **Revisión de la placa física: pendiente de confirmación del usuario.** No hay evidencia clasificada como “observado en hardware”.

## Preguntas de versión pendientes

1. ¿Qué texto exacto de versión/revisión aparece en la serigrafía de ambas caras de la placa física y en cualquier etiqueta/empaque?
2. ¿Hay fotografías legibles de ambas caras, conectores y etiquetas que permitan vincularla a una revisión?
3. ¿Qué firmware y versión están instalados actualmente en RP2040 o SAMD21, CC1352P/P7 y cualquier firmware asociado al SX1262?
4. ¿La modificación local del submódulo TI en `Board.md` es intencional o solo un efecto de finales de línea?
5. ¿Qué fuente debe considerarse canónica si la placa resulta ser v3.3 pero los archivos actuales se etiquetan v3.2/2.0?
