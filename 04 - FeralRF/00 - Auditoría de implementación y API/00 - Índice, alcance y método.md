# FeralRF — Fase II: auditoría de implementación y API

## Resultado ejecutivo

La implementación actual forma una cadena de tres procesadores: una biblioteca Python en el PC, el firmware stock del RP2040 como puente USB/UART y controlador de la ruta RF, y un firmware TI-RTOS en el CC1352P7 que ejecuta las operaciones de radio. FeralRF implementa control PHY, RX/TX de tramas crudas, modos de prueba, criptografía y jamming básico. No implementa pilas completas Zigbee, Thread, Wi-SUN, Sidewalk, Wireless M-Bus o MIOTY; sus presets y emuladores sólo producen configuración PHY o tramas preparadas.

La auditoría encontró, entre otros resultados, un defecto confirmado en el orden de los metadatos de RX propietario, valores de modulación reservados expuestos como MSK/4-FSK/4-GFSK, pérdida de eventos RX durante la espera de respuestas síncronas, semántica de ACK insuficiente para operaciones diferidas y una dependencia operativa no abstraída de Cat-Shell. Los detalles y su grado de certeza están en los informes enlazados abajo.

## Navegación

1. [[01 - Arquitectura real de FeralRF]]
2. [[02 - Inventario de implementación funcional]]
3. [[03 - Auditoría de API Python y contrato firmware]]
4. [[04 - Implementación frente a evidencia experimental]]
5. [[05 - Brechas funcionales y limitaciones]]
6. [[06 - Preguntas abiertas y entrega a fases III-IV]]

## Alcance y restricciones

Esta Fase II es una auditoría estática. Se inspeccionaron fuente, configuración de compilación, declaraciones, implementaciones, sitios de llamada, historial Git y evidencia ya preservada. No se compiló, instaló, flasheó ni ejecutó código; no se abrió ningún puerto serie ni se generó RF. No se modificó FeralRF, CatSniffer-Firmware, la consolidación de Fase I ni las notas preexistentes del Vault.

La autoridad para afirmar qué está implementado es el árbol fuente actual. La autoridad para afirmar qué ocurrió en hardware son los registros y reportes de Fase I. Un ACK sólo se interpreta según el punto exacto en que lo emite el handler; no se toma como prueba de radiación, frecuencia, modulación, potencia ni conmutación del front-end.

## Instantánea auditada

| Elemento | Revisión/estado observado | Uso en la auditoría |
|---|---|---|
| `FeralRF` | rama `feature/hardware-validation`; HEAD `fbff1b7f5077964418fb63a3fdf346a27c35a003` | Autoridad principal de implementación |
| SDK TI | submódulo `firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01` en `5b31d0a4903351e544546e23ef3330eaa4291ceb` (`lpf2-8.30.01.01`) | Definiciones RF y drivers exactos del árbol |
| `CatSniffer-Firmware` | rama `v3.x`; HEAD `c0cd5a45e019dbd14ed11d039aacb13f300e5731`; limpio al inspeccionarlo | Autoridad de Cat-Bridge, Cat-Shell, reset y selección de banda |
| `Understanding-CatSniffer` | rama `main`; limpio antes de crear este directorio | Evidencia histórica y destino exclusivo de escritura |

El árbol de FeralRF ya estaba modificado. Se preservaron sin cambio:

- `python/examples/killerbee_sniff.py` modificado localmente.
- `docs/NEXT_RF_VALIDATION_PLAN.md` y `docs/RF_VALIDATION_RUNBOOK.md` sin seguimiento.
- artefactos sin seguimiento bajo `experimental_artifacts/`.
- registros sin seguimiento bajo `validation_runs/next_rf_*`.

Esos archivos se trataron como evidencia o contexto cuando correspondía, nunca como parte inequívoca de HEAD. La versión Python declarada es `0.3.0` alpha (`FeralRF/pyproject.toml`). El dispositivo devuelve firmware `1.0.0`, capacidades `0x07` y serial constante `FERALRF1` desde `ControlTask_onGetInfo()`; esos campos no identifican un binario ni un commit (`FeralRF/firmware/cc1352/src/control_task.c:L443-L454`). La Fase I tampoco preservó hash del UF2 del RP2040. Por ello no se atribuye automáticamente el comportamiento histórico a HEAD.

## Método

La investigación siguió cuatro pasos:

1. Reconstrucción independiente de los ejecutables, tareas, transportes, estados y dependencias.
2. Trazado de cada familia funcional desde el método Python hasta el driver o servicio final y de regreso.
3. Comparación uno a uno de enums, builders, formatos, handlers, respuestas y errores.
4. Correlación posterior con Fase I y documentación, preservando contradicciones y procedencia.

Para cada hallazgo material se usa una de estas clases:

| Clase | Significado |
|---|---|
| Confirmado por fuente | El flujo o la violación se desprende directamente del código auditado |
| Confirmado experimentalmente | Existe evidencia de ejecución preservada, sin afirmar mecanismo interno |
| Fuente + experimento | El mecanismo de fuente explica directamente el síntoma observado |
| Sospechado por fuente | Patrón plausible, pero falta demostrar que ocurre en el binario/ejecución relevantes |
| Sospechado experimentalmente | Síntoma repetible sin mecanismo establecido |
| No verificable | Falta fuente, identidad de binario, instrumentación o medición independiente |

“Implementado” significa que existe una ruta ejecutable. No significa que el contrato sea completo ni que haya sido validado OTA. “PHY/raw” se distingue expresamente de MAC, red, seguridad y aplicación.

## Convención de referencias

Las referencias `ruta:Lx-Ly` corresponden a la instantánea indicada, con rutas relativas al workspace. Los símbolos son el ancla primaria si una edición futura desplaza líneas. Fuentes centrales:

- Python: `FeralRF/python/feralrf/radio.py`, `protocol.py`, `commands.py`, `enums.py`, `presets.py`.
- Firmware CC1352P7: `FeralRF/firmware/cc1352/src/{main_rtos,data_task,host_if_task,command_processor,control_task,radio_if,output_if,packet_queue,crypto_engine}.c`.
- Contrato C: `FeralRF/firmware/cc1352/include/protocol.h`, `radio_if.h`, `control_task.h`, `crypto_engine.h`.
- RP2040: `CatSniffer-Firmware/app/src/main.c`, `app/boards/catsniffer_v3.overlay`, `app/src/usbd_init.c`, `app/prj.conf`.
- SDK: `FeralRF/firmware/sdk/simplelink_cc13xx_cc26xx_sdk_8_30_01_01/source/ti/devices/cc13x2x7_cc26x2x7/driverlib/rf_prop_cmd.h`.
- Evidencia: [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/00 - Índice, alcance y método|Fase I]].

## Limitaciones

- No hay correspondencia verificable entre el string `1.0.0` observado y un commit de FeralRF.
- El análisis de los drivers usa el submódulo disponible; no prueba el contenido del binario instalado.
- No se midieron ondas, temporización, potencia ni posición física del RF switch.
- No se ejecutaron tests, incluso los potencialmente independientes de hardware.
- El esquema eléctrico no fue usado para confirmar todas las variantes de hardware; la selección de banda se describe desde el firmware RP2040.
- Las condiciones de concurrencia, pérdida de cola y temporización se clasifican como riesgos hasta obtener trazas que establezcan causalidad.

