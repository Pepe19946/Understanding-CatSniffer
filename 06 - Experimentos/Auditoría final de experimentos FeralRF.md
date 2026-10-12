# Auditoría final de experimentos FeralRF

**Corte documental:** 11 de octubre de 2026.

**Naturaleza:** auditoría de evidencia y normalización mínima; no se ejecutaron radios, reset, flashing ni nuevos experimentos.

## Dictamen ejecutivo

La campaña OTA descrita por [[Guía enfocada OTA FeralRF - TX RX con dos CatSniffer]] llegó a un punto de cierre defendible: **completada con elementos `PARTIAL`/`INCONCLUSIVE` documentados**. Esto no significa que todas las funciones hayan pasado.

- IEEE RAW cumplió el criterio físico en ambas direcciones; FRAME lo cumplió en una dirección.
- BURST produjo RF atribuible, pero incumplió el conteo 5/5 y queda `PARTIAL`.
- `TX_CONTINUOUS interval_us=0` produjo recepción OTA repetida. `TX_STOP` quedó demostrado a nivel C de control/ACK; el cese RF inmediato permanece `NOT FULLY VALIDATED`.
- Proprietary incumplió el criterio de recepción en 24 corridas y 19 presets. La causa física/backend permanece `INCONCLUSIVE` porque ACK no prueba emisión y no se midieron CTF/U2/J1 ni energía RF.
- BLE era opcional; KillerBee fue intencionalmente diferido como integración host. Tier C proprietary quedó bloqueado o diferido por condiciones documentadas.

Las afirmaciones anteriores separan resultado y nivel de evidencia. Los stdout, comandos, payloads, timestamps y errores literales permanecen en los informes primarios.

## 1. Inventario

| Archivo original | Archivo final | Experimento / etapa | Posición | Ejecución | Evidencia | Resultado | Modificado | Razón |
|---|---|---|---:|---|---|---|---|---|
| `Checklist OTA FeralRF - dos CatSniffer.md` | [[Checklist OTA FeralRF - dos CatSniffer]] | Hoja de banco / plan | Plan | No es ejecución | F para resultados; D para criterios | N/A | Sí | Aclarar que casillas vacías no significan `NOT TESTED`; actualizar enlaces |
| `Informe de validación OTA FeralRF — transmisión y recepción con dos CatSniffer V3.md` | [[01 - OTA IEEE - RAW FRAME y BURST]] | Baseline, frente 2.4, RX IEEE, RAW bidireccional, FRAME y BURST | 1 | Ejecutado | A/B/C/D | RAW/FRAME `PASS-RF`; BURST `PARTIAL` | Sí | Nombre anterior no indicaba etapa ni orden; título normalizado |
| `Reporte_evidencia_OTA_CONTINUOUS_STOP_FeralRF_2026-10-08.md` | [[02 - OTA IEEE - CONTINUOUS y STOP]] | CONTINUOUS 250000/0 y controles STOP | 2 | Ejecutado | A/B/C | `PASS-RF` de repetición a intervalo 0; bloque `PARTIAL` | Sí | Nombre técnico/underscore ambiguo; etapa y orden explícitos |
| `Reporte OTA Sub-GHz - GFSK 868 y 915 MHz.md` | [[03 - OTA proprietary - GFSK 868 y 915 MHz]] | Primera campaña Tier A/Sub-GHz | 3 | Ejecutado | B/C; A para ausencia de entrega peer | Criterio OTA `FAIL`; causa física `INCONCLUSIVE` | Sí | Orden y alcance proprietary explícitos; H1 añadido |
| `Segunda campaña OTA proprietary FeralRF — Ampliación de presets y verificación del procedimiento.md` | [[04 - OTA proprietary - campaña ampliada]] | Tier A/B ampliado, proprietary 2.4 y control concurrente | 4 | Ejecutado | A/B/C/D | Criterio OTA `FAIL`; causa/backend `INCONCLUSIVE` | Sí | Nombre largo sin orden; H1 y etapa explícitos |
| `Validación dirigida de TX_STOP con RX continuo — FeralRF.md` | [[05 - OTA IEEE - CONTINUOUS STOP dirigido]] | RX continuo alrededor de STOP, ancla RAW, recovery | 5 | Ejecutado | A/B/C/E | `PARTIAL`; STOP control `PASS`, cese RF `NOT FULLY VALIDATED` | Sí | Relación con campaña previa y orden explícitos; referencia corregida |
| `Seguimiento documental OTA Sub-GHz - ruta RF y control de banda.md` | [[Seguimiento documental OTA Sub-GHz - ruta RF y control de banda]] | Síntesis documental posterior | Post-campaña; sin posición experimental | No añade ejecución | D/E | N/A; conserva el dictamen proprietary | Sí | Sólo enlaces actualizados |
| — | [[Auditoría final de experimentos FeralRF]] | Reconciliación y cierre | Auditoría | No añade ejecución | D sobre A–F previas | Campaña completada con `PARTIAL`/`INCONCLUSIVE` | Nuevo | Entregar inventario, cobertura, correcciones y cierre solicitados |

