# EV-04 — Reset y reinicialización

Registro canónico. Resultado manual y API diferenciados; fecha experimental no documentada.

## 1. Contexto de evaluación

Tras EV-03 se necesitaba saber si una orden de Shell permitía abandonar/reanudar FeralRF y si la API seleccionaba la Shell correcta.

## 2. Objetivo de validación

Evaluar recuperación mediante `boot`/`exit` y la selección del puerto Shell usada por `reset_device()`.

## 3. Capacidad o requisito FeralRF evaluado

Interacción RP2040–CC1352P7 y recuperación de estado. [[Arquitectura FeralRF]], [[Protocolo y API Python]] y [[Firmware RP2040]].

## 4. Precondiciones y condiciones

Bridge COM88, Shell real COM87; miniterm a 115200. Esperas controladas de 3,5 s; `init()` con timeout 2,5 s. Binario/commit instalado no identificado. Mapa correspondiente a la etapa inicial, no al mapa posterior de EV-12.

## 5. Resultado esperado

La entrada a bootloader impediría responder a FeralRF; `exit` debería permitir `INIT` sin reconectar USB. La API debería seleccionar COM87. La guía exige más repetición que la única secuencia completa documentada.

## 6. Procedimiento y ejecución cronológica

1. Abrir miniterm durante unos 5 s; el intento manual inicial de `boot` no acredita bytes efectivos.
2. Enviar de forma controlada `boot\r\n` a COM87; registrar OPEN/SENT/CLOSED y esperar 3,5 s.
3. Intentar `init()` en COM88: timeout.
4. Enviar `exit\r\n`, esperar 3,5 s y recuperar `INIT`/estadísticas sin ciclo USB.
5. Después de esa secuencia, consultar `_get_shell_port`: devuelve COM90.
No se invierte este orden para ajustarlo a la narrativa de la auditoría histórica.

## 7. Resultado observado

Recuperación manual funcional en la secuencia registrada. La selección automática produjo COM90 frente a Shell real COM87. No se ejecutó como prueba completa un `reset()` de API sobre el puerto incorrecto.

## 8. Evidencia

Comandos, salidas del helper, traceback de timeout (`radio.py` líneas 415/336) y consulta del puerto preservados en §17.

## 9. Comparación entre lo esperado y lo observado

Manual: comportamiento compatible con bootloader y recuperación. API: selección discrepante; no cumple el requisito de encontrar la Shell real en este mapa.

## 10. Interpretación técnica

La aritmética de puertos no es universal. La secuencia confirma recuperación manual local; no demuestra qué pin físico se accionó. GPIO15 en diagramas previos y GPIO3/reset GPIO2 en descripciones de overlay requieren reconciliación de revisiones.

## 11. Anomalías, desviaciones y limitaciones

Un ciclo completo, no 3/3. Intento inicial ambiguo. Sin medida de pines ni prueba de todas las revisiones del puente. Versiones no fijadas.

## 12. Resultado de la evaluación

NOT FULLY VALIDATED: recuperación manual observada; ruta API bloqueada por selección incorrecta y cobertura de repetición insuficiente.

## 13. Confianza

High sobre COM90 frente a COM87 y la secuencia manual; Medium sobre generalizar la recuperación.

## 14. Preguntas abiertas

¿Qué identificación USB permite asociar Bridge y Shell por dispositivo? ¿Qué overlay/binario y pines corresponden a esta placa?

## 15. Acciones de seguimiento

Corregir descubrimiento por identidad/interfaz y comprobar por placa; repetir ciclos con registro binario y GPIO antes de usar reset como control de todos los EV.

## 16. Trazabilidad

Guía EV-04/EV-40; [[FeralRF - Guía de validación experimental]]; [[FeralRF - Matriz de pruebas]]; [[Matriz de capacidades]]; [[EV-03 — Reconexión limpia entre procesos]]; [[Fuentes firmware oficial]]; [[Registro de validación FeralRF]].

