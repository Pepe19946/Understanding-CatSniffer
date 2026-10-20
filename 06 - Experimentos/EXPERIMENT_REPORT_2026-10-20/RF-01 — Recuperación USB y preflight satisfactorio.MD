# RF-01 — Recuperación USB y preflight satisfactorio

**Fecha:** 2026-10-20
**Estado:** Recuperación de comunicación PASS; recuperación RF pendiente.

## Contexto

RF-01 quedó bloqueado después de registrar 10/10 paquetes experimentales con CRC válido reportado en IEEE 802.15.4, canal 25. El fallo ocurrió durante `RX_STOP`.

Posteriormente, un preflight de recuperación no encontró los puertos COM requeridos.

## Aclaración del operador

El operador confirmó que había desconectado los CatSniffer y olvidado reconectarlos antes de ejecutar el preflight bloqueado.

Esta explicación es consistente con la ausencia de los puertos tanto en Windows como en `pyserial`.

## Recuperación observada

Después de reconectar los dispositivos:

* Windows volvió a enumerar COM33, COM34, COM35, COM86, COM87 y COM88.
* Cat-Shell respondió correctamente en ambos dispositivos.
* Cat-Bridge respondió correctamente en ambos dispositivos.
* Ambos RP2040 reportaron firmware `v3.1.0.1`, commit `c0cd5a4 (clean)`.
* Ambos CC1352P7 reportaron firmware `1.0.0`.
* Los contadores iniciales de recepción y errores estaban en cero.
* Se registró `DISCONNECT_OK` para ambos dispositivos.

El preflight finalizó con `RESULT=PASS` y código de salida `0`.

## Conclusión

La comunicación USB y las interfaces de control quedaron operativas después de reconectar ambos CatSniffer.

La ausencia previa de puertos se explica por la desconexión física declarada por el operador.

No se ha comprobado todavía si el timeout de `RX_STOP` desapareció después de la recuperación.

## Evidencia

`validation_runs/next_rf_20261020_091040/01b_preflight_recovery.jsonl`

**Nota de trazabilidad:** este archivo reemplazó el contenido del intento anterior bloqueado. La evidencia de aquel intento permanece en la salida compartida en el chat.

## Próxima acción

Ejecutar una sola vez RF-01 con `--only-control`, conservando el resultado en un archivo independiente.

No ejecutar RF-02 ni repetir RF-01 completo hasta evaluar el control de recuperación.
