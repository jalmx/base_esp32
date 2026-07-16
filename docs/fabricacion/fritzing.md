# Fritzing - Visualización y prototipado

## Archivos disponibles

| Archivo | Descripción |
|---------|-------------|
| `base_esp32.fzpz` | Parte completa Fritzing (schematic + PCB + breadboard) |
| `base_esp32.fzp` | Definición parte (XML) |
| `board.svg` | Vista PCB vectorial |
| `breadboard.svg` | Vista protoboard |
| `schematic.svg` | Vista esquemático |
| `board_render_*.png` | Renders 3D (rojo/azul/blanco) |
| `icon.svg` | Icono parte |
| `connector.json` | Definición conectores |
| `artifact_validation.json` | Validación FZ |

---

## Uso en Fritzing

### Importar parte
1. **File → Import → Part** → Seleccionar `base_esp32.fzpz`
2. La parte aparece en **My Parts** (corazón)
3. Arrastrar a vista **Breadboard**, **Schematic** o **PCB**

### Vistas disponibles

#### Breadboard (Prototipado)
```
Conexiones físicas reales:
- ESP32 DevKit en socket
- Componentes sueltos en protoboard
- Cables jumper macho-hembra
- Alimentación USB/Jack visible
```

#### Schematic (Esquemático)
```
Símbolos normalizados:
- ESP32 como MCU genérico (38 pines)
- Reguladores, driver, sensores con símbolos IEC
- Nets etiquetados: +5V, +3V3, GND, GPIOxx
- Anotaciones: valores, referencias
```

#### PCB (Layout)
```
Vista real PCB fabricada:
- Capas: Top/Bottom copper, Silk, Mask
- Footprints exactos (incluye SMD 0805, SOP-8)
- Mounting holes M3
- Edge cuts 100×80mm R2mm
```

---

## Renders 3D

| Archivo | Vista |
|---------|-------|
| `board_render_red.png` | PCB rojo (máscara roja) |
| `board_render_blue.png` | PCB azul (máscara azul) |
| `board_render_white.png` | PCB blanco |
| `base_esp32_w.png` | Wireframe |

Útiles para:
- Documentación
- Presentaciones
- Verificación visual pre-fabricación

---

## Personalizar parte

### Editar `base_esp32.fzp` (XML)
```xml
<part>
  <title>Base ESP32 DevKit V4</title>
  <description>Training board for ESP32 DevKit V4</description>
  <author>Xizuth</author>
  <date>2024-11-27</date>
  <views>
    <breadboard>breadboard.svg</breadboard>
    <schematic>schematic.svg</schematic>
    <pcb>board.svg</pcb>
  </views>
  <connectors>
    <!-- Definición pines GPIO, power, etc. -->
  </connectors>
  <bus>
    <name>GPIO</name>
    <description>ESP32 GPIOs</description>
  </bus>
</part>
```

### Actualizar SVGs
- **Inkscape** (gratis) para editar `.svg`
- Mantener `viewBox` y IDs de conectores
- `connector*.svg` en `fritzing/` para pines individuales

---

## Exportar para documentación

### Imágenes alta resolución
```
File → Export → as Image
  → Breadboard / Schematic / PCB
  → Resolution: 300 DPI
  → Transparent background: Yes
```

### PDF vectorial
```
File → Export → as PDF
  → Seleccionar vistas
  → Útil para datasheets, manuales
```

---

## Limitaciones conocidas

| Issue | Workaround |
|-------|------------|
| No simula circuito | Usar SPICE externo (KiCad/Ngspice) |
| Librería SMD limitada | Crear footprints custom SVG |
| No exporta Gerbers | Solo visual - usar KiCad para fabricación |
| Buses parciales | Definir buses en .fzp manualmente |
| No 3D STEP export | Usar KiCad 3D viewer / STEP export |

---

## Integración con KiCad

```
KiCad (Fuente verdad) → Exportar SVG → Fritzing (Visual)
     ↑                                         ↓
     └─────────────── No bidireccional ────────┘
```

**Flujo recomendado**:
1. Diseñar en KiCad (esquemático + PCB)
2. Exportar PCB → SVG (File → Export → SVG)
3. Importar SVG en Fritzing como imagen base
4. Añadir conectores y buses en .fzp
5. Usar Fritzing solo para docs visuales / protoboard

---

## Archivos fuente Fritzing

Ubicación: `/fritzing/` y `/fritzing_out/`

```bash
# Regenerar desde KiCad (si tienes plugin)
# No hay export directo KiCad→Fritzing automático fiable

# Workflow manual:
1. KiCad: Plot → SVG (PCB layers)
2. Inkscape: Limpiar SVG, separar capas
3. Fritzing Parts Editor: Nuevo parte → Importar SVGs
4. Definir conectores (pin mapping ESP32)
5. Guardar .fzpz
```