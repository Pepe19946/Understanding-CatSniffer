# Fase 1 — Preguntas abiertas y brechas de evidencia

## Procedencia y reproducibilidad

1. ¿Qué SHA/hash exacto de CC1352P7 y qué UF2/hash RP2040 estuvieron instalados en cada campaña? `1.0.0`, `FERALRF1` y `v3.1.0.1` no bastan para unir binario con source.
2. ¿Las placas de abril/mayo, octubre temprano, recuperación y 22-oct fueron las mismas revisiones físicas y mantuvieron antenas/ruta equivalentes?
3. No se localizaron los transcripts primarios del baseline de abril ni del claim F29 70/70. Se preservan como claims, pero no puede auditarse cada paquete, reset o exit.
4. El worktree actual contiene artefactos no versionados. Son inspectables localmente, pero su preservación/commit y hash de largo plazo no están garantizados.

## Contradicciones sin causa establecida

- F29: 70/70 reportado; campaña posterior del subconjunto: 0/70; recuperación: 60/60 y 10/10 por preset. **CAUSE NOT ESTABLISHED.** No se sabe si cambió firmware, estado RF, reset, switch, antena, timing o procedimiento.
- Propietarios: 240 ACK/0 peer el 8-oct frente a 14 presets positivos el 15–20 oct. **CAUSE NOT ESTABLISHED.** ACK no permite distinguir fallo de TX, frecuencia, ruta o RX.
- OOK 868: 10/10 histórico frente a 1/10 ACK, 0/10 peer y cleanup fallido el 22-oct. **CAUSE NOT ESTABLISHED.** Falta identidad binaria y observación física.
- Proprietary 2440: 10/10 histórico frente a 0/10 posterior y fallback a 868 confirmado en source actual. No se conoce la frecuencia efectiva de la prueba histórica.
- 433: 0/10, 1/10, 6–10/10 y 10/10 aparecen en documentación distinta. No hay evidencia primaria/hardware suficiente para reconciliarlos.

## Configuración efectiva

1. `SET_PROP_CONFIG` no devuelve resultado de `RF_open`, RadioSetup o `CMD_FS`; ¿qué error/completion real ocurrió por preset?
2. ¿Cuál es la frecuencia efectiva, especialmente en 2440 y 169? Source establece el defecto actual, pero no existe medición ni readback.
3. ¿Son válidas todas las combinaciones de rate word, `rx_bw`, deviation y overrides? Para 396 baud sólo se derivó ~395.203 baud teórico; no se midió.
4. ¿Cuál fue la potencia conducida/radiada y el throw eléctrico de U2? `Band: 0/1` es estado lógico RP2040, no verificación eléctrica.
5. ¿Qué revisión/BOM/front-end tienen las placas v3/P7? El esquema local antiguo no identifica por sí solo el hardware conectado.

## Framing, CRC y metadata

1. El host ve bytes de payload más bytes que parecen longitud/CRC. Falta una captura independiente que ubique preámbulo, sync, header, length y CRC en el aire.
2. `crc_ok=True` propietario depende del parser con layout cuestionado; falta verificarlo con frames deliberadamente válidos/erróneos y referencia externa.
3. RSSI positivo, canal 25/37 en Sub-GHz, LQI 0 y timestamps no calibrados muestran que la metadata no es confiable. Falta definir layout exacto del data entry TI y corregir el orden status/RSSI/timestamp.
4. Los marcadores repetidos de las campañas 15–20 oct no permiten estimar PDR por intento ni detectar duplicados. Sólo OP-MO del 22-oct usó secuencias únicas.
5. `RX_STOP` pierde/espera respuestas bajo tráfico asíncrono. La violación de multiplexado está confirmada en source, pero no se ha probado que sea la única causa de cada timeout.

## Protocolos nominales

- Wi-SUN: faltan framing 802.15.4g, FEC/modos aplicables, MAC/FAN, hopping, RPL, seguridad y peer independiente.
- Sidewalk: sólo existe FSK raw nominal; falta stack y peer. Sidewalk LR/LoRa corresponde al SX1262, fuera de esta ruta.
- Wireless M-Bus: falta construir/validar trama EN 13757, codificación, CRC y equipo independiente; el sync nominal no basta.
- MIOTY: faltan telegram splitting, coding/interleaving, sub-burst scheduling, distribución temporal/frecuencial, decoder/reassembly y base station. El 0/10 raw no evalúa TS-UNB.
- MSK/4-FSK/4-GFSK: falta elegir un comando/patch TI oficialmente compatible. Los valores actuales son reservados.
- BLE/IEEE 802.15.4 aparecen en campañas, pero no son presets `PROP_PRESETS`; su evidencia no valida por extensión ninguna fila propietaria.

## Preguntas que requieren estándares o fabricante en una fase posterior

1. Parámetros y framing exactos por rol/modo de Wi-SUN FAN, Sidewalk FSK, W-MBus S/T/C/N y MIOTY.
2. Comandos/patches TI compatibles con MSK y 4-(G)FSK para CC1352P7/SDK 8.30.01.01, si existen bajo el RF core usado.
3. Configuración SmartRF válida para proprietary 2.4 GHz y compatibilidad con el front-end de la revisión real.
4. Rango de silicio y cadena RF justificable para 169 MHz; la evidencia actual dice no ejecutar.
5. Reglas regulatorias aplicables a 868 y 902–928 en el lugar, potencia, duty cycle y antena del futuro ensayo.

## Medidas experimentales faltantes

- Frecuencia portadora y error.
- Modulación/deviation/rate/ancho ocupado.
- Potencia y armónicos.
- Estado GPIO/CTF/U2 y pérdidas de ruta.
- Captura independiente/decoder de framing y CRC.
- Prueba con peer ajeno a FeralRF para L4.
- Completion TX por intento, especialmente OOK y tasas extremas.
- Ensayo controlado que discrimine la causa de los históricos 0/70 y 0/240, no una repetición indiscriminada.

## Criterio de cierre

Estas preguntas no se resuelven por inferencia desde nombres de preset, ACKs o recepción entre dos copias de FeralRF. Deben permanecer abiertas hasta que exista fuente, artifact primario, medida o implementación independiente adecuada.

## Resoluciones parciales y nuevas preguntas (2026-10-23)

- Resuelto por fuente: el comando activo sólo define `modType` 0/1/2; la capacidad de silicio MSK/4-(G)FSK requiere otro setup/overrides exportados y no justifica 4/5/6.
- Resuelto por fuente: `format_conf=0` se vuelve `0x00A0`; las 31 filas usan framing TI variable con CRC, no framing del protocolo nominal.
- Corregido: TI sí publica soporte/configuración 169 MHz para CC1352P7 con RF design especial. Sigue abierto si la revisión CatSniffer, su frontend, antena, overrides y ruta de firmware pueden usarlo.
- Abierto: qué setup TI dedicado permite proprietary 2.4 GHz en CC1352P7 y cómo debe coexistir con la ruta IEEE/BLE existente.
- Abierto: qué mode IDs/perfiles exactos de Wi‑SUN, W‑M‑Bus y Sidewalk deben representarse y qué campos aporta el host frente al RF Core.
- Abierto: si MIOTY TS‑UNB puede implementarse con comandos/patches disponibles o exige procesamiento/baseband distinto.

## Changelog

- 2026-10-23: se actualizaron las preguntas para distinguir silicio, diseño RF, backend de driver, framing y pila.
