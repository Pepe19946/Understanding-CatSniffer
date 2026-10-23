# Matriz de compatibilidad de hardware y driver

## 1. Capacidad del silicio frente a la placa

TI especifica para CC1352P7 las bandas 287–351, 359–527, 861–1054, 1076–1315 y 2360–2500 MHz, y anuncia 2-(G)FSK, 4-(G)FSK, (G)MSK y OOK. Esto describe el transceiver, no el PCB completo ni el comando usado por FeralRF.

La revisión de esquema disponible (`hardware/CatSniffer.pdf`, rev. 1.0 de 2021) tiene tres limitaciones probatorias:

1. rotula U8 como CC1352P1, no P7;
2. U4 es un balun/filtro `0900PC15A0036`, especificado para 862–928 MHz y 2400–2500 MHz;
3. U2 selecciona entre LoRa, 2.4 GHz del CC y Sub-1 GHz del CC hacia una antena común; la selección depende del RP2040/Cat-Shell.

Los preflights históricos identifican hardware v3 con CC1352P7, pero no existe en este snapshot un esquema/BOM v3 vinculado de forma inequívoca a las dos placas. Por eso 433 MHz no se rechaza por capacidad del chip, sino por falta de evidencia de la ruta completa. A 169 MHz la incompatibilidad es más fuerte: está fuera de las bandas especificadas del P7 y del componente U4 documentado.

Referencias:

- https://www.ti.com/product/CC1352P7
- https://www.johansontechnology.com/datasheets/0900PC15A0036/0900PC15A0036.pdf

## 2. Contrato del driver exacto

FeralRF construye `rfc_CMD_PROP_RADIO_DIV_SETUP_PA_t`. En `rf_prop_cmd.h` del SDK incorporado, `modulation.modType` admite 0=FSK, 1=GFSK y 2=OOK; el resto está reservado. El mismo campo define la desviación en pasos controlados por `deviationStepSz`; FeralRF no lo cambia del valor base 0, equivalente a 250 Hz.

No se encontró validación de frecuencia, modulación, `rxBw`, tasa o consistencia. `SET_PROP_CONFIG` sólo rechaza longitud de payload incorrecta. Tampoco devuelve los estados de Radio Setup/FS dentro del ACK.

## 3. Matriz por familia excluida

| Preset/familia | Silicio CC1352P7 | Ruta CatSniffer | Comando/setup actual | Evidencia histórica | Compatibilidad resultante |
|---|---|---|---|---|---|
| `gfsk_433_50k`, `gfsk_433_10k`, `fsk_433_50k` | 433.92 está en 359–527; FSK/GFSK válidos | U4 de la revisión local no cubre 433; revisión v3 desconocida | setup dedicado `LP_CC1352P7_4`, handle 433 persistente | Claims marginales/contradictorios, sin raw primario | Firmware plausible; hardware completo no establecido |
| `ook_433_4k8`, `ook_433_2k4` | Banda/modulación plausibles | misma incertidumbre 433 | patch genook + overrides 433; lock hasta power-cycle | Claims 0/10 y 10/10 sin primarios | Bloqueo de hardware y recuperación |
| `ook_868_4k8` | Banda/modulación soportadas | U4 documentado cubre 862–928; antena/revisión deben confirmarse | `Prop0_modeOok`, genook y overrides OOK 868 | Claim 10/10; RF-05 no ejecutado | Compatible de forma condicional |
| MSK 433/868 | Silicio anuncia (G)MSK | 868 plausible; 433 incierto | `mod_type=4` reservado; sin patch/ruta alterna | Claims no verifican modulación | No soportado por contrato actual |
| 4-FSK/4-GFSK 433/868 | Silicio anuncia 4-(G)FSK | 868 plausible; 433 incierto | `mod_type=5/6` reservado | Claims no verifican modulación | No soportado por contrato actual |
| W-MBus N 169 | 169.45 fuera de bandas listadas | U4 no cubre 169 | selecciona `Prop0_mode433`; overrides 169 se escriben en la estructura compartida que no usa esa ruta | Control solamente | Incompatible/no defendible en hardware actual |
| `mioty_868_tsunb` | TI lista MIOTY; 868 soportado | ruta 868 plausible | FSK genérico; overrides normales; sin TS-UNB | 0/10 claim, sin evidencia primaria | Silicio plausible, implementación ausente |
| GFSK 2440 50k/250k | 2360–2500 y GFSK soportados | U4 documentado cubre 2.4; exige `band1` y antena 2.4 | defecto de frecuencia + overrides Sub-GHz; no setup 2.4 activo | claim 10/10 vs 0/10 posterior | Bloqueado por implementación, no por silicio |

