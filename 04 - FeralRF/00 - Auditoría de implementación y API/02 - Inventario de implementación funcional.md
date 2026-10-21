# Inventario de implementación funcional

## Matriz de subsistemas

| Capacidad | Entrada principal | Ruta de fuente | Nivel real | Estado de evidencia |
|---|---|---|---|---|
| Conexión/descubrimiento | `Radio.connect`, `list_devices` | `radio.py:L136-L257` | puerto serie y filtro VID/descripción | Usado en Fase I; identidad débil |
| Framing | `_send_command`, `_read_response` | `radio.py:L263-L397`, `protocol.py:L18-L90`, `protocol.c` | COBS + CRC16 + secuencia | Implementado; manejo asíncrono incompleto |
| Init/info/stats | `init`, `get_stats` | `radio.py:L399-L453,L553-L598`; handlers C | control/telemetría parcial | Demostrado |
| PHY/canal/potencia | `set_phy`, `set_channel`, `set_power` | `radio.py:L455-L489`; `command_processor.c`; `radio_if.c` | configuración lógica, aplicada al operar | Demostración indirecta/OTA parcial |
| Propietario | `configure_prop`, presets | `radio.py:L490-L538`, `presets.py`, `RadioIF_setPropConfig` | mutación de estructuras RF; sin resultado | RX/TX observado; defectos de metadatos y enums |
| RX | `start_rx`, `read_packets`, `stop_rx` | `radio.py:L599-L724`; `DataTask`; `RadioIF_start/stopRx` | RX raw y metadatos | IEEE demostrado; stop problemático; prop defectuoso |
| TX raw/frame | `transmit`, `transmit_frame` | `radio.py:L725-L751`; `ControlTask`; `RadioIF_transmitRaw` | una trama raw | Demostrado IEEE y propietario |
| BURST/CONTINUOUS | `burst`, `continuous`, `stop_transmit` | `radio.py:L752-L810`; `control_task.c:L264-L407` | scheduler cooperativo sin completion | Actividad OTA parcial, conteo no verificable |
| CW/PRBS | `tx_cw`, `tx_prbs`, `tx_test_stop` | `radio.py:L811-L863`; `radio_if.c:L2207-L2286` | comandos de prueba TI | ACK demostrado; RF físico no medido |
| Criptografía | métodos `random_*`, AES ECB/CTR/CBC/CCM/GCM, SHA, ECDSA, ECDH | `radio.py:L864-L1149`; `crypto_engine.c` | operaciones por drivers TI | Implementado; sin evidencia de Fase I |
| Jamming | `start_jam`, `stop_jam` | `radio.py:L1151-L1205`; `ControlTask_onJam*`, `RadioIF_startJamSession` | continuo básico BLE/IEEE | Experimental, no validado aquí |
| KillerBee | `KBCatSniffer` | `integrations/killerbee.py` | adapter de sniff/inject/jam | bloqueado por dependencia en Fase I |
| Emulación | módulos `emulation` | `python/feralrf/emulation/*` | envío repetido de payloads preparados | No es stack ni interoperabilidad probada |
| Spectrum | `_spectrum.py`, builders huérfanos | `_spectrum.py`, `commands.py` | tipos/gráficas y builders sin IDs/handlers/API | No implementado extremo a extremo |
| Jamming reactivo/patrones | `_jamming.JammingPolicy`, constantes pending | `_jamming.py`, `enums.py` | estructura host sin contrato de firmware | Pendiente/no implementado |

## Dispositivo y transporte

`list_devices()` filtra por VIDs conocidos y por VID `0x1209` con descripción que contenga “Bridge”; con selección automática toma el primer candidato (`radio.py:L136-L156`). No enlaza bridge y shell por serial físico inequívoco ni comprueba que `FERALRF1` sea único. `connect()` puede resetear y luego llama a `init()`.

`RADIO_INIT` responde ACK después de `ControlTask_onRadioInit()`. `GET_INFO` retorna tres bytes de versión, capacidades y 8 bytes constantes de serial. `GET_STATS` retorna nueve contadores de 32 bits cuando la capacidad LL está disponible. Son estadísticas del backend CC1352P7, no de los buffers del RP2040 ni de la cola de salida.

Los timeouts son host-side y la correlación depende de secuencia. Los frames malformados durante `_read_response()` pueden desaparecer por un `except Exception: continue`, sin contador ni diagnóstico al caller. No hay una operación pública de resynchronization/flush ni estado transport/RF agregado.

## Configuración RF

### PHY, canal y potencia

`set_phy(phy, channel)` serializa dos bytes; el handler valida el enum y llama a `ControlTask_onSetPhy()`. `RadioIF_setPhy()` detiene backends/sesiones y cambia selección, pero no abre ni verifica el nuevo modo. `set_channel()` y `set_power()` almacenan un byte/int8; se aplican cuando una operación RF crea/usa su comando. Los builders enmascaran valores, de modo que algunos enteros fuera de rango se truncan antes de llegar al firmware.

