# RF-03 — Preparación de transición IEEE 802.15.4 ↔ Sub-GHz

**Fecha:** 2026-10-20
**Estado:** PREPARACIÓN
**Referencia:** `docs/RF_VALIDATION_RUNBOOK.md`, sección 7.

## Objetivo

Evaluar la transición IEEE 802.15.4 canal 25 → Sub-GHz 902.2 MHz → IEEE 802.15.4 canal 25, sin reiniciar ni reinicializar el CC1352P7 entre etapas.

## Excepción metodológica

El RUNBOOK exige RF-01 y RF-02 PASS antes de RF-03.

RF-02 terminó en PASS funcional, mientras que RF-01 permanece BLOCKED por timeout reproducible durante `RX_STOP`.

El operador solicita continuar con RF-03 pese a esta condición incumplida.

La ejecución se tratará como una prueba exploratoria controlada, fuera de la secuencia original.

## Configuración prevista

* TX: CatSniffer #2, COM33 / COM35.
* RX: CatSniffer #3, COM88 / COM87.
* Primera etapa: IEEE canal 25, marcador `C3032501`.
* Segunda etapa: Sub-GHz 902.2 MHz, marcador `C3039002`.
* Tercera etapa: IEEE canal 25, marcador `C3032503`.
* Intentos por etapa: 10.
* Umbral mínimo: 9 paquetes válidos reportados por etapa.

## Riesgos conocidos

RF-01 presentó timeout reproducible durante `RX_STOP`.

RF-03 utiliza detención de RX entre etapas, por lo que existe riesgo de bloqueo antes de completar la transición.

No se investigará la causa durante esta campaña.

## Condiciones de ejecución

* Recuperación física inicial.
* Antenas y rutas de ambas bandas confirmadas.
* Preflight PASS.
* Una sola ejecución inicial.
* Detención ante el primer error.
* Conservación íntegra del registro JSONL.

## Criterio de evaluación

PASS únicamente si las tres etapas cumplen sus criterios, todas las detenciones RX se confirman y no existen errores de limpieza.

Las etapas completadas se documentarán individualmente aunque el resultado global sea BLOCKED.

## Limitaciones

Las frecuencias, potencias y metadatos RF no se validarán con instrumentación independiente.

No se realizarán modificaciones de firmware, biblioteca Python ni scripts.
