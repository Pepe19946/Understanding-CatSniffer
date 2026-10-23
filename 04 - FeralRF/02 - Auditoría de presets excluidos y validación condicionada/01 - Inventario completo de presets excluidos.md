# Inventario completo de presets excluidos

## 1. Contrato común del catálogo

`PROP_PRESETS` contiene 31 entradas. Todas llaman `Radio.configure_prop(**preset)` y terminan en `SET_PROP_CONFIG`; ninguna entrada fija `format_conf`, por lo que la API transmite cero y el firmware restaura `0x00A0`: sync de 32 bits, MSB primero, sin FEC ni whitening. La operación real requiere además `set_phy(PHY.PROPRIETARY_GFSK, channel=0)` y después `start_rx()` o `transmit()`.

La desviación no está expresada en Hz en el diccionario. El firmware copia el valor al campo `deviation` y deja `deviationStepSz=0`; en el SDK auditado esto equivale a pasos de 250 Hz. Por ejemplo, `deviation=100` solicita 25 kHz y `deviation=1` solicita 250 Hz.

Leyenda histórica:

- **OTA reciente:** transcript/artefacto primario de la campaña 15–20 oct 2026.
- **Claim histórico:** matriz o commit sin stdout/JSONL primario suficiente.
- **Control:** configuración aceptada, no prueba física.
- **No ensayado:** no se localizó ejecución individual defendible.

## 2. Catálogo vigente completo

