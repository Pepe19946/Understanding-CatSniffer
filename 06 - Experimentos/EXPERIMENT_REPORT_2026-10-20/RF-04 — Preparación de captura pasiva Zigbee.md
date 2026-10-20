# RF-04 — Preparación de captura pasiva Zigbee

**Fecha:** 2026-10-20
**Estado:** BLOCKED — Preparación del entorno.

## Objetivo

Validar la integración `KillerBeeFeralRF` mediante captura pasiva de una red Zigbee autorizada en el canal IEEE 802.15.4 número 25.

## Condiciones

* Sistema operativo: Windows.
* Python: 3.14.7, según revisión previa.
* Red Zigbee autorizada: confirmada por el operador.
* Canal: 25.
* Captura RF: no iniciada.

## Comando ejecutado

`python -m pip install -e '.\python[killerbee]'`

## Resultado observado

`pip` no encontró una distribución compatible con el requisito `killerbee>=3.0.0`.

La instalación terminó con `ERROR: No matching distribution found`.

## Interpretación

El bloqueo ocurrió durante la resolución de dependencias Python, antes de inicializar hardware o ejecutar una captura.

No constituye evidencia de funcionamiento incorrecto del CC1352P7 ni del adaptador KillerBee de FeralRF.

## Limitaciones

No se ha comprobado todavía si existe una instalación previa de KillerBee o una configuración particular del índice de paquetes.

No se ha evaluado la compatibilidad efectiva de KillerBee con Windows y Python 3.14.

## Próxima acción

Comprobar disponibilidad local de KillerBee y configuración de `pip`, sin modificar el entorno.

**PENDIENTE DE CONSOLIDACIÓN — REQUIERE AUDITORÍA POSTERIOR.**