La posición 1 está respaldada porque el propio informe cierra la primera validación OTA y difiere CONTINUOUS y proprietary. Las etapas 2 y 3 pertenecen al corte del 8 de octubre; no hay timestamps suficientes para probar cuál ocurrió primero en tiempo de pared. Se conserva el orden mejor sustentado por la dependencia narrativa y se declara esta incertidumbre. La etapa 4 continúa explícitamente a la 3. La etapa 5 está fechada el 9 de octubre y continúa a la 2. El seguimiento Sub-GHz no recibe número porque es análisis documental, no una corrida.

### Auditabilidad y reproducción

| Registro | Suficiencia | Límite principal |
|---|---|---|
| [[01 - OTA IEEE - RAW FRAME y BURST]] | Alta para repetir comandos, puertos, PHY, canal, marcadores, ACK, bytes/FCS, CRC y conteos | Fecha exacta, distancia/geometría y hash del binario instalado no quedaron cerrados |
| [[02 - OTA IEEE - CONTINUOUS y STOP]] | Alta: comandos y stdout literal por fase, parámetros, ventanas y timestamps del receptor | Fecha exacta de ejecución no está en stdout; terminales sin reloj común; control negativo específico de `a402...` ausente |
| [[03 - OTA proprietary - GFSK 868 y 915 MHz]] | Media: helper completo, condiciones, varios comandos/stdout y consolidación aritmética | La primera corrida no conserva transcripción literal completa; fechas/distancias exactas y medición RF ausentes |
| [[04 - OTA proprietary - campaña ampliada]] | Alta para el procedimiento y resultados; incluye control concurrente con timestamps y estadísticas | Sin instrumentación RF, distancia exacta ni provenance del binario; ACK no da denominador de emisiones |
| [[05 - OTA IEEE - CONTINUOUS STOP dirigido]] | Alta: helpers completos, marcadores, stdout, tiempos host/device, repeticiones y recovery | Sin reloj RF independiente post-STOP ni traza de colas/wire; `0x90` sin decodificar |
| [[Checklist OTA FeralRF - dos CatSniffer]] | Sólo criterios/precondiciones | Campos sin llenar; no usar como registro de ejecución |
| [[Seguimiento documental OTA Sub-GHz - ruta RF y control de banda]] | Alta como síntesis cruzada | No contiene evidencia experimental nueva; remite a los informes primarios |

En conjunto, los registros permiten reproducir el procedimiento lógico y auditar los observables. No permiten reproducir con identidad binaria estricta ni hacer metrología de potencia, alcance, PER o espectro.

### Mapeo EV histórico

| Referencia histórica | Relación con la carpeta final |
|---|---|
| EV-05 | Fuente Zigbee declarada y RX IEEE local; permanece en `05 - Evaluación`, no es una campaña nueva de `06` |
| EV-10 | Configuración/control de ocho PHY; evidencia C, no RF |
| EV-11 | Barrido de presets a nivel control; distinto de las campañas OTA 03/04 |
| EV-12 | Evolucionó en varios experimentos independientes: RAW/FRAME/BURST en 01 y CONTINUOUS/STOP en 02/05; no se fuerza un único filename EV |
| EV-13 | CW/PRBS/STOP por control; fuera de la campaña OTA de bytes de `06` |
| EV-14 | Eventos/firma RX y mocks host; no equivale a la anomalía `RX_STOP/0x90` descubierta después |
| EV-20 | Objetivo IEEE OTA satisfecho por RAW bidireccional y FRAME de 01 |
| EV-22–27 | Antecedentes y objetivos proprietary reconciliados por 03/04; los nombres de protocolo siguen siendo PHY/raw |
| EV-29 | KillerBee diferido como integración host |

