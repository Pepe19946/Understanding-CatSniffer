# EV-10 — Matriz de PHY por control

Registro canónico. PASS de control se mantiene separado de validación RF.

## 1. Contexto de evaluación

Se necesitaba conocer qué enumeraciones PHY respondían al ciclo de control antes de comparar sus capacidades por aire.

## 2. Objetivo de validación

Ejecutar smoke de control para los ocho PHY documentados, con recuperación entre filas.

## 3. Capacidad o requisito FeralRF evaluado

Selección PHY, configuración y RX_START/RX_STOP por API. Este smoke no ejecuta TX; la prueba de TX_RAW está en los presets de EV-11. [[Matriz de capacidades]], [[Protocolo y API Python]].

## 4. Precondiciones y condiciones

COM88, potencia configurada 0 dBm. Filas (PHY,canal): (0,37), (1,9), (2,37), (3,37), (4,25), (5,0), (6,0), (7,0). Reset manual boot/exit sobre COM87 entre filas. Hashes/versiones instaladas no fijados.

## 5. Resultado esperado

ACK/control sin error en cada fila y posibilidad de reiniciar el siguiente caso. El plan de RF independiente pertenece a EV-20/EV-21 y evaluaciones de bandas.

## 6. Procedimiento y ejecución cronológica

1. Ejecutar el smoke/phase 2 para cada una de las ocho parejas documentadas.
2. Entre filas, efectuar recuperación manual mediante Shell COM87 e INIT/estadísticas.
3. Registrar las ocho salidas y una recuperación final.
El canal 9 de BLE2M es el valor efectivamente usado; no se sustituye por 37 del ejemplo upstream.

## 7. Resultado observado

8/8 filas informadas como satisfactorias por control; recuperación registrada entre ellas. No se aportó observación de emisiones para estos ocho casos.

## 8. Evidencia

Comandos y salidas por PHY en §17, incluyendo resets. El smoke no establece completitud RF de TX ni ausencia de todo error asíncrono tardío.

## 9. Comparación entre lo esperado y lo observado

Cumple el objetivo de control de esta matriz. ACK de programación/configuración no equivale a una señal conforme al PHY elegido.

## 10. Interpretación técnica

Se acreditan rutas host/firmware bajo reinicio entre filas. No se acredita cambio de PHY sin reset, ni interoperabilidad BLE2M/Coded/sub-GH z/propietaria.

## 11. Anomalías, desviaciones y limitaciones

Reset entre filas impide evaluar continuidad de estado. Sin instrumento, bytes RF ni recepción independiente de TX. PHY7 por defecto no caracteriza todos los presets.

## 12. Resultado de la evaluación

PASS para las ocho filas de control; NOT FULLY VALIDATED para RF multi-PHY.

## 13. Confianza

High para los registros de control; Low para cualquier inferencia RF no medida.

## 14. Preguntas abiertas

¿Qué PHY emiten y reciben de forma interoperable? ¿Se puede alternar sin reset? ¿Hay eventos tardíos no consumidos por el smoke?

## 15. Acciones de seguimiento

Completar EV-20/21 y EV-41 con marcadores/eventos completos; no promover esta matriz a RF PASS.

## 16. Trazabilidad

Guía EV-10/20/21/41; [[FeralRF - Guía de validación experimental]]; [[FeralRF - Matriz de pruebas]]; [[Arquitectura FeralRF]]; [[EV-06 — Exclusión RX y TX y recuperación de estado]]; [[EV-11 — Control de presets propietarios]]; [[Pruebas y evidencia existente]].