Definición específica: [[FeralRF - Wiki técnica integral#6.2 Conexión, reset y secuencia]].



## 17. Notas originales preservadas y material pendiente

Fuente: `EV-04 — Reset y reinicialización.md`. SHA-256 previo: `9F3E0C4D1ED36C944E04191037B3B05C5B2776FF7CF6A52C3D183C8FAF30DDA9`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
EV-04 — Reset y reinicialización

Estado final actual: BLOQUEADO para Radio.reset_device() por reproducción de KI-15.
Mecanismo de recuperación mediante Cat-Shell explícito: comprobado favorablemente en 1 ejecución.

1. Objetivo y criterio original de EV-04

La guía define EV-04 como una prueba P0 cuyo objetivo es validar el mecanismo de recuperación antes de entrar en modos RF potencialmente frágiles y caracterizar de forma segura KI-15.

La ruta que debía comprobarse era:

PC
 ↓
Cat-Shell @ 115200
 ↓
RP2040
 ↓ GPIO15
RESET_N
 ↓
CC1352P7
 ↓
FeralRF vuelve a ejecutarse
 ↓
Cat-Bridge
 ↓
PC / Radio.init() / GET_STATS

El procedimiento originalmente documentado utiliza:

python -c "from feralrf import Radio; r=Radio(port='COM_BRIDGE'); print(r.init()); r.reset_device(wait=3.5); print(r.get_stats()); r.disconnect()"

Según la guía, reset_device() cierra el Bridge, calcula internamente el puerto Shell como COM(n+2), envía boot y exit, vuelve a abrir la comunicación e inicializa FeralRF. El estado sano requiere que el Shell calculado coincida con el Shell que Catnip atribuye a la misma placa y que INFO/STATS regresen sin desconectar físicamente el USB. El criterio formal es PASS con 3/3 recuperaciones, FAIL si abre otro dispositivo o requiere desconexión, y BLOQUEADO si el Shell calculado no coincide con el Shell verificado.

Esto es importante: la propia fase nos impedía ejecutar ciegamente reset_device() si antes encontrábamos una discrepancia en el mapeo.

2. Estado inicial antes de EV-04

La placa bajo prueba es la DUT-V3-FERAL: CatSniffer V3 con RP2040 + CC1352P7, ejecutando FeralRF en el CC1352P7.

Catnip había identificado:

Cat-Bridge (CC1352): COM88
Cat-LoRa (SX1262):   COM86
Cat-Shell (Config):  COM87

Por tanto, para nuestra placa:

Bridge real identificado = COM88
Shell real identificado  = COM87

Esto ya era sospechoso porque la documentación de FeralRF indicaba que reset_device() deriva el Shell mediante Bridge + 2.

Además, EV-03 acababa de demostrar que FeralRF se encontraba en estado operativo: realizamos 5 procesos independientes de Radio.init()/disconnect() sobre COM88 y los 5 respondieron correctamente, siempre con:

DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')

Por tanto, comenzamos EV-04 con un estado de referencia conocido: COM88 y FeralRF funcionaban correctamente antes de manipular el reset.

3. Comprobación inicial de COM87

Primero abrimos el puerto identificado por Catnip como Shell:

python -m serial.tools.miniterm COM87 115200

Se obtuvo:

--- Miniterm on COM87  115200,8,N,1 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---

En los primeros intentos solamente se esperaron aproximadamente 5 segundos y se salió mediante Ctrl+]. No se presionó ningún botón físico y no se envió ningún comando al dispositivo.

Esas primeras aperturas, por tanto, solo demostraron:

Windows/pySerial puede abrir COM87 @ 115200.

No demostraban todavía que COM87 fuera funcionalmente el Cat-Shell.

Posteriormente se intentó introducir boot manualmente mediante Miniterm, sin obtener una respuesta útil. Decidimos no interpretar esa ausencia como fallo porque la reproducción no controlaba explícitamente los bytes enviados y el análisis previo del código indicaba una secuencia concreta con terminadores \r\n.

4. Reproducción controlada de boot\r\n

Para eliminar la ambigüedad de Miniterm se utilizó directamente pySerial sobre el Shell explícito COM87:

python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'boot\r\n'); s.flush(); print('SENT',repr(b'boot\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"

Resultado exacto:

OPEN COM87
SENT b'boot\r\n'
CLOSED

El cierre ocurrió aproximadamente después de los 3.5 s establecidos.

Qué demuestra

A nivel host demuestra que:

COM87 abrió correctamente
→ pySerial escribió exactamente b'boot\r\n'
→ flush() terminó sin excepción
→ se esperaron ≈3.5 s
→ COM87 cerró normalmente

Es importante no exagerar esta evidencia: OPEN, SENT y CLOSED son mensajes generados por nuestro script. No son respuestas del RP2040.

La evidencia realmente interesante apareció al comprobar posteriormente el CC1352P7.

5. Estado de FeralRF inmediatamente después de boot\r\n

Sin desconectar la CatSniffer ni realizar power-cycle ejecutamos:

python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"

Resultado exacto:

Traceback (most recent call last):
  File "<string>", line 1, in <module>
    from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()
                                                                    ~~~~~~^^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 415, in init
    cmd_id, seq, payload = self._read_response(
                           ~~~~~~~~~~~~~~~~~~~^
        timeout=2.5, expected={Response.ACK, Response.ERROR}
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    )
    ^
  File "C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python\feralrf\radio.py", line 336, in _read_response
    raise TimeoutError("Response timeout")
feralrf.exceptions.TimeoutError: Response timeout

Este resultado cambia sustancialmente el valor de la prueba.

Antes de enviar boot\r\n, EV-03 había obtenido 5/5 inicializaciones correctas. Inmediatamente después de enviarlo por COM87, Radio.init() sobre COM88 dejó de obtener respuesta y agotó sus 2.5 s de timeout.

Por tanto:

Antes de boot:
COM88 → FeralRF responde

boot\r\n por COM87

Después de boot:
COM88 → Response timeout

Esto constituye evidencia experimental de que COM87 no es simplemente un puerto serie que puede abrirse: enviar boot\r\n produce un cambio observable sobre el sistema asociado al CC1352P7/FeralRF.

No podemos afirmar únicamente con este experimento qué estado interno exacto alcanzó el CC1352P7, pero el comportamiento es compatible con la función de boot/reset descrita para Cat-Shell.

6. Envío de exit\r\n

A continuación completamos manualmente la segunda parte de la secuencia documentada:

python -c "import serial,time; s=serial.Serial('COM87',115200,timeout=1); print('OPEN',s.name); s.write(b'exit\r\n'); s.flush(); print('SENT',repr(b'exit\r\n')); time.sleep(3.5); s.close(); print('CLOSED')"

Resultado exacto:

OPEN COM87
SENT b'exit\r\n'
CLOSED

Nuevamente, esto demuestra únicamente que el host pudo transmitir:

b'exit\r\n'

La comprobación funcional debía realizarse otra vez por COM88.

7. Recuperación posterior por COM88

Sin power-cycle ni desconexión USB se volvió a ejecutar:

python -c "from feralrf import Radio; r=Radio(port='COM88'); print('INIT:',r.init()); print('STATS:',r.get_stats()); r.disconnect()"

Resultado exacto:

INIT: DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
STATS: DeviceStats(rx_ok=0, rx_crc_err=0, rx_drop=0, rx_overflow=0, ll_kind_unknown=0, ll_kind_adv=0, ll_kind_scan=0, ll_kind_connect=0, ll_kind_data=0)

La secuencia experimental completa fue, por tanto:

FeralRF operativo
    │
    │ EV-03: 5/5 init correctos
    ▼
COM88 responde
    │
    │ COM87 ← b'boot\r\n'
    ▼
COM88 deja de responder
Response timeout
    │
    │ COM87 ← b'exit\r\n'
    ▼
COM88 vuelve a responder
    │
    ├─ INIT correcto
    └─ GET_STATS correcto

Esto concuerda cualitativamente con el comportamiento esperado por EV-04: se produjo una transición mediante Cat-Shell y posteriormente FeralRF recuperó comunicación por Cat-Bridge sin reconectar físicamente USB. La guía precisamente utiliza INFO/STATS posteriores como evidencia de recuperación.

Los contadores de DeviceStats aparecieron todos en cero. Esto es coherente con un estado reinicializado, pero no debemos convertirlo por sí solo en prueba de un reset eléctrico concreto del CC1352P7. La evidencia fuerte es la transición completa observada, no exclusivamente los ceros.

8. Comprobación directa de _get_shell_port()

Hasta aquí habíamos utilizado COM87 explícitamente. Faltaba demostrar si la instalación real de FeralRF efectivamente cometería la discrepancia anticipada por la documentación.

Ejecutamos:

python -c "from feralrf import Radio; r=Radio(port='COM88'); print('Bridge configurado:',r.port); print('Shell calculado:',r._get_shell_port())"

Resultado exacto:

Bridge configurado: COM88
Shell calculado: COM90

Este resultado es especialmente importante porque ya no estamos infiriendo que COM88 + 2 = COM90: lo estamos obteniendo directamente de _get_shell_port() en la instalación evaluada.

La comparación queda:

Fuente/evidencia	Bridge	Shell
Catnip / identificación de la placa	COM88	COM87
Prueba física boot/exit	—	COM87 funcional
FeralRF _get_shell_port()	COM88	COM90

Por tanto:

Shell funcional demostrado = COM87
Shell calculado por FeralRF = COM90

COM87 ≠ COM90

Esto reproduce experimentalmente el problema que la documentación había identificado como KI-15: inferencia frágil Shell = Bridge + 2.

9. Documentación frente a resultado experimental

La comparación final es especialmente clara.

La documentación decía: FeralRF depende del Cat-Shell para recuperación; reset_device() calcula internamente COM(n+2), envía boot/exit y después debe recuperar INFO/STATS. La guía advertía que la numeración +2 era frágil y exigía comparar el resultado contra el Shell identificado independientemente por Catnip. Si no coincidían, EV-04 debía considerarse BLOQUEADO.

El código ejecutado mostró: _get_shell_port() recibe Bridge COM88 y devuelve COM90.

Catnip mostró: el Cat-Shell de la misma placa es COM87.

La prueba física mostró: COM87 tiene efecto funcional sobre el CC1352P7/FeralRF. boot\r\n provoca que FeralRF deje de responder por COM88; exit\r\n permite recuperar Radio.init() y get_stats() sin power-cycle.

Por tanto, la advertencia documental sobre KI-15 se reproduce en nuestra configuración real.

10. Estado final de EV-04

No corresponde marcar simplemente EV-04 como PASS, porque hacerlo ocultaría precisamente el problema que esta fase estaba diseñada para detectar.

El resultado correcto es:

EV-04 — BLOQUEADO para la API pública Radio.reset_device() debido a KI-15 reproducido experimentalmente.

La causa inmediata demostrada a nivel host es:

Radio(port='COM88')
        ↓
_get_shell_port()
        ↓
COM90

mientras que la placa realmente presenta:

Cat-Bridge = COM88
Cat-Shell  = COM87

No ejecutamos deliberadamente Radio.reset_device() contra COM90 porque el criterio de seguridad de EV-04 establece que la prueba debe bloquearse cuando el puerto calculado no coincide con el Shell previamente verificado.

Al mismo tiempo, el mecanismo subyacente de recuperación no ha fallado en nuestra prueba. Utilizando explícitamente el Shell correcto COM87 observamos una secuencia funcional:

boot → FeralRF deja de responder
exit → FeralRF vuelve a responder

con posterior INIT y GET_STATS correctos y sin desconexión USB.

Por eso conviene conservar dos resultados separados:

API Radio.reset_device() — BLOQUEADO. La selección automática del Shell es incorrecta para la enumeración USB observada.

Ruta física/manual de recuperación — FAVORABLE, 1/1 observado. El Cat-Shell COM87 produjo la transición y recuperación esperadas. Esto no equivale al criterio formal 3/3 de la API ni valida un reset por watchdog.

Conclusión técnica provisional

EV-04 no revela, hasta ahora, un fallo demostrado del mecanismo RP2040→CC1352P7 de recuperación. Lo que sí revela y reproduce es un problema de descubrimiento/selección del puerto en la capa host de FeralRF: la asociación del Shell mediante aritmética Bridge+2 no representa la enumeración USB real de nuestra CatSniffer.

Esto debe conservarse para la fase posterior de evaluación/mejoras como candidato concreto: el comportamiento actual depende de una relación numérica entre puertos COM que Windows no está respetando en nuestra configuración. La solución todavía no debe implementarse durante el baseline; posteriormente habrá que estudiar cómo reemplazar esa inferencia por identificación robusta del Cat-Shell y volver a ejecutar EV-04 hasta obtener el criterio formal de recuperación reproducible.
````
