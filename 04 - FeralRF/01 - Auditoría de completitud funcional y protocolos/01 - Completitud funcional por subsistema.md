# Completitud funcional por subsistema

## Matriz consolidada

| Área | API/entrada | Comportamiento implementado | Límite determinante | Evidencia | Clasificación |
|---|---|---|---|---|---|
| Transporte host | Sí | COBS, delimitador, CRC16, secuencia y despacho | Los eventos asíncronos comparten espacio de secuencia de forma no coherente con el contrato host | Código y pruebas unitarias | Implementado con limitaciones |
| Identidad y ciclo de vida | Sí | Apertura, `HELLO`, reset y consulta de identidad | Identidad reportada constante; no prueba binario/hardware efectivo | Código; F-05 | Incompleto frente al contrato de diagnóstico |
| `set_phy` | Sí | Selecciona PHY, canal `u16` y frecuencia opcional `u32` | No hay consulta de configuración efectiva; validación desigual | Código host/firmware/driver | Implementado con limitaciones |
| `configure_prop` | Sí | Frecuencia, tipo, tasa, desviación, BW, sync y `format_conf` | ACK significa aceptación; 4/5/6 son reservados; no hay *readback* | SDK exacto y código | Parcial para parámetros anunciados |
| Potencia | Sí | Solicita potencia y el driver escoge una entrada | El valor efectivo redondeado no se devuelve | Código; F-17 | Implementado con limitaciones |
| RX | Sí | Inicio/parada, entrega de datos, RSSI/LQI/canal/timestamp según PHY | Orden de metadatos PROP defectuoso; pérdidas de cola no visibles; causa de `RX_STOP` histórica inconclusa | Código + Fase I | Incompleto en observabilidad/contrato PROP |
| TX único | Sí | Aceptación diferida y ejecución por `ControlTask` | No existe evento final correlacionado; límites por PHY difieren | Código + Fase I | Incompleto en finalización |
| TX frame | Sí | Usa el mismo camino de transmisión que TX raw | El nombre no construye MAC; en IEEE el RF core añade PHR/FCS | Código + SDK | Completo sólo como alias de envío de bytes |
| BURST/CONTINUOUS | Sí | Repetición programada y parada | No garantiza al host conteo/fin/errores posteriores | Código + Fase I | Implementado con limitaciones |
| Proprietary FSK/GFSK | Sí | Setup RF, TX/RX de paquete variable, length y CRC por RF core | Parámetros efectivos no se leen; presets no son protocolos | Código + OTA histórica | Completo como herramienta PHY limitada |
| Proprietary OOK | Sí | Selección OOK y camino RF específico | Históricamente no validado; presets de “dispositivo” son estímulos de bytes | Código + Fase I | Implementado no validado |
| MSK/4-FSK/4-GFSK por `mod_type` | Aceptada | El valor se copia al campo `modType` | SDK: sólo 0/1/2 definidos; no patch/comando alterno | SDK exacto | No implementado como modulación demostrable |
| IEEE 802.15.4 | Sí | Canales, RX, TX de PSDU/MAC; RF core añade PHR/FCS | Filtrado y auto-ACK desactivados; sin procedimientos MAC | Código, SDK + OTA | Implementado como PHY/transportador de MAC |
| BLE | Sí, limitada | RX publicitario, TX de advertising data; parseo LL superficial | Sin pila pública de conexión, GAP/GATT o seguridad; funciones previas retiradas | Código + historial | PHY + publicidad, intencionalmente limitado |
| CW/PRBS | Sí | Modos de prueba RF continuos | Son prueba de radio, no tráfico protocolario | Código + Fase I | Completo para el alcance de prueba |
| Jamming | Sí | Emisión continua acotada para BLE/IEEE | Sin reactivo, patrones ni motor general PROP; máximo 30 s | Código | Implementado con limitaciones |
| Spectrum | Builder host residual | No existe implementación firmware equivalente utilizable | No debe anunciarse como capacidad | Código | No implementado |
| Criptografía | Sí | TRNG, AES ECB/CTR/CBC/CCM/GCM, SHA-256, ECDH, ECDSA | Curva ECDSA host/firmware no coincide; límites CCM/GCM; pruebas host no ejercen hardware | Código/pruebas estáticas | Parcial respecto de la API anunciada |
| Emulación IEEE | Sí | Construye dos tramas MAC fijas | Sin asociación, ACK, seguridad ni estado real | Código | Constructor de tramas, no emulador de dispositivo |
| Emulación Wireless M-Bus | Sí | Bytes fijos sobre preset inapropiado | Estructura/longitud/CRC/procedimientos incompletos | Código + OMS | Incompleto frente a su nombre |
| Emulación OOK | Sí | Patrones canónicos fijos | RF core los encapsula con framing propietario; no reproduce temporización/codificación real | Código | Generador de estímulos |
| KillerBee | Adaptador presente | Traduce operaciones básicas IEEE | Empaquetado/dependencias, interpretación y ciclo de vida no son reproducibles integralmente | Código + Fase I | Integración parcial |
| RP2040/Cat-Shell | Externo al CC1352 | Encaminamiento/selección de puerto en la integración | Suposición de puerto/identidad no garantizada | Código + F-06 | Parcial para integración automática |

