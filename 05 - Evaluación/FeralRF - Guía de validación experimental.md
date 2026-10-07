# FeralRF - Guía de validación experimental

> Encabezado original conservado: # FeralRF - Guía de validación experimental

## Enmienda de reconciliación — 7 de octubre de 2026

Esta es la guía canónica. El cuerpo previo conserva íntegros los 38 EV,35 KI, requisitos, comandos y criterios originales/operativos. Esos bloques son registro de intención y evolución, no prueba de ejecución ni un mapa universal de puertos. Para el estado actual prevalece [[FeralRF - Matriz de pruebas]]; para condiciones realizadas, el EV correspondiente. [[Registro de validación FeralRF]] explica la evolución y [[Auditoría técnica de validación FeralRF - EV ejecutadas]] fundamenta prioridades y próximos experimentos.

### Precedencia y condiciones actuales

1. La definición se obtiene de [[FeralRF - Wiki técnica integral]], [[Arquitectura FeralRF]], [[Matriz de capacidades]] y [[Protocolo y API Python]] con sus límites/fechas. La guía no añade soporte a funciones pendientes o retiradas.
2. Entre tarjeta corta y procedimiento expandido, el bloque **Procedimiento operativo completo por EV** tiene la precedencia que ya declara la guía para ensayos futuros. La discrepancia y el criterio realmente adoptado se registran por corrida; no se cambian expectativas retrospectivamente para dar PASS.
3. Puertos/roles del procedimiento corresponden a un montaje anterior. No usar los ejemplos como discovery. En EV-12 OTA del 6-10-2026: DUT COM33/Shell COM35, observador COM88/Shell COM87; antes COM88 era DUT y COM31 peer planificado. La sustitución y el serial constante requieren manifest físico/build. COM33+2 coincide localmente, COM88+2 falló; no son reglas universales.
4. Reset automático no validado en el mapa inicial: EV-04 obtuvo COM90 en lugar de COM87. Recovery manual observada no significa tres ciclos completos ni API reset PASS. Usar identificación por interfaz/placa y comprobar la Shell efectiva antes de adoptar un reset como precondición.
5. ACK/configuración no demuestra emisión, parámetros efectivos, repetición o cese. EV-10 es PASS+C para ocho secuencias aceptadas, no transición física RX ni RF de ocho PHY. EV-11 reporta control de 27 presets:18 transcripciones individuales y nueve resúmenes, sin OTA por preset. El wrapper de F22 que cambia reset no garantiza cambiar potencia+5; EV-13 usó harness adaptado 0 dBm y espera 0,3 s, con medición física diferida.
6. Las prioridades del cuerpo previo son históricas y no vigentes. No hay P0 incondicional aprobado. P1: identidad actual/interfaz, corregir Shell, calificar observador/conteo, diagnosticar repetición, localizar RX_STOP, medir cese TX y definir aceptación/completitud. P2: procedencia histórica/transcripciones y cobertura/contratos secundarios. P3: naming/usabilidad/roadmap. P1 no significa causa conocida.

### Evidencia y criterios actuales