No se inventaron EV nuevos ni se cambió la numeración histórica. Los nombres finales priorizan las etapas realmente ejecutadas; las referencias EV se conservan como provenance en este mapeo y en los documentos canónicos.

## 2. Cobertura de la guía OTA

| Requisito mayor | Evidencia | ¿Ejecutado? | Esperado | Observado | Clasificación final |
|---|---|---|---|---|---|
| Identidad, host/control y puertos | [[01 - OTA IEEE - RAW FRAME y BURST]] | Sí | Dos endpoints inequívocos; INIT/STATS; Shell explícito | COM33/35 y COM88/87; ambos INIT/INFO; serial lógico común no identifica placa | `PASS-CONTROL` B/C, con provenance de binario limitada |
| Configuración de ocho PHY | [[EV-10 — Matriz de PHY por control]] | Sí, antes de esta campaña | Aceptación de selección/configuración | 8/8 secuencias aceptadas con reset entre filas | `PASS-CONTROL` C; no convierte los ocho PHY en PASS RF |
| Selección frente 2.4 GHz | [[01 - OTA IEEE - RAW FRAME y BURST]] | Sí | Forzar `band2→band1` en ambos | Shell aceptó transición; RAW/FRAME posteriores funcionaron OTA | Funcional en la condición ensayada; GPIO/CTF no medido |
| Baseline RX IEEE en ambas placas | [[01 - OTA IEEE - RAW FRAME y BURST]] | Sí | Paquetes físicos antes de TX propio | COM88: 16; COM33: 15; CRC válido reportado | `PASS` B; tráfico ambiental, no atribuido al peer |
| RAW COM33→COM88 | [[01 - OTA IEEE - RAW FRAME y BURST]] | Sí | Negativo 0; 10 ACK; 10 hits len8/CRC | Negativo 0; se enviaron accidentalmente 20; 20 ACK y 20 hits | `PASS-RF` A/C; desviación de cantidad documentada |
| RAW COM88→COM33 | [[01 - OTA IEEE - RAW FRAME y BURST]] | Sí | Negativo 0; 10/10 | Negativo 0; 10 ACK y 10 hits | `PASS-RF` A/C |
| FRAME COM33→COM88 | [[01 - OTA IEEE - RAW FRAME y BURST]] | Sí | Negativo 0; 10/10 | Negativo 0; 10/10 len8/CRC | `PASS-RF` A/C; contrato distinto, backend compartido |
| BURST 5×250 ms | [[01 - OTA IEEE - RAW FRAME y BURST]] | Sí | 5 hits | 1/5; repetición con host abierto 1/5; cero tardíos | `PARTIAL`; `FAIL` del criterio 5/5; número emitido/cause `INCONCLUSIVE` |
| BURST 20×50 ms | Guía/checklist | Intencionalmente no ejecutado | Sólo tras aprobar 5× | Primer punto no funcionó | Diferido conforme a la guía; no es omisión |
| CONTINUOUS 250000 µs | [[02 - OTA IEEE - CONTINUOUS y STOP]] | Sí | ≥5 hits | 1 hit; START/STOP ACK | `PARTIAL`; repetición positiva no validada |
| CONTINUOUS 0 µs | [[02 - OTA IEEE - CONTINUOUS y STOP]], [[05 - OTA IEEE - CONTINUOUS STOP dirigido]] | Sí | Múltiples hits | 30 hits iniciales; 43/131/137 en corridas dirigidas | `PASS-RF` de repetición A/C; tasa y conteo exactos no validados |
| TX_STOP, control posterior | [[02 - OTA IEEE - CONTINUOUS y STOP]] | Sí | STOP ACK; cero hits posteriores | ACK; dos ventanas posteriores de 5 s con cero marcador | `PASS-CONTROL`; ausencia compatible con cese, no prueba inmediata |
| TX_STOP con RX continuo | [[05 - OTA IEEE - CONTINUOUS STOP dirigido]] | Sí | Ordenar RF antes/después sin hueco | Host entregó durante segundos eventos con timestamps RF agrupados en 47–152 ms; ancla B post-STOP 0/3 y 0/10×2 | Cese inmediato `NOT FULLY VALIDATED`; método host-time invalidado |
| RAW y RX_STOP después de sesión cargada | [[05 - OTA IEEE - CONTINUOUS STOP dirigido]] | Sí | RAW posterior observable; RX_STOP ACK | RAW B ACK pero 0 observado; RX_STOP timeout, último tipo `0x90`; nueva sesión sin reset recuperó RAW 9/10 y ACK | Anomalía B/C de sesión/lifecycle/colas; poisoning persistente no demostrado |
| Tier A proprietary 868/915 y 2440 | [[03 - OTA proprietary - GFSK 868 y 915 MHz]], [[04 - OTA proprietary - campaña ampliada]] | Sí: 868 bidireccional, 915 exploratorio en una dirección y 2440 representativo/concurrente | 10/10 por corrida | Control aceptado; cero paquetes; ambas direcciones en presets representativos; control concurrente | `FAIL` del criterio OTA; causa física/backend `INCONCLUSIVE` |
| Tier B proprietary | [[04 - OTA proprietary - campaña ampliada]] | Parcialmente ejecutado y suficiente para detener el barrido | Ampliar sólo si aporta discriminación | 19 presets únicos acumulados; W-MBus/Wi-SUN/Sidewalk como PHY/raw; cero entrega | Criterio OTA `FAIL`; no se afirma interoperabilidad ni regresión |
| Tier C: 433/169/OOK/MIOTY | [[04 - OTA proprietary - campaña ampliada]] | No, por decisión documentada | Sólo con soporte/setup/recovery adecuado | 433 y 169 bloqueados; OOK y MIOTY diferidos | `BLOCKED` o `DEFERRED`, no `FAIL` experimental |
| BLE opcional | Guía; EV-13/14 fuera de esta carpeta | No como OTA peer marcado en esta campaña | Opcional | Sólo antecedentes de RX/control BLE en EV previos | `DEFERRED`; no bloquea cierre |
| KillerBee | Guía | No; intencionalmente diferido | Integración opcional | Adapter estático consume RX e invoca FRAME; sin nueva evidencia física | `DEFERRED / integración no validada`; no brecha core RF |
| CW/PRBS | [[EV-13 — CW PRBS y TX_TEST_STOP por control]] | Sí, sólo control, fuera de `06` | ACK y medición física si se reclama onda | ACK de CW/PRBS/STOP; sin instrumento | `PARTIAL` C; forma de onda/cese F |
| Eventos RF asíncronos | [[EV-14 — Eventos RF asíncronos y firma RX]] y [[05 - OTA IEEE - CONTINUOUS STOP dirigido]] | Sí, con alcances distintos | Inducir/correlacionar evento real | EV-14 no indujo error físico; 3 mocks host pasaron. La campaña STOP vio respuestas inesperadas `0x90`, sin semántica establecida | `NOT FULLY VALIDATED`; no inventar significado de `0x90` |
| Fuente Zigbee / IEEE ch25 | [[EV-05 — Primera observación RF IEEE]] | Sí como fuente declarada, no como protocolo decodificado | Observar RF IEEE de fuente conocida | 41/43/43 paquetes; sin bytes completos, acción disparada, correlación de dirección, parser externo o NWK/APS/ZCL | `PASS` B para RX IEEE local; atribución al dispositivo Medium; Zigbee superior no validado |

