### Estandarización del directorio de ejecución de los procedimientos

**Acción requerida para Codex:**

Revisar y estandarizar todos los comandos de los manuales prácticos de FeralRF, especialmente los documentos 05, 06 y 07, estableciendo **`FeralRF/python` como directorio de trabajo obligatorio** para ejecutar los ejemplos.

Se identificó una inconsistencia: algunos procedimientos utilizan rutas relativas como `.\python\examples\...`, válidas desde `FeralRF/`, mientras que otros utilizan `.\examples\...`, válidas desde `FeralRF/python/`. Esta diferencia provoca errores al copiar y ejecutar los comandos desde un directorio distinto al supuesto por la documentación.

**Convención propuesta:**

Directorio de ejecución:

```powershell
cd C:\ruta\al\proyecto\FeralRF\python
```

Formato estándar de los comandos:

```powershell
python .\examples\lab\ota_rx_probe.py --port COM88 ...
```

Evitar dentro de estos procedimientos:

```powershell
python .\python\examples\lab\ota_rx_probe.py --port COM88 ...
```

**Criterios de corrección:**

1. Declarar explícitamente el directorio de trabajo al inicio de cada procedimiento.
2. Unificar todas las rutas relativas bajo la convención `FeralRF/python`.
3. Revisar los comandos de PowerShell, incluidos los bloques multilínea.
4. Mantener intactos los parámetros experimentales, salvo que exista una corrección justificada y documentada.
5. Comprobar que las rutas de los scripts realmente existen y que los argumentos coinciden con sus interfaces actuales.
6. Identificar cualquier comando que necesite ejecutarse desde otro directorio y justificarlo explícitamente.

**Objetivo:** que el usuario pueda copiar y ejecutar los comandos sin modificar sus rutas internas, teniendo que adaptar únicamente la ubicación local del repositorio y los puertos COM correspondientes.

Esta corrección es documental y de reproducibilidad; no requiere modificar el firmware.