La potencia efectiva se obtiene con `RF_TxPowerTable_findValue()` y fallback a la entrada más cercana (`radio_if.c:L280-L317`). La API no informa si el valor fue exacto, aproximado o si empleó PA normal/alta potencia.

### Configuración propietaria

`configure_prop()` envía frecuencia, symbol rate, deviation, RX bandwidth, modulation, preamble, sync word y whitening. El handler exige sólo longitud 16/18, decodifica y llama a `RadioIF_setPropConfig()`, cuyo retorno es `void`; después emite ACK. La función actualiza `Prop0_cmdFs`, `Prop0_cmdPropRadioDivSetup`, `Prop0_cmdPropRxAdv` y TX, sin pedir al RF core que confirme parámetros. En OOK activo hay cambios que sólo quedan en estructuras hasta una reapertura posterior.

No existe validación coherente de banda, divisor, overflow de cálculo, tasa, desviación, ancho o modulación. El SDK exacto declara `modType` 0=FSK, 1=GFSK, 2=OOK y 3…7 reservados (`.../driverlib/rf_prop_cmd.h:L475-L478,L581-L584`). Python documenta y usa 4, 5 y 6 como MSK, 4-FSK y 4-GFSK (`radio.py:L502-L509`, `presets.py`), por lo que esos nombres no corresponden al contrato del driver auditado.

Los presets son diccionarios de parámetros para `configure_prop`; no implementan Wireless M-Bus, Wi-SUN, Sidewalk o MIOTY. Un nombre de preset sólo expresa una configuración nominal.

## Recepción

### Traza extremo a extremo

1. `Radio.start_rx()` envía `RX_START` y espera ACK.
2. El handler comprueba que TX no esté ocupado, marca RX solicitada y agenda evento; ACK precede al arranque real.
3. `DataTask` llama a `RadioIF_startRx()`, que abre/configura el backend correspondiente.
4. Si falla, firmware emite `RSP_ERROR` asíncrono; si funciona, callbacks/colas TI proporcionan entradas.
5. `RadioIF_process*Packets()` produce `RadioIF_RxPacket` en cola software de profundidad 8.
6. `DataTask` extrae hasta ocho, arma `RSP_RX_PACKET` y `OutputIF` lo encola.
7. `Radio.read_packets()` decodifica timestamp, canal, RSSI firmado, LQI, CRC, payload y metadatos BLE opcionales.

`read_packets()` puede producir `RxStreamError`; `read_one_packet()` lo omite y continúa, por lo que integraciones que sólo usan el helper pierden ese diagnóstico (`radio.py:L636-L724`). No valida de manera estricta que la longitud declarada del paquete coincida con los bytes disponibles.

### Metadatos por PHY

IEEE 802.15.4 extrae RSSI, correlación/CRC y timestamp en el orden configurado, filtra CRC fallido y reporta `s_channel`. BLE usa parsing específico. Propietario configura `bIncludeHdr`, `bIncludeCrc`, `bAppendRssi`, `bAppendTimestamp` y `bAppendStatus` (`radio_if.c:L1631-L1642`), pero el parser supone `status, RSSI, timestamp` (`radio_if.c:L1706-L1754`). El layout del SDK y su uso en Sidewalk es `header, payload, CRC, RSSI, timestamp, status`. En consecuencia:

- usa el RSSI real como status/CRC;
- reporta el primer byte de timestamp como RSSI;
- forma un timestamp desplazado que incorpora status;
- puede descartar paquetes según un bit del RSSI.

Este es un defecto confirmado por fuente y coherente con Fase I. El LQI propietario siempre se fija en cero.

El campo `channel` propietario copia `s_channel`. `configure_prop()` cambia frecuencia pero no ese estado. Además `RadioIF_setPhy(..., channel=0)` conserva el canal previo. Así, cambiar desde IEEE canal 25 a propietario y configurar 902.2 MHz puede reportar `channel=25`: es etiqueta lógica residual, no canal derivado de frecuencia (`radio_if.c:L1733` y setters asociados).

### Stop RX

`stop_rx()` envía `RX_STOP` con hasta tres intentos y espera ACK. Mientras espera, ignora `RX_PACKET`. Firmware agenda stop y ACK; `DataTask` atiende stop antes de nuevas operaciones. La arquitectura permite que paquetes ya producidos compitan con el ACK en colas y transporte. La política host descarta la mayoría de esos paquetes por secuencia. Esto explica los `0x90` “unexpected”, pero no demuestra por qué faltó el ACK histórico; cola llena, carrera interna, pérdida en bridge y diferencia de binario permanecen hipótesis.

## Transmisión

### Una trama

`transmit()` construye `TX_RAW`; `transmit_frame()` usa `TX_FRAME`. Ambos terminan en el mismo tipo de slot de control y `RadioIF_transmitRaw()`, que despacha a BLE, IEEE o propietario. El ACK significa payload aceptado/copiado para ejecución diferida, no TX completado. Un fallo posterior de raw genera `RSP_ERROR` asíncrono. El límite de control es 125 bytes; Python no lo valida explícitamente y el builder usa un campo de longitud de un byte.

### Burst y continuo

