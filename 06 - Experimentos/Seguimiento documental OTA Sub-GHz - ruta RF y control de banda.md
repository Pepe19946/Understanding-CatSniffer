# Seguimiento documental OTA Sub-GHz — ruta RF y control de banda

## Propósito

Vincular los resultados literales de [[Reporte OTA Sub-GHz - GFSK 868 y 915 MHz]] con la investigación de capacidad, firmware y ruta física en [[Sub-GHz en CatSniffer V3 - CC1352P7, ruta RF y control de banda]]. Esta nota no cambia resultados EV ni añade una prueba física.

## Contexto preservado

| equipo | Bridge | LoRa | Shell | RP2040 informado | asociación de sesión, no hash instalado |
|---|---|---|---|---|---|
| CatSniffer #1 | COM33 | COM34 | COM35 | v3.1.0.0 | `8eaa84c` |
| CatSniffer #2 | COM88 | COM86 | COM87 | v3.1.0.1 | `c0cd5a4` |

Ambos Shell aceptaron una transición forzada `band1 → band2` y después mostraron `Band: 1`. En las fuentes comparadas, `Band: 1` es `SUBGIG_1`; `band2` intenta escribir CTF1/2/3=`0,0,1` desde RP2040 GPIO8/9/10. El status refleja un valor cacheado y no lee U2 ni los GPIO.

`Radio: LoRa`, `LoRa: initialized` y `LoRa Mode: Stream` pertenecen al estado del SX1262 controlado por el RP2040. No contradicen que el tráfico del puerto Bridge se dirija al CC1352P7 ni prueban que el SX1262 participe.

## Resultados literales que gobiernan este seguimiento

| # | preset y dirección | marcador | resultado |
|---:|---|---|---|
| 1 | `gfsk_868_50k`, COM33 → COM88 | `b10133008801` | `acks=10, hits=0, errors=0` |
| 2 | `gfsk_868_50k`, COM88 → COM33 | `b10288003301` | `NEG hits=0 errors=0`; `RESULT gfsk_868_50k COM88 -> COM33 acks=10 hits=0 errors=0` |
| 3 | repetición de #2 | `b10288003301` | `NEG hits=0 errors=0`; mismo `RESULT` |
| 4 | `gfsk_915_50k`, COM33 → COM88 | `b11133008801` | `NEG hits=0 errors=0`; `RESULT gfsk_915_50k COM33 -> COM88 acks=10 hits=0 errors=0` |

El stdout completo de la corrida #1 no está en el reporte suministrado. No aparecieron líneas `RX_ITEM` en las capturas positivas disponibles.

Status posterior a la primera prueba:

- #1: `Mode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.0, CC1352 FW: feralrf_cc1352 (custom)`; `uart_overrun=0`, `ring_dropped=0 bytes`.
- #2: `Mode: 0, Band: 1, Radio: LoRa, LoRa: initialized, LoRa Mode: Stream, FW: v3.1.0.1, CC1352 FW: feralrf_cc1352 (custom)`; `uart_overrun=0`, `ring_dropped=326 bytes`.

Los valores previos eran iguales. Por tanto, los 326 bytes no se atribuyen a estas corridas y la igualdad no prueba que ninguna otra cola haya perdido datos.

## Lectura del helper

`Invoke-FeralPresetOta` configura los dos radios, inicia RX, drena 3 s de control negativo, solicita diez TX con pausas de 100 ms y solo entonces lee 5 s del receptor. No hay lector Python de fondo.

- `acks=10`: diez solicitudes llegaron al punto de aceptación/encolado del firmware. La ACK precede a `RF_runCmd()`; no equivale a diez emisiones terminadas.
- `hits=0`: ningún `Packet` CRC-válido devuelto en la ventana positiva contenía el marcador. No equivale a cero paquetes totales ni a pérdida RF del 100 %.
- `errors=0`: no se devolvió un `RxStreamError` en esa iteración. No cubre todos los drops, CRC fallidos, errores no leídos ni cleanup suprimido.
- lista positiva vacía: es consistente con que no se imprimieran `RX_ITEM`; no localiza la causa.

