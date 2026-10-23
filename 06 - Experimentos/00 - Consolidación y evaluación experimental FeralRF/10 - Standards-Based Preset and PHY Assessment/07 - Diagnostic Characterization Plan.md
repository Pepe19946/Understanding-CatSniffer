# Fase 2 — Plan de caracterización diagnóstica

Este plan define preguntas, observables y criterios. **No es un runbook, no contiene comandos y no autoriza transmisiones.** La ejecución futura requiere revisión regulatoria, instrumental y de seguridad separada.

## Principios

- Una prueba tiene una sola afirmación primaria.
- Se registra configuración solicitada y configuración efectiva/readback por separado.
- Toda afirmación física L3 exige instrumento independiente.
- Toda afirmación de protocolo L4 exige peer/decoder independiente.
- Un negativo útil debe discriminar hipótesis; repetir el mismo OTA simétrico no añade nivel.
- Validación positiva y diagnóstico negativo tienen gates distintos.

## Work packages propuestos

| WP | Pregunta | Casos | Observables mínimos | Criterio de cierre | Gate |
|---|---|---|---|---|---|
| D1 contrato | ¿el firmware rechaza configuraciones no interpretables? | P06–P08/P12/P18/P19 | status explícito, backend/setup reportado, ausencia de ACK falso | 4/5/6 rechazados o mapeados a backend versionado | sin RF |
| D2 frecuencia | ¿qué frecuencia programa/emite cada backend? | P09/P21 controles; P22/P23; futuro 169/433 | readback/trace de comando + analizador | error y frecuencia dentro de tolerancia documentada | carga/analizador, regulación |
| D3 modulación | ¿rate/deviation/BW coinciden? | baselines GFSK; candidatos TI | demodulación/IQ, occupied BW, error rate | valores dentro de tolerancia de fuente | setup oficial y señal conducida preferida |
| D4 framing | ¿qué bytes genera host/RF Core? | PROP e IEEE | captura independiente, preámbulo/sync/header/CRC | vector de bits reproducible | decoder/captura |
| D5 lifecycle OOK | ¿falla TX, RX, patch o cleanup? | P11 primero; P04/P05 sólo con HW 433 | eventos RF, completion, espectro, estado antes/después | causa discriminada y recuperación determinista | banco aislado/power-cycle |
| D6 2.4 propietario | ¿backend dedicado funciona sin fallback? | candidatos que reemplacen P22/P23 | setup ID, FS, frecuencia, espectro, peer raw | L3 antes de OTA ampliado | export CC1352P7 + ruta 2.4 verificada |
| D7 169 | ¿variante RF soporta N-mode? | futuro candidato N | BOM/RF design, S11, frecuencia, setting TI, decoder | sólo en hardware 169 trazable | no usar CatSniffer actual sin rediseño |
| D8 protocolo | ¿existe interoperabilidad? | W‑M‑Bus/Wi‑SUN/Sidewalk/MIOTY/Zigbee | peer independiente, vectors, logs de capas | L4 por perfil explícito | después de L3 |

## Predicciones por caso obligatorio

| Caso | Pregunta | Predicción de fuente | Medición independiente requerida | Evidencia que cuenta | Lo que no debe concluirse |
|---|---|---|---|---|---|
| OOK 868 / P11 | ¿hay emisión OOK y dónde falla el lifecycle? | setup OOK existe; patch queda cargado; evidencia reciente falla | espectro/demodulación OOK + eventos/completion + estado post-cleanup | portadora/envelope correcta y transición de estados correlacionada | que ACK sea OOK correcta o que un fallo invalide el silicio |
| MSK / P06/P12 | ¿qué hace `modType=4` reservado? | comportamiento no especificado por contrato | espectro/IQ conducido, frecuencia y eventos RF | observable físico etiquetado “valor reservado 4” | llamarlo MSK sin clasificador independiente |
| 4-FSK / P07/P18 | ¿se rechaza, conserva estado o emite otra señal? | comportamiento no especificado | analizador vectorial/IQ y readback/eventos | cuatro niveles de frecuencia medidos o diagnóstico alternativo explícito | inferir 4-FSK por ACK/delivery |
| 4-GFSK / P08/P19 | idem con valor 6 | comportamiento no especificado | IQ/demodulador capaz de distinguir shaping y cuatro niveles | medición de niveles y shaping con tolerancias | inferir 4-GFSK por nombre |
| proprietary 2.4 / P22/P23 | ¿la ruta actual emite 868, stale o nada? | código aplica fallback 868 antes de RX/TX | observación simultánea/segura de 868 y 2440 + trace setup | frecuencia inequívoca y correlación temporal | llamar 2.4 a un payload recibido sin medida |
| MIOTY-labeled / P27 | ¿qué hace el raw 396 actual? | un paquete TI; faltan splitting/coding/bursts | IQ/decoder del frame TI; comparación con vector ETSI | caracterización del raw actual y diferencias bit a bit | validación MIOTY/TS‑UNB |
| 169 / P16/P17 | ¿es viable en un diseño 169 adecuado? | setting TI especial existe; placa/ruta actuales no demostradas | VNA/ruta, frecuencia/espectro y decode en variante 169 | export, hardware y señal trazables | autorizar OTA o asumir soporte de CatSniffer actual |

## Diseño de evidencia

Cada resultado futuro debe conservar: commit/firmware hash; SDK y export RF; IDs físicos de placas; revisión/BOM; fecha/zona; banda/potencia/antena/carga y autorización; configuración solicitada y efectiva; instrumentos/calibración; stdout/stderr/exit; errores asíncronos; captura primaria; cleanup; criterio predeclarado; resultado `PASS/FAIL/BLOCKED/INCONCLUSIVE`.

Los payloads deben llevar `run_id`, dirección y secuencia única. El receptor debe deduplicar y separar paquetes previos. ACK del comando, completion RF, detección de energía, decode de frame y match de payload se cuentan como observables distintos.

## Priorización diagnóstica

1. **D1 sin RF:** elimina ACKs engañosos y valores reservados.
2. **D5 OOK 868:** existe un fallo reciente reproducible con alto valor causal.
3. **D2 sobre P22/P23:** confirma el fallback sin atribuir 2.4.
4. **D3/D4 sobre P09 y P21:** establece controles instrumentales antes de perfiles estándares.
5. **D6:** sólo después de un export 2.4 dedicado.
6. **D7/D8:** requieren hardware o peers externos y son líneas separadas.

## Exclusiones

No se propone barrer los 31 presets, repetir 10/10 con dos FeralRF, ejecutar 433/169 en la placa actual, ni emitir configuraciones reservadas “para ver qué pasa”. Tampoco se usa un decode IEEE 802.15.4 para inferir Zigbee.

