# Ejecución experimental y resultados

## 1. Estado de ejecución

**No se autorizó ni ejecutó ninguna prueba RF en esta auditoría.**

No se enumeraron ni abrieron puertos serie, no se enviaron comandos Cat-Shell, no se cambió el switch de banda, no se reseteó hardware y no se inició RX o TX. La inspección se limitó a archivos, historial Git, SDK y documentación.

## 2. Registro de experimentos

| Experimento | Preset | Estado | Hardware | Comando | ACK TX | RX | Cleanup | Exit code | Resultado |
|---|---|---|---|---|---|---|---|---|---|
| RF-05/OOK-868 compuesto | `ook_868_4k8` | Planificado, no autorizado | No inspeccionado en esta fase | No ejecutado | N/A | N/A | N/A | N/A | **NOT RUN** |
| MIOTY raw/interoperabilidad | `mioty_868_tsunb` | No propuesto por falta de implementación | N/A | No ejecutado | N/A | N/A | N/A | N/A | **NOT RUN** |
| Proprietary 2.4 | `gfsk_2440_50k`, `gfsk_2440_250k` | Bloqueado por fallback a 868 | N/A | No ejecutado | N/A | N/A | N/A | N/A | **NOT RUN** |
| Modulación reservada | MSK/4-FSK/4-GFSK | Prohibido por contrato actual | N/A | No ejecutado | N/A | N/A | N/A | N/A | **NOT RUN** |
| W-MBus N 169 | ambos presets | Bloqueado por hardware/implementación | N/A | No ejecutado | N/A | N/A | N/A | N/A | **NOT RUN** |
| Familias 433 | FSK/GFSK/OOK | Bloqueadas por evidencia de hardware | N/A | No ejecutado | N/A | N/A | N/A | N/A | **NOT RUN** |

`N/A` indica que no existe medición, no un resultado cero.

## 3. Resultados históricos usados como contexto

| Familia | Observación histórica | Lectura válida | Límite |
|---|---|---|---|
| FSK/GFSK 868–915/902.2 | 14 presets con recepción raw reciente; control 60/60 y reverso 10/10 | entrega raw CatSniffer↔CatSniffer | no protocolos ni medición física |
| Campaña 8-oct | 240 ACK y 0 paquetes peer | comandos host aceptados, peer no observado | causa abierta; resultados posteriores positivos |
| MIOTY F29.b | claim 0/10 en varias configuraciones | no se observó marcador raw según commit | sin transcript, sin TS-UNB ni base station |
| OOK 868 | claim F10 10/10 | claim de entrega raw | sin evidencia primaria localizada; RF-05 reciente no corrió |
| OOK/FSK/GFSK 433 | claims contradictorios | implementación se intentó | sin aislamiento de ruta/antena y sin primarios suficientes |
| Proprietary 2.4 | claim 10/10 antiguo y 0/10 posterior | conflicto documental | source actual sobrescribe 2440 con 868 |
| MSK/4-FSK/4-GFSK | algunos claims 10/10 | a lo sumo bytes recibidos bajo estado no caracterizado | comando usa valores reservados; no prueba modulación |

Ningún claim histórico se promovió a PASS de protocolo.

## 4. Limitaciones pendientes

- No se identificó físicamente la revisión exacta de las placas disponibles.
- No se vinculó el binario CC1352 instalado con un hash reproducible del source HEAD.
- No hay medición de frecuencia, potencia, espectro, modulación ni posición eléctrica de U2.
- El esquema local P1/rev.1 no representa de forma demostrada las placas v3/P7.
- La causa de los 0/10 históricos de MIOTY y de la campaña 8-oct sigue inconclusa.
- Los claims F10 de OOK y modulaciones reservadas carecen de transcript primario localizado.
- No se evaluó cumplimiento regulatorio de una futura transmisión 868 en el lugar físico; es un prerrequisito del operador.

## 5. Gate abierto

El único candidato de esta auditoría es el experimento compuesto RF-05/OOK-868 de `05 - Preparación de validación y plan experimental.md`. Sigue `NOT RUN` hasta recibir aprobación explícita individual. Autorizarlo no autoriza ningún otro preset.

