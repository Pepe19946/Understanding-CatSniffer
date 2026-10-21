# Matriz de capacidades de protocolos y presets

## Convenciones

- `desv.` es el valor de registro entregado a TI; en la configuración base su paso nominal es 250 Hz. Por ejemplo, `100 ≈ 25 kHz`. No constituye medición RF.
- `BW` conserva el byte codificado del comando, porque no hay lectura efectiva pública.
- `sync` es la palabra solicitada; el formato propietario base usa 32 bits.
- Todos estos presets llaman `configure_prop()`: comparten por defecto preámbulo 4 B, longitud variable, CRC del RF core, MSB-first, sin FEC y sin *whitening*, salvo que `format_conf` se altere por otra vía.
- “Soportado por campo” significa que el SDK local define el valor para `CMD_PROP_RADIO_DIV_SETUP`; no implica validación OTA.

## Inventario exacto actual (31 presets)

| Preset | MHz | `mod_type` | tasa bit/s | desv. | BW | sync | Evaluación |
|---|---:|---|---:|---:|---:|---|---|
| `gfsk_433_50k` | 433.920 | 1 GFSK | 50 000 | 100 | `0x52` | `930B51DE` | Genérico soportado por campo |
| `gfsk_433_10k` | 433.920 | 1 GFSK | 10 000 | 50 | `0x52` | `930B51DE` | Genérico soportado por campo |
| `fsk_433_50k` | 433.920 | 0 FSK | 50 000 | 100 | `0x52` | `930B51DE` | Genérico soportado por campo |
| `ook_433_4k8` | 433.920 | 2 OOK | 4 800 | 0 | `0x4C` | `930B51DE` | Modo representable; no validado |
| `ook_433_2k4` | 433.920 | 2 OOK | 2 400 | 0 | `0x4C` | `930B51DE` | Modo representable; no validado |
| `msk_433_50k` | 433.920 | 4 reservado | 50 000 | 100 | `0x52` | `930B51DE` | Nombre no realizado por el comando |
| `4fsk_433_50k` | 433.920 | 5 reservado | 50 000 | 100 | `0x52` | `930B51DE` | Nombre no realizado por el comando |
| `4gfsk_433_50k` | 433.920 | 6 reservado | 50 000 | 100 | `0x52` | `930B51DE` | Nombre no realizado por el comando |
| `gfsk_868_50k` | 868.000 | 1 GFSK | 50 000 | 100 | `0x52` | `930B51DE` | Genérico; evidencia OTA histórica |
| `gfsk_868_100k` | 868.000 | 1 GFSK | 100 000 | 150 | `0x56` | `930B51DE` | Genérico; evidencia histórica del catálogo aplicable sólo si está trazada |
| `ook_868_4k8` | 868.000 | 2 OOK | 4 800 | 0 | `0x4C` | `930B51DE` | Modo representable; no validado |
| `msk_868_50k` | 868.000 | 4 reservado | 50 000 | 100 | `0x52` | `930B51DE` | Nombre no realizado; usado erróneamente por helper W-MBus |
| `wireless_mbus_s_868` | 868.300 | 1 GFSK | 32 768 | 75 | `0x52` | `543D0000` | Parámetros RF nominales, no protocolo |
| `wireless_mbus_t_868` | 868.950 | 1 GFSK | 100 000 | 200 | `0x57` | `543D0000` | Parámetros RF nominales, no protocolo |
| `wireless_mbus_c_868` | 868.950 | 1 GFSK | 100 000 | 180 | `0x57` | `543D0000` | Parámetros RF nominales, no protocolo |
| `wireless_mbus_n_169_2k4` | 169.450 | 1 GFSK | 2 400 | 154 | `0x44` | `543D0000` | Parámetros RF nominales, no protocolo |
| `wireless_mbus_n_169_4k8` | 169.450 | 1 GFSK | 4 800 | 154 | `0x44` | `543D0000` | Parámetros RF nominales, no protocolo |
| `4fsk_868_50k` | 868.000 | 5 reservado | 50 000 | 100 | `0x52` | `930B51DE` | Nombre no realizado por el comando |
| `4gfsk_868_50k` | 868.000 | 6 reservado | 50 000 | 100 | `0x52` | `930B51DE` | Nombre no realizado por el comando |
| `gfsk_915_50k` | 915.000 | 1 GFSK | 50 000 | 100 | `0x52` | `930B51DE` | Genérico; evidencia OTA histórica |
| `gfsk_902_50k` | 902.200 | 1 GFSK | 50 000 | 100 | `0x52` | `930B51DE` | Genérico soportado por campo |
| `gfsk_2440_250k` | 2440.000 | 1 GFSK | 250 000 | 320 | `0x59` | `930B51DE` | Genérico; capacidad efectiva no validada aquí |
| `gfsk_2440_50k` | 2440.000 | 1 GFSK | 50 000 | 100 | `0x52` | `930B51DE` | Genérico; capacidad efectiva no validada aquí |
| `sidewalk_915_fsk_50k` | 915.000 | 0 FSK | 50 000 | 100 | `0x52` | `930B51DE` | No coincide con GFSK/framing Sidewalk |
| `sidewalk_915_fsk_250k` | 915.000 | 0 FSK | 250 000 | 200 | `0x5A` | `930B51DE` | Experimental; rol Sidewalk no establecido |
| `wisun_915_fsk_50k` | 902.200 | 0 FSK | 50 000 | 100 | `0x52` | `930B51DE` | Tasa nominal; no PHY 802.15.4g demostrado |
| `mioty_868_tsunb` | 868.000 | 0 FSK | 396 | 1 | `0x4A` | `930B51DE` | Marcado pendiente; 0/10 registrado |
| `wisun_915_fsk_100k` | 902.200 | 0 FSK | 100 000 | 150 | `0x56` | `930B51DE` | Tasa nominal; no protocolo |
| `wisun_915_fsk_150k` | 902.200 | 0 FSK | 150 000 | 200 | `0x5A` | `930B51DE` | Tasa nominal; no protocolo |
| `wisun_915_fsk_200k` | 902.200 | 0 FSK | 200 000 | 200 | `0x5A` | `930B51DE` | Tasa nominal; no protocolo |
| `wisun_915_fsk_300k` | 902.200 | 0 FSK | 300 000 | 200 | `0x5E` | `930B51DE` | Tasa nominal; no protocolo |