| Preset | Hz | `mod_type` | baud | dev pasos | `rx_bw` | sync | Tecnología pretendida | Evidencia / estado actual |
|---|---:|---:|---:|---:|---:|---:|---|---|
| `gfsk_433_50k` | 433920000 | 1 | 50000 | 100 | `0x52` | `930B51DE` | propietario GFSK | Claim F10 10/10; evidencia primaria ausente; ruta física 433 no establecida |
| `gfsk_433_10k` | 433920000 | 1 | 10000 | 50 | `0x52` | `930B51DE` | propietario GFSK | Claim histórico familiar/marginal; sin evidencia individual suficiente |
| `fsk_433_50k` | 433920000 | 0 | 50000 | 100 | `0x52` | `930B51DE` | propietario FSK | Claims contradictorios 1/10 y posterior 10/10; sin transcript primario localizado |
| `ook_433_4k8` | 433920000 | 2 | 4800 | 0 | `0x4C` | `930B51DE` | OOK/ASK raw | Claims 0/10 y posterior 10/10; lock OOK + ruta 433 incierta |
| `ook_433_2k4` | 433920000 | 2 | 2400 | 0 | `0x4C` | `930B51DE` | OOK/ASK raw | Claim 0/10; lock OOK + ruta 433 incierta |
| `msk_433_50k` | 433920000 | 4 | 50000 | 100 | `0x52` | `930B51DE` | MSK pretendido | Control/claim; valor reservado en el comando actual |
| `4fsk_433_50k` | 433920000 | 5 | 50000 | 100 | `0x52` | `930B51DE` | 4-FSK pretendido | Control/claim; valor reservado |
| `4gfsk_433_50k` | 433920000 | 6 | 50000 | 100 | `0x52` | `930B51DE` | 4-GFSK pretendido | Control/claim; valor reservado |
| `gfsk_868_50k` | 868000000 | 1 | 50000 | 100 | `0x52` | `930B51DE` | propietario GFSK | OTA reciente 10/10 raw |
| `gfsk_868_100k` | 868000000 | 1 | 100000 | 150 | `0x56` | `930B51DE` | propietario GFSK | OTA reciente 10/10 raw |
| `ook_868_4k8` | 868000000 | 2 | 4800 | 0 | `0x4C` | `930B51DE` | OOK/ASK raw | Claim F10 10/10 sin raw; RF-05 2026-10-20 `NOT RUN` |
| `msk_868_50k` | 868000000 | 4 | 50000 | 100 | `0x52` | `930B51DE` | MSK pretendido | Control/claim; valor reservado |
| `wireless_mbus_s_868` | 868300000 | 1 | 32768 | 75 | `0x52` | `543D0000` | PHY parecido a W-MBus S | OTA reciente 10/10 raw; no acredita EN 13757 |
| `wireless_mbus_t_868` | 868950000 | 1 | 100000 | 200 | `0x57` | `543D0000` | PHY parecido a W-MBus T | OTA reciente 10/10 raw; no acredita EN 13757 |
| `wireless_mbus_c_868` | 868950000 | 1 | 100000 | 180 | `0x57` | `543D0000` | PHY parecido a W-MBus C | OTA reciente 10/10 raw; no acredita EN 13757 |
| `wireless_mbus_n_169_2k4` | 169450000 | 1 | 2400 | 154 | `0x44` | `543D0000` | W-MBus N pretendido | Control solamente; silicio/ruta incompatibles o no demostrados |
| `wireless_mbus_n_169_4k8` | 169450000 | 1 | 4800 | 154 | `0x44` | `543D0000` | W-MBus N pretendido | Control solamente; silicio/ruta incompatibles o no demostrados |
| `4fsk_868_50k` | 868000000 | 5 | 50000 | 100 | `0x52` | `930B51DE` | 4-FSK pretendido | Control/claim 10/10; valor reservado, modulación no demostrada |
| `4gfsk_868_50k` | 868000000 | 6 | 50000 | 100 | `0x52` | `930B51DE` | 4-GFSK pretendido | Control/claim 10/10; valor reservado |
| `gfsk_915_50k` | 915000000 | 1 | 50000 | 100 | `0x52` | `930B51DE` | propietario GFSK | OTA reciente 10/10 raw |
| `gfsk_902_50k` | 902200000 | 1 | 50000 | 100 | `0x52` | `930B51DE` | propietario GFSK | OTA reciente 10/10 raw |
| `gfsk_2440_250k` | 2440000000 | 1 | 250000 | 320 | `0x59` | `930B51DE` | propietario 2.4 GHz | Claim abril 10/10; octubre 0/10; ejecución actual cae a 868 MHz |
| `gfsk_2440_50k` | 2440000000 | 1 | 50000 | 100 | `0x52` | `930B51DE` | propietario 2.4 GHz | Claim abril 10/10; octubre 0/10; ejecución actual cae a 868 MHz |
| `sidewalk_915_fsk_50k` | 915000000 | 0 | 50000 | 100 | `0x52` | `930B51DE` | capa FSK parecida a Sidewalk | OTA reciente 10/10 raw; no es stack Sidewalk |
| `sidewalk_915_fsk_250k` | 915000000 | 0 | 250000 | 200 | `0x5A` | `930B51DE` | capa FSK parecida a Sidewalk | OTA reciente 10/10 raw; no es stack Sidewalk |
| `wisun_915_fsk_50k` | 902200000 | 0 | 50000 | 100 | `0x52` | `930B51DE` | PHY parecido a Wi-SUN FAN | 60/60 recuperación + 10/10 reverso; raw, no Wi-SUN |
| `mioty_868_tsunb` | 868000000 | 0 | 396 | 1 | `0x4A` | `930B51DE` | MIOTY TS-UNB pretendido | Claim F29.b 0/10 sin transcript primario; no existe TS-UNB |
| `wisun_915_fsk_100k` | 902200000 | 0 | 100000 | 150 | `0x56` | `930B51DE` | PHY parecido a Wi-SUN FAN | OTA reciente 10/10 raw |
| `wisun_915_fsk_150k` | 902200000 | 0 | 150000 | 200 | `0x5A` | `930B51DE` | PHY parecido a Wi-SUN FAN | OTA reciente 10/10 raw |
| `wisun_915_fsk_200k` | 902200000 | 0 | 200000 | 200 | `0x5A` | `930B51DE` | PHY parecido a Wi-SUN FAN | OTA reciente 10/10 raw |
| `wisun_915_fsk_300k` | 902200000 | 0 | 300000 | 200 | `0x5E` | `930B51DE` | PHY parecido a Wi-SUN FAN | OTA reciente 10/10 raw |

