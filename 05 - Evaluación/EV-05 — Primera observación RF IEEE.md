EV-05 — Primera observación RF IEEE 802.15.4

Estado final: PASS-RF

1. Objetivo original de EV-05

La guía de validación define EV-05 como la primera prueba destinada a separar el funcionamiento del camino de control de una recepción RF física real.

Hasta este punto, respuestas como:

SET_PHY ACK
SET_CHANNEL ACK
RX_START ACK
RX_STOP ACK

demuestran que existe comunicación entre host y firmware y que determinados comandos son aceptados, pero un ACK no demuestra por sí mismo que el CC1352P7 haya recibido una señal por radio.

Por ello EV-05 introduce una fuente RF independiente y conocida:

PROTOCOL-DEVICE-ZIGBEE-CH25
        │
        │ IEEE 802.15.4 / canal 25
        ▼
Antena / front-end RF CatSniffer
        │
        ▼
CC1352P7
FeralRF
        │
        ▼
RP2040 / Cat-Bridge
        │
        ▼
COM88
        │
        ▼
FeralRF Python API

La fuente disponible genera tráfico Zigbee continuamente sobre IEEE 802.15.4 canal 25. No es controlada por FeralRF, por lo que sirve como fuente independiente para comprobar recepción física.

El procedimiento especificado por la guía es ejecutar tres veces:

python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30

registrando paquetes recibidos y, cuando estén disponibles, longitud, RSSI, LQI, CRC y timestamp. El criterio PASS-RF requiere al menos una trama CRC-válida atribuible y reproducible. La propia guía advierte que esto no requiere asociar FeralRF a la red Zigbee.

2. Condiciones de la prueba

La prueba se realizó sobre la misma unidad utilizada en las fases anteriores:

DUT:
CatSniffer V3
RP2040 + CC1352P7

Firmware RF:
FeralRF sobre CC1352P7

Interfaz host:
Cat-Bridge = COM88

Baudrate:
921600

PHY:
IEEE 802.15.4 (phy=4)

Canal:
25

Duración formal:
30 s por ejecución

Fuente RF:
Dispositivo Zigbee conocido con tráfico continuo
en IEEE 802.15.4 canal 25

La prueba se realizó además después de EV-04, donde habíamos ejecutado manualmente el ciclo boot\r\n → exit\r\n. Por ello EV-05 aporta indirectamente información sobre el estado posterior a esa recuperación.

3. Ejecución formal 1
Comando realizado explícitamente
python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
Acción realizada

El script establece comunicación con FeralRF por COM88, obtiene información del dispositivo, selecciona el PHY IEEE 802.15.4, configura el canal 25, inicia RX, mantiene la recepción durante 30 segundos y finalmente detiene RX.

Intención

Comprobar que el sistema no se limita a responder correctamente a comandos de configuración, sino que entrega al host paquetes procedentes de recepción RF real.

Resultado obtenido
FeralRF IEEE 802.15.4 Smoke Test
================================
port=COM88 baudrate=921600 phy=4 channel=25 duration=30.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY IEEE_802_15_4
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=41
[ OK ] first: ts=780546602us ch=25 rssi=-85 lqi=52 crc_ok=True len=52
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS
Análisis

El camino de control funcionó completamente:

RADIO_INIT / GET_INFO → OK
SET_PHY               → ACK
SET_CHANNEL           → ACK
RX_START              → ACK
RX_STOP               → ACK

Más importante para EV-05, durante RX se recibieron:

41 paquetes

y la primera trama reportada presenta:

Canal:     25
RSSI:      -85 dBm
LQI:       52
CRC:       True
Longitud:  52 bytes

Por tanto, esta ejecución proporciona evidencia RF positiva y no únicamente aceptación de comandos.

4. Ejecución formal 2
Comando

Se repitió exactamente:

python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
Resultado obtenido
FeralRF IEEE 802.15.4 Smoke Test
================================
port=COM88 baudrate=921600 phy=4 channel=25 duration=30.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY IEEE_802_15_4
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=43
[ OK ] first: ts=487488162us ch=25 rssi=-79 lqi=57 crc_ok=True len=5
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS
Análisis