`TX_BURST` copia payload, count e intervalo a estado estático. `TX_CONTINUOUS` copia payload e intervalo y permanece activo. En cada poll, `ControlTask_processTxOperations()` transmite a lo sumo una unidad cuando `SysTick` alcanza el deadline (`control_task.c:L264-L407`). Intervalo cero no significa simultaneidad: queda limitado por duración del comando RF y frecuencia de polling.

El ACK sólo confirma aceptación del scheduler. No hay evento de paquete transmitido, progreso, contador final ni completion. Si una transmisión interna falla, el estado burst/continuous se limpia silenciosamente. `TX_STOP` borra operaciones pendientes y jam activo; no devuelve conteos. Por ello un receptor peer con N paquetes no puede usarse como conteo exacto de emisiones y el host no puede distinguir final normal de fallo.

### CW y PRBS

`tx_cw()` / `tx_prbs()` fijan potencia y envían `TX_CW` / `TX_PRBS`. Firmware abre el modo seleccionado, ejecuta FS, aplica potencia y publica un comando de prueba; ACK indica que se pudo programar (`RF_postCmd` handle válido). `tx_test_stop()` cancela sesión. No existe lectura de potencia efectiva, frecuencia medida ni estado RF posterior.

## Criptografía

`crypto_engine.c` envuelve drivers de polling TI para TRNG, AES (ECB/CTR/CBC/CCM/GCM), SHA-256, ECDSA P-256 y ECDH P-256/Curve25519. Los comandos devuelven respuestas específicas `0x95`…`0x9C`, no sólo ACK. ECB, CTR y CBC comparten `RSP_AES`; CCM y GCM tienen responses propios. Python valida varias longitudes de clave/nonce/tag.

Hay dos límites de contrato:

- `ecdsa_sign()`/`ecdsa_verify()` aceptan el selector `curve25519`, pero firmware ECDSA sólo implementa P-256 y responde curva no soportada; Curve25519 sólo aplica a ECDH.
- CCM/GCM permiten longitudes Python mucho mayores, aunque el protocolo tiene payload máximo de 255 bytes. Falta validar el tamaño total serializado antes de enviar.

No existe evidencia experimental de Fase I para certificar resultados contra vectores conocidos.

## Funciones de mayor nivel

### KillerBee

`KBCatSniffer` anuncia sniff, set channel, inject y PHY jam. `pnext()` usa `read_one_packet()`, perdiendo eventos de error y metadatos excepto data/CRC/RSSI/fecha. `inject()` elimina los dos últimos bytes si la entrada tiene al menos cinco, asumiendo FCS presente, y no retorna evidencia de TX. La página 802.15.4 se almacena pero no afecta firmware.

El extra `killerbee = ["killerbee>=3.0.0"]` no bastó para instalar en Fase I y la integración requiere hooks del patch incluido en `docs/killerbee-catsniffer.patch`. Por tanto, el bloqueo observado es de empaquetado/integración; no demuestra un defecto RF.

### Emulación

Los módulos emiten tramas/payloads preconstruidos en bucles: IEEE incluye ejemplos de framing; Sub-GHz incluye cargas de sensor/W-MBus; OOK tiene flujo especial y reset. No hay asociación, MAC completa, seguridad, enrutamiento, commissioning ni aplicación interoperable. El preset W-MBus/MSK usa además `mod_type=4`, reservado por el SDK.

### Jamming y spectrum

Jamming actual acepta sesiones básicas sólo para BLE e IEEE en firmware. Python valida canal/duración, pero no comprueba de antemano el PHY seleccionado. `_jamming.JammingPolicy` describe patrones/reactividad sin un comando extremo a extremo.

`commands.py` contiene builders `spectrum_scan`, `spectrum_stop` y `rx_set_promiscuous`, y `_spectrum.py` contiene resultados/gráficas. No existen IDs correspondientes en `Command`, handlers firmware ni métodos públicos de `Radio`; no constituyen una función implementada. Los únicos IDs pending son jamming reactivo/pattern. `tx_queue.c` tampoco tiene callers en la ruta actual y se considera deuda/remanente, no capacidad pública.

## Profundidad real de protocolos

| Nombre | Implementación observada | No observado en la ruta pública actual |
|---|---|---|
| IEEE 802.15.4 | PHY, RX/TX raw/frame, canal, metadata y adapter KillerBee | MAC completa, asociación, seguridad, Zigbee/Thread |
| BLE | PHY/raw y clasificación LL limitada; jamming básico | host/controller BLE completo, GAP/GATT actual (fue retirado) |
| Proprietary FSK/GFSK/OOK | configuración RF genérica y RX/TX raw | protocolo de aplicación o interoperabilidad específica |
| Wireless M-Bus | presets/payloads de emulación | stack/link/application; modulación nominal MSK válida |
| Wi-SUN/Sidewalk/MIOTY | nombres/presets o intención documental | pilas completas y flujos interoperables |

El historial confirma una refactorización explícita que retiró la pila BLE (`785a093` firmware, `d98e1ce` Python), preservando PHY/LL manager. Las funciones BLE internas residuales de `radio_if.c` no son API ni prueba de una pila accesible.