## 3. Problemas corregidos

- Los cinco informes ejecutados recibieron prefijos cronológicos y nombres por etapa; checklist y seguimiento documental quedaron fuera de la secuencia experimental.
- Se declaró la incertidumbre de orden de pared entre CONTINUOUS inicial y la primera campaña proprietary del 8 de octubre, en vez de inventar hora.
- Se marcó el checklist como plantilla, evitando interpretar campos vacíos como evidencia de no ejecución.
- Se actualizaron enlaces internos a los nombres finales.
- Matriz, registro, auditoría técnica, guía, Wiki y plan dejaron de presentar RAW/FRAME o la campaña OTA como pendientes.
- BURST quedó separado en `FAIL` del criterio 5/5 y `INCONCLUSIVE` para el conteo físico/causa.
- `TX_STOP` quedó separado en `PASS-CONTROL` y cese RF inmediato `NOT FULLY VALIDATED`.
- Proprietary quedó separado en `FAIL` del criterio de recepción OTA y causa/backend físico `INCONCLUSIVE`.
- KillerBee quedó `DEFERRED / integración no validada`, no como experimento core RF faltante.
- Se explicitó que una fuente Zigbee y recepción IEEE 802.15.4 no equivalen a decodificación ni interoperabilidad Zigbee.

## 4. Preguntas técnicas abiertas