Nuevamente se completó correctamente toda la ruta de control y se recibieron:

43 paquetes

La primera trama presenta:

Canal:     25
RSSI:      -79 dBm
LQI:       57
CRC:       True
Longitud:  5 bytes

La longitud de 5 bytes difiere de las tramas de 52 bytes observadas en otras ejecuciones. Esto no constituye por sí mismo una anomalía: el script la reportó como crc_ok=True y IEEE 802.15.4 permite longitudes variables.

Sin disponer en esta salida de los bytes de esa trama y sin decodificarla, no corresponde inferir qué tipo concreto de frame IEEE 802.15.4 o Zigbee representa.

5. Ejecución formal 3
Comando

Por tercera vez:

python examples\smoke_phy4_ieee154.py --port COM88 --channel 25 --duration 30
Resultado obtenido
FeralRF IEEE 802.15.4 Smoke Test
================================
port=COM88 baudrate=921600 phy=4 channel=25 duration=30.0s

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY IEEE_802_15_4
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] Read packets
[ OK ] packets=43
[ OK ] first: ts=540904925us ch=25 rssi=-83 lqi=56 crc_ok=True len=52
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS
Análisis

La tercera ejecución volvió a completar satisfactoriamente tanto el camino de control como el de recepción.

Se obtuvieron:

43 paquetes

con primera trama:

Canal:     25
RSSI:      -83 dBm
LQI:       56
CRC:       True
Longitud:  52 bytes

Por tanto, la recepción física no aparece como un resultado aislado de una sola ejecución.

6. Comparación de las tres ejecuciones formales

Los resultados pueden resumirse así:

Ejecución	Duración	Paquetes	RSSI primera trama	LQI	CRC	Longitud
1	30 s	41	−85 dBm	52	True	52 B
2	30 s	43	−79 dBm	57	True	5 B
3	30 s	43	−83 dBm	56	True	52 B

En las tres ejecuciones:

GET_INFO   → OK
SET_PHY    → ACK
SET_CHANNEL→ ACK
RX_START   → ACK
packets    → > 0
crc_ok     → True en la primera trama mostrada
RX_STOP    → ACK

Por tanto, el comportamiento requerido fue reproducible 3/3.

La cantidad de paquetes fue además relativamente cercana:

41
43
43

pero no debemos convertir esto en una caracterización de rendimiento. No se controlaron con ese objetivo variables como potencia del transmisor, distancia exacta, orientación, ocupación del canal, interferencias o tasa de generación de tráfico.

Lo mismo aplica al RSSI:

-85 dBm
-79 dBm
-83 dBm

La variación observada es perfectamente compatible con una prueba RF real, pero tres valores de la primera trama de cada captura no son suficientes para evaluar sensibilidad, precisión RSSI o calidad RF de CatSniffer.

7. Evidencia complementaria obtenida anteriormente

Antes de completar las tres ejecuciones formales ya se habían realizado otras observaciones.

Una ejecución anterior de 30 s produjo:

packets=48
first: ts=343358331us ch=25 rssi=-77 lqi=58 crc_ok=True len=52

Otra ejecución de 60 s produjo:

packets=100
first: ts=4859346us ch=25 rssi=-74 lqi=60 crc_ok=True len=51

También se realizó una observación en otro canal IEEE 802.15.4 donde previamente no se había detectado actividad mediante las herramientas disponibles; FeralRF tampoco encontró paquetes.

Esta última observación funciona como control negativo complementario, pero no debe elevarse al mismo nivel que las tres pruebas formales porque no conservamos en el registro actual el canal exacto ni stdout completo de aquella ejecución.

Las observaciones adicionales refuerzan la coherencia del resultado, pero el PASS-RF de EV-05 no depende de ellas.

8. Comparación con el comportamiento esperado

La guía establece que EV-05 debe distinguir entre:

ACK de RX_START

y:

recepción física de tramas RF

porque un firmware puede aceptar SET_PHY, SET_CHANNEL y RX_START y aun así no recibir correctamente nada por radio.

Nuestro resultado supera esa condición mínima.

No observamos únicamente:

[ OK ] RX_START ACK

sino posteriormente:

[ OK ] packets=41
[ OK ] packets=43
[ OK ] packets=43

