# Stage G — Recuperación OOK, preflight

## Resultado

**PASS de plano de control; recuperación RF aún pendiente.**

- Los dos grupos USB reaparecieron con la misma asociación Bridge/Shell.
- Ambos RP2040 regresaron a `Band: 0` después de retirar alimentación.
- Los contadores RP2040 `uart_overrun` y `ring_dropped` están en cero.
- Ambos CC1352 respondieron a `RADIO_INIT`, `GET_INFO` y `GET_STATS`.
- Las estadísticas RF observables reiniciaron a cero.
- Los cuatro procesos terminaron con código 0.

## Límite

El preflight demuestra recuperación del acceso y del plano de control después del power-cycle. No demuestra todavía que la ruta GFSK/868 haya recuperado entrega OTA. Para cerrar el objetivo B del ensayo OOK se requiere seleccionar `band2` y ejecutar OP-MO-07, sujeto a autorización RF explícita del operador.
