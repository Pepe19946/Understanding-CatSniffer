# FeralRF RF-01 a RF-05 — runbook de validación

Fecha de preparación: 2026-10-19  
Repositorio auditado: `FeralRF`, rama `feature/hardware-validation`, commit base
`fbff1b7f5077964418fb63a3fdf346a27c35a003`  
Estado: **paquete preparado; ninguna prueba física ejecutada**.

## 1. Alcance, evidencia y exclusiones

Este runbook valida entrega OTA observable entre dos CatSniffer y captura pasiva
IEEE 802.15.4. No mide frecuencia, potencia, espectro ni modulación con un
instrumento independiente. Un ACK sólo significa **comando aceptado**; un
`RX_PACKET` CRC-good con marcador significa **recepción visible por el peer**.

- No se incluyen pruebas BLE, jamming, flashing ni cambios de firmware.
- No se modifica `VALIDATION_MATRIX.md` ni los informes históricos.
- RF-04 es recepción pasiva; no se usa ninguna API de inyección.
- RF-05 no pertenece a la secuencia automática: es opcional, aislada y última.
- Los nombres Wi-SUN/Zigbee describen preset o tráfico; no prueban
  interoperabilidad de stacks.

| Estado | Uso en este runbook |
|---|---|
| `PASS` | Cumple ACK, recepción peer CRC-good, marcador y cleanup exigidos. |
| `PARTIAL` | Hay evidencia OTA, pero falta una condición compuesta (RF-05: recuperación). |
| `FAIL` | La prueba se ejecutó y no cumplió aceptación. |
| `BLOCKED` | No pudo establecerse una precondición o control atribuible. |

## 2. Hardware y rutas fijas

| Placa | Cat-Bridge | Cat-Shell | Rol usual |
|---|---:|---:|---|
| CatSniffer #2 | COM33 | COM35 | TX RF-01/RF-03/RF-05; RX RF-02 |
| CatSniffer #3 | COM88 | COM87 | RX RF-01/RF-03/RF-05; TX RF-02 |

Baseline esperado: RP2040 v3.1.0.1, Git `c0cd5a4`, build
`2026-09-04T00:40:22Z`, board `v3 RP2040 CC1352P7`; CC1352 firmware 1.0.0,
capabilities 7. COM88 debe verificarse, no suponerse. `Band` de Cat-Shell es
estado lógico: no demuestra por sí solo la posición física del switch RF.

## 3. Preparación compartida

### 3.1 Acciones manuales

1. Separar las placas lo suficiente para evitar acoplamiento por proximidad,
   sin retirar las antenas durante transmisión.
2. Inspeccionar que cada placa tenga la antena apropiada para 2.4 GHz y para
   868/915 MHz y conocer el selector físico aplicable.
3. Confirmar que es posible retirar **toda** alimentación USB/externa de ambas.
4. Cerrar terminales, IDEs o procesos que puedan poseer COM33/35/87/88.
5. Consultar el canal Zigbee autorizado para RF-04; no adivinarlo escaneando.

### 3.2 PowerShell e instalación

