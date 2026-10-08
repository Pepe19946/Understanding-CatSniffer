# Seguimiento documental OTA Sub-GHz — ruta RF y control de banda

## Propósito y fuentes experimentales

Reconciliar [[Reporte OTA Sub-GHz - GFSK 868 y 915 MHz]] y [[Segunda campaña OTA proprietary FeralRF — Ampliación de presets y verificación del procedimiento]] con la investigación consolidada en [[Sub-GHz en CatSniffer V3 - CC1352P7, ruta RF y control de banda]]. Esta nota resume evidencia; los comandos, stdout, marcadores y timestamps literales permanecen en sus reportes.

No se supone que las asociaciones de sesión `v3.1.0.0`–`8eaa84c` y `v3.1.0.1`–`c0cd5a4` prueben hashes de los binarios instalados.

## Historia por etapas

| corte | corridas | retornos `transmit()` exitosos | presets únicos acumulados | paquetes/hits | `RxStreamError` expuestos |
|---|---:|---:|---:|---:|---:|
| Primer reporte | 4 | 40 | 2 | 0 | 0 |
| Adición de la segunda campaña | 20 | 200 | 17 nuevos, 19 acumulados | 0 | 0 |
| **Acumulado actual** | **24** | **240** | **19** | **0** | **0** |

La segunda etapa contiene 18 corridas del helper y dos controles concurrentes. Las repeticiones bidireccionales y concurrentes aumentan el número de corridas, pero no el de presets únicos. Cada corrida registró diez retornos exitosos; **240 retornos no prueban 240 emisiones RF**.

En las ventanas positivas documentadas no hubo líneas `RX_ITEM`. “Cero paquetes/hits” significa que ningún paquete proprietary fue entregado al host receptor en esos ensayos; no equivale a pérdida RF del 100 %, ausencia de energía ni cero actividad en toda capa interna.

## Estado lógico de banda

Los Shell aceptaron transiciones forzadas `band1 ↔ band2` y reportaron `Band: 0/1` según lo esperado. En el código comparado, `band1` intenta CTF=`0,1,0` y `band2`, CTF=`0,0,1` desde RP2040 GPIO8/9/10. El valor `Band` es cache, no lectura de GPIO, U2 ni ruta RF.

`Radio: LoRa`, `LoRa: initialized` y `LoRa Mode: Stream` describen el subsistema SX1262 del RP2040. El SX1262 no participa en `configure_prop()` ni `TX_RAW` del CC.

## Control concurrente

El control `gfsk_2440_50k` inverso registró:

| evento | timestamp | relación |
|---|---|---|
| RX listo en COM33 | `12:34:13.916` | inicio de referencia |
| primer ACK TX en COM88 | `12:34:17.980` | `4.064 s` después de RX listo |
| último ACK TX | `12:34:19.241` | `1.261 s` después del primer ACK |
| resultado RX | `12:34:43.931` | `24.690 s` después del último ACK |

Resultado: `events=0, packets=0, hits=0, errors=0`.

**Conclusión metodológica:** las solicitudes TX ocurrieron mientras `read_packets()` estaba activo y quedaron más de 24 s de observación después del último ACK. Ejecutar TX y RX desde una sola terminal no está sustentado como explicación única del patrón; el consumo tardío del helper original pierde fuerza explicativa.

Esto no prueba diez emisiones, llegada a una capa interna, buffering universalmente lossless ni funcionamiento de la ruta RF.

## Estadísticas expuestas

Después del control concurrente, ambos endpoints devolvieron:

`DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=None, ll_kind_adv=None, ll_kind_scan=None, ll_kind_connect=None, ll_kind_data=None)`

Conclusión estrecha:

- en el intervalo post-`init()` pertinente no se contabilizaron RX correctos, CRC erróneos, drops u overflows en esos contadores;
- el resultado es consistente con que ningún paquete proprietary alcanzara las etapas RX contadas;
- no cubre todos los puntos RF, firmware, UART, USB o host;
- no prueba ausencia de energía transmitida.

## Clasificación actual A–D

### A — GFSK/FSK adecuados para diagnóstico físico

| grupo | presets ejecutados | interpretación común |
|---|---|---|
| GFSK genérico | `gfsk_868_50k`, `gfsk_868_100k`, `gfsk_902_50k`, `gfsk_915_50k`, `gfsk_2440_50k`, `gfsk_2440_250k` | frecuencia dentro del silicio y U4 publicado; solicitudes aceptadas; cero paquetes entregados |
| W-MBus PHY/raw | `wireless_mbus_s_868`, `_t_868`, `_c_868` | `mod_type=1` GFSK; no demuestra stack EN 13757 ni interoperabilidad |
| Sidewalk PHY/raw | `sidewalk_915_fsk_50k`, `_250k` | `mod_type=0` FSK; no demuestra stack Sidewalk |
| Wi-SUN PHY/raw | cinco `wisun_915_fsk_*` de 50–300 k | `mod_type=0` FSK; no demuestra Wi-SUN FAN |

