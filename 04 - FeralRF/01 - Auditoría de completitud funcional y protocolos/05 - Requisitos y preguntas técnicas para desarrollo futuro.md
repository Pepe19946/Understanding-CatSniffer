# Requisitos y preguntas técnicas para desarrollo futuro

## Propósito

Este documento no es una hoja de ruta ni ordena trabajo. Separa decisiones de producto, obligaciones del contrato actual, requisitos condicionados a un rol superior y evidencia aún necesaria. Una capacidad sólo se convierte en requisito cuando el proyecto selecciona explícitamente el rol correspondiente.

## Decisiones de alcance

1. **Definición del producto:** ¿herramienta PHY/raw, toolkit de construcción/parseo, emulador de dispositivos o pilas interoperables seleccionadas?
2. **Semántica de los nombres:** ¿los presets con nombre de protocolo se conservarán como “parámetros experimentales” o se pretende conformidad con el protocolo indicado?
3. **Garantía de operación:** ¿éxito significa aceptación del comando, programación del RF core, fin de TX, número de intentos o recepción por un tercero?
4. **Compatibilidad:** ¿la API puede corregir contratos ambiguos/inválidos o debe versionar comandos/presets?
5. **Evidencia:** ¿qué combinaciones de placa, SDK, firmware, región y equipo de referencia constituyen soporte oficial?

## Requisitos del contrato actual, independientemente de protocolos superiores

| Área | Requisito a esclarecer/cumplir | Razón |
|---|---|---|
| Identidad | Identificador real de artefacto, firmware, SDK y unidad/placa | Hace comparables fallos y campañas |
| Async RX | Regla única para secuencia, multiplexado, drenado y callbacks | Evita pérdida determinista durante comandos síncronos |
| TX diferido | Estado observable de inicio, fin, fallo y stop | El ACK temprano no prueba ejecución |
| PROP metadata | Layout conforme al RF command y semántica explícita de CRC/header/canal | Los valores actuales no son fiables |
| Configuración | Separar solicitado, aceptado y efectivo; devolver errores RF | ACK no certifica hardware |
| Modulación | No presentar 4/5/6 como modos soportados por el campo actual | El SDK los reserva |
| Límites | Validación uniforme antes de I/O por comando y PHY | Evita truncamiento y fallos tardíos |
| Colas/stats | Drops y ocupación observables de extremo a extremo | Permite atribuir pérdidas |
| Integración RP2040 | Asociación robusta Bridge/Shell y estado de banda/reset | Evita controlar otra interfaz |
| Documentación | Calificar BLE, IEEE y presets por nivel funcional | Evita inferencias de pila completa |

## Requisitos condicionados por rol

### Si el rol sigue siendo PHY/raw multiprotocolo

No son obligatorias pilas Zigbee/Thread/Wi-SUN/Sidewalk/W-MBus/MIOTY. Sí se necesita documentar con precisión:

- bytes que entrega/espera cada PHY;
- elementos generados por el RF core;
- máximos por modo;
- parámetros efectivos o no verificables;
- metadatos RX y CRC/FCS;
- nivel de validación experimental de cada preset.

Preamble, CRC, *whitening*, FEC y orden de bits configurables serían mejoras opcionales si se desea una herramienta PHY más general.

### Si se requiere IEEE 802.15.4 MAC

Debe definirse el rol (sniffer, inyector, dispositivo, coordinador) y luego seleccionar requisitos: filtrado, FCF/addressing, DSN, ACK, CSMA-CA, temporización, reintentos, asociación, seguridad y tablas de estado. Zigbee y Thread son decisiones adicionales, no sinónimos de ese MAC.

### Si se requiere Wireless M-Bus

Debe fijarse versión de EN 13757, modo (S/T/C/N…), formato y rol (medidor/colector). Para ese rol habrá que establecer PHY/codificación, longitud, L/C/M/A/CI, CRC por bloques, parser/construcción, temporización, dirección y seguridad. La OMS puede perfilar el uso, pero el criterio normativo debe referir la versión aplicable de EN 13757.

### Si se requiere Wi-SUN

Debe identificarse perfil y versión FAN, región y rol. Un PHY MR-FSK conforme a IEEE 802.15.4g es sólo la base; hopping, MAC/LLC, joining, 6LoWPAN, IPv6, RPL, ICMPv6 y 802.1X/EAP-TLS deben evaluarse según dicho perfil. Certificación PHY y certificación de perfil son resultados distintos.

### Si se requiere Amazon Sidewalk

Debe fijarse transporte/región y acceso al ecosistema. Para Sub-G FSK se necesita al menos GFSK y framing/PHR/FCS conforme, rejilla/hopping, enlace y seguridad; registro y operación de red requieren además los mecanismos/ecosistema Sidewalk. El preset de 250 kbit/s necesita primero una fuente normativa que establezca su rol.

### Si se requiere BLE más allá de advertising

Debe decidirse rol y alcance: scanner/advertiser, central/peripheral, conexiones, GAP, GATT, SMP y seguridad. La pila retirada no debe reactivarse por código residual sin un contrato actualizado. Para el alcance actual debe verificarse la semántica de advertising 1M/Coded/2M y del canal auxiliar fijo.