Abrir PowerShell en el repositorio y ejecutar:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF'
git status --short
git rev-parse HEAD
python --version
python -m pip install -e '.\python'
python -c "import feralrf; print(feralrf.__file__)"
$rfRunStamp = Get-Date -Format 'yyyyMMdd_HHmmss'
$rfRunDir = Join-Path (Resolve-Path '.').Path "validation_runs\next_rf_$rfRunStamp"
New-Item -ItemType Directory -Path $rfRunDir -Force | Out-Null
```

El import debe resolver a `FeralRF\python\feralrf\__init__.py`. Registrar HEAD y
el estado sucio; no continuar si hay cambios desconocidos en host/firmware.

Inventario USB independiente:

```powershell
Get-CimInstance Win32_SerialPort | Sort-Object DeviceID | Select-Object DeviceID,Name,PNPDeviceID | Format-List
```

Verificar manualmente la asociación #2/#3 por etiqueta física y PNP/MI. No usar
proximidad numérica entre COMs como prueba de asociación.

### 3.3 Preflight único (abre puertos, no transmite)

Sólo después de las verificaciones manuales:

```powershell
python .\experimental_artifacts\next_rf_validation\preflight.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --antenna-confirmed --recovery-confirmed 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '00_preflight.jsonl')
```

PASS exige los cuatro COM presentes, ambas Shell con identidad RP2040 esperada,
ambos Bridge con firmware 1.0.0/capabilities 7 y `get_stats()` correcto. Los
contadores iniciales se conservan sin exigir que sean cero. El texto completo de
`status` registra modo, banda y contadores de bridge; no infiere la ruta física.

Si falta COM88, una identidad no coincide o un puerto está ocupado: `BLOCKED`.
Desconectar/reconectar, volver a inventariar y repetir sólo el preflight.

## 4. Convención de resultados

- Los scripts RF-01/02/03/05 producen JSONL por stdout con timestamp UTC,
  test/stage, puertos, configuración, ACK TX, paquetes RX, CRC, errores,
  estadísticas, cleanup y resultado.
- `Tee-Object` guarda el transcript; stderr se combina con `2>&1`.
- Un marker puede aparecer dentro del payload/FCS reportado; la comparación es
  deliberadamente por contención porque RX IEEE puede incluir FCS.
- Umbral normal: al menos 9 de 10 markers CRC-good. RF-02 usa 9/10 para PASS del
  paquete automatizado; anotar por separado que la campaña histórica usó 10/10.
- Exit code 0 = criterio del script satisfecho; 2 = FAIL/PARTIAL; 3 = BLOCKED.
- Consultar `$LASTEXITCODE` inmediatamente después de cada pipeline y anotarlo;
  no deducir el resultado sólo del nombre del archivo de log.
- En RF-05, incluso una etapa OTA satisfactoria termina con exit 2/`PARTIAL`
  hasta completar el power-cycle y control IEEE posterior.
- Ninguna RSSI/LQI se convierte en potencia, sensibilidad o margen RF.

## 5. RF-01 — switching IEEE 802.15.4

**Propósito.** Probar `set_channel()` 25→20→25 en una misma sesión, con control
de mismatch TX25/RX20. El firmware actual intenta reiniciar RX si el canal cambia
con RX activo, pero `RadioIF_setChannel()` ignora el retorno de
`RadioIF_restartRfRx()`. Por ello el script cambia canal sólo con RX detenido y
reinicia RX en cada etapa.

**Precondiciones.** Preflight PASS; antenas/ruta 2.4 GHz; ningún OOK desde el
último power-cycle. Roles: #2 TX COM33/COM35, #3 RX COM88/COM87.

```powershell
python .\experimental_artifacts\next_rf_validation\rf01_ieee_channel_switch.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --min-received 9 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '01_rf01_ieee_channel_switch.jsonl')
```

| Etapa | TX/RX | Marker | Aceptación |
|---|---|---|---|
| `CONTROL_CH25` | 25/25 | `C1012501` | ≥9/10 CRC-good |
| `MISMATCH_TX25_RX20` | 25/20 | `C1012502` | 0 markers; no decide PASS por sí sola |
| `MATCHED_CH20` | 20/20 | `C1012001` | ≥9/10 CRC-good |
| `RETURN_CH25` | 25/25 | `C1012503` | ≥9/10 CRC-good |

PASS exige las tres positivas, mismatch cero, ausencia de errores async y ACK de
cada `RX_STOP`. El script detiene la secuencia ante una positiva fallida. Un
mismatch vacío con controles débiles es FAIL, no evidencia de aislamiento.

Cleanup automático: un intento acotado de `RX_STOP`, drenaje de host y cierre de
ambos Bridge. Cat-Shell queda en `band1`. Si hay fatal/timeout, guardar log, no
continuar a RF-02 y aplicar la recuperación de §10.

## 6. RF-02 — Sub-GHz inverso

**Propósito.** Cubrir la dirección no validada #3 TX → #2 RX usando exactamente
la secuencia temporal del runner manual de octubre: RX, espera 0.3 s, 10 TX
separados 0.1 s, espera 1 s, observación RX 2 s y errores TX 0.5 s.

**Precondiciones.** RF-01 PASS; rutas/antenas 868/915 verificadas. El preset
`wisun_915_fsk_50k` usa 902.2 MHz, PHY `PROPRIETARY_GFSK`, canal 0, potencia
solicitada 0 dBm y marker de cuatro bytes `C2028802`.

```powershell
python .\experimental_artifacts\next_rf_validation\rf02_subghz_reverse.py --tx-port COM88 --rx-port COM33 --tx-shell COM87 --rx-shell COM35 --marker-hex C2028802 --min-received 9 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '02_rf02_subghz_reverse.jsonl')
```

PASS: 10 ACK TX, ≥9 packets CRC-good con marker, cero errores async, `RX_STOP`
ACK y cleanup correcto. PARTIAL: 1–8 markers válidos. FAIL: cero markers tras
configuración completa. BLOCKED: init/configuración/puerto impide ejecutar.

**Control positivo sólo si falla el inverso.** No se lanza automáticamente.
Después de recuperación y preflight, comprobar la dirección históricamente
validada #2→#3 con marker distinto:

```powershell
python .\experimental_artifacts\next_rf_validation\rf02_subghz_reverse.py --test-id RF-02-CONTROL --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --marker-hex C2023302 --min-received 9 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '02b_rf02_positive_control.jsonl')
```

Si el control también falla, RF-02 queda BLOCKED por baseline/ruta, no se atribuye
el fallo a la dirección #3→#2. Si control PASS e inverso FAIL repetible una sola
vez tras recuperación, registrar FAIL direccional y detener la campaña.

## 7. RF-03 — IEEE ↔ Sub-GHz sin reset

**Propósito.** Probar IEEE ch25 → `wisun_915_fsk_50k` → IEEE ch25 sin reset ni
re-init CC1352. La fuente usa `RF_MODE_MULTIPLE`, conserva el handle no-433 y
aplica flush + RadioSetup al cambiar PHY. El script crea dos `Radio`, ejecuta
`init()` exactamente una vez por placa y conserva ambos durante las tres etapas.

**Precondiciones.** RF-01 y RF-02 PASS; power-cycle/preflight limpio si hubo un
fallo; no haber cargado OOK. Roles #2 TX y #3 RX.

```powershell
python .\experimental_artifacts\next_rf_validation\rf03_cross_phy_switch.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --min-received 9 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '03_rf03_cross_phy_switch.jsonl')
```

| Etapa | Band/PHY | Marker | PASS de etapa |
|---|---|---|---|
| `IEEE_INITIAL` | band1, IEEE ch25 | `C3032501` | ≥9/10 CRC-good |
| `SUBGHZ_MIDDLE` | band2, preset 902.2 MHz | `C3039002` | ≥9/10 CRC-good |
| `IEEE_RETURN` | band1, IEEE ch25 | `C3032503` | ≥9/10 CRC-good |

RX se detiene antes de cada transición. Ante cualquier etapa fallida el script
no continúa, no restaura banda automáticamente y no llama `reset_device()`.
El evento `FATAL` conserva etapa/completadas. Cleanup sólo cierra los Bridge.
PASS exige las tres etapas, todos los stops y cero errores.

Después de FAIL/BLOCKED, no ejecutar RF-04/RF-05: aplicar §10 y correr únicamente
el control IEEE de recuperación antes de decidir una nueva tentativa.

## 8. RF-04 — captura Zigbee pasiva mediante KillerBee

**Propósito.** Validar el adapter `KillerBeeFeralRF`, `pnext()` y PCAP sobre
tráfico autorizado. Esto demuestra captura IEEE 802.15.4/MAC, no asociación,
descifrado ni interoperabilidad Zigbee.

Instalar el extra y verificar imports sin hardware:

```powershell
python -m pip install -e '.\python[killerbee]'
python -c "import killerbee; from feralrf.integrations.killerbee import KillerBeeFeralRF; print('KillerBee import OK')"
```

**Acciones manuales.** Confirmar el canal desde la administración de la red.
El ejemplo siguiente usa 25 sólo si ése es el canal confirmado. Puede producirse
una acción normal autorizada (p. ej. operar un sensor), nunca inyectar frames.

```powershell
python .\python\examples\killerbee_sniff.py --port COM33 --channel 25 --duration 60 --count 10 --timeout 100 --min-valid-crc 10 --pcap (Join-Path $rfRunDir '04_rf04_zigbee_passive.pcap') 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '04_rf04_zigbee_passive.log')
```

El loop termina al llegar a 10 frames o 60 s. PASS exige salida
`SNIFF DONE packets=10 valid_crc=10`, exit 0, PCAP no vacío y cleanup sin error.
Si hay tráfico insuficiente, exit 2 significa BLOCKED por observabilidad; no es
FAIL del receptor. Errores de dependencia/puerto/protocolo son BLOCKED.

Abrir el PCAP en Wireshark y confirmar link type IEEE 802.15.4 (DLT 195), diez
registros y campos MAC decodificables. No exigir descifrado. Conservar el PCAP y
log juntos. El script sólo llama `sniffer_on`, `pnext`, `sniffer_off` y `close`;
no llama `inject` ni transmite.

## 9. RF-05 — OOK 868 aislado, opcional

**Estado: CONDITIONAL.** `mod_type=2` es válido y carga patches RF genook, pero
éstos no deben descargarse/cambiarse en caliente. Esta prueba se ejecuta última,
fuera de cualquier secuencia y sólo con aprobación inmediata del operador.

### 9.1 Go/no-go manual

Proceder sólo si: RF-01..03 PASS; RF-04 terminó/omitió; antenas/ruta 868 están
confirmadas; se puede quitar toda alimentación; el operador acepta perder la
sesión si el RF Core se bloquea. Si falta una condición: **RF-05 NOT READY**.

1. Apagar procesos seriales.
2. Desconectar USB y toda fuente externa de ambas placas por al menos 15 s.
3. Reconectar, confirmar COMs y ejecutar el preflight §3.3.
4. Aprobar explícitamente RF-05 inmediatamente antes del comando.

```powershell
python .\experimental_artifacts\next_rf_validation\rf05_ook_868.py --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --marker-hex C5050001 --min-received 9 --operator-approved --initial-power-cycle-confirmed 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '05_rf05_ook_868.jsonl')
```

El script selecciona band2, inicializa, configura `ook_868_4k8`, solicita 0 dBm,
envía diez markers y observa el peer. No invoca reset, power-cycle, otro preset o
otro PHY. Tras cargar OOK, incluso ante excepción, **no ejecutar ningún comando
RF adicional**.

La etapa OTA es satisfactoria con 10 ACK, ≥9/10 marker CRC-good, cero errores y
stop/cleanup correctos. El resultado queda `PARTIAL` hasta esta recuperación:

1. Cerrar el proceso y guardar el log.
2. Desconectar **toda** alimentación de ambas placas por ≥15 s; no usar reset
   software como sustituto.
3. Reconectar, inventariar COMs y repetir preflight.
4. Ejecutar un único control IEEE fresco:

```powershell
python .\experimental_artifacts\next_rf_validation\rf01_ieee_channel_switch.py --only-control --tx-port COM33 --rx-port COM88 --tx-shell COM35 --rx-shell COM87 --min-received 9 2>&1 | Tee-Object -FilePath (Join-Path $rfRunDir '05b_rf05_recovery_ieee_control.jsonl')
```

RF-05 compuesto es PASS sólo si OTA OOK fue satisfactoria, se documentó el
power-cycle final y el control IEEE PASS. Si hay hang/error/0 markers, registrar
FAIL o BLOCKED según el punto de fallo, hacer power-cycle completo y no reintentar
OOK en la misma campaña.

## 10. Recuperación general y regla de continuación

Ante timeout, error async, stop fallido, pérdida de COM o transición fallida:

1. Detener; no encadenar el siguiente test ni reintentar automáticamente.
2. Guardar JSONL/log y anotar el último evento/etapa.
3. Cerrar el proceso si siguiera vivo y confirmar que liberó los COM.
4. Retirar toda alimentación de ambas placas ≥15 s y reconectar.
5. Revalidar asociación COM y ejecutar el preflight.
6. Para fallos RF-01/RF-03/RF-05, correr `RF-01 --only-control`.
7. Continuar sólo con preflight y control PASS. No usar RF-05 para diagnosticar.

No se prescribe `Radio.reset_device()`: deriva Shell como Bridge+2 (incorrecto
para COM88→COM87) y un reset software no sustituye el power-cycle exigido por OOK.

## 11. Orden mínimo y decisiones

1. Preparación + preflight.
2. RF-01. Debe PASS antes de cualquier otra OTA.
3. RF-02. Si falla, control #2→#3 una vez tras recuperación.
4. RF-03. Requiere RF-01/RF-02 PASS.
5. RF-04 pasivo; BLOCKED por falta de tráfico no invalida RF-01..03.
6. Revisión go/no-go; RF-05 opcional y siempre último.
7. Power-cycle final + control IEEE si se ejecutó RF-05.

No ejecutar: proprietary 2.4 GHz (fallback actual a 868 MHz), MSK/4-FSK/4-GFSK
(valores reservados), 169 MHz (ruta/setup incoherentes), 433 MHz sin hardware RF
adecuado, MIOTY sin patch, BURST/STOP como nueva campaña, ni ninguna prueba BLE.

## 12. Checklist de cierre por corrida

- [ ] Commit, `git status`, Python e import registrados.
- [ ] Puertos Bridge/Shell asociados físicamente y pasaron preflight.
- [ ] Antena y banda manual documentadas; no inferidas sólo de Shell.
- [ ] Comando exacto y stdout/stderr preservados.
- [ ] ACK TX separado de RX peer; markers y CRC contados.
- [ ] Errores async, stats y cleanup preservados.
- [ ] Resultado PASS/PARTIAL/FAIL/BLOCKED asignado sin inventar métricas.
- [ ] Si hubo fallo, recuperación y control registrados antes de continuar.
- [ ] Si hubo OOK, power-cycle final y control IEEE registrados.
- [ ] PCAP RF-04, si existe, revisado sólo como captura IEEE/MAC pasiva.
