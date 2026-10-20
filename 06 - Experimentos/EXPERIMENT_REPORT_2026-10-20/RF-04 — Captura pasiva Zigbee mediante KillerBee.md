# RF-04 — Captura pasiva Zigbee mediante KillerBee

**Fecha:** 2026-10-20
**Clasificación:** BLOCKED
**Causa operativa identificada:** Dependencia KillerBee no disponible en el entorno Python.

## 1. Objetivo

Validar la integración `KillerBeeFeralRF` de FeralRF mediante recepción pasiva IEEE 802.15.4 sobre una red Zigbee autorizada.

El experimento debía evaluar recepción de tramas, información CRC reportada, generación de PCAP y cierre de recursos.

## 2. Condiciones

* Hardware previsto: CatSniffer v3, RP2040 + CC1352P7.
* Dispositivo de captura previsto: CatSniffer #2.
* Puerto Cat-Bridge previsto: COM33.
* Banda: 2.4 GHz.
* Canal IEEE 802.15.4: 25.
* Red Zigbee autorizada: confirmada por el operador.
* Sistema operativo: Windows.
* Python: 3.14.7.
* Biblioteca FeralRF: instalación editable desde el repositorio local.

## 3. Procedimiento ejecutado

Se intentó instalar la dependencia opcional:

`python -m pip install -e '.\python[killerbee]'`

El instalador devolvió:

`ERROR: Could not find a version that satisfies the requirement killerbee>=3.0.0`

`ERROR: No matching distribution found for killerbee>=3.0.0`

Posteriormente se realizaron comprobaciones sin modificar el entorno.

## 4. Resultados

* `python --version`: Python 3.14.7.
* `python -m pip show killerbee`: paquete no encontrado.
* `importlib.util.find_spec('killerbee')`: `None`.
* `python -m pip config list`: sin salida.

No se inició una captura RF.

No se ejecutaron `KillerBeeFeralRF`, `pnext()` ni `sniffer_off()`.

No se generó un archivo PCAP.

## 5. Interpretación técnica

El entorno no satisface las dependencias necesarias para ejecutar RF-04 mediante el procedimiento establecido en el RUNBOOK.

La ausencia de KillerBee explica el bloqueo operativo.

No se ha determinado por qué el índice consultado por pip no ofrece una distribución que satisfaga el requisito declarado.

El resultado no demuestra un defecto de recepción IEEE 802.15.4, del firmware CC1352P7 ni de la implementación del adaptador KillerBee.

## 6. Evaluación

**Resultado: BLOCKED.**

La funcionalidad permanece sin validar, no reprobada.

## 7. Limitaciones

No se evaluaron instalaciones alternativas, otras versiones de Python ni otros sistemas operativos.

No se investigó el código de integración.

No se ejecutaron operaciones RF durante RF-04.

## 8. Seguimiento posterior

Durante la fase de evaluación y mejoras:

* Revisar la estrategia de distribución de la dependencia KillerBee.
* Determinar los entornos oficialmente compatibles.
* Preparar un procedimiento reproducible de instalación.
* Repetir RF-04 cuando exista un entorno funcional.

Estas actividades quedan fuera del alcance de la campaña actual.

## 9. Referencias

* `docs/RF_VALIDATION_RUNBOOK.md`, sección 8.
* Salidas de PowerShell registradas el 2026-10-20.
* Red Zigbee autorizada, canal 25, confirmada por el operador.

**PENDIENTE DE CONSOLIDACIÓN — REQUIERE AUDITORÍA POSTERIOR.**
