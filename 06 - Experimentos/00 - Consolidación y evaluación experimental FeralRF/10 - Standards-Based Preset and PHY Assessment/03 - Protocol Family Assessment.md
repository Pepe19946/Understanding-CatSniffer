# Fase 2 — Evaluación por familia de protocolo

## Tabla ejecutiva

| Familia | PHY/framing esperado | Implementación actual | Evidencia máxima | Brecha crítica | Disposición |
|---|---|---|---|---|---|
| RF propietaria FSK/GFSK | definida por configuración TI propia | presente | L2 en 14 presets | medición física | mantener baselines |
| OOK | setup + patch OOK y lifecycle seguro | parcial/inestable | L1 reciente | lock, completion y cleanup | diagnóstico prioritario |
| MSK/4-(G)FSK | setup/overrides oficiales específicos | valores reservados en setup FSK | L1 de control | backend incorrecto | no validar; investigar export TI |
| W‑M‑Bus | EN 13757‑4, modos y coding/framing propios | raw TI | L2 raw asociado | no framer/peer | catálogo candidato separado |
| Wi‑SUN FAN | IEEE 802.15.4 SUN FSK + MAC/hopping/IP/security | raw TI | L2 raw asociado | PHY exacta y stack ausentes | catálogo candidato separado |
| Sidewalk | stack Sidewalk + BLE/FSK/LoRa según enlace | raw TI FSK | L2 raw asociado | perfil exacto/stack/peer ausentes | no usar claim Sidewalk |
| MIOTY | ETSI TS‑UNB, telegram splitting y radio-bursts | un paquete FSK TI | L1/0 peer | arquitectura de aire ausente | retirar como preset funcional |
| IEEE 802.15.4 | PHY/MAC 802.15.4 por comandos dedicados | backend dedicado | L2 histórico | validación externa/metadata | mantener separado |
| Zigbee | 802.15.4 + NWK/APS/ZDO/aplicación/seguridad | no existe pila | ninguno | stack completo | fuera de claims actuales |

## Capas que el nombre no puede ocultar

```text
Aplicación/perfil      Zigbee clusters | Wi-SUN apps | W-MBus app | Sidewalk app | MIOTY service
Red/seguridad          NWK/APS          | IPv6/RPL   | EN/OMS     | Sidewalk     | TS-UNB MAC
MAC/framing            802.15.4 MAC     | SUN MAC     | EN13757-4  | Sidewalk     | split bursts
PHY/comandos RF        IEEE dedicated   | SUN FSK     | mode S/T/C/N| FSK/LR      | TS-UNB
Hardware RF            CC1352P7 + frontend + antenna + switch + regulación
```

Hoy FeralRF alcanza la PHY/comandos propietarios para P01–P31 y una ruta IEEE dedicada fuera del catálogo. Las capas superiores nominales no están presentes.

## Criterios mínimos para afirmar cada familia

- **W‑M‑Bus:** perfil/mode explícito, framing/coding/CRC estándar, frecuencia y espectro medidos, peer independiente.
- **Wi‑SUN:** operating mode/channel plan explícitos, SUN FSK exacta, whitening/FEC según modo, MAC/hopping y peer certificado para L4.
- **Sidewalk:** configuración obtenida de SDK/especificación autorizada, stack y peer Sidewalk; un enlace FSK genérico no basta.
- **MIOTY:** vector ETSI reproducible, splitting/coding/interleaving/burst scheduling y recepción por implementación independiente.
- **Zigbee:** pila certificable o de referencia sobre IEEE 802.15.4, formación/asociación/red/seguridad y peer Zigbee.

## Priorización

1. Conservar FSK/GFSK raw como controles instrumentales.
2. Resolver OOK por valor diagnóstico inmediato.
3. Corregir 2.4 y el contrato de modulaciones sólo tras exportación TI trazable.
4. Tratar Wi‑SUN/W‑M‑Bus como líneas de producto separadas, no como simples filas de parámetros.
5. No dedicar más OTA simétrica a MIOTY/Sidewalk/Zigbee hasta disponer de su arquitectura/peer.