## Propiedad de responsabilidades

| Componente | Responsabilidades verificadas | No debe atribuírsele |
|---|---|---|
| Python | API, validación parcial, serialización, espera de respuestas, callbacks, presets y helpers | Configuración RF efectiva, CRC en aire o procedimientos no codificados |
| CC1352P7 firmware | Estado, despacho, colas, selección de RF command, preparación de TX/RX, parseo mínimo y crypto | Confirmación automática al host de toda operación diferida |
| TI RF core/SDK | Modulación soportada por el comando/patch activo, preámbulo/sync/length/CRC PROP, framing IEEE/BLE según comando | Construcción de una pila MAC/red/aplicación completa |
| RP2040/Cat-Shell | Puente y exposición física del dispositivo en la arquitectura CatSniffer | Semántica de la API FeralRF o verificación del protocolo aéreo |
| Aplicación usuaria | Bytes MAC/protocolo cuando FeralRF sólo transporta payload, políticas y manejo de eventos | Asumir que el nombre del preset crea los campos faltantes |

## Configuración RF

`Radio.configure_prop()` crea un payload con frecuencia, `mod_type`, tasa, desviación, ancho de banda, palabra de sincronía y `format_conf`. El handler valida longitud, copia la configuración y emite ACK; no devuelve el estado del comando RF ni una lectura efectiva. Esto separa tres hechos:

1. **Aceptación por API:** el valor pasó validaciones Python.
2. **Aceptación por firmware:** la estructura tuvo longitud válida y se almacenó.
3. **Configuración RF efectiva:** el setup/patch/comando soportó la combinación.

Sólo los dos primeros pueden deducirse del ACK. La tercera requiere contrato exacto del SDK y, para combinaciones sensibles, medición. El mismo criterio aplica a potencia: una tabla puede seleccionar el valor más cercano sin reportarlo al host.

## RX

El camino funcional es RF core → entrada de cola → `RadioIF_Packet` → `DataTask` → cola de salida → USB/host → callback. La entrega básica está implementada y IEEE/GFSK tienen evidencia OTA histórica. Sus límites funcionales son:

- PROP interpreta en orden incorrecto los bytes RSSI y estado/LQI anexados por el RF core (F-01).
- La cola de salida puede descartar sin un contador o evento observable por el usuario (F-08).
- Los eventos asíncronos no ofrecen una semántica de secuencia enteramente compatible con el correlador host (F-02).
- Filtrado IEEE y auto-ACK están desactivados; FeralRF entrega tramas, no ejecuta automáticamente la función MAC de un dispositivo.
- `RX_STOP` tuvo fallos históricos reproducibles, pero el material disponible no prueba una única causa; se conserva como inconcluso en causalidad (F-14/F-16).