Modelo canónico: [[FeralRF - Matriz de pruebas#Modelo de evidencia A–F]]. A=RF física independiente; B=comportamiento directo del dispositivo; C=control/API (mocks identificados); D=fuente/documentación; E=inferencia/hipótesis; F=dimensión no evaluada. Nivel, estado, confianza y procedencia se informan por separado. D no demuestra conducta del binario instalado; F no es FAIL.

Un umbral receptor puede FAIL mientras la semántica de conteo/repetición del DUT es INCONCLUSIVE. C sólo acredita aceptación; A acredita la dimensión RF observada, no automáticamente conteo, precisión o cese. El observador FeralRF separado aporta A aunque comparta implementación. Los reportes locales del DUT aportan B; FakeSerial aporta C host/mock.

### Cambios y ejecución respecto del plan

| Ítem | Intención / criterio anterior | Procedimiento o evolución posterior | Estado de reconciliación |
|---|---|---|---|
|EV-00/01/02|Preflight, init/stats, RX stop inicial|Evidencia embebida en registro; no archivos separados|00/01 parciales;02 PASS control, no universal|
|EV-04|Reset por API y repetición|Manual boot/exit y selector equivocado|NOT FULLY VALIDATED; no redefinir manual como API|
|EV-05|RXIEEE local repetible|41/43/43 en 3×30s; primer paquete crc_ok=True en las tres salidas|PASS+B; atribución independiente/bytes completos/rendimiento pendientes; observador concurrente suplementario|
|EV-11|27 presets control excluyendo OOK/MIOTY|Tres bloques complementarios; bloque 868 MHz:6→8, nueve 902/915 sólo resumen|27 control reportados/18 stdout individuales/9 resúmenes; PARTIAL; sin OTA por preset; no sufijos A/B ni renumeración|
|EV-12|Cuatro modos TX/STOP; etapa inicial 01020304|Dos placas, marcadores distintos; min_hits1→40/5/2; intervalos/host abierto|RAW/FRAME PASS+A/C marcador, conteo exacto no establecido; umbral receptor FAIL y semántica DUT INCONCLUSIVE; CONT0:99 registros coincidentes CRC-válidos, conteo físico no establecido; cese F|
|EV-13|CW/PRBS instrumentados|Control con harness adaptado 0 dBm/0,3 s; instrumento declarado disponible pero diferido|PARTIAL; sin frecuencia/potencia/patrón/cese medidos|
|EV-14|Error RF/firma|Baselines bytes y transición;3 mocks SEQ|No error físico inducido; NOT FULLY VALIDATED, no “fallo eliminado”|
|EV-21|Tarjetas 1M/Coded 10/10,2M8/10|Operativo todas≥8/10|Discrepancia explícita; declarar criterio antes del ensayo; aún sin OTA completo|
|EV-41|Tarjeta: diez ciclos|Operativo tres ciclos|EV-14 transición mínima no cierra ninguno de los dos conjuntos|
|EV-44|Tarjeta: burst 100/tasas progresivas|Operativo: baseline 40|Necesita emisión/conteo conocidos; umbral receptor fallido de EV-12 no caracteriza capacidad RX|
|EV-46|Tarjeta: veinte init con RF|Operativo: veinte init solamente|Criterios cubren preguntas distintas;5 procesos EV-03 no 20 init en la misma instancia|
|EV-20/41/42|Campañas dedicadas completas|Evidencia parcial proveniente de 12/14/resets 10–11|Cobertura indirecta marcada, no renombrar como EV ejecutados completos|
|EV-29/51–54|Integración/roadmap/retirada|Dependencias o implementación pendientes;BLE stack retirado|Bloqueado/no ejecutado/no aplica no equivale a FAIL |

### Registro obligatorio para nuevas corridas

Adoptar las 17 secciones visibles en los EV canónicos: contexto, objetivo, capacidad, condiciones, esperado, ejecución, observado, evidencia, comparación, interpretación, límites, resultado, confianza, preguntas, seguimiento, trazabilidad y original. Definir observable y criterio antes de ejecutar. Marcar incógnitas, preservar errores de harness y cambios de procedimiento. PASS/PARTIAL/FAIL/INCONCLUSIVE/NOT FULLY VALIDATED siempre llevan alcance; nivel A–F se añade en Evidencia, separado de High/Medium/Low y procedencia. Fechas de auditoría, commits de fuentes y fecha experimental se informan por separado. Guardar todos los eventos, comandos y salida por caso, identidad física/USB y hashes. La ausencia de firma/error en una ventana no confirma eliminación de un fallo.

Los comandos del cuerpo previo se preservan como registro y requieren verificar roles, versiones y condiciones antes de reutilizarlos. Esta auditoría no los ejecutó ni reflasheó dispositivos. Camino crítico aprobado, sin nuevos EV ni roadmap adicional: **Identidad actual → observación calificada → localización de repetición y STOP → caracterización RF representativa → cobertura más amplia.** La observación se califica antes de atribuir el conteo al DUT; repetición y STOP pueden investigarse en ramas. Recuperar todo el historial no es prerrequisito para fijar la sesión actual. Plan acotado en [[Auditoría técnica de validación FeralRF - EV ejecutadas#Próximas evaluaciones que reducen más incertidumbre]].

## Registro documental previo — conservar su fecha y alcance


> Plan ejecutable; **no se ejecutó ninguna prueba física** al preparar esta nota.

## Evidencia de versión

| Campo | Valor inspeccionado |
|---|---|
| Repositorio | `C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF` |
| Rama / upstream | `main` / `origin/main` |
| Commit | `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea` |
| Fecha / asunto | `2026-07-22 12:34:03 -0600` — `docs: fix remaining 'RF_open at boot' folklore in architecture layer rules` |
| Árbol | No limpio: `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01` aparece modificado. En un control intermedio también era visible el archivo no rastreado y vacío `python/catnip` (timestamp 2026-10-01), pero dejó de existir antes del control final sin que esta revisión emitiera ningún borrado/clean/reset; no se recreó. |
| Baseline host aportado | Windows, Python 3.14.7, pytest 9.1.1: `python -m pytest -rs` → **422 passed, 1 skipped**; el skip es `test_killerbee_dispatch.py` por ausencia de `killerbee`. `pytest` solo falla al importar `tests`: observación de entorno, no defecto funcional confirmado. |

Fuentes principales: `README.md`; `docs/{ARCHITECTURE,PYTHON_API,VALIDATION_MATRIX,protocol,TESTING-ON-LINUX}.md`; `hardware/PINOUT.md`; `python/feralrf/`; `python/tests/`; `python/examples/`; `python/examples/lab/`; `firmware/cc1352/{include,src}/`. Contexto del Vault: [[Arquitectura FeralRF]], [[Protocolo y API Python]], [[Matriz de capacidades]] y [[Pruebas y evidencia existente]].

### Evidencia adicional: CatSniffer-Tools/Catnip

Se inspeccionó `CatSniffer-Tools` sin cambiar de rama ni modificarlo: rama local `fix/CLI_control`, commit `126f13bc0441ad3f37fe0b029160a3c526e4d309`, fecha `2026-09-28 10:54:48 -0600`, tres commits detrás de `origin/fix/CLI_control`. El archivo no rastreado preexistente `py` fue visible en un control intermedio, pero dejó de existir antes del control final sin que esta revisión emitiera ningún borrado/clean/reset; no se recreó. Las afirmaciones siguientes proceden de implementación, no sólo del README:

- `catnip/modules/core/usb_connection.py`, `USBConnection.find_devices()`, `_group_ports_by_device()` y `_map_roles()`: VID/PID `1209:BABB`; agrupa las tres interfaces de una placa por serial/HWID/location y asigna Cat-Bridge, Cat-LoRa y Cat-Shell por descripción, atributo `interface` o índice USB; sólo como último recurso ordena los COM. El `Device ID` es numérico y se reasigna durante cada enumeración: no es identidad persistente.
- `catnip/modules/device/cli.py`, comandos Click `devices`, `identify` y `status`: `catnip devices --debug` muestra campos USB crudos; `catnip identify --device N` hace identificación segura por Cat-Shell; `catnip status --device N --diagnostics` muestra revisión, tres COM, firmware CC reconocido y diagnóstico. Un FeralRF custom puede aparecer como firmware desconocido: confirmar además con `Radio.get_info()`.
- `catnip/modules/firmware/board.py`, `BoardDefinition`, `detect_board()` y `get_board_definition()`: V2 usa SAMD21E17 como host USB y CC1352P1 como MCU RF; V3 usa RP2040 y CC1352P7. Ambos exponen tres CDC. La detección consulta Cat-Shell (`fw_version`, `Board: v2/v3`) y no debe adivinar una revisión desconocida.
- `catnip/modules/firmware/cli.py`, comando `flash`; `catnip/modules/firmware/flasher.py`, `flash_cc1352()`; `catnip/modules/core/device_session.py`, `require_firmware_for_board()`: `catnip flash FIRMWARE --device N` es posicional y destructivo para el CC1352; entra al bootloader por Shell, borra/escribe/verifica por Bridge y reinicia. `catnip update` reflashea el host SAMD21/RP2040. `catnip restore` es recuperación CC por CMSIS-DAP sólo en V3, no una restauración stock rutinaria.
- `catnip/modules/firmware/fw_aliases.py`, tablas por revisión: V3 dispone de `ti_sniffer`; V2 dispone de imágenes P1 como Sniffle, pero no de `ti_sniffer`. `catnip/modules/sniff/cli.py` confirma que `sniff zigbee/thread` exige `ti_sniffer`, `sniff ble` exige Sniffle y que esos comandos pueden auto-flashear si falta la imagen. `sniff fsk` usa el SX1262 mediante el host stock y no convierte al CC1352 en un stack W-MBus/Wi-SUN/Sidewalk.

**Contradicciones/deuda documental de Catnip:** el README superior no refleja toda la compatibilidad V2 presente en implementación; `docs/devices.md` recomienda Device ID frente a COM fijo, pero `find_devices()` asigna esos números de nuevo en cada enumeración, por lo que no son identificadores persistentes. Además, la documentación conceptual de dispositivos inválidos no cambia que `find_devices()` descarte actualmente grupos con menos de tres CDC. Para esta guía manda la implementación del commit anterior y se conserva el diagnóstico como candidato de tooling, no como fix.

### Inventario, arquitectura y compatibilidad

| Placa | Host USB | Radio principal | Familia oficial observada en repos | Control/flasheo | Uso en esta campaña |
|---|---|---|---|---|---|
| CatSniffer V2.0 | SAMD21E17 | CC1352P1 | host `SAMD21/catsniffer`; imágenes/tag V2/P1 | tres CDC; bootloader SAMD/UF2 y bootloader serie CC; recuperación CC externa por cJTAG si se cruza variante | `OBS-V2-A-STOCK`, `OBS-V2-B-STOCK`; conservar conocidas-buenas |
| CatSniffer V3 | RP2040 | CC1352P7 | host `RP2040/catsniffer`; imágenes V3/P7 | tres CDC; ROM BOOTSEL/UF2 RP2040; CC por Shell+Bridge; recuperación CMSIS-DAP disponible | #1 `DUT-V3-FERAL`: Bridge `COM88`, LoRa `COM86`, Shell `COM87`; #2 `PEER-V3-FERAL`/`AUX-V3-FERAL`: Bridge `COM31`, LoRa `COM32`, Shell `COM30` |

FeralRF declara CC1352P7 + RP2040. `firmware/cc1352/CMakeLists.txt` usa por defecto `DEVICE_VARIANT=CC1352P7`; SysConfig/SmartRF y bibliotecas actuales son de la familia CC13x2x7. Aunque existe una cadena alternativa `CC1352P`, no hay configuración de placa V2/SAMD21 ni target explícito CC1352P1. La coincidencia parcial de familia CC13xx no demuestra compatibilidad.

> **FeralRF on CatSniffer V2: compatibility not established; do not flash as part of the current baseline.**

Un HEX P7 en un P1 puede inutilizar el bootloader serie y exigir cJTAG. Los dos V2 se mantienen stock; sólo se considerará cambiar su firmware con documentación explícita V2, beneficio concreto, procedimiento de recuperación y una imagen original identificada/restaurable.

### Modelo de roles

| Rol | Significado |
|---|---|
| `DUT-V3-FERAL` | CatSniffer #1 V3, FeralRF: Bridge `COM88`, LoRa `COM86`, Shell `COM87`. |
| `PEER-V3-FERAL` / `AUX-V3-FERAL` | CatSniffer #2 V3, FeralRF: Bridge `COM31`, LoRa `COM32`, Shell `COM30`; peer/observador RF simétrico disponible. |
| `AUX-V3-STOCK` | rol alternativo futuro de la placa #2 con firmware oficial; sólo después de conservar evidencia y autorizar un cambio destructivo. |
| `OBS-V2-A-STOCK`, `OBS-V2-B-STOCK` | V2 conocidas-buenas con firmware oficial; observadores/generadores sólo para capacidades verificadas. |
| `RF-OBSERVER` | SDR, analizador de espectro, contador de frecuencia, medidor de potencia o sniffer dedicado. |
| `PROTOCOL-DEVICE` | dispositivo real Zigbee, Thread, W-MBus, BLE, Wi-SUN, etc. |
| `PROTOCOL-DEVICE-ZIGBEE-CH25` | equipo existente que genera tráfico Zigbee continuo en IEEE 802.15.4 canal 25; fuente independiente, no controlada por FeralRF. |
| `HOST-TOOL` | Catnip, Wireshark, SmartRF Packet Sniffer, Sniffle, KillerBee u otro software aplicable; requiere radio compatible para emitir/recibir RF. |

**Validación simétrica** (`DUT-V3-FERAL ↔ PEER-V3-FERAL`) reproduce el baseline OTA histórico y ejercita ambos endpoints, pero un defecto compartido puede pasar inadvertido. Aunque la placa #2 observa RF físicamente, no es una implementación independiente mientras ejecute el mismo FeralRF. **Validación independiente** (`DUT-V3-FERAL ↔ AUX-V3-STOCK/OBS-V2-STOCK/PROTOCOL-DEVICE/RF-OBSERVER`) es preferible para interoperabilidad, frecuencia/canal y contenido físico real.

## Preparación Catnip segura

Estos comandos existen exactamente en el commit Catnip inspeccionado y no deben sustituirse por nombres inventados:

```powershell
catnip devices --debug
catnip identify --device 1
catnip status --device 1
catnip status --device 1 --diagnostics
catnip flash --list --device 1
```

Los cuatro primeros son de descubrimiento/consulta (la identificación actúa por Cat-Shell); `flash --list` sólo enumera opciones. Registrar tabla y datos USB, usar `identify` de una sola placa a la vez y volver a enumerar tras reconexiones. Catnip es una alternativa más segura a asumir `Bridge+2`, aunque `_map_roles()` conserva un fallback final por orden COM; confirmar siempre descripción/interfaz/location y placa física antes de reset.

Comandos con efecto y alcance:

| Comando verificado | Controla | Revisión | Efecto/riesgo |
|---|---|---|---|
| `catnip flash FIRMWARE --device N` | CC1352 por Shell+Bridge | V2/V3 sólo con imagen compatible | destructivo: boot, mass erase, write, CRC, exit/reset; no ejecutar durante preparación |
| `catnip update` | host SAMD21/RP2040 | V2/V3 | destructivo: actualiza/reflashea host; fuera de alcance |
| `catnip restore` | CC1352 vía RP2040 CMSIS-DAP | V3 | recuperación destructiva, no “volver a stock” rutinario |
| `catnip verify` | Shell/LoRa y RF | según subprueba | cambia configuración y puede transmitir; no usar como inventario pasivo |
| `catnip sniff zigbee -c 25 --device N -w capture.pcap` | CC1352 TI sniffer por Bridge | V3; V2 sólo si ya ejecuta `ti_sniffer` verificable | puede auto-flashear CC1352 si falta firmware; no usar en V2 stock preservado sin verificar primero |
| `catnip sniff ble --device N -c 37 -m passive_scan --wireshark` | CC1352 Sniffle | V2/V3 con imagen compatible | puede auto-flashear CC1352 |
| `catnip sniff fsk --device N --frequency F --bitrate B --fdev D --bandwidth W --sync-word S -w capture.pcap` | SX1262 mediante host stock | V2/V3 stock | configura RX FSK/GFSK Sub-GHz; no prueba OOK, 4FSK ni un stack de protocolo |

Para convertir más adelante `AUX-V3-STOCK` en `AUX-V3-FERAL`, primero conservar salida de `devices/status`, revisión, firmware y hash; terminar las comparaciones stock; verificar `Board: v3`; y sólo entonces, con autorización explícita, usar `catnip flash "C:\ruta\feralrf_cc1352.hex" --device N`. Catnip rechaza revisión desconocida y comprueba tamaño de flash/variante cuando el nombre lo permite, pero sigue siendo una operación destructiva. Para volver al TI sniffer oficial se usa el flash serial normal; `catnip restore` sí selecciona esa imagen por defecto, pero es la ruta JTAG de recuperación de V3 y no un “undo” rutinario de cualquier imagen anterior.

## Procedimiento operativo de Catnip CLI

Este procedimiento corresponde a `CatSniffer-Tools`, rama local `fix/CLI_control`, commit inspeccionado `126f13bc0441ad3f37fe0b029160a3c526e4d309` (`2026-09-28`), que estaba tres commits detrás de `origin/fix/CLI_control`. No cambiar ni actualizar la rama durante esta campaña. La forma `python .\catnip.py ...`, ejecutada desde el directorio indicado abajo, fuerza el uso de ese checkout; la forma instalada `catnip ...` sólo se acepta después del preflight.

### 1. Preflight de instalación e invocación

Ejecutar en PowerShell. No hace falta conectar una placa; todos estos comandos son read-only para hardware.

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\CatSniffer-Tools'
git branch --show-current
git rev-parse HEAD
git status --short --branch

Set-Location .\catnip
where.exe catnip
(Get-Command catnip -ErrorAction SilentlyContinue) | Format-List Source,Path,CommandType
python -m pip show catnip
python -c "import importlib.util; s=importlib.util.find_spec('catnip'); print(s.origin if s else 'NOT FOUND')"
catnip --help
python .\catnip.py --help
```

**Interpretación exacta:** los tres primeros comandos deben mostrar `fix/CLI_control`, SHA `126f13bc0441ad3f37fe0b029160a3c526e4d309` y el estado ya registrado (`behind 3`, `?? py`). `where.exe`/`Get-Command` muestran qué ejecutable resolverá PowerShell; `pip show` aporta `Version` y `Location`; `find_spec` muestra el módulo Python importado. El CLI **no implementa `catnip --version`**. `--help` imprime el encabezado, que en este checkout dice `v3.3.3.0`, y los comandos superiores ensamblados en `modules/core/cli.py`: `devices`, `identify`, `status`, `flash`, `update`, `restore`, `verify`, `sniff`, `cativity`, `meshtastic` y `lora` en Windows.

**Normal:** `catnip --help` y `python .\catnip.py --help` muestran la misma versión y árbol de comandos; `pip show`/`find_spec` apuntan al entorno esperado o se decide usar exclusivamente `python .\catnip.py`. **STOP:** SHA/rama distintos, encabezado distinto de `3.3.3.0`, múltiples `catnip.exe` ambiguos, o el ejecutable instalado carece de `devices --debug`, `identify` o `status`. **Registrar en Obsidian:** fecha/hora, Python/venv, salidas de rama/SHA/status, rutas de `where.exe` y `find_spec`, versión del encabezado y forma elegida de invocación. No usar `pip install`, `git pull` ni `flash --refresh` como parte de este preflight.

En el resto de esta sección, `python .\catnip.py` presupone que PowerShell sigue en:

```text
C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\CatSniffer-Tools\catnip
```

### 2. Catálogo operativo y efectos

| Comando exacto recomendado                                                  |         ¿Placa conectada? |                                                                                                ¿Read-only? |                                 ¿Reset? |                         ¿Flash CC1352? |                           ¿Flash host? |                                                 ¿Seguro en baseline? |
| --------------------------------------------------------------------------- | ------------------------: | ---------------------------------------------------------------------------------------------------------: | --------------------------------------: | -------------------------------------: | -------------------------------------: | -------------------------------------------------------------------: |
| `python .\catnip.py --help`                                                 |                        no |                                                                                                         sí |                                      no |                                     no |                                     no |                                                                   sí |
| `python .\catnip.py devices`                                                |       sí para hallar algo |                                                                        sí; consulta `fw_version` por Shell |                                      no |                                     no |                                     no |                                                                   sí |
| `python .\catnip.py devices --debug`                                        |                        sí |                                                                          sí; igual y añade enumeración USB |                                      no |                                     no |                                     no |                                                                   sí |
| `python .\catnip.py identify --device N`                                    |                        sí |                                                                        no: escribe `identify` en Cat-Shell |                                      no |                                     no |                                     no |                                                        sí; sólo LEDs |
| `python .\catnip.py status --device N`                                      |                        sí | **no estrictamente**: consulta Shell y, sin metadata útil, escribe sondas STOP/PING TI y Sniffle en Bridge |                                      no |                                     no |                                     no | sí sólo con DUT idle y antes de EV-01; cerrar otros usuarios del COM |
| `python .\catnip.py status --device N --diagnostics`                        |                        sí |                                                              igual que `status`; añade lectura diagnóstica |                                      no |                                     no |                                     no |                   sí con DUT idle; detalle ampliado sobre todo en V2 |
| `python .\catnip.py flash --list --device N`                                | sí para filtrar por placa |                                                  hardware read-only; puede consultar/cachear catálogo host |                                      no |                                     no |                                     no |                                                  sí, sin `--refresh` |
| `python .\catnip.py sniff zigbee --device N -c 25 -w .\capture.pcap`        |                        sí |                                                                                                         no | reinicia/sale del bootloader si flashea | **sí, si `ti_sniffer` no se verifica** |                                     no |                                                 no en primera sesión |
| `python .\catnip.py sniff thread --device N -c 25 -w .\thread.pcap`         |                        sí |                                                                                                         no |                                   igual |          **sí, si falta `ti_sniffer`** |                                     no |                                                 no en primera sesión |
| `python .\catnip.py sniff ble --device N -c 37 -m passive_scan --wireshark` |                        sí |                                                                                                         no | reinicia/sale del bootloader si flashea |      **sí, si Sniffle no se verifica** |                                     no |                                                 no en primera sesión |
| `python .\catnip.py sniff fsk --device N ...`                               |                        sí |                                                                       no: cambia switch/config/modo SX1262 |                                      no |                                     no |                                     no |                               sólo en EV autorizado; no en preflight |
| `python .\catnip.py flash "C:\ruta\imagen.hex" --device N`                  |                        sí |                                                                                                         no |                                      sí |           **sí: mass erase/write/CRC** |                                     no |                                                                   no |
| `python .\catnip.py flash zigbee --device N`                                |                        sí |                                                                                                         no |                                      sí |             **sí: TI sniffer oficial** |                                     no |                                                                   no |
| `python .\catnip.py update --device N`                                      |              sí o BOOTSEL |                                                                                                         no |                                      sí |                                     no |     **puede actualizar SAMD21/RP2040** |                                                                   no |
| `python .\catnip.py restore --device N`                                     |                V3/BOOTSEL |                                                                                                         no |                                      sí |         **sí, vía JTAG y luego serie** | **sí temporalmente y restaura bridge** |                                    sólo recuperación, nunca baseline |
| `python .\catnip.py verify --device N`                                      |                        sí |                                                                            no: ejecuta comandos Shell/LoRa |                 posible por las pruebas |                                     no |                                     no |                    no como inventario; `--test-all` puede transmitir |

`device_session()` primero verifica la imagen requerida. Si coincide, Zigbee/Thread/BLE no flashean; si no coincide, validan la revisión y llaman al flasher automáticamente. En V2, `ti_sniffer` ya instalado manualmente puede verificarse y usarse, pero si falta, `require_firmware_for_board()` bloquea antes de flashear porque no existe imagen P1 en el catálogo. Sniffle sí tiene imágenes P1 y P7, por lo que BLE puede auto-flashear ambas revisiones.

### 3. Descubrimiento e identificación desde placa desconectada

1. Desconectar todas las CatSniffer y cerrar monitores seriales, Wireshark/extcap y procesos FeralRF.
2. Ejecutar `python .\catnip.py devices`. **Normal:** `No CatSniffer devices found.` **STOP:** aparece una placa: queda otra conectada o hay enumeración stale que debe resolverse. Registrar la salida.
3. Conectar sólo la placa candidata y esperar a que Windows cree tres COM.
4. Ejecutar:

   ```powershell
   python .\catnip.py devices
   python .\catnip.py devices --debug
   ```

   `devices` muestra `Device` (`CatSniffer #N`), `Board`, `Cat-Bridge (CC1352)`, `Cat-LoRa (SX1262)` y `Cat-Shell (Config)`. `Board` proviene de `fw_version` por Cat-Shell: identifica V2/V3 y MCU, no la revisión menor exacta del PCB. `--debug` añade por interfaz `Port`, `Description`, `HWID`, `Location`, `Interface` y `Serial#` desde pyserial.
5. Elegir el `N` de la fila candidata y ejecutar:

   ```powershell
   python .\catnip.py identify --device N
   ```

   Catnip abre sólo Cat-Shell y manda `identify`. El host stock hace parpadear los tres LEDs diez veces, con pasos de 100 ms (aproximadamente dos segundos), y responde `Identifying board...`. No toca ni reinicia el CC1352, por lo que es seguro aunque éste ejecute FeralRF; requiere que el host RP2040/SAMD conserve el comando Shell oficial/compatible. **Normal:** sólo la placa física esperada parpadea y aparece `Identification command sent successfully!`. **STOP:** parpadea otra placa, Shell no responde, o el `N` cambió.
6. Registrar la fila, todos los campos debug y qué placa física parpadeó. Repetir con una sola placa conectada para CatSniffer #2 (`PEER-V3-FERAL`) y, sólo si una EV lo requiere, para `OBS-V2-A-STOCK`/`OBS-V2-B-STOCK`.

   Ejemplo ilustrativo probado por las cadenas de la implementación:

   ```text
   Sending 'Identify' command to CatSniffer #1 on port COM9...
   Response: Identifying board...
   Identification command sent successfully!
   ```

Los IDs `N` se asignan en cada enumeración. Desconectar, reconectar o cambiar el conjunto de placas puede reasignarlos. Nunca registrar sólo `--device N`: conservar también serial/HWID/location, rol físico y los tres COM.

**Ejemplo ilustrativo, sólo con campos que la implementación realmente imprime:**

```text
Found 1 CatSniffer device(s)
Device          Board                         Cat-Bridge (CC1352)  Cat-LoRa (SX1262)  Cat-Shell (Config)
CatSniffer #1   v3 (RP2040 + CC1352P7)        COM7                 COM8                COM9

Raw USB port info (debug)
Port  Description  HWID  Location  Interface  Serial#
COM7  ...          ...   ...       ...        ...
COM8  ...          ...   ...       ...        ...
COM9  ...          ...   ...       ...        ...
```

Los valores `COM7/8/9` son ilustrativos, no una predicción. Un campo debug vacío significa que Windows/driver no lo expuso; si faltan metadatos y la asignación sólo parece depender del orden COM, detener y contrastar físicamente con `identify`.

### 4. Estado y verificación de firmware sin flashear

Con la placa conectada y el `N` recién obtenido:

```powershell
python .\catnip.py status --device N
python .\catnip.py status --device N --diagnostics
```

Ninguno flashea o reinicia, pero **no son estrictamente read-only en el wire**. `status` obtiene la revisión mediante `fw_version` en Cat-Shell, el estado/cache del host mediante `status`, la identidad conocida del firmware CC1352 mediante metadata o sondas de `FirmwareVerifier`, y las configuraciones cacheadas LoRa/FSK del SX1262 mediante `lora_config`/`fsk_config`. Si no hay metadata, `FirmwareVerifier.detect()` escribe sondas directas al Bridge: para TI envía STOP+PING y para Sniffle una consulta base64. Ejecutarlo sólo con FeralRF idle, sin RX/TX activo y antes de EV-01; no repetirlo dentro de una sesión FeralRF. `--diagnostics` añade trace y threads cuando el firmware los reporta, principalmente V2/SAMD21; no crea datos ausentes.

| Campo probado por implementación | Componente/fuente | Qué registrar / interpretar |
|---|---|---|
| `Board` | Cat-Shell `fw_version` | debe ser `v3 (RP2040 + CC1352P7)` para DUT/AUX V3 o `v2 (SAMD21 + CC1352P1)` para OBS V2; `unknown` bloquea cualquier flash |
| `Bridge port`, `LoRa port`, `Shell port` | agrupación USB Catnip | los tres deben existir y coincidir con discovery/debug |
| `Firmware`, `Detected via`, `Capabilities` | `FirmwareVerifier` + registro Catnip | sólo aparecen si la imagen CC1352 es reconocida; FeralRF puede aparecer `unknown` y debe confirmarse con GET_INFO en EV-01 |
| `Board can` | tabla estática de capacidades por revisión | capacidad del hardware/tooling, no estado activo |
| `FW` | respuesta `status` del host | versión RP2040/SAMD21 reportada por Shell |
| `Radio`, `LoRa`, `LoRa Mode`, `CC1352 FW` | respuesta del host | estado/cache declarado; el ID CC puede ser `unset`, `unknown` o metadata oficial, no prueba funcional por sí solo |
| `loss: uart_overrun`, `loss: ring_dropped`, `loss: dma_regress` | contadores host disponibles | ausencia no equivale a cero; registrar valores presentes |
| `stack unused`, `Last fault`, `Threads`, trace | diagnóstico V2 cuando existe | bajo headroom/last fault distinto de `none` detiene el baseline hasta investigar |
| `Radio configuration (SX1262)` LoRa/FSK | configuración cacheada en host | no prueba recepción RF ni presencia del chip; registra el estado que una captura heredaría |

Catnip no muestra un campo “bootloader activo”. Un Shell ausente/no responsive o Bridge que no verifica sólo permite registrar estado desconocido; no autoriza inferir bootloader. Ejemplo ilustrativo para un DUT FeralRF:

```text
Status for CatSniffer #1
Field                   Value
Board                   v3 (RP2040 + CC1352P7)
Bridge port (CC1352)    COM7
LoRa port (SX1262)      COM8
Shell port (Config)     COM9
Firmware                unknown

Firmware diagnostics
FW                      v3.x.y.z
Radio                   LoRa
LoRa                    initialized
LoRa Mode               Stream
CC1352 FW               <valor reportado por el host>
loss: uart_overrun      0
loss: ring_dropped      0 bytes
```

**Normal DUT:** Board V3, tres COM presentes, host responde y `Firmware unknown` es aceptable provisionalmente para FeralRF; EV-01 debe aportar GET_INFO. **STOP:** Board unknown/V2 para el DUT, COM faltante/duplicado, hardware físico equivocado, Shell sin respuesta, fault no limpio, o firmware oficial inesperado donde debía estar FeralRF. **Registrar:** tabla completa, detección/confianza/capabilities si aparecen, diagnóstico y configuraciones SX1262; no afirmar que `unknown` significa FeralRF.

### 5. Preflight obligatorio de `DUT-V3-FERAL` antes de EV-00/EV-01

Con sólo DUT conectado, ejecutar exactamente:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\CatSniffer-Tools\catnip'
python .\catnip.py --help
python .\catnip.py devices
python .\catnip.py devices --debug
python .\catnip.py identify --device N
python .\catnip.py status --device N
```

Sustituir `N` únicamente después de leer `devices`; no reutilizar un ID de una sesión previa. Estos comandos no flashean ni reinician. Completar:

| Campo | Valor observado |
|---|---|
| Catnip device ID (sólo esta enumeración) | |
| Identidad física confirmada por LEDs | `DUT-V3-FERAL` / otro |
| Board reportada | |
| Cat-Bridge COM | |
| Cat-LoRa COM | |
| Cat-Shell COM | |
| Serial / HWID / Location / Interface | |
| Catnip Firmware / Detected via / Capabilities | |
| Host `FW` y `CC1352 FW` reportados | |
| Puerto Bridge elegido para FeralRF | |
| Shell que FeralRF calculará (`Bridge + 2`) | |
| ¿`Bridge + 2 == Shell` real? | sí / no |
| Resultado | PASS preflight / BLOCKED |
| Fecha/hora | |

### 6. Mapeo seguro de reset para EV-04

Después del preflight, copiar los COM reales en este bloque PowerShell; no obtiene ni abre puertos, sólo calcula la suposición de FeralRF:

```powershell
$Bridge = 'COM7'   # reemplazar por Cat-Bridge de Catnip
$Shell  = 'COM9'   # reemplazar por Cat-Shell de la misma fila/placa
$BridgeNumber = [int]($Bridge -replace '^COM','')
$AssumedShell = 'COM' + ($BridgeNumber + 2)
[pscustomobject]@{
  Bridge = $Bridge
  ActualShell = $Shell
  FeralRFAssumedShell = $AssumedShell
  Match = ($AssumedShell -eq $Shell)
}
```

**Normal:** `Match=True` y `identify --device N` confirmó la misma placa. Registrar tabla Catnip, cálculo y timestamp; entonces EV-04 puede invocar `Radio.reset_device()`. **STOP/BLOCKED:** `Match=False`, Board/roles unknown, cualquier COM pertenece a otra placa, o el ID cambió. No probar “a ver si funciona”: EV-04 queda `BLOCKED` y se usa power-cycle manual si una recuperación fuese necesaria. Catnip no corrige internamente `Radio.reset_device()`.

### 7. Observador IEEE 802.15.4/Zigbee y Thread — fuera del baseline inicial

Primero conectar sólo el observador, repetir discovery/identify/status y guardar su firmware anterior. Para `AUX-V3-STOCK`, la secuencia exacta de canal 25 es:

```powershell
python .\catnip.py devices --debug
python .\catnip.py identify --device N
python .\catnip.py status --device N
python .\catnip.py sniff zigbee --device N --channel 25 --write .\aux-v3-zigbee-ch25.pcap
```

`-c 25` es equivalente a `--channel 25`; `-w` a `--write`. Sin `-ws`, imprime cada paquete como número, longitud, RSSI y hex y conserva PCAP. `-ws` abre Wireshark con el perfil `Zigbee`; ejemplo combinado: `python .\catnip.py sniff zigbee --device N -c 25 -ws -w .\aux-v3-zigbee-ch25.pcap`. Detener con `Ctrl+C`: Catnip envía STOP al TI sniffer, cierra Bridge/PCAP y reporta pérdidas del bridge. Si el PCAP ya existe, aborta antes de tocar radio salvo que se añada deliberadamente `--force`.

**Efecto:** requiere `ti_sniffer`. Si no se verifica, Catnip puede entrar al bootloader, hacer mass erase y flashearlo automáticamente. Por tanto, sólo ejecutar tras preservar la imagen/estado anterior y autorizar el cambio. **Normal:** V3 reconocido, mensaje de firmware encontrado o flash exitoso, `Sniffing Zigbee at channel: 25`, `Capture running`, tramas variables del `PROTOCOL-DEVICE-ZIGBEE-CH25`, y PCAP cerrada limpiamente. **STOP:** placa V2 sin TI sniffer ya instalado, board unknown, intento de usar DUT-V3-FERAL, error de flash, canal fuera de 11–26, FCS/tráfico sistemáticamente inválido o pérdidas no explicadas. Registrar firmware antes/después, rol, COM, comando, PCAP/hash, duración, paquetes y loss report.

**V2:** `fw_aliases.py` confirma de nuevo que `ti_sniffer` no existe en `OFFICIAL_ID_TO_FILENAME_BY_BOARD['v2']`. El comando es **BLOCKED para provisionar** `OBS-V2-*-STOCK`. Sólo si `status`/verifier demuestra que un V2 ya ejecuta un TI sniffer P1 instalado previamente, `device_session()` lo conserva y permite la captura; no asumirlo.

Thread sí tiene comando distinto, pero usa la misma imagen TI, el mismo canal IEEE 802.15.4 y el mismo `run_bridge`; la diferencia operativa es la etiqueta/perfil Wireshark `Thread`, no un stack Thread dentro de Catnip:

```powershell
python .\catnip.py sniff thread --device N --channel 25 --write .\aux-v3-thread-ch25.pcap
```

La restauración posterior no es automática: Catnip no memoriza la imagen previa. Si se registró una imagen oficial, reflashear explícitamente su alias/archivo; para volver al TI sniffer oficial de V3 usar `python .\catnip.py flash zigbee --device N`. No usar estos comandos sobre DUT-V3-FERAL.

### 8. Observador BLE stock — fuera del baseline inicial

Comando verificado:

```powershell
python .\catnip.py status --device N
python .\catnip.py sniff ble --device N --channel 37 --mode passive_scan --wireshark
```

Usa Sniffle en el **CC1352** (imagen P7 en V3 o P1 en V2), Bridge como transporte, Cat-Shell para seleccionar la ruta RF 2.4 GHz y el plugin externo `sniffle_extcap` para Wireshark. Sin `--wireshark`, el comando sólo deja Sniffle listo e imprime instrucciones para configurar manualmente interfaz, Bridge, canal y modo; no ofrece `--write` propio. Con Wireshark, la captura termina al cerrar Wireshark. Puede auto-flashear Sniffle si falta; no usar en la primera sesión ni sobre un V2 conocido-bueno sin autorización para cambiar su CC1352. Registrar revisión, imagen antes/después, canal 37–39, modo y resultado del plugin.

### 9. Observador SX1262/Cat-LoRa FSK/GFSK — V2 o V3 stock

Este flujo no cambia el firmware del CC1352 ni del host. Sí selecciona la ruta SX1262, cambia su configuración FSK/GFSK, pone el host en modo stream y al salir intenta restaurar modo command. Usa Cat-Shell para configuración y Cat-LoRa para paquetes. Ejemplo concreto para un FeralRF `gfsk_868_50k` — ejecutar sólo donde 868 MHz esté autorizado y después de confirmar los parámetros de la emisión:

```powershell
python .\catnip.py devices --debug
python .\catnip.py identify --device N
python .\catnip.py status --device N
python .\catnip.py sniff fsk --device N --frequency 868000000 --bitrate 50000 --fdev 100000 --bandwidth 312.0 --sync-word 930B51DE --bt 0.5 --no-crc --no-whitening --pktlen variable --payload 255 --write .\ev22-gfsk868-observer.pcapng
```

Rangos implementados: frecuencia 137–1020 MHz, bitrate 600–300000 bps, desviación 600–200000 Hz, 21 anchos discretos de 4.8 a 467.0 kHz, sync word de 1–8 bytes hex, BT `off/0.3/0.5/0.7/1.0`, CRC/whitening on/off y longitud fixed/variable. El ancho debe cubrir aproximadamente `bitrate + 2*fdev`; si no, el firmware lo sustituye por 187.2 kHz y Catnip lo advierte. Ajustar los valores al preset exacto bajo prueba; el ejemplo no convierte incompatibilidad de framing en fallo RF.

La terminal muestra índice, longitud, RSSI, hex y ASCII; `--write` produce PCAP/PCAPNG LoRaTap. Detener con `Ctrl+C`; el reporte final incluye duración, paquetes, errores de parseo, líneas no reconocidas/truncadas, frames cortados por firmware y distribución RSSI. El firmware host puede truncar a 40 bytes, lo cual debe registrarse. No hay subcomando superior Catnip de TX FSK necesario para estos EV; no inventar uno ni escribir al Cat-LoRa en stream, porque esos bytes se interpretarían como payload TX.

Aplicación: EV-11/12 sólo para FSK/GFSK Sub-GHz compatible; EV-22 para filas FSK/GFSK 868/915, no MSK/4FSK; EV-25 sólo evidencia PHY FSK compatible, no protocolo W-MBus; EV-26 sólo capa FSK compatible, no Wi-SUN/Sidewalk stack. No sirve para OOK, MSK, 4FSK ni propietario 2.4 GHz. **STOP:** `Some settings were not confirmed`, respuesta inesperada al cambiar stream/command, COM faltante, firmware host incompatible, tramas persistentemente truncadas o parámetros desconocidos. Registrar configuración completa, firmware host, rol/serial/COM, PCAP/hash y reporte final.

### 10. Cambio de rol documentado de la V3 #2 — no ejecutar en esta fase

#### AUX-V3-STOCK → AUX-V3-FERAL

La V3 #2 ya existe y actualmente es `PEER-V3-FERAL`; este procedimiento queda como referencia para una futura conversión desde stock y sólo se ejecutaría con autorización explícita:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\CatSniffer-Tools\catnip'
python .\catnip.py devices --debug
python .\catnip.py identify --device N
python .\catnip.py status --device N

$FeralHex = 'C:\ruta\verificada\feralrf_cc1352.hex'
Test-Path -LiteralPath $FeralHex
Get-FileHash -Algorithm SHA256 -LiteralPath $FeralHex
python .\catnip.py flash $FeralHex --device N

python .\catnip.py devices --debug
python .\catnip.py identify --device N
python .\catnip.py status --device N
```

Antes de flash: `Board` debe ser V3/P7, identificación física inequívoca, tres COM presentes, `Test-Path=True`, extensión `.hex` y hash registrado. FeralRF documenta `.hex`, no `.bin`. `flash` entra al bootloader CC por Shell, usa Bridge, hace mass erase, escribe, verifica CRC, sale/reinicia y trata de actualizar metadata en el RP2040. **Normal:** erase/write/CRC y restart exitosos, placa reenumerada, Bridge disponible; `status` puede decir `Firmware unknown` porque FeralRF no está en el registro Catnip. La confirmación final es EV-01/GET_INFO, no la etiqueta Catnip. **STOP:** Board unknown/V2, archivo ausente/`.bin`, chip/imagen incompatibles, CRC/error de bootloader, o la placa identificada no es AUX.

#### Restaurar AUX-V3 a stock

Si el bootloader serie funciona, la ruta normal soportada para instalar el TI multiprotocolo oficial es:

```powershell
python .\catnip.py devices --debug
python .\catnip.py identify --device N
python .\catnip.py flash --list --device N
python .\catnip.py flash zigbee --device N
python .\catnip.py status --device N
```

`zigbee` resuelve a `ti_sniffer`/`sniffer_fw_Catsniffer_v3.x.hex`; confirmar en `status` `TI Multiprotocol Sniffer (ti_sniffer)`. Esto restaura esa imagen oficial, no cualquier firmware previo arbitrario. Si el bootloader CC está roto, V3 ofrece la recuperación destructiva `python .\catnip.py restore --device N`: usa OpenOCD, convierte temporalmente el RP2040 en CMSIS-DAP con Free-DAP, borra CC por JTAG preservando bootloader, restaura el bridge RP2040 y flashea por defecto `sniffer_fw_Catsniffer_v3.x.hex`. Requiere intervención BOOTSEL y no es el método rutinario. V2 no soporta `restore` interno y necesita cJTAG externo.

`flash --list --device N` imprime `Board` y una tabla `Alias / Firmware Name / Description` filtrada por revisión. Es normal que oculte imágenes de otra generación. **STOP:** Board `unknown`, variante contraria o ausencia de la imagen oficial esperada. Registrar alias y nombre exacto antes de cualquier flash.

**V2:** no ejecutar ningún flash FeralRF. No existe target FeralRF P1/SAMD21 establecido ni imagen V2 aceptada por esta campaña.

### 11. Mapa Catnip → EV

| EV | Preparación Catnip | Comando(s) exactos | ¿Modifica firmware? | Rol requerido |
|---|---|---|---|---|
| EV-00 | discovery completo | `devices`; `devices --debug`; `identify --device N`; `status --device N` | no | `DUT-V3-FERAL` |
| EV-04 | preflight + comparación `Bridge+2` | mismos de EV-00 + bloque PowerShell §6 | no; EV-04 sí resetea después vía FeralRF | DUT |
| EV-05 | sólo preflight DUT | `status --device N`; sin Catnip de captura en primera sesión | no | DUT + `PROTOCOL-DEVICE-ZIGBEE-CH25` |
| EV-12 | preflight; observer opcional | `sniff zigbee ...` para IEEE o `sniff fsk ...` para FSK/GFSK | Zigbee puede; FSK no | `AUX-V3-STOCK`/V2 compatible/`RF-OBSERVER` |
| EV-13 | No Catnip action required after preflight. | `status --device N` antes/después únicamente | no | DUT + instrumento RF |
| EV-20 | AUX-Feral no necesita Catnip tras preflight; AUX-Stock observer | `sniff zigbee --device N -c 25 -w .\ev20.pcap` | puede auto-flashear TI sniffer | AF o AS |
| EV-21 | observer BLE opcional | `sniff ble --device N -c 37 -m passive_scan --wireshark` | puede auto-flashear Sniffle | AF o AS/V2 autorizado |
| EV-22 | observer SX1262 parcial | comando completo `sniff fsk` §9 | no firmware; cambia config SX1262 | AF o V2/AS stock |
| EV-25 | observer PHY FSK parcial | `sniff fsk` con parámetros W-MBus compatibles | no firmware | V2/AS + dispositivo W-MBus para interop |
| EV-26 | observer PHY FSK parcial | `sniff fsk` con parámetros exactos Wi-SUN/Sidewalk | no firmware | V2/AS + dispositivo de protocolo para interop |
| EV-29 | preflight; observer stock separado | `sniff zigbee --device N -c 25 -w .\ev29.pcap` | puede auto-flashear TI sniffer | DUT+KillerBee; AS observer |
| EV-40 | discovery/reset mapping de ambas Feral | EV-00/04 por placa; después No Catnip action required. | no | D+AF |
| EV-44 | discovery de DUT/generador | EV-00 por placa; después No Catnip action required. | no | D+AF o generador controlado |

### 12. ¿Qué ejecuto ahora?

Con ambas V3 disponibles, haga el preflight **una placa a la vez**; mantenga los V2 desconectados. Primero conecte sólo CatSniffer #1:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\CatSniffer-Tools'
git branch --show-current
git rev-parse HEAD
git status --short --branch

Set-Location .\catnip
where.exe catnip
python -m pip show catnip
python -c "import importlib.util; s=importlib.util.find_spec('catnip'); print(s.origin if s else 'NOT FOUND')"
python .\catnip.py --help

python .\catnip.py devices
python .\catnip.py devices --debug
python .\catnip.py identify --device N
python .\catnip.py status --device N
```

Sustituya `N` por el ID mostrado en **esa misma enumeración**. Registre #1=`Bridge COM88 / LoRa COM86 / Shell COM87`. Desconecte #1, conecte sólo #2 y repita desde `devices`; vuelva a elegir el `N` recién mostrado y registre #2=`Bridge COM31 / LoRa COM32 / Shell COM30`. Finalmente conecte ambas, ejecute otra vez `devices --debug` y confirme que cada `identify --device N` hace parpadear la placa prevista.

La fase estrictamente read-only para hardware termina en `devices --debug`. `identify` escribe sólo la orden temporal de LEDs en Shell; `status` es no destructivo pero puede escribir las sondas Bridge descritas en §4. Ambos se ejecutan con cada placa idle, antes de abrir FeralRF. Después, rellene la tabla de §5 para ambas y ejecute sólo la comparación PowerShell de §6. En esta máquina ambos resultados `Bridge+2 == Shell` deben ser `False`; eso **bloquea `Radio.reset_device()`**, pero no EV-01 ni los procedimientos que usan Shell explícito. Cierre Catnip antes de abrir COM88/COM31 desde FeralRF. Si revisión, identidad o roles difieren, detenga la validación. **No ejecutar ahora:** `sniff`, `flash`, `update`, `restore` ni `verify`.

### Evidencia de implementación para este procedimiento

- `catnip/modules/core/cli.py:build_cli/main_cli/print_header` — comandos superiores, ayuda y versión de encabezado.
- `catnip/modules/device/cli.py:devices/_print_raw_port_debug/identify/status` — campos y operaciones de discovery/status.
- `catnip/modules/core/usb_connection.py:_group_ports_by_device/_map_roles/find_devices` — agrupación, roles e IDs locales.
- `CatSniffer-Firmware/{RP2040,SAMD21}/catsniffer/src/shell_commands.c:cmd_identify` — diez parpadeos/100 ms y LEDs.
- `catnip/modules/firmware/fw_status.py:read_status/read_radio_configs` y `firmware_verifier.py:FirmwareVerifier` — origen de los campos status.
- `catnip/modules/core/device_session.py:device_session` — verificación y auto-flash condicionado.
- `catnip/modules/sniff/cli.py:sniff_ble/sniff_zigbee/sniff_thread/sniff_fsk`; `catnip/modules/core/bridge.py:run_bridge/run_fsk_bridge/_run_sx_capture` — opciones, captura y cleanup.
- `catnip/modules/firmware/cli.py:flash/update/restore/verify`; `flasher.py:find_flash_firmware`; `restore.py:restore_cc1352`; `fw_aliases.py:OFFICIAL_ID_TO_FILENAME_BY_BOARD` — efectos destructivos, restauración y soporte por revisión.

## Escenarios de hardware disponibles

| Escenario | Combinación | Ejecutable / valor |
|---|---|---|
| A — disponible ahora | `DUT-V3-FERAL + PEER-V3-FERAL + OBS-V2-A/B-STOCK` | Todos los controles de una placa y los OTA simétricos entre #1 y #2; RX IEEE real con `PROTOCOL-DEVICE-ZIGBEE-CH25`; observación Sub-GHz FSK/GFSK por SX1262 V2 si los parámetros encajan. V2 sólo observa IEEE si `status` confirma que ya lleva `ti_sniffer`; Catnip actual no ofrece esa imagen para V2. |
| B — dos V3 FeralRF | `DUT-V3-FERAL + PEER-V3-FERAL` | **Disponible ahora.** Reproduce `smoke_ota_txrx.py`, matriz de presets/modulación, BLE raw, propietario Sub-GHz, OOK, propietario 2.4 GHz, carga/burst y helpers de emulación mediante los wrappers seguros de esta guía. Es baseline simétrico, no certificación independiente; EV-40 completo queda bloqueado porque el script oficial deriva Shell aritméticamente. |
| C — FeralRF + V3 stock | `DUT-V3-FERAL + AUX-V3-STOCK` | Más fuerte para observar IEEE 802.15.4, canal, PCAP y frames con `ti_sniffer`; útil para compatibilidad con tooling oficial y diagnóstico. Stock no expone automáticamente todos los PHY FeralRF: BLE requiere Sniffle y Sub-GHz FSK usa SX1262; OOK/4FSK/propietario 2.4 no quedan cubiertos. |
| D — FeralRF + V2 stock | `DUT-V3-FERAL + OBS-V2-A-STOCK` | También disponible. Fuente/observador IEEE sólo si el firmware instalado lo permite; `sniff fsk` sí ofrece observación SX1262 FSK/GFSK Sub-GHz. No es peer FeralRF ni generador genérico confirmado por Catnip. |
| E — instrumento independiente | `DUT-V3-FERAL + RF-OBSERVER` | SDR confirma frecuencia/ocupación/contenido compatible; analizador, contador y power meter aportan pureza, potencia y error de frecuencia que otra CatSniffer no mide. Necesario para CW/PRBS y preferible en OOK, 433, 2.4 propietario y jamming. |

### Decisión de firmware para AUX-V3

| Objetivo de validación | Firmware AUX-V3 | Por qué |
|---|---|---|
| Reproducir baseline OTA / recepción FeralRF→FeralRF | `AUX-V3-FERAL` | misma ruta que los scripts y resultados históricos |
| Verificar IEEE 802.15.4 de forma independiente | `AUX-V3-STOCK` + TI sniffer | evita defecto compartido y produce captura/PCAP; aceptar auto-flash CC antes de `sniff zigbee` |
| Investigar un bug que puede afectar ambos peers | `AUX-V3-STOCK` o `RF-OBSERVER` | una segunda implementación separa el bug común de RF real |
| Protocolo host FeralRF, colas y burst controlado | `AUX-V3-FERAL` | permite comandos/eventos y generación reproducible del repo |
| Interoperabilidad con software oficial | `AUX-V3-STOCK` | valida Catnip/TI sniffer/Sniffle/SX1262 según capacidad |
| BLE raw / Sub-GHz raw del baseline | `AUX-V3-FERAL` primero | reproduce PHY simétrica; añadir stock/SDR para independencia donde exista soporte |
| Diagnosticar si una falla es específica de FeralRF | `AUX-V3-STOCK` | referencia independiente; no asumir soporte de PHY no expuesto |

Secuencia recomendada, no ejecutada: preservar stock y hashes → pruebas independientes → registrar firmware → flash verificado de FeralRF → OTA simétrica → restaurar stock únicamente si un objetivo posterior lo requiere.

### Fuentes externas de tráfico

| Capacidad | Fuente práctica | Clasificación actual |
|---|---|---|
| IEEE 802.15.4 | `PROTOCOL-DEVICE-ZIGBEE-CH25`; AUX-V3 TI sniffer como observador; V2 sólo si ya lleva TI sniffer | **easy with existing hardware** para RX del DUT; comparación V2 condicional |
| Zigbee | dispositivo CH25 real + Wireshark/sniffer compatible | PHY/RX y decode externo prácticos; no demuestra stack Zigbee en FeralRF |
| Thread / Matter sobre 802.15.4 | border router/nodo Thread/Matter o simulador más radio IEEE compatible | **requires protocol-specific third-party device** para interoperabilidad; PHY-only con frames raw |
| BLE raw PHY | `AUX-V3-FERAL`; Sniffle/SDR compatible para independencia | **possible with software/emulation plus suitable radio hardware**; no stack BLE FeralRF |
| Propietario Sub-GHz | AUX-Feral o generador/SDR; V2 stock SX1262 para FSK/GFSK compatible | **PHY-only validation currently practical** |
| Wireless M-Bus | AUX-Feral para markers; medidor/receptor W-MBus para protocolo | **requires protocol-specific third-party device**; V2 SX1262 sólo PHY FSK compatible |
| Wi-SUN / Amazon Sidewalk | AUX-Feral para markers; nodo/gateway real o radio programable compatible | **requires protocol-specific third-party device**; actualmente PHY-only |
| Propietario 2.4 GHz | AUX-Feral; SDR/generador 2.4 GHz para independencia | **possible with software/emulation plus suitable radio hardware**; V2 stock no confirmado |

Un PC normal no genera RF por sí solo: todo software/simulador requiere una radio capaz de la banda, modulación, tasa y framing correspondientes.

## Cómo interpretar resultados

- **PASS:** cumple el criterio observable definido; indicar si fue sólo control, OTA o instrumento.
- **FAIL:** incumple el criterio sin coincidir con una limitación ya documentada.
- **LIMITACIÓN CONOCIDA REPRODUCIDA:** coincide con firma y condiciones de un `KI-xx`.
- **REGRESIÓN NO REPRODUCIDA:** el caso históricamente fallido supera todas las repeticiones; no elimina el riesgo fuera de esas condiciones.
- **MARGINAL:** hay RF real, pero no alcanza la razón de éxito fijada o depende sensiblemente de posición/orientación.
- **INCONCLUSO:** falta observador independiente, tráfico, logs o control de variables.
- **BLOQUEADO / NO PROBADO:** falta hardware, software, autorización RF o una ruta de código completa.

Ruta común: PC/`feralrf.Radio` → USB CDC Cat-Bridge → RP2040 stock (puente transparente) → UART 921600 sin flow control → procesador de comandos FeralRF en CC1352P7 → `control_task`/`data_task` → `radio_if` → TI RF driver/radio core. `reset_device()` se desvía a Cat-Shell 115200, infiere `COM(n+2)` y conmuta RESET_CC mediante el RP2040. Crypto termina en aceleradores TI del CC1352P7. El SX1262/Cat-LoRa no participa.

## Seguridad RF

Todo test marcado **TX RF** debe realizarse sólo en un banco autorizado, con antena/carga/atenuación apropiada y cumpliendo regulación local. Use `0 dBm` o menos y la duración mínima indicada. CW, PRBS, TX continuo y jamming requieren idealmente conexión conducida atenuada o recinto apantallado. Jamming queda fuera de la primera sesión.

## First validation session — una placa, Windows, bajo riesgo

Ejecutar desde `FeralRF\python` en PowerShell, sustituyendo `COM_BRIDGE`. No iniciar por OOK, CW, PRBS, TX continuo o jamming.

1. `EV-00`: con sólo `DUT-V3-FERAL` conectado, ejecutar `catnip devices --debug`, `catnip identify --device N` y `catnip status --device N`; contrastar con enumeración serial de Windows y registrar Device ID, Board, Bridge, LoRa, Shell, serial/HWID/location.
2. Mapear los tres COM por identidad/interfaz, no por aritmética. El ID Catnip sólo vale para la enumeración actual.
3. `EV-01`: `init()`, INFO y STATS por el Bridge explícito; usar GET_INFO FeralRF para confirmar el firmware custom.
4. `EV-02`: IEEE 802.15.4, canal 25, RX cinco segundos y parada.
5. `EV-03`: desconectar/reconectar cinco veces y repetir el inventario si Windows reasigna puertos.
6. `EV-04`: reset sólo después de verificar Cat-Shell con Catnip; comprobar aparte que FeralRF intenta internamente `Bridge+2`.
7. `EV-05`: recibir durante 30 s la fuente conocida `PROTOCOL-DEVICE-ZIGBEE-CH25`; repetir tres veces.
8. `EV-06`: comprobar rechazo de TX mientras RX está activo; no transmite si el firmware rechaza correctamente.

Registro mínimo por sesión: fecha/hora, commit, hash del `.hex` si está disponible, versión Python/paquete, modelo/revisión/ID de placa, COM Bridge/Shell/LoRa, firmware RP2040, antena/CTF/U2, alimentación, comando y stdout íntegro.

## Comandos base para Windows

```powershell
cd C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python
python -c "from feralrf import Radio; print(Radio.list_devices())"
python examples\smoke_phy4_ieee154.py --port COM_BRIDGE --channel 25 --duration 5
```

Para fragmentos multilínea se usa sintaxis PowerShell nativa:

```powershell
@'
from feralrf import Radio
r = Radio(port="COM_BRIDGE")
print(r.init())
print(r.get_stats())
r.disconnect()
'@ | python -
```

`run_validation_baseline.sh` es Bash: en Windows use Git Bash (`bash python/examples/run_validation_baseline.sh ...`) o ejecute los scripts Python enumerados en `EV-40`. WSL sólo es adecuado si expone correctamente los puertos USB/serial; para COM nativos se prefieren Python y PowerShell.

## Nivel 0 — entorno y conexión

### EV-00 — Enumeración e identidad de puertos (P0)

**Objetivo:** identificar revisión, placa física y Bridge/Shell/LoRa sin depender de números contiguos. **Origen:** baseline y riesgo `KI-15`. **Evidencia/fuente:** FeralRF `radio.py:Radio.list_devices/_get_shell_port`; Catnip `usb_connection.py:find_devices/_group_ports_by_device/_map_roles`; `device/cli.py:devices/identify/status`. **Ruta:** Windows SetupAPI/pyserial → tres CDC del RP2040 V3 o SAMD21 V2; todavía no usa RF. **Roles mínimos:** `DUT-V3-FERAL`; conectar primero una placa. **Comandos:** `catnip devices --debug`; `catnip identify --device N`; `catnip status --device N`; `Get-CimInstance Win32_SerialPort | Select DeviceID,Name,PNPDeviceID`; luego `Radio.list_devices()`. **Configuración:** anotar Device ID temporal, Board, COM, descripción, HWID, serial, location e interface; repetir después de reconectar. **Qué hace:** Catnip agrupa por identidad USB y asigna roles por metadatos/interfaz; el fallback final ordena COM. FeralRF sólo busca Bridge y aún infiere Shell por suma. **Sano:** placa V3 y tres roles inequívocos, con identificación física coincidente. **Aceptación:** PASS sólo con identidad inequívoca; INCONCLUSO si Catnip cae a orden posicional o hay grupos ambiguos; FAIL si la placa presente no aparece. **Ahora:** plenamente ejecutable; los V2 se inventarían por separado. **AUX-V3:** repetir por placa y no reutilizar IDs entre enumeraciones. **Evidencia:** salida completa Catnip/Windows. **Límites:** `status` puede no reconocer la imagen FeralRF custom y no prueba UART CC. **Recuperación:** cerrar programas que ocupen COM; reconectar USB.

### EV-01 — Init, GET_INFO y GET_STATS (P0)

**Objetivo:** establecer PC↔RP2040↔CC1352. **Origen:** capacidad estable/baseline. **Evidencia:** `radio.py:init/get_stats`; `command_processor.c` casos `RADIO_INIT/GET_INFO/GET_STATS`; `protocol.h`. **Ruta:** ruta común, sin operación RF activa. **Prerrequisitos:** EV-00. **Comando:** fragmento PowerShell base anterior. **Configuración:** 921600; COM explícito. **Qué hace:** `init` reintenta tres veces, exige ACK e INFO ≥12 B; stats acepta 16/36 B. **Sano:** INFO consistente y cuatro contadores enteros; sin timeout. **Falla conocida:** timeout, payload corto, versión/capabilities inesperados. **Aceptación:** PASS en 3/3 ejecuciones; FAIL ante error repetible; INCONCLUSO si otro proceso ocupaba el puerto. **Evidencia:** `repr(info)`, stats y duración. **Límites:** no prueba RF. **Indicadores:** versión distinta, contadores que retroceden sin reset, `8E89BE` no aplica hasta RX. **Recuperación:** EV-04 o power-cycle.

### EV-02 — RX IEEE controlado y parada (P0)

**Objetivo:** probar configuración, RX_START/RX_STOP y retorno a idle. **Origen:** capacidad estable. **Evidencia:** `smoke_phy4_ieee154.py`; `Radio.set_phy/start_rx/stop_rx`; `command_processor.c`; `RadioIF_startRx/stopRx`. **Ruta:** ruta común → SmartRF IEEE RX. **Prerrequisitos:** una placa con antena 2.4 GHz. **Comando:** segundo comando base. **Configuración:** PHY 4, canal 25, 5 s. **Qué hace:** configura y abre RX; paquetes=0 sigue siendo control-path válido. **Sano:** INFO, SET_PHY/RX ACK, STOP y salida normal. **Falla conocida:** error asíncrono `ERR_RF_INIT_FAILED` o timeout de STOP. **Aceptación:** PASS-control si finaliza; PASS-RF sólo si se ve una trama real coherente; FAIL si no recupera. **Evidencia:** stdout, paquetes/RSSI. **Límites:** cero paquetes no condena RF. **Indicadores:** stream idéntico `8e89be`, bloqueo, reset. **Recuperación:** EV-04.

### EV-03 — Reconnect limpio (P0)

**Objetivo:** detectar fugas/estado serial residual. **Origen:** stress/lifecycle. **Evidencia:** `Radio.connect/disconnect/init`; reintentos en `init`. **Ruta:** ruta común repetida. **Prerrequisitos:** una placa. **Comando:** `1..5 | % { python -c "from feralrf import Radio; r=Radio(port='COM_BRIDGE'); print(r.init()); r.disconnect()" }`. **Configuración:** cinco procesos independientes. **Qué hace:** abre, vacía buffers, inicializa y cierra. **Sano:** 5/5 sin demora creciente. **Falla conocida:** COM ocupado, respuesta stale, timeout. **Aceptación:** PASS 5/5; FAIL cualquier repetición confirmada; INCONCLUSO si antivirus/otro proceso interviene. **Evidencia:** tiempos y número de ciclo. **Límites:** no prueba sesión larga. **Indicadores:** secuencia sólo falla tras N ciclos. **Recuperación:** cerrar proceso, EV-04.

### EV-04 — Reset y reinicialización (P0)

**Objetivo:** validar el mecanismo de recuperación antes de modos frágiles y caracterizar de forma segura `KI-15`. **Origen:** workaround `KI-15`. **Evidencia:** FeralRF `radio.py:reset_device`; `run_validation_baseline.sh:_reset_one`; `CatSniffer-Firmware/RP2040/catsniffer/boards/rpi_pico.overlay` (alias ejecutable `pin-reset` en GPIO3 y `pin-boot` en GPIO2); Catnip `usb_connection.py` y `device/cli.py`. **Ruta:** PC→Cat-Shell 115200→RP2040 `change_mode()`→líneas boot/reset del CC1352→Bridge. `FeralRF/hardware/PINOUT.md` menciona GPIO15, pero no coincide con el overlay ejecutable inspeccionado. **Roles mínimos:** `DUT-V3-FERAL`. **Prerrequisitos:** EV-00 con Shell verificado por Catnip y Windows; no basta observar puertos próximos. **Comando inseguro, sólo para referencia y no ejecutar en estas placas:** `Radio(port='COM_BRIDGE').reset_device(wait=3.5)`. **Configuración:** el baseline histórico espera 3.5 s. **Qué hace:** FeralRF cierra Bridge, calcula internamente `COM(n+2)`, manda `boot` y `exit`, reabre e inicializa. Catnip no cambia ese código, pero ofrece el mapa independiente previo. **Estado actual:** `COM88→COM90` y `COM31→COM33` no coinciden con los Shell reales `COM87`/`COM30`; por ello la API queda `BLOCKED`. Use únicamente `Reset-Cc1352` con mapeo explícito en la sección operativa. **Aceptación:** PASS-recovery manual si INFO/STATS regresan; la API no puede recibir PASS en esta configuración. **Evidencia:** mapa Catnip, COM calculado, Shell explícito, tiempos y errores. **Límites:** Catnip no permite pasar Shell a `reset_device()` ni valida watchdog. **Recuperación:** power-cycle y reidentificar puertos si la ruta manual falla.

### EV-05 — Primera observación RF IEEE (P1)

**Objetivo:** separar ACK de recepción física con una fuente independiente conocida. **Origen:** advertised Stable/missing current validation. **Evidencia:** README Quick Start; `smoke_phy4_ieee154.py`; `RadioIF_processIeee154Packets`. **Ruta:** `PROTOCOL-DEVICE-ZIGBEE-CH25` → antena/U2 → CC1352P7 → ruta común inversa. **Roles mínimos:** `DUT-V3-FERAL + PROTOCOL-DEVICE-ZIGBEE-CH25`; observador stock opcional. **Prerrequisitos:** EV-02; fuente Zigbee continua disponible en canal 25. **Comando:** `python examples\smoke_phy4_ieee154.py --port COM_BRIDGE --channel 25 --duration 30`, tres ejecuciones. **Procedimiento:** iniciar RX; durante cada ventana registrar todos los bytes raw, longitud, RSSI, LQI, CRC y timestamp si se expone; comprobar variabilidad y atribución plausible; opcionalmente variar distancia/orientación. No se requiere asociar el DUT a la red. **PASS-RF:** ≥1 trama CRC-válida atribuible y reproducible. **INCONCLUSO:** cero tramas si no se confirmó actividad simultánea de la fuente. **FAIL candidate:** V2/AUX stock u otro sniffer confirma tráfico fuerte simultáneo en canal 25 y FeralRF falla repetidamente. **FAIL:** estado irrecuperable, errores RF repetidos o tráfico claramente sintético/inválido. **Ahora:** plenamente ejecutable para RX; `OBS-V2-*-STOCK` sólo compara si `catnip status --device N` confirma `ti_sniffer` ya instalado; no ejecutar `sniff zigbee` a ciegas porque puede intentar flashear y Catnip no ofrece TI sniffer V2. **AUX-V3:** stock con `catnip sniff zigbee -c 25 --device N -w aux_ch25.pcap` es el observador independiente preferido, tras aceptar su posible cambio de firmware; Feral ofrece comparación simétrica menos fuerte. **Comparación:** presencia, tasa aproximada, longitudes, bytes cuando sea posible y tendencia RSSI, sin exigir RSSI idéntico. **Límites:** sólo IEEE raw/recepción; Wireshark Zigbee no demuestra stack Zigbee en FeralRF. **Recuperación:** STOP o EV-04.

### EV-06 — Exclusión RX/TX (P1)

**Objetivo:** verificar `ERR_INVALID_STATE` sin emitir RF. **Origen:** negativo/state machine. **Evidencia:** `protocol.md` RX/TX state rules; `command_processor.c`. **Ruta:** API→handlers de estado; TX debe rechazarse antes de radio. **Prerrequisitos:** una placa. **Comando:** PowerShell multilínea que ejecute `init(); set_phy(PHY.IEEE_802_15_4,25); start_rx();` y capture la excepción de `transmit(b'\x01', power_dbm=-20)`; `finally: stop_rx(); disconnect()`. **Configuración:** -20 dBm por cautela. **Qué hace:** intenta conflicto. **Sano:** `CommandError` código `0x05`; dispositivo sigue respondiendo a STATS. **Falla conocida:** ACK/TX durante RX o estado irrecuperable. **Aceptación:** PASS sólo con rechazo y recuperación; FAIL si transmite/timeout. **Evidencia:** excepción y stats posterior. **Límites:** no prueba simultaneidad real. **Indicadores:** error asíncrono perdido. **Recuperación:** STOP y EV-04.

## Nivel 1 — control de una placa

### EV-10 — Matriz de PHY por control (P1)

**Objetivo:** comprobar PHY 0–7, canal y potencia sin afirmar RF. **Origen:** baseline histórico. **Evidencia:** `smoke_phase2.py`; enum `PHY`; handlers SET_PHY/CHANNEL/POWER. **Ruta:** común→selección backend TI. **Prerrequisitos:** una placa; reset entre filas. **Comandos:** `python examples\smoke_phase2.py --port COM_BRIDGE --phy N --channel C --power 0`, con `(N,C)=(0,37),(1,9),(2,37),(3,37),(4,25),(5,0),(6,0),(7,0)`. **Configuración:** reset EV-04 entre PHY. **Qué hace:** ACK/config/RX breve. **Sano:** 8/8 terminan; posibles cero paquetes. **Falla conocida:** cambio sin reset se trata en EV-41. **Aceptación:** PASS-control por fila; no PASS-RF. **Evidencia:** stdout por fila. **Límites:** PHY 7 requiere `configure_prop` para una modulación concreta. **Indicadores:** PHY aceptado pero error RF asíncrono. **Recuperación:** EV-04.

### EV-11 — Presets propietarios, sólo control (P1/P2)

**Objetivo:** comprobar que cada preset alcanzable configura backend. **Origen:** baseline/pending validation. **Evidencia:** `presets.py:PROP_PRESETS`; `smoke_prop_phase1.py`; `RadioIF_setPropConfig`. **Ruta:** común→SET_PROP_CONFIG→SmartRF/patch seleccionado. **Prerrequisitos:** una placa; antena adecuada; no ejecutar OOK hasta EV-24. **Comando:** `python examples\smoke_prop_phase1.py --port COM_BRIDGE --preset NOMBRE --power 0`; usar todos salvo `ook_*` y `mioty_868_tsunb` inicialmente. **Configuración:** reset entre bandas. **Qué hace:** config, RX/TX smoke según script; **TX RF breve**. **Sano:** ACK y salida del script. **Falla conocida:** 433 marginal, W-MBus N no validado, MIOTY falla. **Aceptación:** PASS-control; RF queda INCONCLUSO sin receptor. **Evidencia:** preset, frecuencia, salida. **Límites:** preset no es protocolo. **Indicadores:** aceptación de frecuencia imposible, timeout al cambiar banda. **Recuperación:** EV-04.

**Roles corregidos para EV-11:** mínimo `DUT-V3-FERAL`; ahora sólo PASS-control. `OBS-V2-*-STOCK` puede observar independientemente FSK/GFSK Sub-GHz con `catnip sniff fsk` si frecuencia/tasa/desviación/ancho/sync son compatibles, pero no OOK, MSK, 4FSK ni 2.4 GHz. `AUX-V3-FERAL` habilita OTA simétrica de presets; `AUX-V3-STOCK` sólo es preferible en PHY que su firmware oficial exponga. Preparación: `catnip devices --debug` y `catnip status --device N`; no flashear V2.

### EV-12 — TX raw/frame/burst/continuous y stop (P1)

**Objetivo:** validar estado/ACK y, con observador, emisiones. **Origen:** Stable/baseline. **Evidencia:** `smoke_tx_{phase1,frame_phase1,burst_phase1,continuous_phase1}.py`; `control_task.c`. **Ruta:** común→TX handlers→RadioIF TX. **Prerrequisitos:** banco autorizado; observador recomendado. **Comandos:** los cuatro scripts con `--port COM_BRIDGE --phy 4 --channel 25 --power 0`; para continuo añadir `--run-seconds 1`. **Configuración:** 0 dBm, 1 s máximo continuo. **Qué hace:** RAW y FRAME one-shot, burst finito, continuo+STOP. **Sano:** scripts terminan y observador ve conteo/framing esperado. **Falla conocida:** ACK confirma aceptación, no TX_DONE (`KI-14`). **Aceptación:** PASS-control por ACK/STOP; PASS-RF sólo con observador; FAIL si STOP no cesa energía. **Evidencia:** stdout + captura/espectro. **Límites:** payload efectivo 125 B; BLE ADV 31 B. **Indicadores:** emisiones después de STOP, conteo incompleto, frecuencia errónea. **Recuperación:** STOP, luego EV-04/power-cycle.

**Roles corregidos para EV-12:** mínimo `DUT-V3-FERAL` para control; PASS-RF requiere `RF-OBSERVER`, `AUX-V3-FERAL` como receptor simétrico o un receptor independiente compatible. Ahora los V2 sólo pueden observar FSK/GFSK Sub-GHz por SX1262, no generar la matriz ni validar STOP de todas las PHY. AUX-Feral permite los scripts; AUX-Stock es más fuerte para IEEE/Sniffle/SX1262 cuando el firmware correspondiente está verificado. Preparación Catnip: sólo inventario/status; `sniff zigbee` o `sniff ble` puede auto-flashear el CC y exige autorización separada.

### EV-13 — CW y PRBS instrumentados (P2)

**Objetivo:** verificar frecuencia, potencia relativa y patrón; no sólo wire-level. **Origen:** histórico F22/`KI-07`. **Evidencia:** `lab/smoke_f22_tx_test.py`; `Radio.tx_cw/tx_prbs`; `RadioIF_runTxTest`. **Ruta:** común→CMD_TX_TEST→RF; **TX RF**. **Prerrequisitos:** segundo receptor para smoke; analizador/frecuencímetro/power meter para medición. **Comando:** `python examples\lab\smoke_f22_tx_test.py --tx-port COM_TX --rx-port COM_RX` (dos placas); para una placa use API sólo en banco conducido y detenga en ≤1 s. **Configuración:** potencia ≤0 dBm; PRBS15/32, no PRBS9 BLE DTM. **Qué hace:** CW/PRBS hasta STOP. **Sano:** energía sólo durante ventana, frecuencia correcta, STOP idempotente. **Falla conocida:** High-PA no enruta; docs/API discrepan entre +5 y +14 dBm. **Aceptación:** PASS-instrumento sólo con medición; wire-level queda parcial. **Evidencia:** span/RBW, pico, potencia, tiempo. **Límites:** receptor de paquetes sólo prueba caída de recepción, no pureza. **Indicadores:** armónicos, potencia no monótona, emisión persistente. **Recuperación:** `tx_test_stop`, EV-04, cortar alimentación si persiste.

**Roles corregidos para EV-13:** mínimo físico `DUT-V3-FERAL + RF-OBSERVER`; `AUX-V3-FERAL` sólo aporta el smoke de caída/recuperación de paquetes y no mide frecuencia, potencia ni pureza. V2/AUX stock no sustituyen analizador, contador o medidor. Catnip sólo se usa para mapear/status antes y después; no usar `verify`. El observador independiente sigue siendo preferible incluso con dos V3.

### EV-14 — Error RF asíncrono y firma sintética (P0/P1)

**Objetivo:** caracterizar fallo de inicio sin confundirlo con tráfico. **Origen:** `KI-12/KI-31`. **Evidencia:** `data_task.c:DataTask_poll` emite `ERR_RF_INIT_FAILED`; `radio.py` bufferiza seq 0/FF; `radio_if.c` conserva backend sintético; Linux doc menciona `8E 89 BE`. **Ruta:** RX_START ACK→inicio diferido RF→RSP_ERROR asíncrono. **Prerrequisitos:** una placa; no forzar daño. **Comando:** durante EV-02/10 conservar objetos `RxStreamError` y bytes. **Configuración:** 10 s. **Qué hace:** observa respuesta tardía. **Sano:** no hay error; si RF falla, aparece error explícito y no se cuenta `8e89be` como RF. **Falla conocida:** docs históricas afirman fallback visible mientras código actual dice reportar error. **Aceptación:** PASS si comportamiento es inequívoco; LIMITACIÓN si aparece firma documentada; FAIL si falla silenciosamente. **Evidencia:** orden temporal ACK/error/paquetes. **Límites:** quizá no sea reproducible sin una falla real. **Indicadores:** stream sintético tras error. **Recuperación:** EV-04.

### EV-15 — Límites y parámetros inválidos (P2)

**Objetivo:** validar rechazo host/firmware sin lock-up. **Origen:** negative test/`KI-29`. **Evidencia:** builders/API; `command_processor.c`; `protocol.md`. **Ruta:** validación Python o RSP_ERROR firmware. **Prerrequisitos:** una placa; usar API pública, no bytes corruptos al inicio. **Comandos:** probar payload 0/125/126, BLE frame 31/32, IEEE 125/126, canales 10/27, potencia -21/+15 y `random_bytes(0/241)` capturando excepciones. **Configuración:** no repetir el caso que realmente transmita sin observador; priorizar errores locales. **Qué hace:** delimita fronteras. **Sano:** valores fuera de rango fallan rápido; el siguiente GET_STATS funciona. **Falla conocida:** límites host y firmware no son uniformes. **Aceptación:** PASS si rechazo y recuperación coinciden; FAIL si crash/hang/aceptación peligrosa. **Evidencia:** entrada, capa que rechazó, código. **Límites:** malformed wire frames requieren harness separado. **Indicadores:** truncamiento silencioso. **Recuperación:** EV-04.

## Niveles 2–4 — evidencia RF, OTA y casos de uso

### EV-20 — IEEE 802.15.4 OTA de dos placas (P1)

**Objetivo:** reproducir 10/10 markers y framing raw. **Origen:** histórico 2026-04-08. **Evidencia:** `smoke_ota_txrx.py`; matriz §2. **Ruta:** TX CC1352→aire canal 25→RX CC1352→host. **Prerrequisitos:** dos CatSniffer, antenas 2.4 GHz. **Comando:** `python examples\smoke_ota_txrx.py --tx-port COM_TX --rx-port COM_RX --phy 4 --channel 25`. **Configuración:** documentar separación/orientación, 0 dBm; reset ambas antes. **Qué hace:** emite markers DEADBEEF. **Sano:** 10/10. **Falla conocida:** selector RF externo/U2 no está controlado por API (`KI-32`). **Aceptación:** PASS ≥10/10; MARGINAL 1–9/10; FAIL 0/10 con control-path sano. **Evidencia:** stdout de ambos y setup. **Límites:** no interoperabilidad Zigbee. **Indicadores:** asimetría al invertir placas. **Recuperación:** reset ambas.

**Roles corregidos para EV-20:** el baseline de `smoke_ota_txrx.py` requiere exactamente `DUT-V3-FERAL + AUX-V3-FERAL`. Ahora, `DUT-V3-FERAL + PROTOCOL-DEVICE-ZIGBEE-CH25` valida de forma independiente la mitad RX, pero no el marker TX ni la simetría. Un V2 sólo observa canal 25 si `catnip status` confirma `ti_sniffer` ya instalado; no hay imagen TI-sniffer V2 en el catálogo actual. `AUX-V3-STOCK` con `catnip sniff zigbee -c 25 --device N -w aux_ch25.pcap` es preferible para verificar independientemente TX/canal/frame, aceptando que el comando puede reflashear el CC1352. No demuestra stack Zigbee FeralRF.

### EV-21 — BLE raw 1M/2M/Coded OTA (P1)

**Objetivo:** validar cuatro PHY y workaround 2M. **Origen:** histórico/missing current validation; `KI-09`. **Evidencia:** matriz §2/13; `radio_if.c` extended ADV; bt5 patch. **Ruta:** TX BLE raw→aire→RX raw. **Prerrequisitos:** dos placas o sniffer BLE PHY compatible; decodificador opcional. **Comandos:** `smoke_ota_txrx.py` con `--phy 0/1/2/3 --channel 37`; RX 2M escucha internamente/según script ch9. **Configuración:** reset entre filas. **Qué hace:** 2M usa ADV_EXT 1M ch37→AUX 2M ch9; demás markers raw. **Sano:** historia: 1M/S8/S2 10/10, 2M 8/10. **Falla conocida:** patch multi_protocol cuelga AUX y defaultPhy=2M cuelga segundo TX si regresan workarounds. **Aceptación:** PASS ≥8/10 en 2M y 10/10 restantes; MARGINAL por debajo; FAIL lock-up. **Evidencia:** conteos y segundo TX explícito. **Límites:** no advertising stack, scan activo, conexión ni GATT. **Indicadores:** primer TX sí/segundo no. **Recuperación:** reset.

**Roles corregidos para EV-21:** OTA del repositorio requiere `DUT-V3-FERAL + AUX-V3-FERAL`. Ahora no es reproducible con los V2 preservados; Sniffle V2 sólo participaría si `status` confirma que ya está instalado, y cambiarlo se difiere. `AUX-V3-STOCK` + Sniffle/SDR es referencia independiente para las PHY que capture, pero no se presume cobertura de todos los modos 2M/Coded. Preparación: inventario/status; `catnip sniff ble ...` puede auto-flashear y no se ejecuta sin registrar/autorizar firmware.

### EV-22 — Matriz Sub-1 GHz 868/915 y 4FSK (P1)

**Objetivo:** reproducir presets estables. **Origen:** baseline 2026-04-08. **Evidencia:** baseline script; presets; matriz §5. **Ruta:** SET_PROP_CONFIG→TI proprietary TX/RX OTA. **Prerrequisitos:** dos placas, antenas apropiadas; usar la unidad fuerte como TX. **Comandos:** `smoke_ota_txrx.py --tx-port COM_TX --rx-port COM_RX --preset PRESET` para `gfsk_868_50k`, `gfsk_915_50k`, `msk_868_50k`, `4fsk_868_50k`, `4gfsk_868_50k`. **Configuración:** reset entre banda/preset. **Qué hace:** marker PHY, no stack. **Sano:** 10/10 cada uno. **Falla conocida:** placa histórica #2 TX débil 868 (`KI-06`). **Aceptación:** PASS 10/10; MARGINAL 1–9; invertir roles para localizar hardware. **Evidencia:** rol/serial/antena/conteo. **Límites:** no certifica espectro. **Indicadores:** falla sólo un rol. **Recuperación:** reset.

**Roles corregidos para EV-22:** matriz completa requiere `AUX-V3-FERAL`; V2 stock puede observar independientemente sólo FSK/GFSK 868/915 con `catnip sniff fsk` y parámetros exactos, no MSK/4FSK. `AUX-V3-STOCK` tiene la misma vía SX1262 FSK, no una cobertura automática de presets CC. Un SDR/analizador sigue siendo preferible para frecuencia y espectro. Preparación Catnip V2: `devices/status`, luego `sniff fsk` únicamente si se decide capturar y sin modificar su firmware CC.

### EV-23 — Caracterización 433 MHz (P2)

**Objetivo:** cuantificar la marginalidad, no dar PASS binario. **Origen:** `KI-04/KI-05`. **Evidencia:** matriz: GFSK 6–10/10, FSK/MSK 1/10, OOK 0/10; `smoke_f9_phy_matrix_ota.py`. **Ruta:** proprietary 433 OTA. **Prerrequisitos:** dos placas; ideal antena 433 o atenuador/conducido. **Comando:** baseline OTA por presets `gfsk_433_50k`, `fsk_433_50k`, `msk_433_50k`; 10 corridas de 10 markers en dos orientaciones. **Configuración:** misma distancia/potencia, luego invertir placas; OOK separado en EV-24. **Qué hace:** estima tasa y sensibilidad a geometría. **Sano:** resultado estable compatible con setup; 433 no se etiqueta estable por éxito aislado. **Falla conocida:** bajo link budget/antena 868–915. **Aceptación:** PASS caracterización si hay 100 ensayos registrados; MARGINAL <95%; FAIL 0 sostenido; INCONCLUSO sin antena conocida. **Evidencia:** éxitos/100, RSSI, orientación. **Límites:** no separa antena de FW sin instrumento. **Indicadores:** fuerte asimetría. **Recuperación:** reset entre presets.

**Roles corregidos para EV-23:** OTA cuantitativo requiere `AUX-V3-FERAL`; ahora V2 stock puede aportar observación independiente sólo para FSK/GFSK 433 que el SX1262 y antena/configuración admitan, no MSK/OOK. Un `RF-OBSERVER` y antena 433 son preferibles para separar firmware, front-end y antena. AUX-Stock no cubre por sí solo todas las modulaciones.

### EV-24 — OOK 868/433, lock y recovery (P1/P2)

**Objetivo:** confirmar OTA y delimitar bloqueo `KI-01/KI-05`. **Origen:** limitación/bug histórico. **Evidencia:** `RadioIF_getPropMode/setPropConfig`; `smoke_prop_phase1.py --auto-reset`; matriz §§5,10. **Ruta:** carga patches genook no descargables→OOK TX/RX; luego reset por Shell. **Prerrequisitos:** dos placas; OOK siempre último; banco autorizado. **Comandos:** `smoke_ota_txrx.py ... --preset ook_868_4k8`; después intentar una sola configuración BLE/IEEE controlada, registrar firma, ejecutar EV-04 y repetir EV-01; OOK433 sólo con antena/equipo adecuado. **Configuración:** 10 markers, 0 dBm; no cambiar frecuencia OOK antes del reset. **Qué hace:** prueba operación, transición fallida y recuperación. **Sano:** OOK868 10/10; operación posterior puede bloquear según historia; reset restaura 3/3 init. **Falla conocida:** lock/hang; OOK433 0/10 histórico. **Aceptación:** LIMITACIÓN REPRODUCIDA si lock+recovery exactos; REGRESIÓN NO REPRODUCIDA si 10 ciclos OOK→IEEE sin reset funcionan; FAIL si reset no recupera. **Evidencia:** última respuesta, timeout y recovery. **Límites:** evitar insistencia ante hang. **Indicadores:** configuración ACK pero frecuencia OOK anterior. **Recuperación:** reset físico/power-cycle.

**Roles corregidos para EV-24:** `AUX-V3-FERAL` es necesario para OTA OOK simétrica; Catnip stock V2/V3 no expone un workflow OOK equivalente. `RF-OBSERVER` es preferible y requerido para validar portadora/espectro; OOK433 permanece condicionado a antena/equipo. Preparación Catnip se limita a inventario/status; no alterar V2.

### EV-25 — Wireless M-Bus S/T/C/N (P1/P2)

**Objetivo:** distinguir preset OTA de interoperabilidad W-MBus. **Origen:** advertised Stable/pending N. **Evidencia:** presets; matriz §§2,5,14. **Ruta:** proprietary GFSK raw; no parser/stack. **Prerrequisitos:** dos placas para markers; medidor/receptor W-MBus para interoperabilidad. **Comandos:** `smoke_ota_txrx.py ... --preset wireless_mbus_{s,t,c}_868`; N con `wireless_mbus_n_169_2k4` sólo con RF 169 MHz adecuada. **Configuración:** reset por preset; registrar modo/frecuencia. **Qué hace:** demuestra compatibilidad de parámetros raw. **Sano:** S/T/C 10/10; N es nueva evidencia. **Falla conocida:** N no probado y antena/front-end inciertos. **Aceptación:** PASS-PHY por markers; PASS-interoperabilidad sólo si tercero decodifica una trama conforme; BLOCKED N sin antena/observer. **Evidencia:** captura decodificada. **Límites:** markers no son telegramas W-MBus. **Indicadores:** README “Stable” entendido como stack. **Recuperación:** reset.

**Roles corregidos para EV-25:** markers requieren `AUX-V3-FERAL`; V2/AUX stock con SX1262 y `sniff fsk` puede aportar evidencia PHY FSK si todos los parámetros/framing caben, nunca decodificación W-MBus garantizada. PASS-interoperabilidad requiere `PROTOCOL-DEVICE` W-MBus o receptor independiente conforme; N169 requiere además cadena RF/antena adecuada. Observador independiente preferible.

### EV-26 — Wi-SUN y Sidewalk FSK (P2)

**Objetivo:** reproducir carrier/preset sin atribuir stack. **Origen:** experimental/histórico F29. **Evidencia:** presets y `smoke_f29_subg_915.py`; demos `demo_wisun_scan.py`, `demo_sidewalk_subg.py`; matriz 70/70. **Ruta:** proprietary FSK 902.2/915 MHz. **Prerrequisitos:** dos placas o SDR; autorización regional. **Comando:** `python examples\smoke_f29_subg_915.py --tx-port COM_TX --rx-port COM_RX` (ver `--help` antes y conservar valores por defecto); alternativamente `smoke_ota_txrx.py` por siete presets. **Configuración:** potencia mínima; reset por tasa. **Qué hace:** transfiere markers con parámetros FSK. **Sano:** 10/10 por siete presets. **Falla conocida:** no FAN MAC/security ni Sidewalk; LR/LoRa reside en SX1262 fuera de FeralRF. **Aceptación:** PASS-PHY, nunca PASS-protocolo; FAIL si current HEAD no reproduce. **Evidencia:** conteo por preset y espectro opcional. **Límites:** un SDR sólo prueba energía/modulación. **Indicadores:** rate/frecuencia distinta al preset. **Recuperación:** reset.

**Roles corregidos para EV-26:** `AUX-V3-FERAL` reproduce los 70/70 raw; V2/AUX stock mediante SX1262 puede observar FSK compatible 902/915 como PHY, no FAN MAC/security ni Sidewalk. La interoperabilidad requiere nodo/gateway de protocolo; SDR/radio programable confirma PHY. Catnip: `sniff fsk` sólo con parámetros explícitos y banda legal.

### EV-27 — Propietario 2.4 GHz (P1/P2)

**Objetivo:** resolver README experimental vs matriz 10/10 GFSK2440. **Origen:** discrepancia `KI-08`. **Evidencia:** README Protocols; matriz §§2,5,11; preset `gfsk_2440_50k`. **Ruta:** proprietary backend→front-end 2.4→OTA. **Prerrequisitos:** dos placas y/o SDR 2.4. **Comando:** `smoke_ota_txrx.py ... --preset gfsk_2440_50k`; repetir `gfsk_2440_250k`. **Configuración:** 10 ciclos y reset. **Qué hace:** prueba round-trip modulado, no sólo CW. **Sano:** 10/10 repetible y centro en 2440 MHz. **Falla conocida:** README dice round-trip pendiente. **Aceptación:** PASS sólo con OTA+frecuencia; INCONCLUSO si ACK/CW; registrar contradicción resuelta para este commit. **Evidencia:** markers y captura SDR. **Límites:** no certifica cualquier GFSK 2.4. **Indicadores:** energía por ruta Sub-1/U2. **Recuperación:** reset.

**Roles corregidos para EV-27:** OTA requiere `AUX-V3-FERAL`; el tooling stock inspeccionado no confirma recepción propietaria 2.4 GHz en V2 ni en la ruta SX1262. `AUX-V3-STOCK` sólo ayuda si se instala/verifica firmware CC capaz; no asumirlo. SDR 2.4 GHz o analizador es el observador independiente preferido.

### EV-28 — Emulación PHY-level (P2)

**Objetivo:** verificar que helpers emiten firmas, sin llamarlos emulador de stack. **Origen:** histórico F17. **Evidencia:** `python/feralrf/emulation/`; `smoke_f17_emulation.py`; matriz §8. **Ruta:** helper host→config/TX burst→observer. **Prerrequisitos:** dos placas o receptor compatible. **Comando inicial seguro:** `python examples\smoke_f17_emulation.py --tx-port COM_TX --rx-port COM_RX --count 20 --skip-ook --skip-433`; luego retirar `--skip-433` para caracterización 433 y, sólo al final con recovery disponible, retirar también `--skip-ook`. **Configuración:** personalidades IEEE, 868, 433/OOK. **Qué hace:** envía payloads/presets predefinidos. **Sano:** payload observado coincide byte a byte. **Falla conocida:** sin auth/framing/encryption; WMBus helper usa fallback MSK según tests. **Aceptación:** PASS-firma, no PASS-interoperabilidad; BLOCKED OOK sin recovery. **Evidencia:** bytes capturados. **Límites:** nombre de personalidad no prueba dispositivo real. **Indicadores:** parser tercero rechaza. **Recuperación:** reset, obligatorio tras OOK.

**Roles corregidos para EV-28:** el script completo requiere `AUX-V3-FERAL`; stock sólo observa subconjuntos soportados: IEEE con TI sniffer verificado y Sub-GHz FSK/GFSK por SX1262. V2 no es peer FeralRF. Para afirmar emulación/interoperabilidad se requiere `PROTOCOL-DEVICE` o decodificador independiente; markers entre dos FeralRF prueban sólo firma simétrica.

### EV-29 — KillerBee real: sniff e inject (P2)

**Objetivo:** pasar de mocks a integración física. **Origen:** missing validation/host limitation. **Evidencia:** `integrations/killerbee.py`; `killerbee_sniff.py`; Linux runbook. **Ruta:** KillerBee→adapter→Radio→IEEE PHY. **Prerrequisitos:** Linux/WSL USB serial fiable, fork/parche KillerBee, pyusb/libusb; red propia y segundo sniffer para inject. **Comandos:** en Linux `zbid`; `zbdump -i /dev/ttyACM0 -d feralcat -c 25 -w cap.pcap`; inject según snippet del runbook. En Windows, primero `python examples\killerbee_sniff.py --port COM_BRIDGE --channel 25 --count 10`. **Configuración:** canal propio; no jamming. **Qué hace:** pnext, FCS, PCAP e inject. **Sano:** Wireshark DLT_IEEE802_15_4_WITHFCS válido y segundo observador ve 3 requests. **Falla conocida:** dependencia opcional ausente; reset_on_init puede romper stock bridge. **Aceptación:** PASS sólo end-to-end; host skip no es fallo FW. **Evidencia:** PCAP, versiones, stdout. **Límites:** no prueba ataque/key capture. **Indicadores:** FCS malo universal, autodetección errónea. **Recuperación:** cerrar KillerBee y EV-04.

**Roles corregidos para EV-29:** ahora `DUT-V3-FERAL + PROTOCOL-DEVICE-ZIGBEE-CH25 + HOST-TOOL` permite el primer sniff real, pero KillerBee falta en el baseline host y la integración completa sigue pendiente. Registrar canal 25, FCS, PCAP y decode Wireshark; no confundir decode Zigbee con stack FeralRF. `OBS-V2-*-STOCK` sólo sirve como captura paralela si ya ejecuta TI sniffer; `AUX-V3-STOCK` preparado con `catnip sniff zigbee -c 25 --device N -w aux_ch25.pcap` es la referencia preferida. AUX-Feral sirve para inject/observación simétrica, menos independiente.

## Nivel 5 — crypto

### EV-30 — Vectores independientes en aceleradores CC1352 (P1)

**Objetivo:** reproducir F25 9/9 con oráculo host. **Origen:** Stable/histórico 2026-04-30. **Evidencia:** `lab/smoke_f25_crypto.py`; `crypto_engine.c`; `test_crypto_vectors.py`. **Ruta:** Python→comandos 0x59–0x62→drivers AES/SHA/TRNG/ECC del CC1352. **Prerrequisitos:** una placa; paquete `cryptography`. **Comando:** `python examples\lab\smoke_f25_crypto.py --port COM_BRIDGE`. **Configuración:** sin RF. **Qué hace:** FIPS/NIST vectors, round-trips y verificación cruzada host/chip. **Sano:** 9/9; TRNG cambia; P-256/Curve25519 ECDH, ECDSA P-256. **Falla conocida:** ECDSA Curve25519 no aplica/rechazo 0x05; AES-GCM evita longitud problemática documentada. **Aceptación:** PASS sólo si bytes igualan oráculo independiente; FAIL por mismatch/hang. **Evidencia:** stdout completo. **Límites:** un smoke no certifica RNG. **Indicadores:** reset requerido tras GCM/TRNG. **Recuperación:** EV-04.

### EV-31 — Límites y repetición crypto (P2)

**Objetivo:** detectar hangs, límites one-shot y estado residual. **Origen:** workaround/stress. **Evidencia:** `crypto_engine.c` dependencia TRNG; API limita TRNG/SHA a 240 B; comentario GCM hang. **Ruta:** misma EV-30 repetida. **Prerrequisitos:** una placa. **Comando:** ejecutar EV-30 20 veces; añadir límites 1/240 y rechazo 0/241 para TRNG/SHA, tags CCM/GCM válidos/erróneos. **Configuración:** registrar latencia. **Qué hace:** estresa dominios de potencia y drivers. **Sano:** 20/20, resultados deterministas salvo TRNG, sin latencia creciente. **Falla conocida:** hang AESGCM en caso evitado por smoke; auth failure debe ser error, no plaintext. **Aceptación:** PASS con recuperación normal; FAIL timeout/reset/mismatch. **Evidencia:** ciclo, primitiva, tamaño, ms. **Límites:** no test estadístico profundo de RNG. **Indicadores:** primera llamada tras RF difiere. **Recuperación:** EV-04.

## Nivel 6 — estrés y regresión

### EV-40 — Baseline completo actual (P1)

**Objetivo:** reproducir historia completa contra HEAD. **Origen:** repositorio validation baseline. **Evidencia:** `run_validation_baseline.sh`; matriz pide recórrela. **Ruta:** todos los handlers/PHY, reset por step. **Prerrequisitos:** dos placas; Git Bash; roles/antenas confirmados. **Comando:** desde raíz en Git Bash: `bash python/examples/run_validation_baseline.sh --port COM_TX --rx-port COM_RX`; en una placa omitir `--rx-port` (sólo control). **Configuración:** script usa 921600 y Shell `port+2`; OOK último. **Qué hace:** 18 control steps y OTA soportado; no cubre OTA de TX_FRAME/BURST/CONTINUOUS ni scans BLE. **Sano:** summary PASS y conteos históricos como referencia. **Falla conocida:** resets pueden fallar; 433 marginal; OOK433 OTA omitido. **Aceptación:** guardar control y OTA separados; cualquier ACK no sustituye OTA. **Evidencia:** log íntegro, commit/binario. **Límites:** Bash/COM y supuestos de puertos. **Indicadores:** step pasa sólo por reset previo. **Recuperación:** reset/power-cycle de ambas.

**Roles corregidos para EV-40:** una placa permite sólo baseline de control. El baseline OTA histórico completo exige `DUT-V3-FERAL + AUX-V3-FERAL`; ni “dos CatSniffers” genéricos ni V2 stock satisfacen los puertos FeralRF del script. Ejecutar EV-00/04 en ambas antes; el script conserva su propio supuesto `port+2`, por lo que hay que verificarlo placa por placa. AUX-Stock se reserva para capturas independientes separadas, no como `--rx-port` FeralRF.

### EV-41 — Cambio PHY sin reset (P1)

**Objetivo:** reproducir/falsar deadlock del segundo ciclo. **Origen:** histórico `KI-02`. **Evidencia:** matriz §10; reglas RF/RF_close. **Ruta:** SET_PHY→RF_flush/yield/reconfig sobre handles persistentes. **Prerrequisitos:** una placa; recovery EV-04 ya probado. **Comando:** script PowerShell/API que repita 10 veces `BLE1M→IEEE→BLE1M`, cada fase con start/stop RX y GET_STATS, sin reset. **Configuración:** canales 37/25; timeout 3 s; detener al primer hang. **Qué hace:** fuerza transición documentada. **Sano:** 10/10 ciclos. **Falla conocida:** deadlock `RF_close` en segundo ciclo/timeout. **Aceptación:** LIMITACIÓN si firma histórica; REGRESIÓN NO REPRODUCIDA si 10/10; FAIL nuevo si crash distinto. **Evidencia:** transición/ciclo/último ACK. **Límites:** sólo una secuencia. **Indicadores:** latencia creciente. **Recuperación:** EV-04; power-cycle si Shell falla.

### EV-42 — Cambios con reset y entre bandas (P1)

**Objetivo:** validar workaround y 433↔868. **Origen:** `KI-03`. **Evidencia:** API reset recommendations; matriz §10. **Ruta:** reset físico entre cada configuración. **Prerrequisitos:** una placa. **Comando:** 10 ciclos BLE→reset→IEEE→reset→GFSK433→reset→GFSK868→reset→BLE, con GET_STATS. **Configuración:** no OOK. **Qué hace:** baseline de transición protegida. **Sano:** 10/10 sin timeout. **Falla conocida:** sintetizador inconsistente sin reset. **Aceptación:** PASS-workaround; FAIL si reset no protege. **Evidencia:** tiempos/reintentos. **Límites:** no demuestra que reset sea necesario. **Indicadores:** sólo falla al cruzar 861 MHz. **Recuperación:** power-cycle.

### EV-43 — RX soak y contadores (P2)

**Objetivo:** descubrir drops, overflow, starvation y stats no monótonas. **Origen:** stress/colas. **Evidencia:** `lab/canary_regression.py`; `packet_queue.c` depth 32; DataTask procesa máximo 8/poll. **Ruta:** `PROTOCOL-DEVICE-ZIGBEE-CH25`→RX RF→colas estáticas→UART→host. **Roles mínimos:** `DUT-V3-FERAL + PROTOCOL-DEVICE-ZIGBEE-CH25`. **Prerrequisitos:** EV-05 estable. **Comando inicial:** `python examples\lab\canary_regression.py --port COM_BRIDGE --phy 4 --channel 25 --soak-duration 60 --report-every 15 --profile quiet`; después repetir 5 minutos; sólo alargar tras ambos niveles estables. **Registro:** paquetes, fallos CRC si se exponen, drops/overflow, stats antes/después, monotonicidad, y que STOP/reconnect funcionen. **Sano:** sin hang y contadores monotónicos. **Aceptación:** PASS-soak si termina y recupera; los drops se interpretan con cautela porque la tasa Zigbee no está controlada. **Ahora:** ejecutable con la fuente existente; V2/AUX stock puede confirmar actividad simultánea bajo las condiciones de EV-05/20. **Límites:** no inferir capacidad de cola de este tráfico no controlado; usar EV-44. **Recuperación:** STOP/EV-04.

### EV-44 — Presión de cola/burst (P2)

**Objetivo:** localizar capacidad y recuperación de overflow. **Origen:** `KI-13`. **Evidencia:** `PACKET_QUEUE_DEPTH=32`; RX buffer 16 KiB; IEEE rearm tras overflow. **Ruta:** generador/segunda placa→RX rápido→colas→UART. **Prerrequisitos:** segunda placa/generador con tasa controlable. **Comando TX:** `python examples\lab\ota_tx_burst.py --port COM_TX --phy 4 --channel 25 --power 0 --payload-hex DEADBEEF --count 100 --interval-us 25000`; repetir con 10000, 5000 y 1000 µs. **Comando RX:** `python examples\lab\canary_regression.py --port COM_RX --phy 4 --channel 25 --soak-duration 60 --report-every 5 --min-packets 0`. **Configuración:** escalones, no RF continuo innecesario. **Qué hace:** eleva presión mientras consulta stats. **Sano:** drops/overflow contabilizados y RX continúa después de bajar tasa. **Falla conocida:** descarte silencioso en output queue; rearm IEEE especial. **Aceptación:** PASS caracterización si umbral y recovery repetibles; FAIL si lock/reset. **Evidencia:** enviados/recibidos/drop/overflow. **Límites:** clocks no sincronizados. **Indicadores:** contadores no explican pérdida. **Recuperación:** STOP/reset.

**Roles corregidos para EV-44:** requiere `DUT-V3-FERAL + AUX-V3-FERAL` o un generador RF independiente de tasa controlable. Los V2 stock no son generadores burst IEEE confirmados por la implementación Catnip inspeccionada; la fuente Zigbee CH25 tampoco tiene tasa controlable. AUX-Stock sólo vale si una herramienta oficial demuestra generación programable, no observación pasiva. Observador independiente adicional es deseable para separar pérdida de TX/RX. Preparación: mapear ambas placas con Catnip; no usar stock como `COM_TX` del script FeralRF.

### EV-45 — Wrap de secuencia (>253) (P2)

**Objetivo:** confirmar corrección del timeout histórico al TX 253. **Origen:** pre-fix bug en `test_radio_seq.py`. **Evidencia:** `_next_seq` reserva 0xFF; DataTask también lo salta. **Ruta:** 300 requests control con correlación SEQ. **Prerrequisitos:** una placa; evitar 300 TX: usar GET_STATS. **Comando:** script API `init()` y `for i in range(300): get_stats()` registrando ciclo/latencia. **Configuración:** sin RF. **Qué hace:** cruza wrap sin interferencia. **Sano:** 300/300. **Falla conocida:** antiguo timeout en transmisión 253. **Aceptación:** PASS si ningún timeout/stale response; FAIL reproducible alrededor de 253–255. **Evidencia:** índice y error. **Límites:** no prueba TX queue, sólo SEQ. **Indicadores:** respuesta stale filtrada. **Recuperación:** reconnect/reset.

### EV-46 — Re-init y ciclo de vida (P1/P2)

**Objetivo:** probar `RADIO_INIT` a media sesión y cierres guardados. **Origen:** workaround RF_close/state. **Evidencia:** `RadioIF_init` fuerza close de non433 handle; `init` reabre serial en retry. **Ruta:** sesión RF→RADIO_INIT→teardown/reopen. **Prerrequisitos:** una placa. **Comando:** 20 ciclos: init, set IEEE, start/stop RX, init de nuevo, GET_STATS; además 20 procesos EV-03. **Configuración:** timeout 3 s. **Qué hace:** fuerza ruta de excepción a “no RF_close”. **Sano:** 20/20. **Falla conocida:** deadlock `SemaphoreP_pend`. **Aceptación:** FAIL al primer hang; REGRESIÓN NO REPRODUCIDA 20/20. **Evidencia:** ciclo/última llamada. **Límites:** podría alterar contadores. **Indicadores:** close tarda progresivamente. **Recuperación:** EV-04.

### EV-47 — Recuperación tras interrupción/error (P2)

**Objetivo:** verificar que STOP/reset restauran baseline. **Origen:** recovery regression. **Evidencia:** retries `stop_rx`, fallback `stop_jam→TX_STOP`, reset docs. **Ruta:** host termina durante RX/TX finito; nuevo host reabre. **Prerrequisitos:** una placa; no cortar durante flash; TX sólo 1 s autorizado. **Comando:** interrumpir `smoke_phy4_ieee154.py` con Ctrl-C, luego EV-01/02; repetir tras error inválido. **Configuración:** no jamming en esta prueba. **Qué hace:** simula host abortado. **Sano:** reconexión sin power-cycle. **Falla conocida:** estado TX/RX stale. **Aceptación:** PASS si baseline vuelve; FAIL si sólo power-cycle recupera. **Evidencia:** proceso/estado antes y después. **Límites:** no desconexión eléctrica. **Indicadores:** COM abre pero comandos timeout. **Recuperación:** EV-04/power-cycle.

## Nivel 7 — experimental, incompleto, retirado

### EV-50 — Jamming continuo, criterio físico (P3, laboratorio solamente)

**Objetivo:** distinguir ACK de interferencia real. **Origen:** experimental `KI-17/KI-18`. **Evidencia:** `_jamming.py`; `Radio.start_jam/stop_jam`; handlers 0x30/0x33; `smoke_jam_phase1.py`. **Ruta:** comando→sesión RF TX repetida→canal; **TX RF interferente**. **Prerrequisitos:** recinto apantallado o conexión conducida, víctima/generador y observador propios, autorización. **Comando:** `python examples\lab\smoke_jam_phase1.py --port COM_BRIDGE --channel C --power -20 --duration-ms 100 --wait-ms 200` tras confirmar `--help`. **Configuración:** 100 ms inicial; máximo API 30 s, no usarlo por defecto. **Qué hace:** inicia/detiene sesión; medir PER/RSSI antes-durante-después. **Sano:** energía/efecto sólo en ventana y recovery total. **Falla conocida:** STOP reintenta y cae a TX_STOP; reset recomendado. **Aceptación:** PASS experimental sólo con cambio físico repetible y cese; ACK solo=INCONCLUSO. **Evidencia:** PER/espectro y legal setup. **Límites:** no jamming reactivo/pattern. **Indicadores:** energía persiste. **Recuperación:** STOP, EV-04 y power off.

**Roles corregidos para EV-50:** mínimo `DUT-V3-FERAL + víctima/generador independiente + RF-OBSERVER`, siempre en recinto o conexión conducida y con autorización. AUX-Feral puede ser víctima controlada, pero comparte implementación; AUX-Stock o un `PROTOCOL-DEVICE` es preferible para efecto independiente. V2 sólo observa si su firmware/radio soporta exactamente la señal; no reemplaza medición espectral. Catnip sólo inventaría/status; `verify` no es preparación pasiva.

### EV-51 — Spectrum/RSSI scan (P3)

**Objetivo:** documentar ausencia de data path, no intentar demostrar soporte. **Origen:** planned/incomplete `KI-16`. **Evidencia:** `_spectrum.py`/`_responses.py` dataclasses/builders; sin Command enum, ID, handler ni método público; IDs spectrum no asignados. **Ruta:** incompleta antes del wire protocol. **Prerrequisitos:** ninguno. **Comando:** ninguno físico válido. **Configuración:** no conectar hardware ni asignar IDs hipotéticos. **Qué hace:** inspección estática únicamente. **Sano:** BLOCKED/NOT IMPLEMENTED en HEAD. **Falla conocida:** scripts externos podrían importar estructuras huérfanas y confundirlas con función. **Aceptación:** sólo cambiar estado si una futura revisión aporta API+handler+respuesta+medición. **Evidencia:** referencias negativas. **Límites:** RSSI por paquete sí existe; scan de espectro no. **Indicadores:** documentación anuncia comando inexistente. **Recuperación:** no aplica.

### EV-52 — MIOTY TS-UNB (P3)

**Objetivo:** conservar el FAIL histórico sin confundir preset con soporte. **Origen:** planned/known failure `KI-19`. **Evidencia:** `presets.py` comentario pending; `demo_mioty_listen.py`; matriz 0/10. **Ruta:** preset FSK 396 baud→backend que no implementa TS-UNB/CPE requerido. **Prerrequisitos:** dos placas/equipo MIOTY y antena 868 sólo si se autoriza repetir. **Comando:** `smoke_ota_txrx.py ... --preset mioty_868_tsunb` en test aislado. **Configuración:** 10 markers; reset. **Qué hace:** caracteriza el límite actual. **Sano esperado:** fallo documentado; un ACK es sólo configuración. **Falla conocida:** 0/10 sin CPE patch custom. **Aceptación:** LIMITACIÓN REPRODUCIDA con 0/10 y control sano; REGRESIÓN NO REPRODUCIDA requiere OTA decodificable, no sólo marker accidental. **Evidencia:** espectro/conteos. **Límites:** no implementar patch. **Indicadores:** tasa real no 396. **Recuperación:** reset.

### EV-53 — RSA, AIS, 802.15.4g y High-PA (P3)

**Objetivo:** registrar roadmap y distinguir ausencias. **Origen:** planned/incomplete `KI-07/KI-21`. **Evidencia:** README/matriz §1/14; `ti_rf_config_min.c` DIO29; sin APIs/handlers RSA/AIS/15.4g. **Ruta:** RSA/AIS/15.4g no tienen ruta; High-PA no se enruta en hardware/selector. **Prerrequisitos:** ninguno para evaluación estática; analizador sólo cuando exista ruta. **Comando:** ninguno válido hoy. **Configuración:** no intentar potencia >0 dBm hasta caracterizar la ruta estándar; no inventar comandos. **Qué hace:** evita falsas pruebas por ACK de configuraciones genéricas. **Sano esperado:** BLOCKED/NOT IMPLEMENTED. **Falla conocida:** la ausencia es de implementación/ruteo, no un fallo de una función prometida como disponible. **Aceptación:** no marcar FAIL funcional; confirmar sólo ausencia/incompletitud. **Evidencia:** enum/handler/API ausentes. **Límites:** std PA útil históricamente -20..+14 dBm, mientras docstring `tx_cw` dice +5: resolver instrumentalmente antes de potencia alta. **Indicadores:** cualquier claim “supported” sin data path. **Recuperación:** no aplica.

### EV-54 — BLE protocol stack retirado (P3)

**Objetivo:** confirmar alcance deliberadamente removido. **Origen:** removed/out-of-scope `KI-23`. **Evidencia:** Architecture/Python API/protocol §9; no IDs 0x40–0x54; quedan structs/funciones internos en `radio_if.c/.h` y SmartRF. **Ruta:** sólo BLE PHY raw es alcanzable; scan activo, initiator, master, GATT no tienen comandos públicos. **Prerrequisitos:** ninguno. **Comando:** EV-21 valida lo que sí queda; Sniffle valida el caso de uso externo, no FeralRF. **Configuración:** tratar BLE como PHY raw; no buscar perfiles/GATT en FeralRF. **Qué hace:** análisis de reachability. **Sano:** raw RX/TX funciona y operaciones de stack no se anuncian. **Falla conocida:** restos internos pueden sugerir erróneamente que las rutas retiradas siguen alcanzables. **Aceptación:** retirada no es bug; registrar candidato de docs si restos internos inducen a error. **Evidencia:** API/enum/handler. **Límites:** `set_adv_hop` es hopping pasivo raw, no active scan. **Indicadores:** herramientas que llamen símbolos retirados. **Recuperación:** no aplica.

## Procedimiento operativo completo por EV

Esta sección es la instrucción de ejecución vigente. Las fichas resumidas anteriores explican el diseño; cuando exista una diferencia, prevalece esta sección. Está verificada contra `FeralRF@0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`, `CatSniffer-Firmware@c0cd5a45e019dbd14ed11d039aacb13f300e5731` y Catnip `fix/CLI_control@126f13bc0441ad3f37fe0b029160a3c526e4d309`. El EXE Catnip instalado mostró `v3.3.3.0`, pero su commit de build no está establecido.

### Convenciones obligatorias para todas las EV

Abra PowerShell en:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python'
$DutBridge  = 'COM88'
$DutShell   = 'COM87'
$PeerBridge = 'COM31'
$PeerShell  = 'COM30'
$CatnipExe  = 'C:\Program Files\Catnip\catnip.exe'
```

No derive ningún Shell del Bridge. Defina en cada terminal que vaya a efectuar recuperaciones:

```powershell
function Reset-Cc1352 {
    param([Parameter(Mandatory)][string]$Shell,
          [Parameter(Mandatory)][string]$Bridge)
    python -c "import serial,time; s=serial.Serial('$Shell',115200,timeout=1,write_timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT boot'); time.sleep(.5); s.write(b'exit\r\n'); s.flush(); print('SENT exit'); time.sleep(.3); s.close(); print('CLOSED')"
    if ($LASTEXITCODE -ne 0) { throw "Reset serial falló en $Shell" }
    Start-Sleep -Seconds 4
    $env:EV_BRIDGE=$Bridge
    @'
import os
from feralrf import Radio
r=Radio(port=os.environ['EV_BRIDGE'])
try:
    print('RECOVERY_INFO:', r.init())
    print('RECOVERY_STATS:', r.get_stats())
finally:
    r.disconnect()
'@ | python -
    if ($LASTEXITCODE -ne 0) { throw "El Bridge $Bridge no recuperó" }
}
```

Para usar el script oficial OTA sin su cálculo inseguro `Bridge+2`, defina esta función. No modifica archivos: importa `examples.smoke_ota_txrx`, sustituye sólo en memoria su función de reset por el mapa explícito y ejecuta su `main()` original.

```powershell
function Invoke-FeralOta {
    param([string]$Tx='COM88',[string]$Rx='COM31',
          [Nullable[int]]$Phy=$null,[int]$Channel=0,[string]$Preset='',
          [int]$Power=0,[int]$Count=10,[int]$MinMarkers=10)
    $env:EV_TX=$Tx; $env:EV_RX=$Rx; $env:EV_PHY="$Phy"; $env:EV_CH="$Channel"
    $env:EV_PRESET=$Preset; $env:EV_POWER="$Power"; $env:EV_COUNT="$Count"
    $env:EV_MIN="$MinMarkers"
    @'
import os,sys,time,serial
from examples import smoke_ota_txrx as app
shell={'COM88':'COM87','COM31':'COM30'}
def safe_reset(bridge):
    if bridge not in shell: raise RuntimeError(f'No explicit Shell mapping for {bridge}')
    with serial.Serial(shell[bridge],115200,timeout=1,write_timeout=1) as s:
        s.write(b'boot\r\n'); s.flush(); time.sleep(.5)
        s.write(b'exit\r\n'); s.flush(); time.sleep(.3)
    time.sleep(3.5)
app.reset_cc1352=safe_reset
argv=['smoke_ota_txrx.py','--tx-port',os.environ['EV_TX'],'--rx-port',os.environ['EV_RX'],
      '--channel',os.environ['EV_CH'],'--power',os.environ['EV_POWER'],
      '--count',os.environ['EV_COUNT'],'--min-markers',os.environ['EV_MIN']]
if os.environ['EV_PRESET']:
    argv += ['--preset',os.environ['EV_PRESET']]
else:
    argv += ['--phy',os.environ['EV_PHY']]
sys.argv=argv
raise SystemExit(app.main())
'@ | python -
    if ($LASTEXITCODE -ne 0) { throw "OTA falló: TX=$Tx RX=$Rx PHY=$Phy preset=$Preset" }
}
```

`Invoke-FeralOta` conserva la semántica del script upstream. No convierte el segundo FeralRF en implementación independiente; sólo da evidencia OTA simétrica. Para cada EV, inicie `Start-Transcript -Path .\EV-XX-AAAAmmdd-HHMM.txt` antes del primer comando y `Stop-Transcript` al final. Preserve siempre comando completo, stdout/stderr completo, roles/COM, fecha, distancia/orientación/antena, firmware/hash conocido, observación física y recuperación. Pare ante puerto ambiguo, error RF asíncrono, timeout no recuperable, calentamiento, emisión fuera del entorno autorizado o discrepancia de identidad.

**Mapa de implementación común.** Salvo que una EV diga lo contrario, el camino es PowerShell → script bajo `FeralRF/python/examples[/lab]` → `FeralRF/python/feralrf/radio.py` → serialización en `commands.py`/`protocol.py` → Cat-Bridge del RP2040 (`CatSniffer-Firmware/RP2040/catsniffer/src/main.c`) → `FeralRF/firmware/cc1352/src/host_if_task.c` → `command_processor.c` → `control_task.c` o `data_task.c` → `radio_if.c` → TI RF driver → antena. El PC/RP2040 sólo transporta el protocolo; el CC1352P7 ejecuta el control y la RF. Catnip (`CatSniffer-Tools/catnip/modules/{device,firmware,sniff}`) se usa para identidad/estado o workflows stock explícitos, no como sustituto de la API FeralRF. Un ACK demuestra aceptación síncrona hasta el handler; cada EV declara aparte si se observó scheduling, intento RF, recepción, framing, timing o medición instrumental.

### EV-00 — Enumeración e identidad de puertos

**Objetivo e implementación.** Confirmar las dos placas y sus tres CDC antes de usar RF. Catnip corre en PC, agrupa puertos en `usb_connection.py` y consulta el RP2040 por Cat-Shell; no prueba FeralRF. **Equipo:** ambas V3, conectadas primero una por una. **Terminal:** una, cualquier directorio.

**Ejecución:** con ambas desconectadas ejecute `Get-Command catnip | Format-List Source,Path,CommandType`; conecte sólo #1 y ejecute `& $CatnipExe devices`, `& $CatnipExe devices --debug`, `& $CatnipExe identify --device 1`, `& $CatnipExe status --device 1`, y `Get-CimInstance Win32_SerialPort | Select-Object DeviceID,Name,PNPDeviceID`. Desconecte #1, conecte sólo #2 y repita; después conecte ambas, vuelva a enumerar y use los IDs recién mostrados, nunca IDs recordados.

**Esperado/evidencia/PASS.** Deben quedar físicamente identificados #1=`COM88/COM86/COM87` y #2=`COM31/COM32/COM30`, con Board V3. Preserve tablas y debug íntegros. `PASS-identity` exige roles inequívocos y LED de la placa elegida; `PARTIAL` si sólo hay tabla; `FAIL/STOP` ante V2/unknown, puerto faltante o LED equivocado. **No prueba:** firmware CC1352 funcional ni RF. **Estado previo/acción:** EV-00 anterior fue `PARTIAL`; requiere suplemento completo, no borrar el mapa anterior.

### EV-01 — Init, GET_INFO y GET_STATS

**Objetivo/base/ruta.** Confirmar PC→COM88→RP2040 passthrough→CC1352P7 `CMD_RADIO_INIT/GET_INFO/GET_STATS`. `Radio.init()` reintenta hasta tres veces; firmware reinicia métricas en `ControlTask_onRadioInit`. **Equipo:** DUT #1. **Terminal:** una en `FeralRF\python`; Catnip cerrado.

**Ejecución:** ejecute tres veces el bloque:

```powershell
1..3 | ForEach-Object {
  $env:EV_I="$_"
  @'
import os,time
from feralrf import Radio
t=time.perf_counter(); r=Radio(port='COM88')
try:
 print('RUN',os.environ['EV_I'],'INFO',r.init())
 print('RUN',os.environ['EV_I'],'STATS',r.get_stats())
 print('RUN',os.environ['EV_I'],'SECONDS',round(time.perf_counter()-t,3))
finally: r.disconnect()
'@ | python -
  if ($LASTEXITCODE -ne 0) { break }
}
```

**Criterios.** `PASS-control`=3/3 INFO coherente y STATS decodificable sin timeout; `FAIL`=fallo repetible tras cerrar ocupantes; `PARTIAL`=menos de tres. Preserve todo. Firmware `1.0.0` y serial `FERALRF1` no identifican el hash del HEX. **No prueba:** RF. **Estado previo/acción:** evidencia fuerte pero formalmente `PARTIAL`; completar esta serie una vez.

### EV-02 — RX IEEE controlado y parada

**Objetivo/base.** Probar configuración y start/stop. `CMD_RX_START` ACK sólo agenda; `DataTask_poll` abre RF después y puede emitir `ERR_RF_INIT_FAILED`. **Equipo:** DUT #1. **Terminal:** una.

```powershell
python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 5
```

**Criterios/evidencia.** `PASS-control`=INFO, SET_PHY/CHANNEL ACK, RX_START, ventana y RX_STOP sin error; paquetes no son obligatorios. `PASS-RF` requiere frames reales y pertenece a EV-05. Preserve stdout y cualquier `RxStreamError`. **No prueba:** stack Zigbee, sensibilidad ni que ACK por sí solo abriera RF. **Fallo/STOP:** error asíncrono, STOP timeout o stream sintético; recuperar por COM87. **Estado previo/acción:** ya `PASS-control`; no repetir salvo regresión o después de recuperación significativa.

### EV-03 — Reconnect limpio

**Objetivo/base.** Verificar `connect/init/disconnect` en cinco procesos independientes. **Equipo:** DUT #1. **Terminal:** una.

```powershell
1..5 | ForEach-Object { Write-Host "CYCLE $_"; python -c "from feralrf import Radio; r=Radio(port='COM88'); print(r.init()); r.disconnect()"; if ($LASTEXITCODE -ne 0) { break } }
```

**Criterios.** `PASS-control`=5/5, sin COM ocupado ni demora creciente manifiesta; cualquier fallo confirmado=`FAIL`. Preserve ciclos completos. **No prueba:** sesión larga, USB unplug/replug o misma instancia. **Estado/acción:** `PASS-control 5/5`; no repetición necesaria.

### EV-04 — Reset y reinicialización

**Objetivo/base.** Validar recuperación y KI-15. `Radio.reset_device()` y varios scripts calculan Shell=`Bridge+2`; ambos mapas actuales lo contradicen. **Equipo:** DUT #1; luego #2 por separado. **Terminal:** una.

**Ejecución segura:** no ejecute `Radio.reset_device()`. Registre primero:

```powershell
python -c "from feralrf import Radio; print('DUT calculated=',Radio(port='COM88')._get_shell_port()); print('PEER calculated=',Radio(port='COM31')._get_shell_port())"
Reset-Cc1352 -Shell COM87 -Bridge COM88
Reset-Cc1352 -Shell COM30 -Bridge COM31
```

**Criterios.** La API permanece `BLOCKED` porque calcula COM90/COM33, no COM87/COM30. La secuencia explícita obtiene `PASS-recovery` por placa si INFO/STATS regresan; repetir 3 veces sólo si se desea cerrar recuperación manual 3/3. **No prueba:** que la API sea segura, watchdog ni nivel eléctrico exacto. **STOP:** nunca abra COM90/COM33 como supuesto Shell. **Estado/acción:** preservar KI-15 y recuperación DUT 1/1; suplementar peer y, opcionalmente, 3/3 manual.

### EV-05 — Primera observación RF IEEE

**Objetivo/base.** Recibir tráfico real de `PROTOCOL-DEVICE-ZIGBEE-CH25`. `RadioIF_processIeee154Packets` entrega CRC/RSSI/LQI/timestamp. El smoke oficial sólo imprime la metadata del primer paquete; el bloque mínimo siguiente usa la misma API y conserva **todos** los frames requeridos. **Equipo:** DUT #1 y fuente Zigbee; #2 como segundo receptor FeralRF simultáneo, no como implementación independiente. **Terminales:** A peer RX, B DUT RX, ambas en `FeralRF\python`.

En A pegue el bloque completo con `COM31` y 35 s:

```powershell
$env:EV_PORT='COM31'; $env:EV_SECONDS='35'
@'
import os
from feralrf import Radio,PHY,RxStreamError
r=Radio(port=os.environ['EV_PORT']); started=False; n=0; good=0
try:
 print('INFO',r.init()); r.set_phy(PHY.IEEE_802_15_4,25); r.start_rx(); started=True
 print('RX_START_ACK',os.environ['EV_PORT'])
 for p in r.read_packets(timeout=float(os.environ['EV_SECONDS'])):
  if isinstance(p,RxStreamError): print('ASYNC_ERROR',p); continue
  n+=1; good+=int(p.crc_ok)
  print('PACKET',n,'ts_us',p.timestamp_us,'ch',p.channel,'rssi_dbm',p.rssi_dbm,
        'lqi',p.lqi,'crc_ok',p.crc_ok,'len',len(p.data),'raw_hex',p.data.hex())
 print('TOTAL',n,'CRC_VALID',good)
finally:
 if started:
  try: r.stop_rx(); print('RX_STOP_ACK')
  except Exception as e: print('RX_STOP_ERROR',repr(e))
 r.disconnect()
'@ | python -
```

Espere `RX_START_ACK COM31`. Dentro de cinco segundos, pegue en B el mismo bloque cambiando sólo la primera línea por `$env:EV_PORT='COM88'; $env:EV_SECONDS='30'`. No transmita durante ambas ventanas y espere `RX_STOP_ACK` en las dos terminales. Repita el par tres veces sólo si se está cerrando la evidencia formal; guarde ambos transcripts con el mismo número de corrida.

**Criterios.** `PASS-RF` DUT=al menos un frame CRC válido reproducible, con bytes variables y plausibles, en las tres ventanas formales; coincidencia temporal de actividad en #2 fortalece atribución, pero dos FeralRF no son independencia de implementación. `INCONCLUSIVE` si ambos ven cero sin confirmar la fuente; `FAIL candidate` si #2 ve tráfico fuerte y DUT no en repeticiones. Preserve cada línea `PACKET`, no sólo totales. **No prueba:** Zigbee stack, asociación, PER/RSSI calibrado. **Estado/acción:** `PASS-RF` ya preservado con 41/43/43 paquetes, pero faltan bytes completos y observación simultánea; este procedimiento es suplemento, no reemplazo del resultado.

### EV-06 — Exclusión RX/TX

**Objetivo/base.** Confirmar que `ControlTask_canStartTx` rechaza TX cuando `s_rx_enabled`; no llega a `RadioIF_transmitRaw`. **Equipo:** DUT #1. **Terminal:** una.

```powershell
@'
from feralrf import Radio,PHY
from feralrf.exceptions import CommandError
r=Radio(port='COM88')
try:
 print('INFO',r.init()); r.set_phy(PHY.IEEE_802_15_4,25); r.start_rx()
 try: r.transmit(b'\x01',power_dbm=-20); raise RuntimeError('TX accepted unexpectedly')
 except CommandError as e: print('EXPECTED_ERROR',e.error_code,hex(e.error_code)); assert e.error_code==5
 r.stop_rx(); print('STATS',r.get_stats())
finally: r.disconnect()
'@ | python -
```

**Criterios.** `PASS-control`=`0x05`, STOP y STATS; aceptación TX/timeout=`FAIL`. **No prueba:** RX físicamente activo ni RF simultánea; prueba el flag lógico. **Estado/acción:** ya PASS; no repetir.

### EV-10 — Matriz de PHY por control

**Objetivo/base.** Recorrer PHY 0–7 con `smoke_phase2.py`; ACK y start/stop, no RF. **Equipo:** DUT #1. **Terminal:** una. Use reset explícito entre filas; el canal 9 de PHY1 pertenece al plan local, mientras el baseline upstream usa 37.

```powershell
$rows=@(@(0,37),@(1,9),@(2,37),@(3,37),@(4,25),@(5,0),@(6,0),@(7,0))
foreach($row in $rows){ python examples\smoke_phase2.py --port COM88 --phy $row[0] --channel $row[1] --power 0; if ($LASTEXITCODE -ne 0) { break }; Reset-Cc1352 -Shell COM87 -Bridge COM88 }
```

**Criterios.** `PASS-control`=8/8; error asíncrono/timeout=`FAIL`. Preserve fila y reset. **No prueba:** frecuencia, potencia, modulación o RX física; PHY7 requiere preset. **Estado/acción:** ya 8/8; no repetir salvo regresión.

### EV-11 — Presets propietarios, sólo control

**Objetivo/base.** Ejecutar `smoke_prop_phase1.py` para 27 presets no OOK/no MIOTY. SET_PROP_CONFIG ACK no confirma backend; RX/TX son diferidos y el script puede no mostrar error asíncrono. **Equipo:** DUT #1; transmisión breve a 0 dBm. **Terminal:** una. Reset explícito entre bandas, no `--auto-reset`.

Para recuperar evidencia individual faltante de 902/915:

```powershell
$presets='gfsk_915_50k','gfsk_902_50k','sidewalk_915_fsk_50k','sidewalk_915_fsk_250k','wisun_915_fsk_50k','wisun_915_fsk_100k','wisun_915_fsk_150k','wisun_915_fsk_200k','wisun_915_fsk_300k'
foreach($p in $presets){ Write-Host "PRESET $p"; python examples\smoke_prop_phase1.py --port COM88 --preset $p --power 0; if ($LASTEXITCODE -ne 0) { break } }
```

**Criterios.** `PASS-control` por preset=script termina y GET_STATS responde; RF sigue `INCONCLUSIVE`. Preserve stdout individual. Cero drop/overflow con cero tráfico no caracteriza colas. **No prueba:** protocolo, OTA, frecuencia ni TX efectivo. **STOP:** no incluir `ook_*`/MIOTY; reset COM87 al cambiar banda. **Estado/acción:** 27/27 reportado, 18 con evidencia individual; repetición parcial sólo de nueve si no se recuperan logs.

### EV-12 — TX raw/frame/burst/continuous y STOP

**Objetivo/base.** Separar aceptación, scheduling y observación OTA. En `examples/lab/ota_tx_burst.py`, `--count 40` llega a `Radio.transmit_burst(packet,count,interval_us)` (`feralrf/radio.py`), que valida `1..65535`; `CommandBuilder.tx_burst()` (`commands.py`) lo serializa como `<HI`. `command_processor.c:CMD_TX_BURST` lo decodifica como `uint16_t`; `ControlTask_onTxBurst()` lo copia a `s_tx_burst_remaining` y devuelve el ACK de scheduling. `ControlTask_processTxBurst()` llama `RadioIF_transmitRaw()` y sólo decrementa después de un retorno exitoso. Si una transmisión falla, pone el restante en cero y cancela silenciosamente el burst: no envía al host el error asíncrono que sí produce la ruta TX_RAW. Por eso 40 significa **máximo solicitado de transmisiones exitosas según el retorno interno**, no 40 ondas demostradas. `interval_us` fija el siguiente instante de inicio desde el poll previo, no garantiza separación on-air exacta. `RadioIF_transmitRaw()` elige la ruta TI IEEE/BLE/propietaria correspondiente. **Equipo:** DUT TX #1, peer RX #2. **Terminales:** A RX primero, B TX.

Para RAW, A: `python examples\lab\ota_rx_probe.py --port COM31 --phy 4 --channel 25 --duration 15 --marker-hex DEADBEEF --min-hits 1`; espere `RX_START`, y B: `python examples\smoke_tx_phase1.py --port COM88 --phy 4 --channel 25 --power 0 --packet-hex DEADBEEF`.

Para FRAME repita A con `--marker-hex A1B2C3D4 --min-hits 10 --duration 15`; B: `python examples\lab\ota_tx_frame.py --port COM88 --phy 4 --channel 25 --power 0 --payload-hex A1B2C3D4 --count 10 --interval-us 100000`.

Para BURST, A con `--marker-hex C0FFEE01 --min-hits 40 --duration 15`; B: `python examples\lab\ota_tx_burst.py --port COM88 --phy 4 --channel 25 --power 0 --payload-hex C0FFEE01 --count 40 --interval-us 25000`.

Para CONTINUOUS, A con `--marker-hex F00DBA5E --min-hits 1 --duration 8`; B: `python examples\smoke_tx_continuous_phase1.py --port COM88 --phy 4 --channel 25 --power 0 --packet-hex F00DBA5E --interval-us 100000 --run-seconds 2`.

Después de las cuatro modalidades, repita cada par con A recibiendo en `COM88` y B transmitiendo por `COM31`, conservando marker, PHY, canal, duración, count e intervalo de la modalidad correspondiente.

**Criterios/evidencia.** `PASS-control` por ACK; `PASS-RF` si el receptor observa marker; burst-count sólo pasa si observa 40; timing/power/frequency requieren instrumento. CONTINUOUS exige que markers cesen después de TX_STOP (una cola residual acotada debe anotarse). **FAIL/STOP:** transmisión persiste, menos de 40 sin interferencia explicable, error/timeout; ejecute STOP y reset explícito. **Estado:** no ejecutado; ejecutar completo.

### EV-13 — CW y PRBS instrumentados

**Objetivo/base.** `CMD_TX_TEST` usa TI `CMD_TX_TEST`: CW=`bUseCw=1`; PRBS15/32=`whitenMode 2/3`. ACK indica que `RF_postCmd` aceptó el comando, no frecuencia/potencia/espectro. `smoke_f22_tx_test.py` es inseguro sin wrapper porque deriva ambos Shell. **Equipo:** DUT #1, peer #2 y, para PASS fuerte, analizador/espectro/carga autorizada. **Terminal:** una más instrumento.

```powershell
$env:EV_TX='COM88';$env:EV_RX='COM31'
@'
import os,sys,time,serial
from examples.lab import smoke_f22_tx_test as app
shell={'COM88':'COM87','COM31':'COM30'}
def reset(p):
 with serial.Serial(shell[p],115200,timeout=1) as s: s.write(b'boot\r\n');time.sleep(.5);s.write(b'exit\r\n')
 time.sleep(3.5)
app.reset=reset;sys.argv=['smoke_f22_tx_test.py','--tx-port',os.environ['EV_TX'],'--rx-port',os.environ['EV_RX']]
raise SystemExit(app.main())
'@ | python -
```

**Criterios.** `PASS-control`=5/5 del script; la caída de BLE ambiental es sólo evidencia RF indirecta y puede ser inconclusa si baseline≤30. `PASS-instrument` requiere centro, espectro y stop observados; potencia solicitada sólo queda validada con medidor calibrado. **STOP:** no irradiar CW/PRBS fuera de banco autorizado; siempre `tx_test_stop`/reset. **Estado:** no ejecutado; equipo independiente aún requerido para cierre.

### EV-14 — Error RF asíncrono

**Objetivo/base.** Verificar que ACK puede ir seguido de `RxStreamError`; `DataTask_poll` emite `ERR_RF_INIT_FAILED` seq 0 si `RadioIF_startRx` falla. No debe inducirse una avería. **Equipo:** DUT #1. **Terminal:** una.

```powershell
@'
from feralrf import Radio,PHY,RxStreamError
r=Radio(port='COM88')
try:
 print('INFO',r.init());r.set_phy(PHY.IEEE_802_15_4,25);r.start_rx();print('RX_START_ACK')
 for x in r.read_packets(timeout=10): print('ASYNC' if isinstance(x,RxStreamError) else 'PACKET',repr(x))
 r.stop_rx();print('STATS',r.get_stats())
finally:r.disconnect()
'@ | python -
```

**Criterios.** Si no hay error y RX/STOP responde, registrar `ASYNC FAILURE NOT TRIGGERED`; eso es un control sano, pero no valida el camino de reporte. Si aparece un `RxStreamError`, `PASS-async-reporting` sólo si código/contexto quedan inequívocamente registrados y el DUT se recupera; no es PASS RF. Stream fijo `8e89be` sería limitación/regresión documental. **No prueba:** manejo de todas las fallas posibles. **Estado:** no ejecutado; ejecutar como observación no destructiva.

### EV-15 — Límites y parámetros inválidos

**Objetivo/base.** Confirmar validación host sin emitir. `random_bytes` admite 1..240; burst count 1..65535; frame vacío se rechaza; payload efectivo firmware TX es 125. **Equipo:** DUT #1 sólo para recovery final. **Terminal:** una.

```powershell
@'
from feralrf import Radio
r=Radio(port='COM88')
cases=[('random0',lambda:r.random_bytes(0)),('random241',lambda:r.random_bytes(241)),
       ('burst0',lambda:r.transmit_burst(b'X',0)),('burst65536',lambda:r.transmit_burst(b'X',65536)),
       ('frame-empty',lambda:r.transmit_frame(b'')),('sha241',lambda:r.sha256(b'X'*241))]
try:
 print('INFO',r.init())
 for name,fn in cases:
  try: fn(); print(name,'UNEXPECTED_ACCEPT')
  except ValueError as e: print(name,'EXPECTED',repr(e))
 print('STATS',r.get_stats())
finally:r.disconnect()
'@ | python -
```

**Criterios.** `PASS-host-boundaries`=todos `ValueError` y STATS; cualquier aceptación=`FAIL` y no continuar con esa entrada. **No prueba:** límites wire/firmware ni emisión a valores válidos; por ello EV-15 queda `PARTIAL` hasta una fase negativa firmware cuidadosamente aislada. **Estado:** no ejecutado.

### EV-20 — IEEE 802.15.4 OTA de dos placas

**Objetivo/base.** Reproducir `smoke_ota_txrx.py` con markers `DEADBEEF`: TX_RAW se agenda en #1 y la ruta RF/CRC/RX de #2 debe entregarlos. **Equipo:** #1 y #2 FeralRF, antenas 2.4 GHz. **Terminal:** una; el script coordina RX antes de TX.

```powershell
Invoke-FeralOta -Tx COM88 -Rx COM31 -Phy 4 -Channel 25 -Power 0 -Count 10 -MinMarkers 10
Invoke-FeralOta -Tx COM31 -Rx COM88 -Phy 4 -Channel 25 -Power 0 -Count 10 -MinMarkers 10
```

**Criterios.** `PASS-RF-symmetric`=10/10 en ambas direcciones; 1–9/10=`PARTIAL`, cero repetible=`FAIL`. Preserve total_rx, markers, roles y geometría. **No prueba:** interoperabilidad independiente, frame IEEE válido, potencia/frecuencia calibradas o Zigbee. El marker puede aparecer dentro de raw PHY. **STOP:** cualquier reset debe mostrar los Shell explícitos del wrapper; no ejecute el script directamente. **Estado:** no ejecutado; ahora desbloqueado por segunda V3.

### EV-21 — BLE raw 1M/2M/Coded OTA

**Objetivo/base.** Validar markers raw entre FeralRF en cuatro PHY; no hay GAP/GATT/stack. `smoke_ota_txrx` usa TX_RAW, no BLE scan. **Equipo:** dos V3 FeralRF, 2.4 GHz. **Terminal:** una.

```powershell
$rows=@(@(0,37),@(1,9),@(2,37),@(3,37))
foreach($r in $rows){ Invoke-FeralOta -Tx COM88 -Rx COM31 -Phy $r[0] -Channel $r[1] -Power 0 -Count 10 -MinMarkers 8 }
foreach($r in $rows){ Invoke-FeralOta -Tx COM31 -Rx COM88 -Phy $r[0] -Channel $r[1] -Power 0 -Count 10 -MinMarkers 8 }
```

**Criterios.** `PASS-RF-symmetric`=≥8/10 cada fila/dirección; guardar ratios. Cualquier fila baja se repite sólo después de documentar orientación/interferencia. **No prueba:** advertising conforme, CRC interoperable, conexiones, GATT ni recepción por Sniffle. Un sniffer BLE independiente sigue siendo preferible. **STOP:** error RF, reset incorrecto o emisión no autorizada. **Estado:** no ejecutado.

### EV-22 — Matriz Sub-1 GHz 868/915 y 4FSK

**Objetivo/base.** OTA simétrica de presets, no stacks. `configure_prop` carga frecuencia/modulación; TX_RAW ACK no basta. **Equipo:** dos V3 FeralRF y antenas adecuadas. **Terminal:** una.

```powershell
$presets='gfsk_868_50k','gfsk_868_100k','msk_868_50k','4fsk_868_50k','4gfsk_868_50k','gfsk_915_50k','gfsk_902_50k'
foreach($p in $presets){ Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset $p -Power 0 -Count 10 -MinMarkers 10 }
foreach($p in $presets){ Invoke-FeralOta -Tx COM31 -Rx COM88 -Preset $p -Power 0 -Count 10 -MinMarkers 10 }
```

**Criterios.** `PASS-RF-symmetric`=10/10 por preset/dirección; `PARTIAL` si sólo una dirección. Preserve antena/banda. **No prueba:** frecuencia, desviación, espectro o protocolo; 4FSK sólo compatibilidad entre implementaciones idénticas. **STOP:** confirme legalidad de 868/902/915 en el banco; no use antena desconocida. **Estado:** no ejecutado.

### EV-23 — Caracterización 433 MHz

**Objetivo/base.** Cuantificar el antecedente marginal, no obtener un PASS aislado. **Equipo:** dos V3, antenas 433 o montaje conducido; instrumento preferible. **Terminal:** una. Para cada preset y dirección ejecute diez veces 10 markers, guardando cada ratio:

```powershell
$presets='gfsk_433_50k','fsk_433_50k','msk_433_50k'
foreach($p in $presets){1..10|%{Write-Host "$p DUT->PEER run $_";Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset $p -Power 0 -Count 10 -MinMarkers 1}}
foreach($p in $presets){1..10|%{Write-Host "$p PEER->DUT run $_";Invoke-FeralOta -Tx COM31 -Rx COM88 -Preset $p -Power 0 -Count 10 -MinMarkers 1}}
```

**Criterios.** Reporte caracterización=éxitos/100 por preset/dirección, no simple PASS. ≥95/100 estable puede etiquetarse `PASS-RF` bajo esa geometría; menor=`MARGINAL`; 0 con instrumento confirmando emisión=`FAIL`; sin antena/instrumento=`INCONCLUSIVE`. **No prueba:** causa firmware frente a matching/antena. **STOP:** OOK pertenece a EV-24. **Estado:** no ejecutado; equipo 433 sigue siendo prerrequisito de conclusión fuerte.

### EV-24 — OOK 868/433, lock y recovery

**Objetivo/base.** Confirmar OTA OOK y recuperación. `RadioIF_setPropConfig` carga patches genook no descargables; OOK debe ser lo último antes de reset. **Equipo:** dos V3, antenas/banco correctos; analizador preferible. **Terminal:** una.

```powershell
Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset ook_868_4k8 -Power 0 -Count 10 -MinMarkers 10
Reset-Cc1352 -Shell COM87 -Bridge COM88; Reset-Cc1352 -Shell COM30 -Bridge COM31
Invoke-FeralOta -Tx COM31 -Rx COM88 -Preset ook_868_4k8 -Power 0 -Count 10 -MinMarkers 10
Reset-Cc1352 -Shell COM87 -Bridge COM88; Reset-Cc1352 -Shell COM30 -Bridge COM31
```

Ejecute OOK433 sólo con antena/equipo 433, sustituyendo preset por `ook_433_4k8`, y recupere ambas placas inmediatamente. **Criterios.** `PASS-RF` por markers; `LIMITATION REPRODUCED` si cambiar de modo sin reset bloquea exactamente y el reset recupera; `FAIL` si reset explícito no recupera. **No prueba:** decodificación OOK de terceros ni sensibilidad. **STOP:** no use `--auto-reset`, `demo_emulate_ook_garage.py` ni scripts sin wrapper: llaman reset inseguro. **Estado:** no ejecutado.

### EV-25 — Wireless M-Bus S/T/C/N

**Objetivo/base.** Distinguir OTA de markers con presets de interoperabilidad W-MBus. **Equipo:** dos V3; dispositivo/decoder W-MBus para conclusión de protocolo; antena 169 para N. **Terminal:** una.

```powershell
$presets='wireless_mbus_s_868','wireless_mbus_t_868','wireless_mbus_c_868'
foreach($p in $presets){Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset $p -Count 10 -MinMarkers 10}
foreach($p in $presets){Invoke-FeralOta -Tx COM31 -Rx COM88 -Preset $p -Count 10 -MinMarkers 10}
# Sólo con antenas/equipo 169 MHz:
Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset wireless_mbus_n_169_2k4 -Count 10 -MinMarkers 10
Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset wireless_mbus_n_169_4k8 -Count 10 -MinMarkers 10
Invoke-FeralOta -Tx COM31 -Rx COM88 -Preset wireless_mbus_n_169_2k4 -Count 10 -MinMarkers 10
Invoke-FeralOta -Tx COM31 -Rx COM88 -Preset wireless_mbus_n_169_4k8 -Count 10 -MinMarkers 10
```

**Criterios.** `PASS-RF-PHY`=10/10 markers; `PASS-protocol` sólo si un dispositivo/decoder W-MBus independiente acepta frames conformes, algo que estos comandos no generan. Role-swap requerido para simetría. **No prueba:** framing, CRC, cifrado o modos W-MBus completos. **STOP:** N queda `BLOCKED` sin antena/banco 169. **Estado:** EV-11 sólo control; OTA pendiente.

### EV-26 — Wi-SUN y Sidewalk FSK

**Objetivo/base.** Reproducir siete presets OTA de `smoke_f29_subg_915.py`; MIOTY está excluido por el propio script. **Equipo:** dos V3, antenas 902/915; dispositivo independiente para protocolo. **Terminal:** una. Use el script original con reset parcheado sólo en memoria:

```powershell
$env:EV_TX='COM88';$env:EV_RX='COM31'
@'
import os,sys,time,serial
import examples.smoke_f29_subg_915 as app
shell={'COM88':'COM87','COM31':'COM30'}
def reset(p):
 with serial.Serial(shell[p],115200,timeout=1,write_timeout=1) as s:s.write(b'boot\r\n');time.sleep(.5);s.write(b'exit\r\n');s.flush()
 time.sleep(3.5)
app.reset_cc1352=reset
sys.argv=['smoke_f29_subg_915.py','--tx-port',os.environ['EV_TX'],'--rx-port',os.environ['EV_RX'],'--count','10','--min-markers','10','--power','0']
raise SystemExit(app.main())
'@ | python -
```

**Criterios.** `PASS-RF-PHY`=70/70. Para la dirección inversa, vuelva a pegar el bloque completo sustituyendo exactamente la primera línea por `$env:EV_TX='COM31';$env:EV_RX='COM88'`; no cambie el mapa `shell`. **No prueba:** Wi-SUN FAN, Sidewalk networking/LR, interoperabilidad o espectro. `PASS-protocol` exige nodos terceros. **STOP:** sólo banda autorizada; no confundir nombres de preset con stack. **Estado:** no ejecutado actual; antecedente histórico 70/70.

### EV-27 — Propietario 2.4 GHz

**Objetivo/base.** OTA de `gfsk_2440_50k` y `_250k`. **Equipo:** dos V3; SDR/analizador 2.4 GHz para independencia. **Terminal:** una.

```powershell
foreach($p in 'gfsk_2440_50k','gfsk_2440_250k'){Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset $p -Count 10 -MinMarkers 10}
foreach($p in 'gfsk_2440_50k','gfsk_2440_250k'){Invoke-FeralOta -Tx COM31 -Rx COM88 -Preset $p -Count 10 -MinMarkers 10}
```

**Criterios.** 10/10 ambas direcciones=`PASS-RF-symmetric`; instrumento confirmando 2440 MHz/modulación=`PASS-instrument`. **No prueba:** interoperabilidad propietaria externa ni exactitud de potencia. **STOP:** U2/CTF o antena no establecidos hacen un cero `INCONCLUSIVE`, no fallo inmediato. **Estado:** no ejecutado.

### EV-28 — Helpers de emulación PHY-level

**Objetivo/base.** Verificar firmas de helpers, no emulación de stacks. `smoke_f17_emulation.py` deriva Shell; use wrapper. **Equipo:** dos V3. **Terminal:** una.

```powershell
$env:EV_TX='COM88';$env:EV_RX='COM31'
@'
import os,sys,time,serial
import examples.smoke_f17_emulation as app
shell={'COM88':'COM87','COM31':'COM30'}
def reset(p):
 with serial.Serial(shell[p],115200,timeout=1,write_timeout=1) as s:s.write(b'boot\r\n');time.sleep(.5);s.write(b'exit\r\n');s.flush()
 time.sleep(3.5)
app.reset_cc1352=reset
sys.argv=['smoke_f17_emulation.py','--tx-port',os.environ['EV_TX'],'--rx-port',os.environ['EV_RX'],'--count','20','--skip-ook','--skip-433']
raise SystemExit(app.main())
'@ | python -
```

**Criterios.** Todos los helpers incluidos superan su threshold=`PASS-signature`. Para invertir roles, vuelva a pegar el bloque completo sustituyendo exactamente la primera línea por `$env:EV_TX='COM31';$env:EV_RX='COM88'`; conserve el mapa `shell`. 433/OOK permanecen en EV-23/24 y no se habilitan aquí inicialmente. **No prueba:** dispositivo/protocolo real, autenticación o framing interoperable. **STOP:** no quite `--skip-ook` mientras reset no esté controlado por wrapper y EV-24 cerrado. **Estado:** no ejecutado.

### EV-29 — KillerBee real: sniff e inject

**Objetivo/base.** Probar adapter→Radio→IEEE y PCAP. El baseline host registró KillerBee opcional ausente; no instalar dependencias durante la sesión sin decisión separada. **Equipo:** DUT, Zigbee CH25, Wireshark/KillerBee. **Terminal:** una.

Preflight: `python -c "import killerbee; print(killerbee.__file__)"`. Si falla, EV-29=`BLOCKED` y se detiene. Si existe:

```powershell
python examples\killerbee_sniff.py --port COM88 --channel 25 --count 20 --pcap .\EV-29-ch25.pcap
Get-FileHash .\EV-29-ch25.pcap -Algorithm SHA256
```

**Criterios.** `PASS-sniff`=20 frames variables, FCS/validcrc coherente, PCAP abre como DLT 195; `PASS-inject` requiere además flujo KillerBee externo y segundo observador, no cubierto por este comando. **No prueba:** stack Zigbee en FeralRF. **STOP:** no flashear stock/sniffer ni ejecutar Catnip sniff sobre DUT. **Estado:** `BLOCKED` por dependencia ausente hasta verificar/corregir el entorno; EV posteriores no RF-integración pueden continuar.

### EV-30 — Vectores crypto en hardware

**Objetivo/base.** Comparar aceleradores CC1352 con vectores/oráculo host usando el script original F25. **Equipo:** DUT #1 y paquete `cryptography`. **Terminal:** una.

```powershell
python -c "import cryptography; print(cryptography.__version__)"
python examples\lab\smoke_f25_crypto.py --port COM88
```

**Criterios.** `PASS-crypto`=9/9; skip Curve25519 sólo puede dar `PARTIAL`, no PASS. Preserve bytes/comparaciones y versiones. **No prueba:** side channels, generación certificada o seguridad de protocolo. **STOP:** mismatch de vector; no continuar stress. **Estado:** no ejecutado.

### EV-31 — Crypto stress y límites

**Objetivo/base.** Repetición y límites después de EV-30. No hay script upstream completo; use F25 veinte veces, que conserva oráculos. **Equipo:** DUT. **Terminal:** una.

```powershell
1..20 | % { Write-Host "CRYPTO RUN $_"; python examples\lab\smoke_f25_crypto.py --port COM88; if ($LASTEXITCODE -ne 0) { break } }
```

**Criterios.** `PASS-repeat`=20×9/9 sin degradación; guardar duración por corrida si se desea latencia. Límites locales 0/241 SHA/TRNG ya pertenecen a EV-15. **No prueba:** throughput sostenido, concurrencia o side-channel. **STOP:** primer mismatch/timeout. **Estado:** no ejecutado.

### EV-40 — Baseline completo actual

**Objetivo/base.** Reproducir `run_validation_baseline.sh`. El script deriva Shell=`Bridge+2` dentro de Bash/Python y no acepta Shell explícito. **Equipo:** dos V3 y Git Bash.

**Ejecución:** **BLOCKED: no ejecutar el script sin cambios** con COM88/COM31. Tampoco generar una copia ad hoc sin revisión, porque EV-40 busca reproducibilidad del baseline oficial. EV-10–28 pueden ejecutarse individualmente con los wrappers seguros anteriores.

**Criterios.** Se desbloquea sólo cuando exista una versión revisada que acepte `--tx-shell COM87 --rx-shell COM30` o un mecanismo equivalente auditable; entonces se preservará log completo y hash del script. **No prueba aun ejecutado:** independencia, instrumentación ni protocolos. **Estado:** no ejecutado/BLOCKED por KI-15; posteriores independientes pueden continuar.

### EV-41 — Cambio PHY sin reset

**Objetivo/base.** Determinar si el deadlock histórico persiste en HEAD. **Equipo:** DUT; reset explícito preparado. **Terminal:** una.

```powershell
@'
from feralrf import Radio,PHY
rows=[(PHY.BLE_1M,37),(PHY.IEEE_802_15_4,25),(PHY.SUB_1GHZ_868,0),(PHY.BLE_1M,37)]
r=Radio(port='COM88')
try:
 print('INFO',r.init())
 for cycle in range(1,4):
  for phy,ch in rows:
   print('STEP',cycle,phy.name,ch);r.set_phy(phy,ch);r.set_channel(ch);r.start_rx();r.stop_rx();print('STATS',r.get_stats())
finally:r.disconnect()
'@ | python -
```

**Criterios.** 3 ciclos completos=`REGRESSION NOT REPRODUCED` (no demuestra ausencia); timeout en transición repetible=`LIMITATION REPRODUCED`; incapacidad de recuperar=`FAIL`. Preserve último ACK/paso. **No prueba:** RF. **STOP:** al primer hang, no reintente antes de guardar evidencia y `Reset-Cc1352 COM87 COM88`. **Estado:** no ejecutado.

### EV-42 — Cambios con reset y entre bandas

**Objetivo/base.** Validar workaround explícito diez ciclos. **Equipo:** DUT. **Terminal:** una.

```powershell
$rows=@(@(0,37),@(4,25),@(7,0))
1..10 | % { $cycle=$_; foreach($r in $rows) { Write-Host "CYCLE $cycle PHY $($r[0])"; python examples\smoke_phase2.py --port COM88 --phy $r[0] --channel $r[1] --power 0; if ($LASTEXITCODE -ne 0) { throw 'step failed' }; Reset-Cc1352 -Shell COM87 -Bridge COM88 } }
```

PHY7 aquí sólo prueba control default; GFSK433↔868 requiere `smoke_prop_phase1` y reset explícito entre ambos, sin OOK. **Criterios.** 10/10=`PASS-workaround`; fallo pese a reset=`FAIL`. **No prueba:** necesidad del reset ni RF. **Estado:** EV-10 aportó menos ciclos; requiere ejecución dedicada.

### EV-43 — RX soak y contadores

**Objetivo/base.** Soak real CH25 y monotonicidad. `canary_regression.py` lee paquetes/estadísticas y STOP final. **Equipo:** DUT + Zigbee CH25. **Terminal:** una.

```powershell
python examples\lab\canary_regression.py --port COM88 --phy 4 --channel 25 --power 0 --soak-duration 60 --report-every 15 --profile quiet --min-packets 1
python examples\lab\canary_regression.py --port COM88 --phy 4 --channel 25 --power 0 --soak-duration 300 --report-every 30 --profile quiet --min-packets 1
```

**Criterios.** `PASS-soak` por etapa=stats monotónicas, ≥1 frame, STOP/final stats; luego reconecte con EV-01 una vez. Drops/overflow se reportan, no se exige cero sin tasa controlada. **No prueba:** capacidad de cola o PER. **STOP:** no iniciar 5 min si 60 s falla. **Estado:** no ejecutado.

### EV-44 — Presión de cola/burst

**Objetivo/base.** Relacionar 40 solicitados con observados y counters. En firmware count es máximo de intentos exitosos; al primer `RadioIF_transmitRaw` fallido el burst se cancela sin error asíncrono al host. **Equipo:** #1 TX, #2 RX. **Terminales:** A RX primero, B TX.

1. A: `python examples\lab\ota_rx_probe.py --port COM31 --phy 4 --channel 25 --duration 15 --marker-hex CAFE4401 --min-hits 40 --print-limit 40`.
2. Espere `RX_START`; B: `python examples\lab\ota_tx_burst.py --port COM88 --phy 4 --channel 25 --power 0 --payload-hex CAFE4401 --count 40 --interval-us 25000`.
3. Dirección inversa: A=`python examples\lab\ota_rx_probe.py --port COM88 --phy 4 --channel 25 --duration 15 --marker-hex CAFE4401 --min-hits 40 --print-limit 40`; espere `RX_START`; B=`python examples\lab\ota_tx_burst.py --port COM31 --phy 4 --channel 25 --power 0 --payload-hex CAFE4401 --count 40 --interval-us 25000`.
4. Reduzca el intervalo sólo en una progresión posterior documentada.

**Criterios.** 40 observados=`PASS-RF-count` para esa tasa; menos=`PARTIAL/pressure observed`, no culpar RX sin tercer observador. Timing exacto requiere instrumento. **No prueba:** capacidad máxima con una sola tasa. **STOP:** overflow creciente, STOP/reconnect fallido. **Estado:** no ejecutado.

### EV-45 — Wrap de secuencia

**Objetivo/base.** `_next_seq` salta 0xFF; probar >253 comandos reales. **Equipo:** DUT. **Terminal:** una.

```powershell
@'
from feralrf import Radio
r=Radio(port='COM88')
try:
 print('INFO',r.init())
 for i in range(300):
  s=r.get_stats()
  if i%25==0: print('SEQ_RUN',i,'STATS',s)
 print('PASS 300/300')
finally:r.disconnect()
'@ | python -
```

**Criterios.** `PASS-control`=300/300 en una conexión; timeout/mismatch=`FAIL`. **No prueba:** TX wrap histórico ni RF, pero sí secuencia de protocolo actual. **STOP:** primer fallo, preservar índice. **Estado:** no ejecutado.

### EV-46 — Re-init y ciclo de vida

**Objetivo/base.** Repetir RADIO_INIT en una instancia; cada init detiene RX y reinicia stats. **Equipo:** DUT. **Terminal:** una.

```powershell
@'
from feralrf import Radio
r=Radio(port='COM88')
try:
 for i in range(1,21): print('REINIT',i,r.init(),r.get_stats())
finally:r.disconnect()
'@ | python -
```

**Criterios.** 20/20=`PASS-lifecycle`; preservar iteración/tiempo. **No prueba:** reconnect USB ni conservación de métricas (se reinician por diseño). **STOP:** timeout o estado que no recupera. **Estado:** no ejecutado; EV-03 no lo sustituye.

### EV-47 — Recuperación tras interrupción/error

**Objetivo/base.** Confirmar cleanup tras Ctrl-C, no provocar corrupción. **Equipo:** DUT + Zigbee. **Terminales:** A prueba, B recuperación.

1. A: `python examples\lab\canary_regression.py --port COM88 --phy 4 --channel 25 --soak-duration 300 --report-every 15 --profile quiet --min-packets 1`.
2. Tras el primer `[RPT]`, pulse Ctrl-C una vez y espere retorno al prompt.
3. B, sólo después: `python -c "from feralrf import Radio;r=Radio(port='COM88');print(r.init());print(r.get_stats());r.disconnect()"`.
4. Si B falla, preserve salida y ejecute reset explícito; repita EV-01, no el soak.

**Criterios.** `PASS-recovery`=cleanup y nueva INIT/STATS sin reset; `PARTIAL` si requiere reset; `FAIL` si reset no recupera. **No prueba:** power loss/USB disconnect. **Estado:** no ejecutado.

### EV-50 — Jamming continuo

**Objetivo/base.** Medir interferencia real, no ACK. `smoke_jam_phase1.py` limita 1..30000 ms y firmware tiene timeout; ACK no prueba PER. **Equipo:** recinto RF o conexión conducida, víctima independiente e instrumento. Las dos V3 al aire no satisfacen por sí solas seguridad/autorización.

**Ejecución:** `BLOCKED` hasta documentar recinto/conexión, carga/atenuación y autorización. Comando reservado, no ejecutar en espacio abierto: `python examples\lab\smoke_jam_phase1.py --port COM88 --phy 4 --channel 25 --power 0 --duration-ms 1000 --wait-ms 500`.

**Criterios.** `PASS-control` sólo ACK/STOP; `PASS-instrument` exige energía confinada, cese tras STOP y cambio de PER de víctima con baseline. **STOP:** cualquier fuga/emisión no autorizada o STOP dudoso; cortar alimentación. **No prueba:** eficacia general o legalidad. **Estado:** BLOCKED.

### EV-51 — Spectrum/RSSI scan

**Objetivo/base.** Validar scan cuando exista data path. En HEAD hay dataclasses/ideas, pero no comando público completo ni handler E2E. **Ejecución:** ninguna; `BLOCKED`. **PASS futuro:** API+wire+handler+datos comparados con instrumento. **No prueba actualmente:** nada físico. EV-52/53 no dependen de inventar esta ruta.

### EV-52 — MIOTY TS-UNB

**Objetivo/base.** Reproducir limitación del preset 396 baud, no declarar soporte. **Equipo:** dos V3 y, para conclusión física, analizador/dispositivo MIOTY. **Terminal:** una.

```powershell
Invoke-FeralOta -Tx COM88 -Rx COM31 -Preset mioty_868_tsunb -Power 0 -Count 10 -MinMarkers 1
Reset-Cc1352 -Shell COM87 -Bridge COM88; Reset-Cc1352 -Shell COM30 -Bridge COM31
```

**Criterios.** 0/10 con ambos controles sanos=`LIMITATION REPRODUCED`, no PASS; marker recibido sólo es `REGRESSION NOT REPRODUCED` hasta que instrumento/protocolo confirme TS-UNB. **No prueba:** MIOTY interoperable. **STOP:** no iterar tras lock/error; recuperar. **Estado:** no ejecutado actual, FAIL histórico 0/10.

### EV-53 — RSA, AIS, 802.15.4g y High-PA

**Objetivo/base.** Conservar límites pendientes. No existen rutas públicas completas ni banco High-PA caracterizado para estas capacidades. **Ejecución:** ninguna; `BLOCKED`. No use `test_pa_characterization.py` como sustituto: caracteriza configuraciones externas y no implementa RSA/AIS/15.4g. **PASS futuro:** implementación y oráculo/instrumento por capacidad. **Estado:** BLOCKED; no impide EV ya implementadas.

### EV-54 — BLE protocol stack retirado

**Objetivo/base.** Confirmar alcance: FeralRF conserva PHY raw y retiró scan/conexión/GATT el 2026-07-20; Sniffle es herramienta separada. **Ejecución:** revisión estática, no test FeralRF. `rg -n "BLE protocol|GATT|scan mode|removed" ..\docs\VALIDATION_MATRIX.md ..\README.md`. **Criterio:** `NOT APPLICABLE` a FeralRF actual; usar AUX stock/Sniffle sería otra campaña y puede implicar flash. **No prueba:** BLE stack. **Estado:** N/A; no convertir en FAIL.

## Resumen operativo y acción requerida

| EV | Propósito | DUT | Observador/equipo | Script/herramienta exacta | Evidencia alcanzable | Estado previo | Acción |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 00 | identidad/COM | #1/#2 | Catnip/Windows | `catnip devices --debug/identify/status` | identidad | PARTIAL | suplemento |
| 01 | init/info/stats | #1 | — | bloque Python 3× | control | PARTIAL | completar |
| 02 | RX start/stop | #1 | — | `smoke_phy4_ieee154.py` | control | PASS-control | no repetir |
| 03 | reconnect | #1 | — | loop 5× | control | PASS 5/5 | no repetir |
| 04 | reset | #1/#2 | Shell explícito | `Reset-Cc1352` | recovery | API BLOCKED; manual 1/1 | suplementar |
| 05 | RX IEEE real | #1 | Zigbee+#2 | bloque RX exhaustivo (`Radio.read_packets`) | RF RX + bytes/metadata | PASS-RF | suplemento simultáneo opcional |
| 06 | exclusión RX/TX | #1 | — | bloque Python | control/estado | PASS | no repetir |
| 10 | PHY 0–7 | #1 | — | `smoke_phase2.py` | control | 8/8 | no repetir |
| 11 | 27 presets | #1 | — | `smoke_prop_phase1.py` | control | 27/27 reportado | 9 logs si faltan |
| 12 | TX modos | #1 | #2 | `ota_rx_probe` + `smoke/ota_tx_*` | control+RF simétrica | NOT TESTED | ejecutar |
| 13 | CW/PRBS | #1 | #2+instrumento | F22 wrapper | control/indirecta/instrumento | NOT TESTED | instrumento |
| 14 | error async | #1 | — | bloque Python | observación | NOT TESTED | ejecutar |
| 15 | límites | #1 | — | bloque Python | host-boundaries | NOT TESTED | ejecutar parcial |
| 20 | IEEE OTA | #1↔#2 | peer | `Invoke-FeralOta` | RF simétrica | NOT TESTED | ejecutar |
| 21 | BLE raw OTA | #1↔#2 | peer/sniffer | `Invoke-FeralOta` | RF simétrica | NOT TESTED | ejecutar |
| 22 | Sub-G matrix | #1↔#2 | peer/instrumento | `Invoke-FeralOta` | RF simétrica | NOT TESTED | ejecutar |
| 23 | 433 | #1↔#2 | antena/instrumento | `Invoke-FeralOta` 100 ensayos | caracterización | NOT TESTED | condicionado |
| 24 | OOK | #1↔#2 | instrumento | `Invoke-FeralOta`+reset | RF/recovery | NOT TESTED | último |
| 25 | W-MBus | #1↔#2 | dispositivo tercero | `Invoke-FeralOta` | PHY; protocolo bloqueado | NOT TESTED | ejecutar PHY |
| 26 | Wi-SUN/Sidewalk | #1↔#2 | nodo tercero | F29 wrapper | PHY; protocolo bloqueado | NOT TESTED | ejecutar PHY |
| 27 | prop 2.4 | #1↔#2 | SDR | `Invoke-FeralOta` | RF simétrica | NOT TESTED | ejecutar |
| 28 | helpers | #1↔#2 | receptor conforme | F17 wrapper | firma | NOT TESTED | ejecutar subset |
| 29 | KillerBee | #1 | Zigbee/host | `killerbee_sniff.py` | PCAP | BLOCKED dep. | verificar dependencia |
| 30 | crypto | #1 | oracle host | F25 | vectores HW | NOT TESTED | ejecutar |
| 31 | crypto repeat | #1 | oracle host | F25×20 | repetición | NOT TESTED | tras EV30 |
| 40 | baseline | #1↔#2 | Git Bash | baseline oficial | completa si corregido | NOT TESTED | BLOCKED KI-15 |
| 41 | switch sin reset | #1 | — | bloque Python | reproducción límite | NOT TESTED | ejecutar cauteloso |
| 42 | switch con reset | #1 | Shell COM87 | smoke+reset | workaround | NOT TESTED | ejecutar |
| 43 | RX soak | #1 | Zigbee | canary 60/300 | soak RF | NOT TESTED | ejecutar escalonado |
| 44 | cola/burst | #1↔#2 | peer; tercero ideal | ota probe/burst | count RF | NOT TESTED | ejecutar |
| 45 | seq wrap | #1 | — | 300 STATS | protocolo | NOT TESTED | ejecutar |
| 46 | re-init | #1 | — | 20 init | lifecycle | NOT TESTED | ejecutar |
| 47 | interrupción | #1 | — | canary+Ctrl-C | recovery | NOT TESTED | ejecutar |
| 50 | jamming | #1 | víctima+instrumento+recinto | jam smoke | instrumento | BLOCKED | no ejecutar abierto |
| 51 | scan | — | instrumento futuro | ninguno | ninguna | BLOCKED | esperar implementación |
| 52 | MIOTY | #1↔#2 | MIOTY/instrumento | `Invoke-FeralOta` | limitación/PHY | histórico FAIL | opcional controlado |
| 53 | pendientes | — | futuro | ninguno | ninguna | BLOCKED | esperar implementación |
| 54 | BLE stack | — | Sniffle externo | revisión estática | alcance | N/A | no ejecutar FeralRF |

## Open validation gaps

- No hay trazabilidad hash del HEX que ejecutan ambas V3 al commit fuente.
- Dos FeralRF dan compatibilidad simétrica, no interoperabilidad independiente ni detección de defectos compartidos.
- Faltan analizador/SDR calibrado, medidor de potencia y montajes/antenas conocidos para frecuencia, espectro, potencia, 169/433 MHz, CW/PRBS y jamming.
- Faltan dispositivos conformes W-MBus, Wi-SUN, Sidewalk y MIOTY; markers raw no sustituyen esos protocolos.
- KillerBee sigue bloqueado hasta confirmar dependencia/host; Catnip instalado no tiene provenance de commit demostrada.
- `run_validation_baseline.sh`, F17, F22, F29 y F9 contienen reset `Bridge+2`; sólo los wrappers en memoria documentados son seguros. EV-40 permanece bloqueado porque su objetivo exige el baseline completo oficial.
- TX_BURST cancela silenciosamente al primer fallo de `RadioIF_transmitRaw`; sin observador no se conoce el número transmitido. Ningún ACK valida timing/potencia/frecuencia.
- El control externo U2/CTF y la discrepancia GPIO de reset requieren reconciliación antes de atribuir fallos multibanda al CC1352/FeralRF.

## Inventario de problemas, limitaciones y trabajo pendiente

| ID | Problema / limitación | Tipo | Fuente (ruta / función o sección) | Capa | Histórico o actual | Evidencia en código actual | Workaround | Test | Estado físico | Notas |
|---|---|---|---|---|---|---|---|---|---|---|
| KI-01 | OOK carga patches que no pueden descargarse y bloquea cambios | limitación confirmada | `radio_if.c:RadioIF_getPropMode/setPropConfig`; API §4 | TI RF/CC1352 | actual | guard `s_prop_ook_active`, reconfig silenciosa | reset/power-cycle | EV-24 | no ejecutado | hacer OOK último |
| KI-02 | Switch PHY sin reset falló por deadlock en segundo ciclo | fallo histórico | matriz §10; reglas RF | TI RF | histórico, alcanzable | lifecycle evita `RF_close`, pero hay excepciones | reset entre PHY | EV-41 | no ejecutado | no asumir corregido |
| KI-03 | Cambio 433↔868 puede dejar sintetizador inconsistente | limitación/workaround | API `reset_device` | TI RF | actual documentado | paths/handles separados alrededor de 861 MHz | reset entre bandas | EV-42 | no ejecutado | delimitar necesidad |
| KI-04 | 433 GFSK/FSK/MSK marginal | marginal RF | matriz §§2,5 | RF hardware/front-end | histórico | presets/ruta existen | antena 433, roles, conducido | EV-23 | no ejecutado | medir ratio, no PASS aislado |
| KI-05 | OOK433 0/10; sensibilidad/antena | fallo/limitación HW | matriz §5 | RF hardware | histórico | OOK preset activo | antena/instrumento; reset | EV-23/24 | no ejecutado | baseline lo omite OTA |
| KI-06 | Device #2 TX débil a 868 | issue de unidad | matriz §13 | RF hardware | histórico | no determinable por código | usar #1 TX e invertir roles | EV-22 | no ejecutado | serializar placas |
| KI-07 | High-PA DIO29 no enrutado; rango útil/documentado inconsistente | limitación | matriz §§1,13; `ti_rf_config_min.c`; `tx_cw` docstring | RF HW/FW/docs | actual | sin ruta High-PA | std PA, potencia baja | EV-13/53 | no ejecutado | +5 vs +14 requiere medida |
| KI-08 | Prop 2.4 dice pending en README pero matriz dice 10/10 | contradicción | README Protocols vs matriz §§2,5 | docs/validación | actual | preset y backend existen | revalidar OTA | EV-27 | no ejecutado | no decidir por prosa |
| KI-09 | BLE2M requiere bt5 patch/extended ADV; workarounds anti-hang | workaround | `radio_if.c` TX BLE; `smartrf_ble5_0.c` | TI RF | actual | comentarios y código especial | reset; probar segundo TX | EV-21 | no ejecutado | ch37→ch9 |
| KI-10 | `RF_runCmd(FS)` puede colgar, especialmente loDivider 0x0A | workaround | Architecture §5; `radio_if.c` | TI RF | actual | usa `RF_postCmd` | regresión por switching | EV-41/46 | no ejecutado | no invocar driver directo |
| KI-11 | `RF_close` puede deadlock; se evita salvo rutas guardadas | workaround/hazard | Architecture §5; `RadioIF_init` | TI RF | actual | close todavía en re-init | tests lifecycle+reset | EV-41/46 | no ejecutado | ruta de alto riesgo |
| KI-12 | Docs describen stream sintético 8E89BE; código actual reporta error RF asíncrono | discrepancia/histórico | Linux troubleshooting vs `data_task.c:120` | FW/docs | incierto | backend SYNTH existe, start failure emite error | no aceptar patrón como RF | EV-14 | no ejecutado | establecer conducta HEAD |
| KI-13 | Colas estáticas pueden descartar; output depth 32, sin backpressure | limitación | Architecture memory; `packet_queue.c`; `data_task.c` | CC1352/UART | actual | drop counters y 8 paquetes/poll | controlar tasa | EV-43/44 | no ejecutado | RX buffer 16 KiB |
| KI-14 | ACK de TX no demuestra finalización/emisión RF | limitación de observabilidad | API TX; control task | protocolo/FW | actual | respuesta precede/ no incluye TX_DONE | observador independiente | EV-12/20 | no ejecutado | separar control/OTA |
| KI-15 | Reset infiere Shell=`Bridge+2`; doc KillerBee dice que puede romper init en stock bridge | fragilidad/contradicción | `Radio._get_shell_port/reset_device`; `killerbee-catsniffer.md` | Python/RP2040 | actual/incierto | aritmética COM literal | confirmar puerto; power-cycle | EV-00/04 | no ejecutado | no automatizar a ciegas |
| KI-16 | Spectrum/RSSI scan son dataclasses/builders sin data path | incompleto | `_spectrum.py`, `_responses.py`, `commands.py`; API §3 | Python/protocolo/FW | actual | sin enum, ID asignado, handler o método | ninguno | EV-51 | bloqueado | RSSI por paquete sí existe |
| KI-17 | Jamming sólo ACK/control; interferencia no validada | experimental | API §2; matriz | RF/FW | actual | handlers/sesión existen | recinto+medición+reset | EV-50 | no ejecutado | legal/lab only |
| KI-18 | Jamming reactivo/pattern sólo IDs reservados | planned | `PENDING_COMMAND_IDS` | Python/protocolo | actual | no handlers/API | ninguno | EV-50/51 | bloqueado | no confundir con continuous |
| KI-19 | MIOTY TS-UNB 396 no soportado sin CPE patch | fallo conocido/planned | `presets.py`; matriz §9 | TI RF | actual | preset existe, data path insuficiente | patch futuro, no aquí | EV-52 | no ejecutado | 0/10 histórico |
| KI-20 | W-MBus N 169 nunca OTA-validado | missing validation | matriz §§5,14 | RF HW/PHY | actual | presets existen | equipo/antena 169 | EV-25 | bloqueado | S/T/C no implican N |
| KI-21 | RSA, AIS162, 802.15.4g Sub-GHz no implementados | planned/incomplete | README Roadmap; matriz §14 | API/FW | actual | no handler/API | ninguno | EV-53 | bloqueado | no es bug por sí solo |
| KI-22 | Nombres Zigbee/Thread/Matter/W-MBus/Wi-SUN/Sidewalk exceden alcance raw/preset si se leen como stacks | limitación de alcance | README notas; matriz §11 | docs/API | actual | sólo PHY/raw/config | parser/stack externo | EV-20/25/26 | no ejecutado | interoperabilidad separada |
| KI-23 | BLE conexión/GATT/active scan retirado, pero quedan restos internos | removed + deuda documental | Architecture §2; `radio_if.c/.h`, SmartRF | FW/docs | actual | no handler/API alcanzable | usar Sniffle | EV-54 | no probado | retirada deliberada, no bug |
| KI-24 | `protocol.md` omite TX-test/crypto y error 0x07 visibles en código | discrepancia docs/código | protocol vs `protocol.h/enums.py/command_processor.c` | docs/protocolo | actual | comandos handlers activos | usar código como fuente | EV-13/30 | no aplica | inventario existente lo confirma |
| KI-25 | Versiones FeralRF v2.0, GET_INFO 1.0.0 y Python 0.3.0 divergen | discrepancia | Architecture, control payload, pyproject | docs/tooling | actual | constantes distintas | registrar las tres | EV-01 | no ejecutado | no inferir compatibilidad |
| KI-26 | `pytest` vs `python -m pytest` difiere en Windows; KillerBee opcional causa skip | entorno/reproducibilidad | baseline aportado; `importorskip` | tooling | actual en host | no defecto RF demostrado | usar módulo; registrar cwd/env | host ya observado | 422/1 skip | investigar path sólo si se desea |
| KI-27 | Validaciones fechadas no tienen logs/hash/commit y full OTA es anterior a HEAD | evidencia histórica stale | matriz §2/14 | documentación | actual | no artefactos crudos | EV-40 con paquete de evidencia | EV-40 | no ejecutado | jamás contar como “nuestra” |
| KI-28 | Flashear `.bin` causa boot failure; debe usarse `.hex` | limitación tooling | README/Architecture | boot/tooling | actual documentado | formatos build existen | catnip + `.hex` | fuera de ejecución | no probado | no reflashear en esta fase |
| KI-29 | Límites: TX efectivo 125, RX emitido 239, BLE AdvData 31, SHA/TRNG 240 | limitación | protocol §§4,5,7; API | protocolo/FW | actual | buffers/validación | validar fronteras | EV-15/31 | no ejecutado | no truncar silenciosamente |
| KI-30 | Curve25519 sirve ECDH, no ECDSA; auth/bounds crypto específicos | limitación | crypto smoke/API/engine | crypto CC1352 | actual | unsupported curve code 5 | vectores independientes | EV-30/31 | no ejecutado | no contar skip como PASS |
| KI-31 | RX_START puede ACK y fallar después con error asíncrono seq 0/FF | limitación/protocolo | protocol; `data_task.c`; `radio.py:_read_response` | wire/FW/Python | actual | buffer compatibility implementado | consumir RxStreamError/reset | EV-14 | no ejecutado | orden temporal importa |
| KI-32 | Control externo U2/CTF por RP2040 no está expuesto por FeralRF | cuestión de hardware | notas Vault; firmware `ti_rf_config_min.c` sólo DIO28/29/30 | RP2040/RF front-end | actual/incierto | no comandos `band1/2/3` en API | fijar/registrar estado CTF | EV-20/22/27 | no ejecutado | posible causa de banda errónea |
| KI-33 | IDs Catnip no son persistentes y el mapeo de roles tiene fallback final por orden COM | tooling/identidad | `CatSniffer-Tools/catnip/modules/core/usb_connection.py:_group_ports_by_device/_map_roles/find_devices` @ `126f13b` | host/tooling | actual | agrupación por serial/HWID/location es más segura, pero no infalible | `devices --debug` + `identify` + Windows; reenumerar | EV-00/04 | no ejecutado | mejora candidata: exponer identidad persistente al reset FeralRF |
| KI-34 | `catnip sniff` puede auto-flashear el CC; TI sniffer no tiene imagen V2 soportada | riesgo tooling/compatibilidad | `device_session.py:require_firmware_for_board`; `fw_aliases.py`; `sniff/cli.py` @ `126f13b` | Catnip/CC1352 | actual | V2 con TI sniffer preinstalado puede conservarse, pero Catnip no lo provisiona | `status` antes de sniff; preservar V2 | EV-05/20/29 | no ejecutado | stock no significa automáticamente “sniffer IEEE” |
| KI-35 | No existe target FeralRF explícito para CatSniffer V2/CC1352P1/SAMD21 | compatibilidad no establecida | FeralRF README/CMake/SysConfig/SmartRF; Catnip `board.py` | board/FW | actual | target actual P7/x7 y arquitectura RP2040 | no flashear V2; usar stock | todos OTA | no aplica | CC13xx común no basta |

## Resultados históricos que deben reproducirse

| Fecha repo | Evidencia declarada | Reproducción |
|---|---|---|
| 2026-04-08 | 18/18 control; OTA BLE/IEEE/Sub-1/W-MBus/OOK; 433 marginal, OOK433 0/10 | EV-20–25 y EV-40 |
| 2026-04-29 | CW/PRBS/stop PASS wire-level | EV-13 con instrumento |
| 2026-04-30 | Crypto 9/9 en hardware | EV-30/31 |
| 2026-05-03 | Wi-SUN/Sidewalk FSK 70/70; MIOTY 0/10 | EV-26/52 |
| 2026-05-04 | Emulación 7/7 wire-level | EV-28 con observador |
| sin fecha completa | PHY switch sin reset falla en segundo ciclo | EV-41 |

## Candidates for investigation

| Candidato | Evidencia / KI | Confianza | Prueba que decide | Capa probable |
|---|---|---|---|---|
| Reset no portable por aritmética de COM | KI-15 | alta en diseño, conducta pendiente | EV-00/04 | Python/RP2040/tooling |
| Estado real del selector U2/CTF por banda | KI-32 | alta como hueco de observabilidad | EV-20/22/27 + inspección RP2040/medición | RP2040/RF HW |
| Propietario 2.4 está más validado que README | KI-08 | alta discrepancia | EV-27 | docs/validación |
| Fallback sintético quizá ya no coincide con troubleshooting | KI-12/31 | media | EV-14 | FW/docs |
| `protocol.md` no representa toda la superficie actual | KI-24 | alta | contraste estático + EV-13/30 | docs/protocolo |
| Tres versiones visibles impiden inferir compatibilidad | KI-25 | alta | EV-01 + política de versionado futura | docs/tooling |
| Workarounds RF esconden hazard de lifecycle | KI-02/03/09/10/11 | alta | EV-41/42/46 | TI RF/CC1352 |
| Drops/overflow no están caracterizados contra tasa | KI-13 | alta | EV-43/44 | CC1352/UART |
| Claims de protocolo pueden leerse como stack | KI-22 | alta | EV-20/25/26/29 | docs/API |
| Integrar el mapa Catnip en `reset_device()` para evitar `Bridge+2` | KI-15/33 | alta | EV-00/04 y diseño futuro | Python/tooling |
| Preflight explícito antes de auto-flash de comandos `sniff` | KI-34 | alta | revisión UX/tooling futura | Catnip/firmware |
| Compatibilidad FeralRF V2 no documentada ni configurada | KI-35 | alta | sólo un target P1 explícito y revisado podría cambiarla | board/build |
| Rango real de potencia y High-PA | KI-07 | alta | EV-13 instrumentado | RF HW/FW/docs |
| Baseline histórico no es auditable ni actual | KI-27 | alta | EV-40 con hashes/logs | validación/tooling |

## Preguntas antes de pruebas físicas

1. ¿Qué mapa muestra `catnip devices --debug` y coincide el Shell de la misma placa con el `Bridge+2` que FeralRF calculará?
2. ¿Qué versión/hash corre en RP2040 y cuál es el SHA-256 del HEX FeralRF flasheado?
3. ¿Cuál es revisión/serial de cada CatSniffer, antena montada y estado U2/CTF por banda?
4. Cuando llegue AUX-V3, ¿qué objetivo exige conservarla stock primero y qué imagen/hash exactos permitirían restaurarla después de usar FeralRF?
5. ¿Qué equipo hay: SDR, analizador, power meter, atenuadores/cables, antenas 169/433/868/915/2.4?
6. ¿Qué bandas/potencias están autorizadas en el laboratorio y existe recinto apantallado?
7. La fuente Zigbee CH25 ya existe; ¿qué otras fuentes BLE, W-MBus, Thread/Matter, Wi-SUN o Sidewalk están disponibles para interoperabilidad?
8. ¿Se instalará KillerBee en Linux/WSL o se deja esa validación bloqueada?

## Orden recomendado

P0: EV-00–04, 14. P1: EV-05/06, 10–12, 20–22, 24, 30, 40–42, 46. P2: EV-13, 15, 23, 25–29, 31, 43–47. P3: EV-50–54. No continuar tras un hang hasta capturar evidencia y recuperar baseline EV-01/02.
