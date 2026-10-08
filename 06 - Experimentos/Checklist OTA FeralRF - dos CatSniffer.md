# Checklist OTA FeralRF — dos CatSniffer

Hoja de banco para [[Guía enfocada OTA FeralRF - TX RX con dos CatSniffer]]. La guía contiene comandos completos, semántica de bytes y criterios; no improvisar variantes aquí.

## Identidad de sesión

- Fecha/hora: ____________________ Operador: ____________________
- Lugar/entorno RF autorizado: ____________________
- Distancia/acoplamiento: ____________________
- Antena #1/banda: ____________________ Antena #2/banda: ____________________
- FeralRF commit esperado: `0178721cbd4f0d0f6f8eba5ae919ca46066d5dea`
- Hash/versiones observadas #1: ____________________ #2: ____________________

| Placa | Bridge | LoRa (no usar) | Shell | Rol inicial |
| --- | ---: | ---: | ---: | --- |
| CatSniffer #1 | `COM33` | `COM34` | `COM35` | TX/DUT |
| CatSniffer #2 | `COM88` | `COM86` | `COM87` | RX/observador |

- [ ] Enumeración Windows verificada; etiquetas físicas correlacionadas.
- [ ] `COM35` corresponde a #1 y `COM87` a #2; no se calculó `Bridge+2`.
- [ ] `init()`/`get_stats()` responden en `COM33` y `COM88`.
- [ ] Ningún puerto está abierto por otro programa.
- [ ] No se ejecutará Catnip sniff/flash/reset ni helper con auto-reset.
- [ ] Potencia inicial 0 dBm, ventanas cortas y antena/carga adecuada.

Directorio de todos los comandos:

```powershell
Set-Location 'C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF'
$env:PYTHONPATH = (Resolve-Path '.\python').Path
```

Carpeta de evidencia de esta sesión: ____________________

## Orden obligatorio

### Preflight y frente RF

- [ ] Ejecutadas las consultas Shell `identify`, `fw_version`, `status` de la guía.
- [ ] Baseline RX IEEE ch25 en `COM88`: paquetes ______ errores ______.
- [ ] Baseline RX IEEE ch25 en `COM33`: paquetes ______ errores ______.
- [ ] Para 2.4 GHz se forzó `band2`→`band1` en `COM35` y `COM87`.
- [ ] Respuesta Shell literal guardada; hash RP2040 conocido: sí / no.
- [ ] Si ruta/firmware/antena no quedan establecidos, cero paquetes se marcará `INCONCLUSIVE`, no `FAIL`.

### RAW IEEE — ambas direcciones

| Dirección | Negativo hits | TX requests | ACK | RX marker | CRC OK | len=8 | RSSI rango | Resultado |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | --- | --- |
| `COM33 → COM88`, `a10133008801` | ___ | 10 | ___ | ___ | ___ | ___ | ___ | ___ |
| `COM88 → COM33`, `a10288003301` | ___ | 10 | ___ | ___ | ___ | ___ | ___ | ___ |

- [ ] RX comenzó antes de TX en ambas direcciones.
- [ ] Bytes completos y dos bytes FCS finales guardados por paquete.
- [ ] Las asimetrías se registraron por separado; no se promediaron.

### FRAME

| Dirección/marcador | Negativo | Requests/ACK | RX marker | CRC/len | Resultado |
| --- | ---: | --- | ---: | --- | --- |
| `COM33 → COM88`, `a20133008802` | ___ | 10 / ___ | ___ | ___ | ___ |

- [ ] Se conservó como prueba contractual distinta; no se afirmó camino RF distinto de RAW.

### BURST — sólo tras one-shot

| Marcador | Count pedido | Intervalo µs | ACK | RX markers | Duplicados | Errores | Ventana | Resultado |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | --- | --- |
| `a30133008803` | 5 | 250000 | ___ | ___ | ___ | ___ | 12 s | ___ |
| `a30233008803` | 20 | 50000 | ___ | ___ | ___ | ___ | 12 s | ___ |

