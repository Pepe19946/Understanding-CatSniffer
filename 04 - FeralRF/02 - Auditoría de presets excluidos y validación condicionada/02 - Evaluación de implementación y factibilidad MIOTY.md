# Evaluación de implementación y factibilidad MIOTY

## 1. Conclusión

`mioty_868_tsunb` no implementa MIOTY ni TS-UNB. Es una configuración nominal de un módem FSK genérico a 868 MHz, aproximadamente 396 baud, con un paquete propietario convencional. El CC1352P7 es una plataforma físicamente plausible y TI lo presenta como compatible con MIOTY, pero FeralRF carece de las funciones de radio y protocolo necesarias. Una nueva prueba de marcadores raw entre dos CatSniffer sería engañosa y no debe ejecutarse como validación MIOTY.

Readiness: **BLOCKED BY IMPLEMENTATION**, confianza alta.

## 2. Definición actual del preset

```text
frequency_hz = 868000000
mod_type     = 0       # FSK
symbol_rate  = 396
deviation    = 1       # 1 paso; hoy equivale a 250 Hz
rx_bw        = 0x4A
sync_word    = 0x930B51DE
format_conf  = 0       # implícito; firmware aplica 0x00A0
```

La tasa se convierte a `rateWord` con truncamiento entero. Para 396 baud y `preScale=15`, resulta `rateWord=259`, aproximadamente 395.2 símbolos/s según la fórmula del driver. El valor es serializable y no se vuelve cero, pero FeralRF no comprueba si la combinación completa `rateWord`, `rxBw`, desviación y overrides es una configuración SmartRF válida ni observa el estado final del Radio Setup.

El comentario en `presets.py` que sugiere que “CC1352 puede no soportar TS-UNB” no es una conclusión válida para CC1352P7: TI lista MIOTY entre las tecnologías del producto. La conclusión correcta es que **esta ruta FeralRF no implementa MIOTY**.

## 3. Traza completa

1. `PROP_PRESETS['mioty_868_tsunb']` proporciona siete campos.
2. `Radio.configure_prop()` llama `CommandBuilder.set_prop_config()`.
3. Python serializa 18 bytes con `<IBIHBIH>` y envía `SET_PROP_CONFIG`.
4. `command_processor.c` sólo valida que el payload mida 18 o 16 bytes, lo deserializa, llama `RadioIF_setPropConfig()` y responde ACK incondicionalmente.
5. `RadioIF_setPropConfig()` calcula frecuencia, `loDivider=5`, `rateWord=259`, copia `mod_type=0`, `deviation=1`, `rx_bw=0x4A`, sync y el formato genérico.
6. Al ser 868 MHz y no OOK, usa `Prop0_mode`, `Prop0_pOverrides` y los overrides TX normales generados alrededor de la configuración Sub-GHz general; no carga patch MIOTY ni override ultra-narrowband específico.
7. RX sólo empieza después con `RadioIF_startRx()` → `RadioIF_startSub1ghzRfBackend()` → Radio Setup, `CMD_FS` y `CMD_PROP_RX`.
8. TX se ACKea al encolarse. Después `ControlTask_processTxRaw()` ejecuta `CMD_FS` y `CMD_PROP_TX`; un fallo llega, si se observa, como `RSP_ERROR` asíncrono con secuencia cero.

Procesadores y componentes:

| Procesador | Papel |
|---|---|
| Host PC | API Python, payload raw y temporización gruesa del script |
| RP2040 | bridge UART y selección externa `band2`; no implementa TS-UNB |
| Cortex-M4F del CC1352P7 | parser, estado y scheduling de comandos |
| RF Core Cortex-M0 | ejecuta la imagen/patch propietario genérico y los comandos RF |

## 4. Qué significa el 0/10 histórico

El commit `6e34ef2` del 3-may-2026 registra:

- FSK, `rx_bw=0x4A`: 0/10;
- GFSK, `rx_bw=0x4A`: 0/10;
- anchos `0x46`, `0x42` y `0x4E`: todos 0/10.

No se localizó stdout, JSONL, identidad de binarios, lectura de estados RF ni medición espectral primaria. El plan F29/F29.b buscaba diez marcadores `DEADBEEF` entre dos CatSniffers sin cambio de firmware y usaba el helper genérico `smoke_f29_subg_915.py`. Éste resetea mediante la suposición Bridge+2, suprime excepciones de reset/stop y sólo verifica recepción del marcador raw.

Por tanto, 0/10 significa únicamente: **bajo la ejecución reportada, el receptor FeralRF no devolvió marcadores raw coincidentes**. No demuestra que:

- el RF Core realmente operara a los parámetros pretendidos;
- hubiera bursts TS-UNB en el aire;
- el CC1352P7 sea incompatible;
- una base station MIOTY rechazara un telegrama;
- fallara telegram splitting o reensamble.

