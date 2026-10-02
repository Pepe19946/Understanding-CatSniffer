Registro — EV-03: reconexión limpia

Comando realizado explícitamente, desde PowerShell en FeralRF\python:

1..5 | % { Write-Host "`n===== CICLO $_ / 5 ====="; python -c "from feralrf import Radio; r=Radio(port='COM88'); print(r.init()); r.disconnect()" }

Acción que realiza. Se crean cinco procesos Python independientes. Cada uno abre una nueva sesión con FeralRF mediante COM88, ejecuta Radio.init(), recibe la información del dispositivo y finalmente ejecuta disconnect(). Al terminar el proceso, el siguiente ciclo comienza desde una nueva instancia.

Intención de la prueba. EV-03 busca comprobar que una sesión puede cerrarse y volver a abrirse repetidamente sin dejar el puerto ocupado ni provocar timeouts, respuestas incorrectas o problemas evidentes de estado entre conexiones. La matriz establece precisamente como evidencia esperada 5/5 ciclos.

Resultado obtenido a nivel de terminal (sin modificar):

PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python> 1..5 | % { Write-Host "`n===== CICLO $_ / 5 ====="; python -c "from feralrf import Radio; r=Radio(port='COM88'); print(r.init()); r.disconnect()" }

===== CICLO 1 / 5 =====
DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')

===== CICLO 2 / 5 =====
DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')

===== CICLO 3 / 5 =====
DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')

===== CICLO 4 / 5 =====
DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')

===== CICLO 5 / 5 =====
DeviceInfo(firmware_version='1.0.0', capabilities=7, serial='464552414c524631')
PS C:\Users\Support\Documents\ec-projects\catsniffer-feralrf\FeralRF\python>

Interpretación. Los cinco ciclos consiguieron inicializar FeralRF y devolvieron exactamente la misma identidad:

firmware_version = 1.0.0
capabilities     = 7
serial           = 464552414c524631

No aparecieron excepciones, timeouts ni errores de puerto ocupado. El hecho de que cada nuevo proceso pudiera volver a abrir COM88 también indica que disconnect()/finalización del proceso dejó disponible la interfaz para la siguiente conexión.

El resultado concuerda con el esperado: 5/5 ciclos satisfactorios.

Breve análisis. EV-03 puede marcarse provisionalmente como PASS. No encontramos indicios de un problema básico de reconnect en cinco procesos independientes. Esto tampoco demuestra todavía estabilidad de largo plazo ni múltiples reconexiones dentro de una misma instancia de Python; demuestra específicamente el escenario que acabamos de ejecutar.

Hay además un pequeño hallazgo de presentación que merece conservarse para la evaluación posterior: Radio.init() devuelve un DeviceInfo(...) perfectamente útil para desarrollo, pero es una representación bastante directa del objeto Python. Más adelante podremos decidir si la CLI/scripts deberían presentar esa información de una forma más amigable. No es un defecto funcional.

Con esto, nuestra progresión queda coherente: RX físico funciona y es repetible → cerrar/reabrir comunicación funciona 5/5. El siguiente candidato natural es EV-06, que prueba algo más interesante del firmware: qué ocurre si intentamos transmitir mientras RX ya está activo y si FeralRF rechaza correctamente ese estado inválido sin quedar dañado. La matriz espera ERROR 0x05 + recovery.

Antes de ejecutarlo, te daré el comando y te explicaré exactamente qué va a provocar, porque ahí empezamos deliberadamente a pedirle al firmware una operación inválida.