- [ ] No se infirió count emitido únicamente desde count recibido.
- [ ] Segunda fila omitida si la primera no funcionó.

### CONTINUOUS + STOP — sólo tras BURST

| Marcador | Intervalo | Duración TX | ACK START | RX activo | ACK STOP | Hits post-STOP/5s | Resultado |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| `a40133008804` | 250000 µs | 3 s | ___ | ___ | ___ | ___ | ___ |
| `a40233008804` | 0 | 1 s | ___ | ___ | ___ | ___ | ___ |

- [ ] Timestamps/hits guardados.
- [ ] Control post-STOP comenzó después de terminar TX/observación.
- [ ] Cero post-STOP requerido para cese físico.

### Presets Tier A

> **Estado 2026-10-08:** las filas siguientes se conservan como plantilla histórica. Sus ejecuciones y controles posteriores están en [[Reporte OTA Sub-GHz - GFSK 868 y 915 MHz]] y [[Segunda campaña OTA proprietary FeralRF — Ampliación de presets y verificación del procedimiento]]. La campaña acumula cero paquetes entregados y la expansión está pausada hasta observar CTF/U2/J1 con `gfsk_868_50k`. `4fsk_868_50k` no se interpreta como 4FSK efectivo porque `modType=5` está reservado en el setup auditado.

- [ ] Se definió `Invoke-FeralPresetOta` exactamente como en la guía.
- [ ] Para Sub-G se forzó `band1`→`band2` en ambos Shell.
- [ ] Banda autorizada elegida: 868 / 915 / ninguna.

| Preset/dirección | Marcador | Negativo | ACK | Hits | Errores | Resultado |
| --- | --- | ---: | ---: | ---: | ---: | --- |
| `gfsk_868_50k` o `gfsk_915_50k`, #1→#2 | __________ | ___ | ___ | ___ | ___ | ___ |
| mismo preset, #2→#1 | __________ | ___ | ___ | ___ | ___ | ___ |
| `4fsk_868_50k`, si autorizado | `b20133008802` | ___ | ___ | ___ | ___ | ___ |
| `gfsk_2440_50k`, tras volver a band1 | `b30133008803` | ___ | ___ | ___ | ___ | ___ |

- [ ] PDU proprietary completo guardado; marcador buscado como subsecuencia.
- [ ] Ningún preset protocol-named se describió como interoperabilidad de protocolo.
- [ ] Tier B se difirió hasta aprobar Tier A.
- [ ] OOK/MIOTY/169/433 quedaron Tier C salvo autorización y setup explícito.

## Opcional

- [ ] BLE1M ch37 ejecutado / diferido. Resultado: ____________________
- [ ] KillerBee diferido; no se instalaron dependencias ni se afirmó compatibilidad.

## Campos mínimos por corrida

- Puertos/roles: ____________________ PHY/preset/canal: ____________________
- Frente RF/antenas: ____________________ potencia: ______ dBm
- Marcador solicitado: ____________________ bytes recibidos: ____________________
- Negativo: ______ ACK: ______ RX: ______ CRC: ______ RSSI/LQI: ____________________
- Duplicados/errores: ____________________ inicio/fin: ____________________
- Evidencia A/B/C/D/E/F: ______ Estado: ____________________
- Ruta de stdout/log: ____________________ Observaciones: ____________________

## STOP conditions

Detener, cerrar procesos y registrar el estado si ocurre cualquiera:

- [ ] Puerto/placa no inequívoco.
- [ ] Bootloader, reset o flashing inesperado.
- [ ] Antena/carga/banda/autorización inadecuada.
- [ ] Calentamiento o comportamiento eléctrico anómalo.
- [ ] Errores repetidos, desconexión o STOP no confirmado.
- [ ] Ruta U2/CTF no establecible: clasificar `BLOCKED/INCONCLUSIVE`.

Resultado global de sesión: ____________________

No ejecutar pruebas adicionales ni recuperación destructiva fuera de [[Guía enfocada OTA FeralRF - TX RX con dos CatSniffer]].
