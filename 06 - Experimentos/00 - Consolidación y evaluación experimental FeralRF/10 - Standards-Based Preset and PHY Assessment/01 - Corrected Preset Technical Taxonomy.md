# Fase 2 — Taxonomía técnica corregida de presets

## Base auditada

- Checkout: rama `feature/hardware-validation`, commit `f2744e400f743df30499e20f03f156f08ad15bd8`.
- SDK local: `simplelink_cc13xx_cc26xx_sdk_8_30_01_01`, submódulo `5b31d0a4903351e544546e23ef3330eaa4291ceb`.
- Catálogo: 31 entradas de `FeralRF/python/feralrf/presets.py`.
- Trabajo documental: no se modificó firmware, Python, hardware ni configuración RF.

## Modelo obligatorio

Cada preset debe describirse en capas, sin usar una etiqueta como sustituto de otra:

1. **Identidad:** ID estable y clave Python.
2. **Intención:** tecnología nominal y modo/perfil nominal.
3. **Solicitud:** frecuencia, `mod_type`, symbol/data rate, desviación almacenada, `rxBw`, sync y `format_conf`.
4. **Interpretación:** unidades reales; aquí `deviation × 250 Hz` y `rxBw` continúa como código.
5. **Capacidad:** capacidad anunciada del CC1352P7, soporte del comando/estructura concreta y compatibilidad del RF design/placa.
6. **Ejecución:** PHY seleccionada, setup y comandos RF, frecuencia efectiva programada/medida.
7. **Framing:** campos host y campos generados/consumidos por RF Core.
8. **Pila:** MAC/red/aplicación realmente presentes.
9. **Evidencia:** experimento, nivel L0–L4, fuerza/procedencia y contradicciones.
10. **Decisión:** elegibilidad de validación y de diagnóstico por separado.

La matriz 31/31 que implementa este modelo está en `../09 - Preset Inventory and Historical Validation Audit/03 - Preset Validation Coverage Matrix.md` y forma parte normativa de esta Fase 2.

## Niveles y vocabulario

| Nivel | Afirmación máxima |
|---|---|
| L0 | existe/configuración declarada |
| L1 | comando aceptado o control ejecutado; no prueba RF física |
| L2 | payload identificable entre dos endpoints bajo la ruta real |
| L3 | frecuencia/modulación/framing verificados independientemente |
| L4 | interoperabilidad con implementación independiente y perfil estándar |

Se prohíbe promover evidencia: `ACK ≠ TX`, `TX aceptado ≠ frecuencia`, `payload raw ≠ PHY estándar`, `dos FeralRF ≠ interoperabilidad`, `IEEE 802.15.4 ≠ Zigbee`.

## Arquitecturas efectivas

| Selector | Backend | Framing RF | Stack visible | Relación con presets |
|---|---|---|---|---|
| `PHY.PROPRIETARY_GFSK` | `CMD_PROP_RADIO_DIV_SETUP` + `CMD_PROP_TX/RX` | propietario variable, sync 32-bit, CRC | ninguno | las 31 entradas |
| `PHY.IEEE_802_15_4` | `Ieee154_0_mode`, `CMD_IEEE_TX/RX` | IEEE 802.15.4 | no Zigbee | fuera del diccionario |
| BLE PHYs | comandos BLE5 | BLE | funcionalidades BLE parciales | fuera del diccionario |

## Consecuencia de diseño documental

Los presets protocol-associated deben presentarse como **RF candidates**, no como implementaciones de protocolo, hasta L3/L4. Los nombres incompatibles con su backend deben deprecarse o renombrarse; nunca se corrigen sólo con una nota de README.