Para todos: el plano de control aceptó configuración/TX; no se entregó un paquete; TX físico, RX físico y causa siguen sin confirmar.

### B — preset aceptado, modulación nombrada no demostrada

- `msk_868_50k` (`mod_type=4`)
- `4fsk_868_50k` (`mod_type=5`)
- `4gfsk_868_50k` (`mod_type=6`)

El `rfc_CMD_PROP_RADIO_DIV_SETUP_PA_s` del SDK auditado documenta 0=FSK, 1=GFSK, 2=OOK y reserva los demás valores. Por ello solo se afirma existencia del preset, aceptación de configuración/TX y cero paquetes observados. No son pruebas físicas interpretables de MSK/4FSK/4GFSK ni establecen éxito o fallo OTA de esas modulaciones.

### C — no ejecutados por falta de una prueba defendible

- **Ocho presets 433 MHz:** el CC1352P74 sí cubre 433, pero U4 publicado solo especifica 862–928/2400–2500 MHz. El asesor indicó no probar el grupo o que no estaba soportado actualmente; se registra como restricción operacional coherente con la limitación publicada, no como prueba de una causa eléctrica exacta. La revisión ensamblada continúa imperfectamente cerrada.
- **W-MBus N 169 MHz:** `wireless_mbus_n_169_2k4` y `_4k8` quedan fuera de las bandas del CC1352P74 y fuera de U4. Su presencia en `presets.py` no los vuelve ejecutables.

### D — diferidos por implementación o ciclo de vida

- `ook_868_4k8`: cargar el camino OOK puede exigir reset/power cycle; la API de reset conserva el problema conocido de derivación del Shell.
- OOK 433: suma al lifecycle la falta de cobertura 433 en U4 publicado.
- `mioty_868_tsunb`: implementación pending/incompleta, antecedente histórico `0/10` y probable necesidad de soporte CPE custom.

## Reconciliación histórica

- W-MBus S/T/C conserva claims OTA históricos favorables, sin trazabilidad suficiente para convertirlos en evidencia vigente.
- La campaña actual obtuvo `0/70` paquetes entregados para los mismos cinco nombres Wi-SUN y dos Sidewalk asociados al claim histórico `70/70` del 2026-05-03.
- **La campaña actual no reprodujo el claim histórico Wi-SUN/Sidewalk `70/70` bajo las condiciones documentadas presentes.** No se declara regresión: faltan logs crudos históricos, hash de imagen, identidad de dispositivos y condiciones completas que demuestren equivalencia.
- Proprietary 2.4 GHz conserva antecedentes históricos contradictorios; los ensayos actuales no los resuelven porque no confirman emisión.

## Conclusión acumulada acotada

- La comunicación de control host–dispositivo funciona en ambas unidades.
- Los comandos de banda fueron aceptados y el cache cambió como se esperaba; no hubo readback físico.
- El control concurrente confirma solapamiento entre observación RX y aceptación TX.
- Ningún paquete proprietary llegó al host receptor en 24 corridas y 19 presets únicos.
- Los casos GFSK/FSK válidos abarcan 868, 902.2, 915 y 2440 MHz, cubiertos por el silicio y U4 publicado.
- Los casos nominales MSK/4FSK/4GFSK no demuestran esas modulaciones por los `modType` reservados.
- La campaña no demuestra 240 emisiones, pérdida RF del 100 %, fallo de TX, fallo de RX ni defecto firmware específico.
- El dominio abierto sigue incluyendo configuración compartida, ejecución TX, ejecución RX, selección/ruta RF externa y entrega visible al host.
- La expansión ciega de presets queda pausada.

## Siguiente observación física discriminante

Usar una sola solicitud `gfsk_868_50k` y establecer, en orden:

1. niveles RP2040 `CTF1/2/3` durante la transición forzada `band1 → band2`;
2. puerto seleccionado por U2 conforme a la tabla de verdad del RFSW8006Q;
3. presencia o ausencia de energía RF en J1 con SDR, analizador de espectro, VNA o receptor acoplado apropiado;
4. solo si hay energía: frecuencia portadora, tasa de símbolos, desviación, sync y marcador.

No usar continuidad DC ordinaria a través del switch semiconductor como prueba de ruta RF.

Árbol de decisión:

- CTF incorrecto → control de banda/RP2040.
- CTF correcto y sin RF en J1 → TX, PA, matching o ruta RF seleccionada.
- RF presente con parámetros incorrectos → configuración proprietary.
- forma de onda y marcador correctos en J1, sin paquete peer → configuración/ruta RX, framing o entrega host.

Es una propuesta futura; no se ejecutó durante esta actualización.
