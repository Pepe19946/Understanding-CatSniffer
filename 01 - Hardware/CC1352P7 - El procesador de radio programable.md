# CC1352P7 — El procesador de radio programable

## En una frase

El CC1352P7 es un segundo computador dentro de CatSniffer: ejecuta su propia imagen y controla una radio integrada capaz de operar en Sub-1 GHz y 2.4 GHz.

## Modelo mental

No pienses en CC1352P7 como “un módulo UART”. UART es solo el camino por el que conversa con [[RP2040 - El procesador de interfaz|RP2040]]. Detrás de ese camino existe otra CPU, otra flash, otro entorno de build y otra aplicación.

**Sub-GHz** significa frecuencias por debajo de 1 GHz, como 433/868/915 MHz. **2.4 GHz** es otra banda usada por tecnologías como BLE e IEEE 802.15.4. Que el chip pueda operar esas PHY no implica que toda pila de protocolo esté implementada.

## Qué hace realmente

- arranca su propia imagen desde flash;
- recibe datos/comandos por UART;
- ejecuta una aplicación RF independiente;
- usa su subsistema de radio integrado y drivers TI;
- puede entrar al bootloader serie mediante señales BOOT/RESET controladas por RP2040.

Un **protocol stack** o pila de protocolo implementa varias capas de una tecnología, no solo la capacidad de modular o recibir paquetes. Una radio IEEE 802.15.4 no equivale por sí sola a una pila Zigbee o Thread.

## Firmware oficial y límite de visibilidad

La imagen normal oficial para CatSniffer v3.x está disponible como HEX, pero su fuente fue retirada del baseline estudiado. Un HEX es **firmware binario**: puede cargarse, pero no permite seguir su implementación interna como código fuente. Otros proyectos TI especializados sí conservan fuente, pero no deben usarse como sustituto del sniffer normal.

[[Firmware CC1352P7]] registra esos proyectos y el ecosistema CCS/SysConfig/SimpleLink/TI-RTOS.

## Relación con FeralRF

[[Arquitectura FeralRF|FeralRF]] ofrece una imagen alternativa, fuente-visible, para este mismo procesador. Mantiene el camino físico:

`PC → USB → RP2040 → Cat-Bridge → UART → CC1352P7`

Lo que cambia es la aplicación que el CC ejecuta y el protocolo que interpreta; no cambia quién termina USB.

## Revisión física y procedencia

- **Observado en hardware:** U8 está marcado `CC1352P74`, tratado como familia P7.
- **Documentado históricamente:** el esquema del tag v3.1 nombra un P1.
- **Conclusión:** para la unidad física se sigue P7, conservando la discrepancia P1/P7 como advertencia de procedencia.

## Concepción errónea común

**“El RP2040 ejecuta el driver del radio CC como ejecuta el driver SX1262.”** No. El firmware que opera la radio integrada del CC corre dentro del propio CC1352P7. RP2040 transporta el tráfico y controla boot/reset.

## Por qué esto importa después

Este límite determina dónde leer código. Los comandos Cat-Bridge normales terminan conceptualmente en CC1352P7. También permite entender el valor de FeralRF: vuelve visible y modificable una parte que el baseline oficial entrega principalmente como binario.

## Evidencia

- **Observado en hardware:** marcaje U8 informado por el usuario.
- **Observado en repositorio:** proyectos y HEX bajo `CatSniffer-Firmware/CC1352P7/`.
- **Fuente de placa:** `CatSniffer:v3.1:hardware/CatSniffer.kicad_sch` y PCB.
- Síntesis: [[Firmware CC1352P7]], [[Arquitectura CatSniffer v3.1]], [[Fuentes firmware oficial]].

## Comprueba tu comprensión

1. ¿Por qué UART no convierte a CC1352P7 en un periférico pasivo?
2. ¿Qué diferencia hay entre una capacidad RF y una pila completa de protocolo?
3. ¿Qué parte del camino oficial se conserva al instalar FeralRF?
4. ¿Por qué los proyectos TI especializados no revelan necesariamente el sniffer normal?

**Anterior:** [[RP2040 - El procesador de interfaz]]  
**Siguiente:** [[SX1262 - El transceptor controlado por RP2040]]  
**Detalle:** [[Firmware CC1352P7]]

