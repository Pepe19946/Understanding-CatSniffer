# Fuentes hardware CatSniffer v3.1

**Conocimiento derivado:** [[Arquitectura CatSniffer v3.1]], [[RP2040, CC1352P7 y SX1262 - Quién hace qué]], [[RP2040 - El procesador de interfaz]], [[CC1352P7 - El procesador de radio programable]] y [[SX1262 - El transceptor controlado por RP2040]].

## Línea base Git

- Repositorio: `CatSniffer/`
- Tag: `v3.1`
- Tipo: tag ligero (ref directa a commit)
- Commit: `5b99d984933a8b8c36e789853c08d3a2a785bed4`
- Fecha: `2024-01-10T19:07:43-05:00`
- Asunto: `Merge pull request #62 from pwnlabmx/master`
- Método de lectura: `git show`, `git ls-tree`, `git cat-file`; no se cambió el checkout.

## Fuentes primarias almacenadas en `CatSniffer:v3.1`

| Fuente | Tipo | Uso | Advertencia |
| --- | --- | --- | --- |
| `hardware/CatSniffer.kicad_sch` | Esquema KiCad, hoja plana | Componentes, nombres de red, conectores y controles | Bloque de título rev `2.0`; algunas etiquetas BOOT/reset difieren del PCB |
| `hardware/CatSniffer.kicad_pcb` | PCB KiCad | Conectividad enrutada, huellas, referencias y serigrafía | Bloque de título rev `v3.2`, serigrafía v3.1 |
| `hardware/CatSniffer.kicad_pro` | Proyecto KiCad | Contexto de proyecto | Difiere del checkout actual |
| `hardware/CatSniffer.pdf` | Exportación de esquema | Referencia visual | Presente en v3.1; mismo blob que HEAD (`a5332e83...`) |
| `README.md` | Documentación oficial en ese commit | Descripción V3, CC1352P1 y puente RP2040 | No sustituye conectividad ni evidencia física |
| `hardware/README.md` | Índice mínimo | Confirma categorías de hardware | Sin detalle técnico |

No se encontró un pinout dedicado ni BOM/Gerbers claramente identificados en el árbol del tag.

## Referencias locales en el tag

- `hardware/Datasheet/0900FM15D0039.pdf` — filtro RF; útil solo para la ruta RF.
- `hardware/Datasheet/LAUNCHXL-CC1352P-2_SCHEMATIC.pdf` — esquema de referencia TI; no es el esquema de CatSniffer.
- `hardware/Datasheet/SX1262MB2xAS_e499v01b_sch_layout.pdf` — referencia de diseño SX1262; no prueba el montaje CatSniffer.
- `hardware/Datasheet/jti-an100.pdf` — referencia Tag-Connect/debug.

## Enlaces de fabricante incrustados en el esquema v3.1

- RP2040: `https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf` (U3).
- CC1352P: `https://www.ti.com/lit/ds/symlink/cc1352p.pdf` (U8; la propiedad exacta del componente dice `CC1352P1F3RGZT`).
- SX1262: `https://www.mouser.mx/datasheet/2/761/SEMT_S_A0008596633_1-2575829.pdf` (U7).
- RFSW8006: `https://www.mouser.mx/datasheet/2/412/fsw8006q_product_data_sheet-1307843.pdf` (U2).
- PE42421: `https://www.mouser.mx/datasheet/2/281/pe42421ds-1671115.pdf` (U6).

Se usaron solo para clasificar a alto nivel los dispositivos; no se hizo estudio detallado de datasheets.

## Fuentes de firmware usadas como contraste

Línea base del repositorio: `CatSniffer-Firmware/` rama `v3.x`, commit `c0cd5a45e019dbd14ed11d039aacb13f300e5731`.

- `RP2040/README.md`
- `RP2040/catsniffer/README.md`
- `RP2040/catsniffer/CMakeLists.txt`
- `RP2040/catsniffer/prj.conf`
- `RP2040/catsniffer/west.yml`
- `RP2040/catsniffer/boards/rpi_pico.overlay`
- `CC1352P7/README.md`
- `CC1352P7/*/*.syscfg`
- `CC1352P7/*/targetConfigs/CC1352P7.ccxml`
- `CC1352P7/*/Release/*.{hex,out}` donde existen

Estas rutas solo establecen correspondencia de proyecto, target y ecosistema; no se analizaron flujos internos.

## Advertencias de revisión y procedencia

### Evidencia física añadida tras la Fase 1

- **Observado en hardware — 2026-09-25:** U8 en la CatSniffer v3.1 del usuario está marcado `CC1352P74`.
- La unidad física se trata por tanto como CC1352P7-family para el análisis de firmware.
- Esto contradice el valor `CC1352P1F3RGZT` del esquema histórico v3.1, pero no establece cuándo ni por qué se produjo el cambio.

1. `v3.1` no es el checkout actual de `CatSniffer/`; se leyó como objeto Git histórico.
2. Los blobs v3.1 de `.kicad_sch`, `.kicad_pcb` y `.kicad_pro` difieren de HEAD. No se usaron los diseños v3.3 como sustituto.
3. El tag v3.1 dice U8=`CC1352P1F3RGZT`; el firmware actual denomina `CC1352P7` a la familia 3.x. Esto queda sin resolver.
4. El commit posterior `08e31d6ef21c7eb41a7c61fc1986f99114e39233` (`2026-02-12`, `feat(HW):CC1352P74T0RGZR_Updated`) no pertenece al historial anterior a v3.1 y solo demuestra una actualización posterior visible, no el componente físico de v3.1.
5. J2 SWD existe en el esquema v3.1, pero no se encontró una huella/referencia J2 en el PCB v3.1; J3 cJTAG sí aparece en ambos.
6. La única evidencia física incorporada es el marcaje `CC1352P74` comunicado tras la Fase 1; no se realizó validación eléctrica ni funcional.
7. `CC1352P7/README.md` menciona `Sniffle_CC1352P_7`, pero ese directorio no está en el checkout de firmware usado; no se contó como proyecto local disponible.
