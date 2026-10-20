# RF-04 — Excepción metodológica de ejecución

**Fecha:** 2026-10-20
**Referencia:** `docs/RF_VALIDATION_RUNBOOK.md`, secciones 8, 10 y 11.
**Estado:** Preparación.

## Contexto

RF-01 quedó clasificado como BLOCKED tras reproducirse un timeout durante `RX_STOP`, pese a observarse recepción OTA de 10/10 paquetes experimentales en dos ejecuciones.

El RUNBOOK original exige RF-01 PASS antes de continuar la campaña.

## Excepción

Se permite evaluar RF-04 de manera independiente, exclusivamente como prueba de recepción pasiva IEEE 802.15.4 mediante KillerBee.

Esta excepción no modifica los resultados de RF-01 ni autoriza RF-02, RF-03 o RF-05.

## Precondiciones

* Red Zigbee autorizada: confirmada por el operador.
* Canal IEEE 802.15.4: 25.
* Antena de 2.4 GHz: confirmada previamente.
* Recuperación física posterior al fallo de RF-01: pendiente.
* Preflight previo a RF-04: pendiente.
* Dependencias KillerBee: pendientes de comprobar.

## Alcance

Evaluar recepción pasiva, integración `KillerBeeFeralRF`, generación PCAP y limpieza de recursos.

No realizar inyección, transmisión ni modificaciones de firmware.

## Limitaciones

El fallo de `RX_STOP` de RF-01 permanece sin diagnóstico causal. Cualquier anomalía durante `sniffer_off()` deberá registrarse sin atribuir automáticamente una causa común.

## Criterio de continuación

Ejecutar RF-04 únicamente después de confirmar las dependencias, completar la recuperación física y obtener un nuevo preflight PASS.