## Matriz de soporte por familia

| Familia | Configuración RF | PHY/*framing* | Construcción | Parseo | Procedimientos | Seguridad de protocolo | Interoperabilidad | Clasificación actual |
|---|---|---|---|---|---|---|---|---|
| PROP FSK/GFSK genérico | Presente | Presente: framing simple TI | Payload opaco | Payload + metadatos; F-01 | Fuera del alcance | Fuera del alcance | CatSniffer↔CatSniffer demostrada en subconjunto | Herramienta PHY/paquetes, con límites |
| OOK genérico | Presente | Implementación disponible, framing de paquete genérico | Bytes fijos opcionales | No protocolario | Ausente | Ausente | No demostrada | Implementado no validado |
| MSK/4-FSK/4-GFSK nominal | Valor aceptado | Ausente con comando actual: valor reservado | No | No | No | No | No | Etiqueta/preset no realizado |
| IEEE 802.15.4 | Presente | Presente; PHR/FCS por RF core | Bytes MAC del usuario; 2 ejemplos | LL/MAC mínimo | Filtrado/auto-ACK/asociación ausentes | Ausente | OTA entre CatSniffer, no tercero | PHY + transporte MAC |
| Zigbee/Thread | Base IEEE utilizable | Base parcial | Ausente | Ausente | Ausente | Ausente | No | No implementado |
| BLE advertising | Presente 1M/2M/Coded | RF core construye framing LL | Datos publicitarios; extensión 2M interna | LL superficial | Sólo advertising limitado | No pila SMP/enlace | No demostrada aquí | PHY + advertising limitado |
| Wireless M-Bus | Presets nominales | Framing específico ausente | Helper fijo incompleto | Ausente | Ausente | Ausente | No | Configuración nominal, no protocolo |
| Wi-SUN FAN | Tasas/frecuencia nominales | 802.15.4g no demostrado | Ausente | Ausente | Hopping/joining/red ausentes | Ausente | No/certificación no realizada | No implementado; presets experimentales |
| Sidewalk Sub-G FSK | Parcial y no coincidente (FSK vs GFSK) | PHR/FCS Sidewalk ausentes | Ausente | Ausente | Hopping/enlace/registro ausentes | Ausente | No | No implementado; etiqueta experimental |
| MIOTY TS-UNB | Nominal 396 baud | Ausente | Sin telegram splitting | Ausente | Ráfagas/hopping ausentes | Ausente | No; comentario registra 0/10 | Pendiente/no implementado |
| PT2262/EV1527/Hörmann | Frecuencia/OOK nominal | Temporización/codificación específica ausente | Patrón fijo | Ausente | Repetición/estado ausentes | N/A | No | Generador de estímulos |

## Cobertura experimental y temporal

La Fase I documentó un subconjunto histórico de 13 presets con resultados 10/10 en condiciones concretas. El diccionario actual tiene 31 entradas. No es válido propagar el resultado a:

- presets añadidos posteriormente;
- valores reservados 4/5/6;
- OOK no observado;
- tasas/frecuencias nuevas;
- etiquetas protocolarias Wi-SUN, Sidewalk, W-MBus o MIOTY;
- interoperabilidad con un equipo de tercero.

Las pruebas estáticas de `test_props.py` comprueban existencia, rangos y serialización. Al aceptar explícitamente 4/5/6 como supuestas modulaciones, reproducen el contrato Python, pero no prueban el contrato del RF core. Tampoco ejercen hardware.

## Requisitos para ascender el nivel de soporte

| Familia | Para PHY verificable | Para protocolo/interoperabilidad |
|---|---|---|
| Genérico PROP | *Readback* o reporte efectivo y medición de combinaciones | Definir externamente el formato de aplicación, si procede |
| OOK | Verificar forma de onda, tasa, framing y recuperación | Codificación/temporización y pruebas con receptor independiente |
| MSK/4-FSK/4-GFSK | Seleccionar comando/patch TI compatible y contrato de parámetros | Construir framing/procedimientos del protocolo objetivo |
| IEEE 802.15.4 | Precisar bytes RX/FCS y validar canales/errores | MAC según rol: ACK, CSMA-CA, asociación, seguridad; tercero independiente |
| BLE | Validación de advertising por PHY | Sólo si se decide: LL/GAP/GATT/SMP y rol concreto |
| Wireless M-Bus | PHY/modo EN 13757 exacto, codificación y CRC | Parser/construcción L/C/M/A/CI, seguridad y rol medidor/colector |
| Wi-SUN | PHY 802.15.4g y plan regional verificables | Perfil FAN, hopping, joining, IPv6/6LoWPAN/RPL y seguridad/certificación |
| Sidewalk | PHY GFSK/PHR/FCS y hopping conforme a región | Enlace, seguridad, registro y compatibilidad del ecosistema |
| MIOTY | Modulación y scheduler TS-UNB, codificación y ráfagas | MAC/seguridad/rol y prueba con infraestructura independiente |

Estas son condiciones para roles superiores, no una lista obligatoria de desarrollo. La decisión de producto se formula en [[05 - Requisitos y preguntas técnicas para desarrollo futuro]].