acompañados de metadata RF y crc_ok=True.

Por ello el resultado concuerda con el criterio documental de PASS-RF: existe recepción CRC-válida reproducible procedente de una fuente independiente conocida en canal 25.

9. Qué recorrido queda validado

La evidencia permite sostener que, bajo las condiciones de esta prueba, funciona el recorrido:

Fuente IEEE 802.15.4 / Zigbee ch25
                │
                │ RF
                ▼
        Antena / RF path
                │
                ▼
           CC1352P7
                │
        recepción IEEE 802.15.4
                │
                ▼
             FeralRF
                │
                ▼
       UART / RP2040 Bridge
                │
                ▼
             COM88
                │
                ▼
       Python FeralRF host
                │
                ▼
 paquetes + RSSI + LQI + CRC

Además, el host pudo configurar el PHY y canal, iniciar RX, recibir paquetes y detener RX limpiamente tres veces.

Esta evidencia física es considerablemente más fuerte que concluir simplemente que "RX_START funciona".

10. Lo que EV-05 NO valida

El dispositivo transmisor utilizado genera tráfico Zigbee, pero FeralRF en esta prueba opera a nivel IEEE 802.15.4 raw.

Por tanto, el resultado no permite afirmar que FeralRF implemente o valide un stack Zigbee.

No hemos demostrado mediante EV-05:

asociación a una red Zigbee;
descubrimiento de dispositivos Zigbee;
procesamiento NWK;
APS;
ZCL;
descifrado Zigbee;
interoperabilidad completa de protocolo;
transmisión IEEE 802.15.4;
sensibilidad RF;
Packet Error Rate;
precisión absoluta de RSSI/LQI;
comportamiento en otros canales;
comportamiento en otros PHY.

La propia definición de EV-05 limita esta prueba a la primera observación RF IEEE reproducible y no exige asociación del DUT a la red.

Por ello, en el informe posterior deberíamos evitar una frase amplia como:

“FeralRF soporta Zigbee correctamente.”

La conclusión sustentada es más concreta:

FeralRF recibió de manera reproducible tráfico RF IEEE 802.15.4 en canal 25 procedente de una fuente Zigbee independiente conocida, entregando al host paquetes con metadata y CRC válido.

11. Relación con EV-04

EV-05 se realizó después de la caracterización de recuperación de EV-04.

En EV-04 habíamos provocado:

COM87 → boot\r\n
       ↓
FeralRF deja de responder

COM87 → exit\r\n
       ↓
FeralRF vuelve a responder

y posteriormente recuperamos correctamente INIT y GET_STATS.

Las ejecuciones actuales demuestran adicionalmente que, después de aquella recuperación, el sistema pudo:

SET_PHY
SET_CHANNEL
RX_START
recibir RF
RX_STOP

repetidamente.

Esto no resuelve KI-15: Radio.reset_device() continúa bloqueado porque calcula COM90 cuando el Shell funcional observado es COM87.

Lo que sí podemos afirmar es que no observamos un deterioro inmediato de la ruta IEEE 802.15.4 RX después de la recuperación manual realizada durante EV-04.

12. Estado final de EV-05
EV-05 — PASS-RF

Se cumplieron las tres ejecuciones formales de 30 segundos y las tres produjeron paquetes recibidos, con una primera trama reportada como CRC válida.

El resultado obtenido concuerda con el esperado por la documentación y no se observó durante estas ejecuciones ningún timeout, error de configuración, pérdida del puerto, estado RF irrecuperable ni fallo de RX_STOP.

La conclusión técnica consolidada para esta fase es:

En la CatSniffer V3 evaluada, con FeralRF ejecutándose sobre el CC1352P7 y utilizando COM88 como Cat-Bridge, se validó físicamente y de forma reproducible la recepción IEEE 802.15.4 en canal 25. Tres ejecuciones independientes de 30 s recibieron 41, 43 y 43 paquetes respectivamente, mostrando tramas con CRC válido y metadata RSSI/LQI. La prueba valida la ruta de recepción PHY IEEE 802.15.4 hasta la API Python del host; no valida un stack Zigbee ni caracteriza cuantitativamente el rendimiento RF.