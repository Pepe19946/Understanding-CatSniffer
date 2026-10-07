# EV-03 — Reconexión limpia entre procesos

Registro canónico de EV-03. Fecha experimental no documentada; evaluación existente al corte histórico del 5 de octubre de 2026.

## 1. Contexto de evaluación

Se necesitaba separar la reconexión serial limpia de la recuperación mediante reset que se estudiaría en EV-04.

## 2. Objetivo de validación

Comprobar que procesos Python independientes pueden abrir, inicializar y cerrar sucesivamente el puente sin dejar el puerto ocupado.

## 3. Capacidad o requisito FeralRF evaluado

Apertura/cierre de `Radio`, `INIT` y respuesta de información de dispositivo; no equivale a reconexión USB física ni a reinicialización repetida de una misma instancia. Véase [[Protocolo y API Python]].

## 4. Precondiciones y condiciones

Puente COM88; cinco procesos independientes. Python/API FeralRF según los comandos preservados. Versiones binarias exactas, hash del firmware cargado, identidad USB física, tiempos entre procesos y entorno RF: no documentados. No consta unplug USB.

## 5. Resultado esperado

Cada proceso debía inicializar y cerrar limpiamente, sin timeout ni puerto ocupado. No consta un criterio temporal cuantitativo.

## 6. Procedimiento y ejecución cronológica

1. Ejecutar un proceso que abre `Radio(port="COM88")` y llama a `init()`.
2. Obtener `DeviceInfo` y cerrar/desconectar.
3. Repetir con otros cuatro procesos independientes.
La secuencia exacta de comandos se conserva en §17.

## 7. Resultado observado

Cinco inicializaciones devolvieron información coincidente: versión 1.0.0, capacidades 7 y serial `464552414c524631`. No se registraron errores de puerto ocupado ni timeouts en estas ejecuciones.

## 8. Evidencia

Cinco resultados textuales del registro original, §17. `464552414c524631` corresponde a la constante `FERALRF1`; no identifica inequívocamente una placa física.

**Nivel de evidencia:** C: cinco inicializaciones/respuestas entre procesos independientes. No acredita desconexión USB ni ciclo de vida de una misma instancia.

Modelo: [[FeralRF - Matriz de pruebas#Modelo de evidencia A–F]]. Nivel, resultado, confianza y procedencia son dimensiones separadas.

## 9. Comparación entre lo esperado y lo observado

La respuesta y cierre observados cumplen el objetivo estrecho de cinco aperturas entre procesos. No se comprobó continuidad USB ni latencia de recuperación.

## 10. Interpretación técnica

Hallazgo confirmado: esta secuencia de procesos pudo reutilizar COM88. La identidad y los valores de `DeviceInfo` no prueban correspondencia entre el binario instalado y el commit de referencia.

## 11. Anomalías, desviaciones y limitaciones

Sin hash binario, tiempos ni trazas seriales completas. No ensaya la misma instancia, Ctrl+C, suspensión, desconexión abrupta ni sesión larga.

## 12. Resultado de la evaluación

PASS + C para reconexión limpia entre cinco procesos independientes. Fuera de ese alcance, no validado.

## 13. Confianza

High para la secuencia registrada; no extrapolable a todos los modos de reconexión.

## 14. Preguntas abiertas

¿Se libera igual el puerto tras una excepción o Ctrl+C? ¿Qué ocurre con varias llamadas a `init()` en una sola instancia?

## 15. Acciones de seguimiento

EV-46 y EV-47 de la guía; registrar identidad y tiempos antes de ampliar la afirmación.

## 16. Trazabilidad

[[FeralRF - Guía de validación experimental]] EV-03, EV-46, EV-47; [[FeralRF - Matriz de pruebas]]; [[Matriz de capacidades]]; [[Arquitectura FeralRF]]; [[EV-04 — Reset y reinicialización]]; [[Registro de validación FeralRF]].

Definición específica: [[FeralRF - Wiki técnica integral#6.2 Conexión, reset y secuencia]].



## 17. Notas originales preservadas y material pendiente

Fuente: `Registro — EV-03 reconexión limpia.md`. SHA-256 previo: `C654653C56CC38FC2C073E8CB56AEADC0E749890D963ADEAA04AD4A04D3396DE`.

Transcripción íntegra, sin corregir comandos, salidas, errores ni conclusiones históricas. Sus estados y recomendaciones deben leerse con el alcance y las correcciones de la parte normalizada. Las fechas de esta auditoría no son fechas de ejecución experimental. Los comandos son evidencia histórica; no se ejecutaron durante esta revisión.

````text
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
````