## TX

La API valida un máximo genérico de 125 bytes y recibe ACK cuando `ControlTask` acepta el trabajo. La ejecución RF ocurre después. No hay respuesta final correlacionada que distinga emisión terminada, fallo tardío o cancelación (F-04). Además:

- BLE advertising admite como máximo 31 bytes en el driver, aunque el límite temprano sea mayor.
- IEEE admite 125 bytes entregados al comando; el RF core maneja PHR/FCS según sus opciones.
- PROP tiene buffer interno de 255 bytes, pero el contrato público común lo limita a 125.
- `transmit_frame()` y `transmit()` convergen en el mismo camino: la primera no agrega campos MAC.
- BURST/CONTINUOUS prueban programación repetida, pero no entregan al host una garantía observable del conteo final ni de cada fallo.

## Criptografía

La implementación usa drivers TI y cubre TRNG; AES ECB, CTR, CBC, CCM y GCM; SHA-256; ECDH P-256/Curve25519; y ECDSA P-256. El alcance no convierte esas primitivas en seguridad de BLE, Wi-SUN, Sidewalk o Wireless M-Bus: faltan integración de claves, nonces, contadores, autenticación, almacenamiento y procedimientos de cada protocolo.

Hallazgos funcionales:

- El host permite seleccionar Curve25519 para ECDSA, pero el firmware ECDSA usa P-256 (F-09).
- Los máximos de CCM/GCM aceptados por host/transporte pueden superar buffers o respuestas internas (matiz de F-10).
- Las pruebas vectoriales Python verifican principalmente la biblioteca `cryptography`; no constituyen evidencia de ejecución del acelerador CC1352P7.
- Existen pruebas *smoke* de hardware en el árbol, pero no fueron ejecutadas ni se incorporan retroactivamente como evidencia de Fase I.

## Emulación e integraciones

“Emular” exige definir el rol. Los helpers actuales son útiles como generadores reproducibles:

- IEEE genera una baliza y una trama de datos con cabeceras MAC plausibles, pero no mantiene asociación, secuencia dinámica, espera de ACK ni seguridad.
- Wireless M-Bus forma algunos campos reconocibles, pero no una trama completa válida para T1; además elige `msk_868_50k` pese a existir el preset T.
- OOK transmite patrones de bits nominales; el modo de paquete del RF core introduce preámbulo, sync, longitud y CRC incompatibles con atribuir interoperabilidad PT2262/EV1527/Hörmann.

KillerBee sí es una intención de adaptación externa y por ello se evalúa frente a su contrato: el mapeo básico IEEE existe, pero los problemas de empaquetado, dependencias y manejo del dispositivo impiden considerarlo reproducible de extremo a extremo.

## Relación con la evidencia experimental

- IEEE raw/frame y PROP GFSK demuestran entrega OTA entre CatSniffer, no interoperabilidad con Zigbee, Thread, Wi-SUN ni otro tercero.
- Los resultados BURST/CONTINUOUS respaldan ejecución básica y también sus límites de control/finalización.
- OOK no debe clasificarse defectuoso sólo por carecer de validación; queda **implementado no validado**, salvo los nombres de emulación que sí exceden lo construido.
- CW/PRBS quedaron demostrados como modos de prueba.
- KillerBee no alcanzó evidencia reproducible integral.
- Los presets añadidos después del subconjunto histórico no heredan automáticamente el estado 10/10 del catálogo de Fase I.

Véase [[../../06 - Experimentos/00 - Consolidación y evaluación experimental FeralRF/03 - Capacidades demostradas y catálogo de presets|catálogo experimental]] y [[04 - Brechas verificadas y limitaciones de implementación]].
