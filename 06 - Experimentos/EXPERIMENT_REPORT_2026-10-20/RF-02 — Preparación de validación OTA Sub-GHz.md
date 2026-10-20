# RF-02 — Preparación de validación OTA Sub-GHz

**Fecha:** 2026-10-20
**Estado:** PREPARACIÓN
**Referencia:** `docs/RF_VALIDATION_RUNBOOK.md`, sección 6.

## Objetivo

Evaluar la comunicación OTA Sub-GHz en dirección CatSniffer #3 TX → CatSniffer #2 RX.

## Excepción metodológica

Se autoriza RF-02 pese a que RF-01 permanece en estado BLOCKED por un timeout reproducible durante `RX_STOP`.

Esta excepción permite obtener evidencia independiente sobre la comunicación Sub-GHz, sin modificar código ni investigar causas raíz.

No altera los criterios de aceptación de RF-02.

## Configuración prevista

* TX: CatSniffer #3, COM88 / COM87.
* RX: CatSniffer #2, COM33 / COM35.
* Preset: `wisun_915_fsk_50k`.
* PHY: `PROPRIETARY_GFSK`.
* Frecuencia efectiva: 902.2 MHz.
* Canal: 0.
* Potencia solicitada: 0 dBm.
* Marcador: `C2028802`.
* Intentos TX: 10.
* Umbral de aceptación: ≥9 paquetes con marcador y CRC válido reportado.

## Precondiciones pendientes

* Confirmación de antenas adecuadas para 902.2 MHz.
* Power-cycle completo de ambos dispositivos.
* Enumeración USB posterior.
* Preflight de RF-02 con resultado PASS.

## Limitaciones

El timeout de `RX_STOP` observado en RF-01 no ha sido diagnosticado.

La prueba de RF-02 no pretende determinar su causa.

La potencia de transmisión y la frecuencia efectiva no se medirán mediante instrumentación RF independiente.