Definición específica: [[FeralRF - Wiki técnica integral#10. Conceptos PHY necesarios]].



## 17. Notas originales preservadas y material pendiente

Fuente: `EV-10 — Matriz de PHY por control.md`. SHA-256 previo: `26FC534557428B37820224010D53D71B54CE97E5B57A3CEE0CF39418031866A4`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
EV-10 — Matriz de PHY por control
1. Objetivo teórico

EV-10 busca comprobar que FeralRF puede seleccionar y operar, a nivel de control, los valores de PHY 0–7 definidos por la interfaz. La guía prescribe examples\smoke_phase2.py, combinaciones concretas (PHY, canal), potencia de 0 dBm y un reset entre filas. El criterio es que las ocho configuraciones completen la secuencia de configuración y un ciclo breve de recepción; incluso cero paquetes recibidos es compatible con el resultado esperado.

Por tanto, la prueba no pretende demostrar que cada PHY haya recibido o transmitido correctamente por RF. Pretende recorrer:

PC / Python API
   ↓
RADIO_INIT + GET_INFO
   ↓
SET_PHY
   ↓
SET_CHANNEL
   ↓
SET_POWER
   ↓
RX_START
   ↓
RX_STOP

Un resultado satisfactorio demuestra que el firmware acepta la configuración correspondiente y puede entrar y salir del estado RX sin error observable por esta prueba.

Hay además una limitación específica: PHY 7 requiere configure_prop para definir una modulación propietaria concreta. Por ello, pasar smoke_phase2.py con PHY 7 no demuestra todavía una configuración RF propietaria funcional.

2. Procedimiento y evidencia obtenida
PHY 0 — canal 37, potencia 0 dBm

Comando realizado explícitamente:

python examples\smoke_phase2.py --port COM88 --phy 0 --channel 37 --power 0

Resultado obtenido, intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=0 channel=37 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Análisis: se obtuvo exactamente la secuencia funcional que EV-10 busca comprobar. SET_PHY, SET_CHANNEL, SET_POWER, RX_START y RX_STOP fueron aceptados. Resultado: PASS-control.

Reset PHY 0 → PHY 1

Se realizó el reset manual porque ya comprobamos en EV-04 que Radio.reset_device() no es fiable con el mapeo actual debido a KI-15. Se utilizó el Shell real, COM87.

Comando:

python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"

Resultado:

OPEN COM87
SENT b'boot\r\n'
CLOSED

Comando:

python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"

Resultado:

OPEN COM87
SENT b'exit\r\n'
CLOSED

Verificación:

python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

La recuperación fue correcta y COM88 volvió a aceptar INIT y GET_STATS.

PHY 1 — canal 9, potencia 0 dBm

Comando:

python examples\smoke_phase2.py --port COM88 --phy 1 --channel 9 --power 0

Resultado intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=1 channel=9 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Resultado: PASS-control.

Reset PHY 1 → PHY 2
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'boot\r\n'
CLOSED
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'exit\r\n'
CLOSED
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

Recuperación correcta.

PHY 2 — canal 37, potencia 0 dBm

Comando:

python examples\smoke_phase2.py --port COM88 --phy 2 --channel 37 --power 0

Resultado intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=2 channel=37 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Resultado: PASS-control.

Reset PHY 2 → PHY 3
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'boot\r\n'
CLOSED
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'exit\r\n'
CLOSED
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

Recuperación correcta.

PHY 3 — canal 37, potencia 0 dBm

Comando:

python examples\smoke_phase2.py --port COM88 --phy 3 --channel 37 --power 0

Resultado intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=3 channel=37 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Resultado: PASS-control.

Reset PHY 3 → PHY 4
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'boot\r\n'
CLOSED
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'exit\r\n'
CLOSED
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

Recuperación correcta.

PHY 4 — canal 25, potencia 0 dBm

Comando:

python examples\smoke_phase2.py --port COM88 --phy 4 --channel 25 --power 0

Resultado intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=4 channel=25 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Resultado: PASS-control.

Aunque PHY 4/canal 25 coincide con la configuración en la que EV-05 obtuvo recepción física IEEE 802.15.4, EV-10 por sí sola no añade evidencia RF, porque este script solamente exige que el flujo de control finalice correctamente.

Reset PHY 4 → PHY 5
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'boot\r\n'
CLOSED
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'exit\r\n'
CLOSED
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

Recuperación correcta.

PHY 5 — canal 0, potencia 0 dBm

Comando:

python examples\smoke_phase2.py --port COM88 --phy 5 --channel 0 --power 0

Resultado intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=5 channel=0 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Resultado: PASS-control.

Reset PHY 5 → PHY 6
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'boot\r\n'
CLOSED
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'exit\r\n'
CLOSED
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

Recuperación correcta.

PHY 6 — canal 0, potencia 0 dBm

Comando:

python examples\smoke_phase2.py --port COM88 --phy 6 --channel 0 --power 0

Resultado intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=6 channel=0 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Resultado: PASS-control.

Reset PHY 6 → PHY 7
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'boot\r\n'
CLOSED
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'exit\r\n'
CLOSED
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

Recuperación correcta.

PHY 7 — canal 0, potencia 0 dBm

Comando:

python examples\smoke_phase2.py --port COM88 --phy 7 --channel 0 --power 0

Resultado intacto:

FeralRF Phase 2 Smoke Test
==========================
port=COM88 baudrate=921600 phy=7 channel=0 power=0

[STEP] Connect + RADIO_INIT + GET_INFO
[ OK ] INFO firmware=1.0.0 capabilities=0x07 serial=464552414c524631
[STEP] SET_PHY
[ OK ] SET_PHY ACK
[STEP] SET_CHANNEL
[ OK ] SET_CHANNEL ACK
[STEP] SET_POWER
[ OK ] SET_POWER ACK
[STEP] RX_START
[ OK ] RX_START ACK
[STEP] RX_STOP
[ OK ] RX_STOP ACK

[ OK ] SMOKE TEST PASS

Resultado: PASS-control, con la limitación ya señalada de que esto no configura ni valida por sí solo una modulación propietaria concreta.

Reset y comprobación final

Aunque la exigencia documental es reset entre filas, después de PHY 7 también se realizó uno, dejándonos además una comprobación explícita de que el dispositivo continuaba recuperándose al terminar EV-10.

python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'boot\r\n'
CLOSED
python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"
OPEN COM87
SENT b'exit\r\n'
CLOSED
python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"
INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

El dispositivo quedó nuevamente accesible mediante FeralRF.

3. Comparación contra lo esperado

La prueba realizada sí concuerda con el procedimiento definido para EV-10: se utilizó el script original smoke_phase2.py, se recorrieron las ocho combinaciones establecidas y se hicieron resets entre filas. La guía establece que la finalidad es comprobar PHY/canal/potencia sin afirmar RF y que una ejecución puede ser satisfactoria aunque no se reciban paquetes.

PHY	Canal	Potencia	Esperado	Observado
0	37	0 dBm	PASS-control	PASS-control
1	9	0 dBm	PASS-control	PASS-control
2	37	0 dBm	PASS-control	PASS-control
3	37	0 dBm	PASS-control	PASS-control
4	25	0 dBm	PASS-control	PASS-control
5	0	0 dBm	PASS-control	PASS-control
6	0	0 dBm	PASS-control	PASS-control
7	0	0 dBm	PASS-control, con limitación PROP	PASS-control, con limitación PROP

Resultado global: 8/8 PASS-control.

No apareció durante las ocho ejecuciones ningún timeout, ERROR, rechazo de parámetro, fallo de RX_START, fallo de RX_STOP ni pérdida persistente de comunicación. Después de cada reset comprobado, INIT y GET_STATS volvieron a responder.

Hay además un detalle metodológico favorable: no creamos un harness específico para esta prueba. Utilizamos examples\smoke_phase2.py del propio repositorio, que es precisamente el script indicado por la guía para EV-10.

4. Qué hemos validado y qué no

Validado experimentalmente en EV-10: el DUT acepta los ocho valores de PHY bajo las combinaciones prescritas; acepta sus respectivos SET_CHANNEL y SET_POWER; completa RX_START → RX_STOP; y puede recuperarse entre filas mediante el reset manual utilizado.

No validado por EV-10: recepción RF efectiva de los ocho PHY, transmisión RF efectiva, frecuencia/potencia real en antena, sensibilidad, PER, interoperabilidad, calidad de demodulación ni corrección de una modulación propietaria concreta para PHY 7. La matriz advierte expresamente que ACK ≠ RF.

Esto evita una conclusión demasiado fuerte: un SET_PHY ACK demuestra que el comando fue aceptado por la cadena software/firmware, no que un analizador externo haya confirmado que el CC1352P7 está produciendo o recibiendo exactamente la forma de onda correspondiente.

5. Conclusión de EV-10

EV-10 queda cerrada como PASS-control 8/8.

La evidencia obtenida concuerda con el resultado esperado por la documentación de validación. Las ocho configuraciones completaron satisfactoriamente RADIO_INIT/GET_INFO → SET_PHY → SET_CHANNEL → SET_POWER → RX_START → RX_STOP, empleando el script original previsto para esta prueba.

Los resets entre filas también resultaron satisfactorios mediante el procedimiento manual COM87 ya establecido debido a KI-15, y no se observó que alguna de las configuraciones dejara el DUT inaccesible para la siguiente prueba.

La conclusión que podemos defender es, por tanto:

En el CatSniffer V3 evaluado con FeralRF, los PHY 0–7 superaron la matriz EV-10 a nivel de control (PASS-control 8/8) bajo las combinaciones de canal y potencia prescritas. Este resultado valida la aceptación y el recorrido básico de configuración/RX de los ocho valores de PHY, pero no constituye validación RF individual de cada PHY. PHY 7 permanece además limitado a una comprobación de control hasta configurar y validar una modulación propietaria concreta mediante configure_prop.
````
