# Fase 2 — Preguntas abiertas y registro de decisiones

## Decisiones

| ID | Decisión | Base | Impacto |
|---|---|---|---|
| DEC-01 | La matriz corregida 31/31 supersede columnas ambiguas de Fase 1 | source + estándar | separa intención y ejecución |
| DEC-02 | Los 14 L2 se conservan como **raw propietario**, no protocolo | artifacts históricos + framing | no se pierde evidencia; se reduce el claim |
| DEC-03 | `modType=4/5/6` no se interpreta como MSK/4-(G)FSK | header TI local | presets afectados se deprecan hasta backend real |
| DEC-04 | Capacidad del silicio y soporte del firmware se registran por separado | datasheet vs código | evita falsos “chip no soporta” y “chip soporta, luego funciona” |
| DEC-05 | P22/P23 son error de backend actual, no imposibilidad de 2.4 | guía TI + fallback en fuente | HOLD hasta setup dedicado |
| DEC-06 | 169 MHz es soporte especial/condicionado, no banda demostrada de CatSniffer | SmartRF/RF design vs frontend local | P16/P17 fuera del catálogo activo |
| DEC-07 | Elegibilidad de validación y de diagnóstico son columnas independientes | metodología | permite investigar fallos sin promover claims |
| DEC-08 | IEEE 802.15.4 y Zigbee se documentan como arquitecturas distintas | fuente + CSA | no usar “Zigbee” para frames IEEE raw |
| DEC-09 | MIOTY no es representable como un único preset PROP actual | ETSI TS‑UNB | P27 se retira como preset funcional |
| DEC-10 | Los nombres protocol-associated requieren sufijo `candidate` hasta L4 | estándares + evidencia | catálogo honesto y trazable |

## Preguntas abiertas priorizadas

| ID | Pregunta | Qué la resuelve | Dueño sugerido | Bloquea |
|---|---|---|---|---|
| Q01 | ¿Qué revisión/BOM tienen exactamente las dos placas históricas? | IDs, fotos, BOM y netlist concordantes | hardware | claims por banda |
| Q02 | ¿Cuál es la tabla real CTF/switch/PA/antena por banda? | esquemático + medición | hardware/RF | 433/868/915/2.4 |
| Q03 | ¿Qué setup CC1352P7 genera proprietary 2.4? | export SmartRF/SysConfig versionado | firmware/RF | reemplazo P22/P23 |
| Q04 | ¿Cómo se generan MSK y 4-(G)FSK en este dispositivo/SDK? | export TI, comando/patch/overrides | firmware/RF | P06–P08/P12/P18/P19 |
| Q05 | ¿Puede corregirse OOK sin power-cycle y con completion observable? | análisis TI/driver + diagnóstico instrumentado | firmware | P04/P05/P11 |
| Q06 | ¿Qué setting 169 y RF design se usarían? | export CC1352EM-XS169-XS24 + nueva placa | hardware/RF | P16/P17 |
| Q07 | ¿Qué versión de EN 13757‑4 y modos W‑M‑Bus son objetivo? | licencia/norma y product requirement | producto/protocolo | candidatos S/T/C/N |
| Q08 | ¿Qué Wi‑SUN FAN version, PHY mode ID y channel plan son objetivo? | especificación/PICS y certificación | producto/protocolo | P26/P28–P31 |
| Q09 | ¿Qué perfil Sidewalk y SDK autorizado están disponibles? | AWS Sidewalk SDK/docs/peer | producto | P24/P25 |
| Q10 | ¿TS‑UNB se implementará con stack TI/tercero o custom baseband? | arquitectura y prueba de factibilidad | producto/firmware | P27 |
| Q11 | ¿El objetivo IEEE es sniffing raw, MAC 802.15.4 o Zigbee? | requerimiento de producto | producto | alcance y nombres |
| Q12 | ¿Cómo exponer configuración efectiva/readback al host? | diseño de contrato | firmware/API | todos los L3 |
| Q13 | ¿Qué tolerancias y regulación aplican por país/laboratorio? | revisión legal/RF vigente | compliance | cualquier emisión futura |
| Q14 | ¿Cuál fue la causa de 0/240, 0/70 y de los resets C-mode? | evidencia primaria adicional o experimento discriminante | validación | explicación histórica |

## Riesgos si no se resuelven

- Claims de protocolo imposibles de defender.
- Emisión en banda distinta de la solicitada.
- Comparaciones PDR inválidas por ACKs, metadata o markers repetidos.
- Daño reputacional al presentar capacidad de silicio como feature implementada.
- Ensayos regulatorios o de hardware sobre una revisión incorrecta.

## Criterio para cerrar decisiones

Una pregunta se cierra sólo con una fuente primaria versionada, artefacto generado reproducible, medición independiente o requisito de producto explícito. “Funcionó entre dos CatSniffer” puede cerrar una pregunta L2, nunca una L3/L4.