## 4. Defecto actual de proprietary 2.4 GHz

La configuración parece aceptar 2440 MHz:

- `RadioIF_setPropConfig()` escribe 2440 en `Prop0_cmdFs`, `centerFreq` y `loDivider=0`;
- el parser responde ACK.

Pero antes de cada RX y TX, `RadioIF_applySub1ghzChannelConfig()` sólo acepta `frequency_hz` si es menor de 1000 MHz. Para PHY `PROPRIETARY_GFSK` y 2440 MHz toma el fallback de 868 MHz y reescribe frecuencia, centro y `loDivider=5`. Además, la rama de overrides para cualquier frecuencia >=861 MHz selecciona `Prop0_pOverrides`, documentados como configuración 868 del SDK.

Resultado: la ruta actual no ejecuta el preset a 2440 MHz. Esto explica por qué un ACK no tiene valor RF y hace que una nueva OTA sea improcedente hasta corregir la implementación.

El historial Git contiene una ruta dedicada `smartrf_prop_2_4ghz` en los commits F26 (`04d3011` a `0563ac0`). No está activa en HEAD. Su existencia demuestra una opción de diseño histórica, no que pueda restaurarse sin revisión; el refactor posterior, la coexistencia de handles y el front-end actual deben reevaluarse.

## 5. Problema adicional de 169 MHz

Para cualquier frecuencia menor de 861 MHz, `RadioIF_getPropMode()` selecciona `Prop0_mode433`. `RadioIF_setPropConfig()` sí elige `Prop0_pOverrides169` cuando la frecuencia es menor de 250 MHz, pero lo asigna a la estructura compartida `Prop0_cmdPropRadioDivSetup`; RX/TX de esa rama usan las estructuras `...433`. Por ello el override 169 no llega de forma coherente al setup efectivamente ejecutado.

Aunque se corrigiera esa inconsistencia, 169.45 MHz sigue fuera del rango especificado para CC1352P7. No debe ensayarse en este hardware.

## 6. Switch, antena y potencia

- `band1`/`band2` es estado del RP2040 que controla U2. Su texto de status no prueba eléctricamente la posición del switch.
- El firmware CC1352 no conoce ni confirma U2.
- Una antena “Sub-GHz” no prueba adaptación en 169, 433, 868 y 915 MHz.
- `set_power(0)` es una solicitud. Sin analizador/power meter no demuestra potencia conducida o radiada.
- Nunca debe conectarse directamente TX a RX por coaxial sin atenuación y presupuesto de potencia verificados.
- Cualquier prueba 868 MHz debe realizarse en entorno RF autorizado; de preferencia dentro de caja apantallada. La disponibilidad técnica del preset no sustituye cumplimiento regulatorio local.

## 7. Configuraciones fuera del diccionario

El comentario Sidewalk LR en `presets.py` reconoce que la modulación tipo LoRa corresponde al SX1262 de la placa y está fuera de `PROP_PRESETS`/CC1352. La presencia física del SX1262 no crea una ruta FeralRF equivalente: harían falta control del componente, selección U2, driver y API propios. No se considera un preset excluido y no se propone ensayo.

## 8. Matriz de tipo de brecha

| Familia | Configuración | Firmware | API | Protocolo | Hardware | Validación | Documentación |
|---|---|---|---|---|---|---|---|
| 433 FSK/GFSK | razonable | presente | suficiente para raw | no aplica | no establecido | insuficiente | claims contradictorios |
| OOK | razonable | presente, lifecycle frágil | recuperación mal expresada | sólo raw | 868 plausible; 433 incierto | insuficiente | reset vs power-cycle inconsistente |
| MSK/4FSK/4GFSK | inválida para comando | ausente | IDs engañosos | no aplica | plausible en silicio | no válida | capacidad anunciada excede implementación |
| 169 | incoherente | override no llega a ruta | acepta sin validar | nombre W-MBus no es stack | incompatible/no demostrado | control solamente | debe marcarse no soportado |
| 2.4 propietario | frecuencia sobrescrita | falta ruta dedicada actual | ACK no revela frecuencia efectiva | raw | plausible | resultados contradictorios | catálogo no advierte fallback |
| MIOTY | nominal, no validada | TS-UNB ausente | no expresa bursts/roles | ausente | plausible a 868 | 0/10 inconcluso | nombre sugiere más soporte del real |