### Si se requiere MIOTY

Debe seleccionarse el modo TS-UNB de ETSI TS 103 357-2 y comprobar si el CC1352P7 puede realizar la modulación con RF core/patch disponible. Se requieren división/codificación del telegrama, planificación pseudoaleatoria de ráfagas en tiempo/frecuencia, framing, MAC y seguridad del rol. Un preset FSK de 396 baud no satisface esas condiciones.

### Si se requieren emuladores OOK

Cada personalidad necesita una definición comprobable de pulsos/símbolos, sincronización, repetición, identificador/código y framing. El camino de paquete genérico TI puede no ser apropiado. “Patrón canónico” debe permanecer etiquetado como estímulo hasta validarse con un receptor independiente.

## Preguntas técnicas abiertas

### RF core y framing

- ¿Qué bytes exactos debe exponer `Packet.data` en IEEE y PROP: PHR/length y CRC/FCS incluidos o retirados?
- ¿Qué opciones de `format_conf` serán públicas y cuáles permanecerán internas?
- ¿Se necesita controlar preámbulo, CRC, sync length, orden, FEC y *whitening* mediante tipos versionados?
- ¿Qué comando/patch TI implementaría realmente cada modulación adicional?
- ¿Cómo se verifica y devuelve frecuencia, tasa, desviación, BW y potencia efectivas?

### Estado y observabilidad

- ¿Qué identificador correlaciona una operación diferida y sus eventos?
- ¿Qué significa `count` en BURST: solicitado, programado, completado por RF core u observado OTA?
- ¿Cómo se contabilizan drops en RadioIF, DataTask, OutputIF, RP2040 y host?
- ¿Cuál es la política de paquetes pendientes al ejecutar `stop_rx()`?
- ¿Puede TX coexistir con jam, o toda operación RF debe usar una exclusión común?

### Protocolos e interoperabilidad

- ¿Qué implementación independiente y qué criterio de aceptación se usarán por familia?
- ¿Se evalúa sólo PHY, framing, procedimientos o perfil completo?
- ¿Qué revisión exacta de cada estándar es contractual?
- ¿Cómo se separan conformidad, certificación y una prueba bilateral exitosa?
- ¿Qué funciones se excluyen deliberadamente y cómo se expresan en API/documentación?

### Criptografía

- ¿ECDSA debe aceptar exclusivamente P-256?
- ¿Los máximos de CCM/GCM deben limitarse a una respuesta, fragmentarse o usar streaming?
- ¿Qué vectores conocidos se ejecutarán en el hardware real?
- Si se implementa seguridad de protocolo, ¿quién administra claves, nonces, contadores, persistencia y anti-replay?

### Integraciones y hardware

- ¿Qué distribución/fork/versión de KillerBee forma parte del soporte?
- ¿KillerBee entrega/espera FCS y metadatos bajo qué convención?
- ¿Cómo se asocian interfaces Bridge/Shell de una misma placa en cada SO?
- ¿Cuál es la tabla placa/revisión/front-end/banda/GPIO verificada?
- ¿Qué restricciones regulatorias y de high-PA debe imponer el producto por región?

## Evidencia mínima para cerrar incógnitas

| Tema | Evidencia necesaria | Qué permite concluir |
|---|---|---|
| Reproducibilidad | Manifest con Git SHA, hash de binarios, SDK, placa y configuración | Qué software/hardware fue probado |
| Modulación | Captura/analizador o receptor independiente por modo | Modulación y parámetros físicos efectivos |
| Framing | Captura de aire y vectores conocidos por PHY | División host/RF core y bytes RX |
| RX_STOP/colas | Traza correlacionada y contadores por etapa | Punto de pérdida y causa, no sólo síntoma |
| TX diferido | Status TI, eventos y captura independiente | Inicio, conteo y finalización reales |
| Presets | Matriz individual, no herencia por familia/nombre | Soporte experimental exacto |
| Interoperabilidad | Peer independiente, versión y rol definidos | Compatibilidad limitada al caso ensayado |
| Crypto | Vectores conocidos ejecutados en CC1352P7 y límites | Corrección de primitivas hardware |
| KillerBee | Entorno instalable reproducible + OTA | Soporte real de integración |

## Dependencias conceptuales, no secuencia de implementación

- La definición del rol precede a declarar “faltante” una capa de protocolo.
- El contrato de framing precede a parsers, emuladores e integración KillerBee.
- El comando/patch RF soportado precede al nombre y validación de una modulación.
- La identidad del artefacto precede a comparar campañas.
- La semántica de finalización precede a criterios de BURST/CONTINUOUS.
- Una prueba PHY no sustituye una prueba de perfil o de interoperabilidad.

## Cierre

El proyecto puede ser funcionalmente completo como herramienta PHY/raw sin implementar pilas completas. Para sostener esa posición necesita corregir los contratos actuales que ya promete y etiquetar con precisión los niveles de soporte. Cualquier extensión protocolaria debe partir de un rol, versión normativa, frontera hardware/software y criterio de interoperabilidad explícitos; no del nombre de un preset.