La razón causal del 0/10 histórico es **UNKNOWN** por falta de evidencia primaria. El bloqueo actual, en cambio, sí puede establecerse desde la fuente.

## 5. Cuatro niveles que no deben confundirse

### 5.1 Configuración RF nominal

FeralRF puede almacenar 868 MHz, FSK y una tasa cercana a 396 baud. No valida la combinación contra SmartRF Studio, no tiene readback efectivo de modulación/bandwidth y reutiliza overrides Sub-GHz genéricos. La validez física exacta de ese setup permanece sin demostrar.

### 5.2 Intercambio de paquete propietario funcional

La ruta `CMD_PROP_TX/RX` genera/espera preámbulo, sync de 32 bits, longitud, payload y CRC del RF Core. Si dos CatSniffers intercambiaran ese formato, sólo se validaría un enlace FSK propietario coincidente.

### 5.3 Implementación TS-UNB

No existe. La búsqueda en fuente sólo encuentra el preset, el smoke F29 y un listener de laboratorio. Faltan, al menos:

- segmentación/telegram splitting;
- codificación, interleaving y estructura de bursts conformes;
- programación precisa de tiempos y frecuencias por burst;
- perfiles regionales y parámetros de acceso;
- identidad/seguridad/procedimientos del endpoint;
- recepción de múltiples bursts, asociación y reensamble;
- telemetría para verificar cada etapa.

El script `demo_mioty_listen.py` se declara explícitamente stub “PHY-only”; imprime bytes de paquetes genéricos, no parsea ni reensambla TS-UNB y ni siquiera llama `stop_rx()` en `finally`.

### 5.4 Interoperabilidad MIOTY

Tampoco existe. MIOTY es una arquitectura endpoint–base station; la capa física aplica telegram splitting y la base station reconstruye los bursts. Una prueba CatSniffer↔CatSniffer con un marcador no sustituye una base station, una implementación conforme ni los criterios ETSI.

## 6. Respuestas a las preguntas de factibilidad

| Pregunta | Hallazgo |
|---|---|
| ¿Por qué quedó pending? | F29.b reportó 0/10 en variaciones raw y activó su escape M3; se infirió que hacía falta configuración/patch dedicado. |
| ¿Existe evidencia primaria del 0/10? | No localizada; sólo commit/comentarios y documentación derivada. |
| ¿Los valores se serializan? | Sí; el parser los acepta y los copia. Eso no valida el setup RF. |
| ¿El silicio puede soportar señalización relacionada? | Sí, plausible: TI lista MIOTY y bandas 861–1054 MHz para CC1352P7. |
| ¿La configuración FeralRF es técnicamente suficiente? | No. Es FSK genérico con overrides generales y sin validación SmartRF. |
| ¿Existe telegram splitting? | No. |
| ¿Existe la temporización/frecuencia por burst? | No. Sólo espaciamiento host de paquetes completos. |
| ¿Existe reensamble RX? | No. |
| ¿Puede afirmarse TS-UNB? | No. |
| ¿Puede afirmarse interoperabilidad MIOTY? | No. |

## 7. Ruta de trabajo futura, sin modificar este repositorio

TI publica una página de MIOTY con demo para CC1352R y referencias a una biblioteca `libmioty` para CC1352R/CC1352P/CC1312, además de un stack de producción de Stackforce. El CC1352P7 se beneficia de afinidad de familia, pero la compatibilidad binaria/licencia/port exacta debe confirmarse; no se puede asumir.

MIOTY Alliance también lista implementaciones TS-UNB abiertas para otros MCU/radios (RFM69, RP2040, STM32). Son referencias útiles para framing, timing y pruebas, no código directamente portable al RF Core de TI, y algunas licencias limitan uso comercial.

Dependencias antes de cualquier validación MIOTY:

1. elegir una implementación o especificación de referencia con licencia adecuada;
2. generar/auditar la configuración RF para CC1352P7 y el front-end real;
3. implementar scheduling de bursts y observabilidad;
4. implementar endpoint o base-station role explícito, no un loop raw simétrico;
5. probar primero con instrumentación de frecuencia/espectro;
6. finalmente usar una base station MIOTY conocida y criterios de interoperabilidad.

Referencias:

- https://www.ti.com/product/CC1352P7
- https://www.ti.com/tool/MIOTY
- https://mioty-alliance.com/faq-items/protocol-layers/
- https://developers.mioty-alliance.com/build/

## 8. Decisión

No se propone experimento RF para `mioty_868_tsunb` en el estado actual. Repetir el 0/10 o conseguir 10/10 raw no cerraría ninguna de las brechas TS-UNB. La dependencia correcta es implementación, no más barrido de `rx_bw`.