- Estado eléctrico real de CTF1/2/3, selección de U2 y energía RF en J1 durante `gfsk_868_50k`.
- Si los comandos proprietary producen emisión y con qué frecuencia, tasa, desviación, sync, CRC y potencia.
- Por qué BURST y CONTINUOUS con intervalos positivos no cumplen el conteo observado; relación de `ControlTask_getTimeUs()`/SysTick con TI-RTOS.
- Latencia física exacta y cese RF de `TX_STOP`; el host no sirve como reloj RF bajo el buffering observado.
- Estado/lifecycle posterior a CONTINUOUS/STOP que coincide con RAW aceptado pero no observado dentro de la sesión.
- Ubicación, profundidad y política de las colas desde RF Core hasta Python; prioridad de ACK frente a eventos asíncronos.
- Significado real de la respuesta `0x90`; no fue establecido experimental ni estáticamente en esta auditoría.
- Procedencia exacta del binario CC1352P7 instalado y su vínculo con el commit de fuente analizado.
- Revisión/firmware RP2040 exactos y equivalencia de manejo de banda entre ambas placas.
- Medición física de CW/PRBS y cese de `TX_TEST_STOP`, si se desean reclamar esas formas de onda.

## 5. Conclusión Zigbee

### ¿Did we test Zigbee?

1. **Dispositivo Zigbee usado como fuente RF: sí, con atribución limitada.** EV-05 declara una fuente Zigbee conocida que generaba tráfico continuo en canal 25. No se preservaron modelo/identidad física, una acción Zigbee deliberadamente disparada ni una captura simultánea independiente.
2. **Recepción IEEE 802.15.4 de ese montaje: sí.** FeralRF recibió 41, 43 y 43 paquetes en tres ventanas sobre canal 25 y reportó metadata; el primer paquete de cada ventana tenía `crc_ok=True`. Esto demuestra recepción local IEEE 802.15.4 bajo ese montaje, no la identidad Zigbee de cada frame.
3. **Decodificación o interoperabilidad Zigbee: no.** No se documentaron bytes completos/PCAP, correlación de dirección/fuente, decodificación MAC, identificación NWK, parsing APS/ZCL, claves, Wireshark/KillerBee ni asociación/interoperabilidad.

Formulación aconsejada para el asesor:

> FeralRF fue probado recibiendo tráfico IEEE 802.15.4 en canal 25 en un montaje que contenía un dispositivo Zigbee declarado como fuente. Se observaron tramas raw y metadatos, pero la campaña no validó decodificación de capas Zigbee ni interoperabilidad Zigbee.

La misma restricción aplica a Thread, Matter sobre Thread y otros protocolos basados en IEEE 802.15.4: la recepción raw del PHY/MAC no distingue por sí sola el protocolo superior. Matter también puede usar otros transportes; nada en estas capturas establece Matter.

## 6. Recomendación de cierre

Clasificación recomendada: **campaña completada con elementos `PARTIAL`/`INCONCLUSIVE` documentados**.

El objetivo central —demostrar TX/RX físico de bytes conocidos mediante IEEE 802.15.4 con dos CatSniffer— quedó cumplido. BURST, STOP inmediato y proprietary no están plenamente validados, pero sus límites fueron caracterizados lo suficiente para justificar detener la repetición OTA indiscriminada. Los opcionales/diferidos no bloquean el cierre.

El siguiente trabajo debe cambiar de capa:

1. hardware/front-end: CTF/U2/J1 y observación de energía/forma de onda proprietary;
2. firmware: scheduler/timebase BURST/CONTINUOUS, cancelación/completion de STOP y estado posterior;
3. transporte/eventos: colas, `RX_STOP`, correlación y definición de `0x90`;
4. trazabilidad: binarios, revisiones y manifests instalados.

Sólo después de localizar una causa o modificar implementación conviene repetir un conjunto OTA mínimo como revalidación.

## Fuentes autoritativas usadas

- [[Guía enfocada OTA FeralRF - TX RX con dos CatSniffer]]
- [[Checklist OTA FeralRF - dos CatSniffer]]
- [[FeralRF - Matriz de pruebas]]
- [[Registro de validación FeralRF]]
- [[Auditoría técnica de validación FeralRF - EV ejecutadas]]
- [[FeralRF - Wiki técnica integral]]
- [[Estado y siguientes pasos]]
- los siete archivos que existían en `06 - Experimentos` al iniciar esta auditoría
