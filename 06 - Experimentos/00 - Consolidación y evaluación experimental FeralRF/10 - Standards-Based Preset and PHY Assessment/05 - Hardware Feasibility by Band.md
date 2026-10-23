# Fase 2 — Factibilidad de hardware por banda

## Criterio

La factibilidad exige simultáneamente: rango de silicio/setup, matching/front-end, switch o combiner, antena/conector, potencia/PA apropiado, firmware que seleccione esa ruta y cumplimiento regulatorio. Una frecuencia aceptada por UART no satisface ninguno de esos puntos por sí sola.

## Matriz

| Banda | Silicio/SDK | Evidencia de placa CatSniffer auditada | Ruta firmware | Estado | Acción previa a RF |
|---|---|---|---|---|---|
| 169 MHz | perfil TI especial documentado | frontend/antena 169 no demostrados; componente 0900 no cubre 169 | incoherente entre overrides 169 y modo 433 | no factible en configuración actual | variante RF 169 documentada + export TI + VNA/espectro |
| 287–351 MHz | banda listada por TI | no hay preset ni frontend demostrado | no evaluada | fuera de catálogo | diseño RF específico |
| 359–527 / 433 MHz | rango de silicio | ruta CatSniffer revisada no demuestra matching/antena 433; claims contradictorios | setup 433 dedicado existe | condicional, no habilitada para validación | identificar revisión/BOM/netlist, medir ruta y antena |
| 861–928 / 868–915 MHz | soportada y coincide con frontend 0900 publicado | componente Johanson 0900PC15A0036 asociado a 862–928; ruta exacta aún requiere revisión/BOM | backend activo y L2 raw | banda de mayor plausibilidad | verificar revisión, CTF/switch, potencia y regulación |
| 902–928 MHz | soportada | igual 868/915 | backend activo; 902.2 solicitado | plausible | channel plan, antena y reglas locales |
| 1076–1315 MHz | banda listada por TI | sin frontend/preset demostrado | ramas de `loDivider` existen | no establecida | diseño RF específico |
| 2360–2500 / 2.4 GHz | soportada por silicio | CatSniffer tiene ruta 2.4 para BLE/IEEE, pero no se ha probado su uso propietario | P22/P23 usan backend incorrecto y caen a 868 | hardware potencial; software no factible hoy | setup proprietary 2.4 dedicado + routing y medida |

## Revisión de placa y front-end

Los archivos locales contienen esquemáticos/PCB CatSniffer y referencias al balun/front-end Johanson `0900PC15A0036`, cuyo rango publicado es 862–928 MHz, además de rutas 2.4 GHz. También existen documentos históricos con revisiones/MCU distintos. No se debe mezclar un esquemático antiguo CC1352P1 con el target actual CC1352P7 ni asumir que dos placas de campaña comparten BOM sin identidad preservada.

El dictamen por banda es por ello deliberadamente asimétrico: 868/902/915 es **plausible y respaldado por L2 raw**, pero aún no medido físicamente; 433 es capacidad de silicio/firmware condicionada por frontend; 169 requiere variante especial; 2.4 es posible en el chip y en alguna ruta de placa, pero P22/P23 no la seleccionan.

## Requisitos de cierre

- fotografía/ID y revisión de cada placa;
- esquemático, PCB, BOM y población efectiva concordantes;
- tabla GPIO/CTF/switch y PA por banda;
- S11/pérdidas de ruta o documentación equivalente;
- frecuencia, potencia, ancho ocupado y armónicos medidos;
- antena/carga y límites regulatorios del lugar del ensayo.

Hasta entonces se usa “frecuencia solicitada”, nunca “frecuencia transmitida”.

