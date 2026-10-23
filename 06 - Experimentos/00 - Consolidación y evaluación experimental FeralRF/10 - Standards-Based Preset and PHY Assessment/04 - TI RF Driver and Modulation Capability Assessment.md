# Fase 2 — Capacidad TI RF, driver y modulaciones

## Tres niveles que deben separarse

| Nivel | Evidencia | Conclusión |
|---|---|---|
| Silicio CC1352P7 | datasheet/product page | soporta 2-(G)FSK, 4-(G)FSK, MSK y OOK; también proprietary sub‑1 GHz y 2.4 GHz |
| Comando/patch/setup | headers y guía RF del SDK | cada estructura acepta un contrato concreto; el setup activo sólo enumera FSK/GFSK/OOK |
| Implementación FeralRF | `radio_if.c`, SysConfig generado | muta un setup sub‑GHz y pasa 4/5/6 sin validación; 2.4 cae a fallback; OOK usa patches con lock |

Por ello, “el chip soporta X” no significa “`modType=X` es válido”. La vía correcta para MSK/4-(G)FSK es generar/exportar una configuración TI válida para el dispositivo, RF design, banda y SDK concretos, identificar comandos/patches/overrides y sólo entonces exponer una API.

## Backend activo

`Radio.configure_prop()` serializa frecuencia, `mod_type`, symbol rate, desviación, `rx_bw`, sync y `format_conf`. `command_processor.c` deserializa, llama `RadioIF_setPropConfig()` y responde ACK sin readback del RF Core. `radio_if.c` muta `Prop0_cmdPropRadioDivSetup`, calcula `rateWord`, copia desviación/código BW, selecciona overrides y usa `CMD_PROP_TX/RX`.

Consecuencias:

- el ACK es de control, no de aplicación RF;
- 0/1/2 son interpretables como FSK/GFSK/OOK en el header auditado;
- 4/5/6 son reservados y deben rechazarse o mapearse a backends reales, nunca enviarse tal cual;
- `format_conf=0` no significa “sin formato”: restaura `0x00A0`;
- `rxBw` es un índice/código de configuración, no MHz/kHz documentados por el catálogo;
- la frecuencia no tiene readback y no fue medida.

## 2.4 GHz propietario

TI documenta que `CMD_PROP_RADIO_SETUP` es sólo 2.4 GHz y que `CMD_PROP_RADIO_DIV_SETUP` puede configurarse para distintas bandas cuando se genera adecuadamente. FeralRF conserva un setup/overrides sub‑GHz y, al iniciar RX/TX, `RadioIF_applySub1ghzChannelConfig()` rechaza `frequency_hz >= 1000 MHz` y selecciona 868 MHz. P22/P23 no son fallos del silicio: son un error de selección/configuración de backend.

## 169 MHz

Existe una tensión documental que no debe ocultarse: la tabla general de bandas del producto CC1352P7 no enumera 169 MHz, pero SmartRF Studio añadió una configuración CC1352P7 169 MHz N‑mode y la guía SysConfig enumera el RF design `CC1352EM-XS169-XS24`. La lectura prudente es soporte **condicionado a configuración y diseño RF especiales**, no una banda universal de cualquier placa CC1352P7.

El firmware actual calcula un `loDivider` y asigna overrides 169 en la estructura compartida, pero la selección de modo para `<861 MHz` toma la ruta/estructura 433; esa incoherencia impide afirmar que los overrides 169 usados sean los pretendidos.

## OOK

OOK usa patches genOOK y un modo separado. El propio código documenta que el patch no se descarga con seguridad y bloquea cambio de frecuencia/modo hasta power-cycle. El 22-oct produjo 1/10 ACK, 0/10 peer y error al parar RX. Es una implementación presente pero operacionalmente no validada.

## IEEE 802.15.4

La ruta usa `Ieee154_0_mode`, `CMD_IEEE_TX/RX`, channel-to-frequency 2405 + 5×(canal−11) MHz y buffers IEEE. Es arquitectura distinta de PROP. El host entrega PSDU; el RF Core maneja detalles PHY/MAC soportados por los comandos. FeralRF no añade NWK/APS/ZDO Zigbee.

## Decisiones de contrato recomendadas

- Rechazar explícitamente valores `mod_type` reservados en el backend actual.
- Representar backends como identificadores tipados, no enteros que aparentan ser universales.
- Hacer que la configuración devuelva setup/backend/banda/framing efectivos o un error verificable.
- No activar un preset si su RF design no coincide con la placa.
- Exportar de SmartRF/SysConfig y versionar los artefactos generados; no inferir overrides a mano.

