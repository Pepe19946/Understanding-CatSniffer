# Análisis de OOK y otras modulaciones restringidas

## 1. OOK: inventario y soporte real

| Preset | Frecuencia | baud | dev | BW | Estado |
|---|---:|---:|---:|---:|---|
| `ook_433_4k8` | 433.92 MHz | 4800 | 0 | `0x4C` | Implementación OOK presente; hardware 433 no demostrado |
| `ook_433_2k4` | 433.92 MHz | 2400 | 0 | `0x4C` | Implementación OOK presente; hardware 433 no demostrado |
| `ook_868_4k8` | 868 MHz | 4800 | 0 | `0x4C` | Candidato condicional aislado |

`mod_type=2` sí está definido como OOK en el comando TI auditado. La implementación selecciona `Prop0_modeOok`, carga `rf_patch_mce_genook` y `rf_patch_rfe_genook`, usa overrides OOK por banda y ejecuta los mismos `CMD_PROP_TX/RX` de paquete. Por tanto, FeralRF implementa **paquetes propietarios OOK**, no un protocolo OOK específico.

## 2. Estado y riesgo de lifecycle

`RadioIF_getPropMode()` documenta que los patches genook no pueden descargarse de forma segura porque `RFCCpePatchReset` es no-op. `s_prop_ook_active` queda activo; una reconfiguración posterior sólo muta estructuras y evita cerrar/reabrir el RF Core. La propia API avisa de lock de modo/frecuencia.

La frase de `presets.py` “call `reset_device()` after OOK” es insuficiente y conflictiva con el firmware: el reset software no garantiza descargar el patch. La recuperación defendible es retirar **toda** alimentación. Además, `Radio.reset_device()` deriva Cat-Shell como Bridge+2; la asociación histórica COM88→COM87 demuestra que esa aritmética no es fiable. Ésta es una brecha de documentación/API, no una razón para declarar OOK defectuoso.

Nunca debe incluirse OOK en un loop ordinario FSK/GFSK ni intentar cambiar de frecuencia después de su primera configuración.

## 3. Evidencia histórica OOK

- Commits F10 (`4970392` y cierre `6138178`) declaran 10/10 para OOK 868 y posteriormente para OOK 433, pero no se localizaron transcripts crudos asociados; son **claims históricos**.
- La consolidación anterior también registra claims 0/10 para OOK 433. La discrepancia no permite inferir causa.
- RF-05 del 20-oct-2026 quedó `NOT RUN`, no `FAIL`: dependía de RF-01..03 y recuperación, y esos gates no se cumplieron.

No hay base para decir que OOK está roto. Sí hay base para decir que no posee una validación reciente reproducible y que su recuperación exige control manual.

## 4. Revisión de RF-05

El script `experimental_artifacts/next_rf_validation/rf05_ook_868.py` es técnicamente adecuado en sus controles esenciales:

- requiere cuatro COM explícitos y flags de aprobación/power-cycle;
- selecciona `band2` en ambos RP2040 y exige respuesta + `status`;
- inicializa dos sesiones nuevas, elige PHY propietario y configura OOK una sola vez;
- solicita 0 dBm, RX primero, diez TX y cuenta únicamente marcador + CRC;
- registra ACK, paquetes, errores asíncronos, stats, stop y cleanup en JSONL;
- no llama reset, cambia de PHY ni carga otro preset después de OOK;
- retorna `PARTIAL` aunque la etapa OTA sea positiva, hasta completar power-cycle + control posterior.

Limitaciones que deben quedar explícitas:

1. Los flags son atestaciones del operador; el script no puede comprobar un power-cycle físico.
2. `safe_disconnect()` sólo cierra handles host; no recupera el RF Core.
3. El `stop_rx()` es un cleanup acotado, pero si falla se debe detener y retirar alimentación, no seguir diagnosticando en esa sesión.
4. La prueba entre pares no mide espectro, profundidad OOK, potencia ni ancho de banda.
5. Debe preservarse `$LASTEXITCODE` además de JSONL.
6. Los gates RF-01..03 completos son conservadores, pero no todos son causalmente necesarios para OOK. RF-01/RF-03 se bloquearon por un problema de `RX_STOP` IEEE. Un gate más específico y seguro es: preflight de identidad, control positivo raw 868 no-OOK con cleanup, power-cycle inicial y RF-05 como última operación. No se modifica el script en esta auditoría.

Decisión: `ook_868_4k8` = **CONDITIONAL**, no ejecutado. `ook_433_*` = **BLOCKED BY HARDWARE EVIDENCE** hasta identificar revisión, balun/ruta y antena 433.

## 5. Valores reservados 4/5/6

Presets afectados:

- `msk_433_50k`, `msk_868_50k` → `mod_type=4`;
- `4fsk_433_50k`, `4fsk_868_50k` → `mod_type=5`;
- `4gfsk_433_50k`, `4gfsk_868_50k` → `mod_type=6`.

En el header exacto del SDK `8.30.01.01`, la estructura usada por FeralRF, `rfc_CMD_PROP_RADIO_DIV_SETUP_PA_t`, define el campo de tres bits así:

```text
0 = FSK
1 = GFSK
2 = OOK
others = Reserved
```

Python, `RadioIF_PropConfig` y sus docstrings anuncian incorrectamente 4=MSK, 5=4-FSK y 6=4-GFSK. El firmware no valida ni traduce el valor: lo copia directamente al bitfield `modType`. `format_conf` controla sync/FEC/whitening; no convierte el demodulador binario en 4-FSK.

Conclusiones:

- Los seis presets son **UNSUPPORTED BY CURRENT DRIVER CONTRACT**.
- No deben transmitirse ni remapearse de forma experimental.
- Un ACK sólo refleja que el byte fue recibido y truncado/copiado al campo, no que la modulación exista.
- Los nombres actuales son engañosos porque prometen una semántica que la ruta no implementa.

## 6. ¿El hardware podría hacerlo por otra ruta?

TI declara para CC1352P7 soporte de 2-(G)FSK, 4-(G)FSK, (G)MSK y OOK. Esto hace plausible el silicio, no el comando actual. La implementación correcta podría requerir:

- otra imagen/patch del RF Core;
- un comando o configuración SmartRF diferente;
- parámetros de símbolo/desviación/formato distintos;
- lógica de RX/TX y parsing específica;
- validación espectral independiente.

No se encontró una ruta alternativa activa en FeralRF que implemente esos modos. El trabajo futuro debe empezar por una configuración oficial generada para CC1352P7 y luego definir un contrato API que no recicle valores reservados.

Referencia TI de capacidades del dispositivo: https://www.ti.com/product/CC1352P7

## 7. Brechas funcionales

| Familia | Configuración | Firmware | API | Protocolo | Hardware | Validación/documentación |
|---|---|---|---|---|---|---|
| OOK 868 | Plausible | Patch y packet mode existen; lifecycle no recuperable en caliente | Advierte lock, pero recomienda reset ambiguo | No se promete stack | Ruta 868 plausible, identidad pendiente | Falta prueba reciente y recuperación demostrada |
| OOK 433 | Plausible | Patch/overrides existen | Igual | No se promete stack | Ruta/antena sin establecer | Claims contradictorios, sin primarios |
| MSK | Valor inválido para esta estructura | No hay ruta MSK | Docstring induce a error | N/A | Silicio plausible | Claims no prueban MSK |
| 4-FSK/4-GFSK | Valor inválido | No hay ruta cuaternaria | Docstring induce a error | N/A | Silicio plausible | Claims no prueban modulación |

