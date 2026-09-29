# Fuentes FeralRF

Registro verificado nuevamente el `2026-09-28` después de la ejecución interrumpida de Fase 4.

**Conocimiento derivado:** [[Arquitectura FeralRF]], [[Protocolo y API Python]], [[Matriz de capacidades]] y [[Pruebas y evidencia existente]].

## Baseline Git

- Repositorio: `FeralRF/`
- Rama: `main` → `origin/main` visible localmente
- HEAD: `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`
- Fecha: `2026-07-22T12:34:03-06:00`
- Asunto: `docs: fix remaining 'RF_open at boot' folklore in architecture layer rules`
- Tags: ninguno visible.
- Python: `python/pyproject.toml` y `python/feralrf/__init__.py` = `0.3.0`.
- Estado: repositorio padre sucio únicamente por submódulo TI preexistente.

## SDK/submódulo

- Ruta: `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/`
- Commit: `5b31d0a4903351e544546e23ef3330eaa4291ceb`
- Descripción: `lpf2-8.30.01.01`, detached HEAD.
- Estado preexistente: `source/ti/boards/CC26X2R1_LAUNCHXL/docs/Board.md` figura `M`; `Board.html` no.
- Diff textual/numstat: vacío, con advertencia LF→CRLF.
- El submódulo abierto no incluye todas las librerías precompiladas requeridas; CMake permite `TI_SDK_FULL` apuntando al instalador completo.

## Build/target

- `firmware/cc1352/CMakeLists.txt`: target P7 default, TI-RTOS7 default, toolchain ARM GCC, fuentes y artefactos.
- `firmware/cc1352/linker/cc1352p7.ld`: memoria/entry P7.
- `firmware/cc1352/linker/cc1352p.ld`: alternativa CC1352P de otra subfamilia.
- `firmware/cc1352/ccfg.c`: CCFG/bootloader.
- `firmware/cc1352/syscfg/`: configuración TI drivers/radio/SYSBIOS.
- `docker/Dockerfile`: entorno host declarativo; no aporta libs completas TI.
- `.github/workflows/{build.yml,release.yml}`: CI Python y release.

## Firmware clave

| Ruta | Función |
|---|---|
| `firmware/cc1352/src/main_rtos.c` | entry app, tasks y boot init actual |
| `src/host_if.c`, `src/host_if_task.c` | UART y framing entrante |
| `src/protocol.c`, `include/protocol.h` | COBS/CRC/IDs/límites |
| `src/command_processor.c` | validación y dispatch |
| `src/control_task.c` | estado/config/TX/jam/info |
| `src/data_task.c` | comandos diferidos, RX y eventos |
| `src/output_if.c`, `src/packet_queue.c` | serialización y cola host |
| `src/phy_manager.c` | tabla PHY/LL |
| `src/radio_if.c` | TI RF driver, RX/TX, SmartRF y callbacks |
| `src/crypto_engine.c` | TI drivers TRNG/AES/SHA/ECDH/ECDSA |
| `src/ll_manager.c` | clasificación BLE PDU raw |
| `include/config.h` | UART, buffers y límites jam |
| `syscfg/ti_drivers_config.c` | drivers y callback DIO28/29/30 |

`src/main.c` es la alternativa NoRTOS. `startup/main_rtos.c` no figura en el conjunto de fuentes actual; no se usó para describir el runtime predeterminado.

## Host Python

- `python/feralrf/protocol.py`: codec.
- `python/feralrf/enums.py`: IDs/PHY/status.
- `python/feralrf/commands.py`: payload builders, incluidos restos spectrum.
- `python/feralrf/_responses.py`: parsers auxiliares.
- `python/feralrf/radio.py`: descubrimiento, sesión y API pública.
- `python/feralrf/presets.py`: parámetros proprietary/protocol-named.
- `python/feralrf/emulation/`: helpers PHY-level.
- `python/feralrf/integrations/killerbee.py`: adapter KillerBee.
- `python/feralrf/_spectrum.py`: estructuras sin data path actual.

## Tests y ejemplos

- Codec/host: `python/tests/test_protocol.py`, `test_radio_*.py`, `test_async_error_surfacing.py`, `test_read_one_packet.py`.
- Contrato/presets: `test_commands_contract.py`, `test_enums_no_collisions.py`, `test_props.py`.
- Crypto/emulación/TX test: `test_crypto*.py`, `test_emulation.py`, `test_tx_test.py`.
- KillerBee: `test_killerbee_integration.py`, `test_killerbee_dispatch.py`.
- Smoke/gates: `python/examples/*.py`, `run_validation_baseline.sh`, `release_gate_multi_phy.py`.
- Laboratorio: `python/examples/lab/`.

No se ejecutó ninguno en Fase 4.

## Documentación consultada

- `README.md`
- `docs/ARCHITECTURE.md`
- `docs/protocol.md`
- `docs/PYTHON_API.md`
- `docs/VALIDATION_MATRIX.md`
- `docs/TESTING-ON-LINUX.md`
- `docs/killerbee-catsniffer.md` y `.patch`
- `hardware/PINOUT.md`, `hardware/CatSniffer.kicad_pcb`

No se consultó documentación TI externa adicional: el SDK local y la evidencia manufacturer ya registrada en Fases 1–2 fueron suficientes.

## Advertencias de procedencia

1. Documentación, código y matriz corresponden a distintas fechas dentro del historial; HEAD es la única implementación base.
2. Firmware reporta 1.0.0; arquitectura dice v2.0; Python es 0.3.0; no hay tag Git.
3. El hardware copiado en FeralRF no es autoridad sobre la placa física y también conecta U2/CTF al RP2040.
4. La matriz registra pruebas físicas, pero no adjunta logs, imágenes, binarios hash ni configuración completa.
5. Las librerías TI precompiladas siguen siendo dependencias binarias aunque la aplicación FeralRF sea visible.
6. No se usaron nombres de branches remotos como evidencia de implementación.