## 3. Clasificación histórica separada

### Éxito OTA con evidencia reciente suficiente para entrega raw

`wisun_915_fsk_50k`, `gfsk_902_50k`, `wisun_915_fsk_100k`, `wisun_915_fsk_150k`, `wisun_915_fsk_200k`, `wisun_915_fsk_300k`, `gfsk_915_50k`, `sidewalk_915_fsk_50k`, `sidewalk_915_fsk_250k`, `gfsk_868_50k`, `gfsk_868_100k`, `wireless_mbus_s_868`, `wireless_mbus_t_868` y `wireless_mbus_c_868`.

La afirmación válida es estrecha: intercambio de paquetes raw compatibles entre dos CatSniffer. No es conformidad Wi-SUN, Sidewalk o Wireless M-Bus.

### Aceptación de control solamente

La campaña EV-11 aceptó por control 27 presets, con sólo 18 salidas individuales preservadas y nueve resumidas. Esto cubre varias entradas excluidas, pero no demuestra frecuencia, modulación ni OTA.

### Parcial o contradictorio

- Familia 433: claims de 0/10, 1/10, 6–10/10 y posteriores commits 10/10 sin transcript crudo localizado.
- OOK 868: claim 10/10 de F10, pero sin artefacto primario; RF-05 reciente nunca se ejecutó.
- Proprietary 2.4: claim 10/10 de abril y 0/10 en octubre; la fuente actual demuestra un defecto que invalida una atribución a 2.4 GHz.

### Fallo histórico observado, pero no causal

- F29.b reportó `mioty_868_tsunb` 0/10 con FSK/GFSK y varios `rx_bw`; sólo se conserva el commit `6e34ef2`, no el transcript.
- La campaña del 8-oct reportó 24 corridas, 240 ACK y cero paquetes peer, incluidos 868/915/2440. Una campaña posterior recuperó 868–915, por lo que ese cero global no prueba incompatibilidad permanente.

### Bloqueado o no ejecutado

- RF-05/OOK 868: `NOT RUN` porque RF-01 y RF-03 quedaron bloqueados en `RX_STOP` y no se cumplieron los gates de recuperación.
- 169 MHz: nunca tuvo una OTA defendible; control solamente.
- Valores 4/5/6: no existe prueba válida de la modulación nombrada, aunque alguna entrega de bytes hubiera ocurrido.

## 4. Razones de exclusión por familia

| Familia | Motivo sustentado | Tipo de brecha |
|---|---|---|
| OOK 868 | patch genook no descargable de forma segura; requiere power-cycle y aislamiento | Firmware/lifecycle + validación |
| OOK 433 | lo anterior + front-end/antena 433 no demostrados | Firmware/lifecycle + hardware |
| FSK/GFSK 433 | silicio plausible; U4 de la revisión documentada sólo especifica 862–928 y 2400–2500 MHz | Evidencia de hardware |
| `mod_type=4/5/6` | el SDK define esos valores como reservados en `CMD_PROP_RADIO_DIV_SETUP_PA` | Contrato del driver |
| 169 MHz | fuera de las bandas de CC1352P7; firmware selecciona setup/handle 433; U4 tampoco cubre 169 | Hardware + implementación |
| Proprietary 2.4 | el runtime rechaza `frequency_hz>=1 GHz` en la función aplicada y cae a 868; no hay configuración SmartRF 2.4 activa | Implementación |
| MIOTY | sólo hay configuración FSK genérica; faltan TS-UNB, telegram splitting, temporización/frecuencia y reensamble | Protocolo + firmware/API |

Cuando la causa histórica exacta no puede establecerse —por ejemplo, los 0/10 de MIOTY sin transcript o los resultados contradictorios de 433— se conserva como **UNKNOWN** y no se atribuye al hardware.

