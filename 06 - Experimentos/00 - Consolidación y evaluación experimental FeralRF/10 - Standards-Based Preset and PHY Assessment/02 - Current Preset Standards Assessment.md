# Fase 2 — Evaluación del catálogo actual frente a estándares

## Resultado por preset

| IDs | Presets | Coincidencia defendible | Desajuste principal | Dictamen |
|---|---|---|---|---|
| P01–P05 | FSK/GFSK/OOK 433 | parámetros RF genéricos solicitados | frontend 433 no demostrado; OOK tiene lock | conservar sólo como experimental/condicionado |
| P06–P08 | MSK/4-FSK/4-GFSK 433 | nombre y capacidad general del silicio | valores 4/5/6 reservados en setup activo | nombre engañoso; deprecar hasta backend oficial |
| P09–P10 | GFSK 868 50/100k | GFSK genérica; L2 raw | frecuencia/modulación no medidas | conservar como baseline raw |
| P11 | OOK 868 4.8k | OOK existe en setup/patch | 0/10 peer reciente y cleanup fallido | conservar sólo para diagnóstico bloqueado |
| P12 | MSK 868 | sólo intención nominal | `modType=4` reservado | deprecar/remapear |
| P13–P15 | W‑M‑Bus S/T/C | bandas/tasas aproximan perfiles conocidos | framing TI, no EN 13757; modulación/sync/coding no acreditados | renombrar a candidatos RF; no claim W‑M‑Bus |
| P16–P17 | W‑M‑Bus N 169 | TI publica 169 N‑mode en diseño especial | desviación actual incoherente; ruta FW y frontend no probados | retirar del catálogo activo de esta placa |
| P18–P19 | 4-(G)FSK 868 | capacidad general de silicio | valores reservados en setup activo | deprecar/remapear |
| P20–P21 | GFSK 915/902.2 | GFSK genérica; L2 raw | falta medición física | conservar como baseline raw |
| P22–P23 | GFSK 2440 | silicio admite proprietary 2.4 | ruta actual cae a 868; setup/overrides erróneos | retirar del catálogo activo hasta backend dedicado |
| P24–P25 | Sidewalk FSK 915 | AWS confirma capa FSK 900 MHz | parámetros exactos/stack no acreditados; framing TI | renombrar candidatos RF; no claim Sidewalk |
| P26/P28–P31 | Wi‑SUN 50–300k | tasas dentro del rango FAN SUN FSK | FSK en vez de 2-GFSK de perfiles TI; desviaciones no mapean limpiamente; sin FEC/whitening/hopping/MAC | renombrar candidatos RF; no claim FAN |
| P27 | MIOTY TS‑UNB | frecuencia/rate nominales aproximados | no telegram splitting, coding, interleaving, bursts ni framing ETSI; 0/10 | retirar como preset MIOTY |

## Wi‑SUN en detalle

La descripción FAN pública sitúa la PHY en IEEE 802.15.4 SUN FSK con 2-FSK, tasas 50–300 kbps, channel spacing 200–600 kHz y FEC opcional. El datasheet TI ofrece ejemplos CC1352P7 2-GFSK: 50 kbps ±12.5/±25 kHz y 100 kbps ±25/±50 kHz según perfil/banda. El catálogo actual pide FSK no gaussiana y usa desviaciones 25, 37.5 o 50 kHz. Que un par FeralRF reciba esos paquetes sólo valida su PHY propietaria simétrica.

`wisun_915_*` programa 902.2 MHz. Esa frecuencia cabe en 902–928 MHz y puede ser un canal del plan NA; el problema taxonómico es el nombre “915” frente al valor real y la ausencia de channel-plan/mode-ID explícitos, no que 902.2 sea automáticamente inválido.

## W‑M‑Bus en detalle

OMS remite a EN 13757‑4 para modos S/T/C/N. Las claves actuales no implementan coding, bloques, CRC, sincronización y temporización de esa norma. P13–P15 son baselines raw con parámetros asociados; P16/P17 ni siquiera cuentan con una ruta 169 demostrada en la placa. No hay evidencia L4 con medidor/peer independiente.

## Sidewalk y MIOTY

AWS documenta Sidewalk como sistema con BLE y enlaces 900 MHz FSK/LoRa. La documentación pública consultada no basta para certificar que 50/250 kbps y las desviaciones del catálogo sean perfiles conformes. MIOTY/ETSI TS‑UNB divide telegramas codificados en radio-bursts distribuidos en tiempo/frecuencia; un único paquete `CMD_PROP_TX` con sync TI no lo reproduce.

## IEEE 802.15.4/Zigbee

La ruta IEEE dedicada puede enviar/recibir PSDU 802.15.4 y tiene evidencia separada. Zigbee requiere NWK, APS, ZDO, seguridad y perfiles/aplicación, ausentes. La documentación debe decir “IEEE 802.15.4 raw/frame” y no “Zigbee” salvo que se incorpore y valide una pila.