El retraso de lectura puede importar porque hay buffers finitos, pero **no se demostró que causara pérdida**. En el CC hay tres entradas RF, ocho paquetes en la cola interna y 32 tramas en la salida; el RP2040 usa rings de 16 KiB; la capacidad posterior USB/OS no se determinó.

El `finally` intenta `stop_rx()` y `disconnect()` pero oculta excepciones. `stop_rx()` se reintenta; `disconnect()` solo cierra el puerto. Estado residual entre corridas es una hipótesis verificable, no un defecto probado.

## Hallazgos documentales que cambian la interpretación

1. El CC1352P7 observado sí tiene capacidad Sub-GHz nativa y la configuración GFSK no depende del SX1262.
2. Los presets 868/915 caen dentro del rango del silicio y del U4 publicado (`862–928 MHz`).
3. U2 conecta una de tres ramas a J1 y lo selecciona el RP2040, no FeralRF en el CC.
4. Los DIO28/29/30 usados por la callback SysConfig de LaunchPad están NC en el PCB v3.1 publicado. `set_phy()` no sustituye `band2`.
5. La transición forzada da evidencia de que el RP2040 ejecutó las escrituras previstas; `Band: 1` no prueba niveles, conmutación ni continuidad.
6. La rama dedicada `TX_20DBM_P/N` de U8 está NC en el diseño publicado. No se puede trasladar la etiqueta +20 dBm del silicio o tabla de firmware a potencia en J1.
7. El helper solicita 0 dBm para estas transmisiones; la potencia efectiva no fue medida.

## Preguntas abiertas priorizadas

| prioridad | pregunta | por qué discrimina |
|---:|---|---|
| 1 | ¿CTF1/2/3 tienen físicamente `0,0,1` durante la prueba? | separa estado cacheado de control real de U2 |
| 2 | ¿U2 presenta continuidad RF3–ANT en ese estado? | separa GPIO correcto de switch/montaje/ruta incorrectos |
| 3 | ¿hay energía GFSK a 868/915 MHz en J1, correlacionada con una solicitud? | separa problema de TX/ruta frente a RX/helper |
| 4 | ¿el frame en J1 tiene tasa, desviación, sync y marcador configurados? | comprueba configuración efectiva, no solo energía |
| 5 | ¿qué revisión/esquema/BOM corresponde a las unidades P7? | cierra la extrapolación desde el diseño P1 publicado |
| 6 | ¿qué binarios exactos están instalados? | las versiones/status informados no son hashes |
| 7 | ¿cambian métricas CC, colas de salida o contadores RP con lector concurrente? | prueba o refuta pérdida de observación en host |

## Siguiente observación propuesta

En una futura sesión autorizada, usar **una sola** solicitud `gfsk_868_50k` y correlacionar tres datos: CTF=`0,0,1`, continuidad U2 RF3–ANT y señal en J1. Si hay señal decodificable en J1 pero no aparece el marcador en el segundo CC, la investigación se concentra en RX/configuración/buffering. Si no hay señal en J1, sondear antes y después de U4/U2 localiza si la ausencia nace en el CC o en la ruta externa.

Una medida de energía por sí sola no prueba framing; una lectura CTF por sí sola no prueba conmutación; un ACK por sí solo no prueba RF.

## Presets posteriores

No debe declararse completa una campaña amplia mientras falten esos controles. En particular:

- excluir 169 MHz: queda fuera de las bandas del CC1352P7;
- separar 433 MHz: el silicio lo admite, pero U4 publicado no;
- tratar 868/902/915 como candidatos compatibles con el frente publicado, aún pendientes de validación proprietary;
- tratar 2440 como otra ruta (`band1`) y campaña distinta;
- no interpretar etiquetas W-MBus, Wi-SUN, Sidewalk o MIOTY como stacks interoperables.

## Estado

**Conclusión documental:** la capacidad de modulación reside en el CC1352P7; la ruta externa exige al RP2040/U2. Las cuatro corridas son negativas para detección del marcador bajo el procedimiento documentado, pero no localizan una causa raíz ni validan o invalidan físicamente el transmisor por sí solas.

**Resultado experimental canónico:** permanece en [[Reporte OTA Sub-GHz - GFSK 868 y 915 MHz]], [[Registro de validación FeralRF]] y [[Auditoría técnica de validación FeralRF - EV ejecutadas]].

