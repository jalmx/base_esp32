# PCB Layout

## Especificaciones físicas

| Parámetro | Valor |
|-----------|-------|
| **Dimensiones** | 100mm × 80mm |
| **Capas** | 2 (Top/Bottom) |
| **Material** | FR-4, 1.6mm |
| **Cobre** | 35µm (1oz) |
| **Via mínimo** | 0.3mm / 0.6mm pad |
| **Track mínimo** | 0.15mm (6mil) |
| **Clearance** | 0.15mm (6mil) |
| **Huecos montaje** | 4× M3 (3.2mm plated) |

## Stackup

```
┌─────────────────────────────────────┐
│  Top Silkscreen (F.SilkS)           │
│  Top Solder Mask (F.Mask)           │
│  Top Copper (F.Cu) ◄── Señales + Power │
├─────────────────────────────────────┤  Core FR-4 1.4mm
│  Bottom Copper (B.Cu) ◄── GND Pour  │
│  Bottom Solder Mask (B.Mask)        │
│  Bottom Silkscreen (B.SilkS)        │
└─────────────────────────────────────┘
```

## Capas de cobre

=== "Top Copper (F.Cu)"
    ![Top Copper](images/pcb/base_esp32-F_Cu.png)
    - Ruteo principal señales
    - Polygons: +3.3V, +5V, VCC_MOTOR
    - Pads SMD componentes

=== "Bottom Copper (B.Cu)"
    ![Bottom Copper](images/pcb/base_esp32-B_Cu.png)
    - **Ground Pour** completa (GND net)
    - Algunas señales de cruce
    - Pads THT

## Planos de alimentación (Polygons)

### Top Layer Polygons

| Net | Área | Props |
|-----|------|-------|
| **+5V** | Zona inferior izq | Thermal relief vias |
| **+3.3V** | Zona central | Zona central | Thermal relief |
| **VCC_MOTOR** | Debajo MX1508 | Direct pour, sin relief |
| **GND** | Relleno resto | Via stitching |

### Bottom Layer
- **GND Pour** completo con **via stitching** cada 5mm
- Conecta a GND pads THT y vias térmicas

## Reglas de diseño (Design Rules)

| Regla | Valor | Aplicación |
|-------|-------|------------|
| Clearance | 0.15mm (6mil) | General |
| Track width (signal) | 0.2mm (8mil) | GPIO, comms |
| Track width (power 5V) | 0.5mm (20mil) | LM7805, MX1508 |
| Track width (power 3.3V) | 0.4mm (16mil) | AMS1117, ESP32 |
| Track width (motor) | 1.0mm (40mil) | Terminal block ↔ MX1508 |
| Via size | 0.3/0.6mm | Signal |
| Via power | 0.5/1.0mm | Power polygons |
| Thermal relief | 0.25mm gap, 4 spokes | Polygon connects |

## Decisiones de layout críticas

### 1. Separación dominios de potencia
```
┌────────────────────────────────────────┐
│  +5V (Motor)    │  +3.3V (Logic)       │
│  ┌──────────┐   │  ┌──────────┐        │
│  │ LM7805   │   │  │ AMS1117  │        │
│  │ MX1508   │   │  │ ESP32    │        │
│  │ Terminal │   │  │ Sensors  │        │
│  └──────────┘   │  └──────────┘        │
│        │        │        │             │
│        └────────┴────────┘             │
│                 GND (common)           │
└────────────────────────────────────────┘
```
- **GND común** (star ground en conector alimentación)
- Polygons separados en Top, unidos por vias en GND
- Evita loops de corriente motor en ground logic

### 2. Ruteo señales sensibles
- **ADC traces** (LM35, LDR, Pot): Cortas, lejos de PWM motor
- **1-Wire (DS18B20)**: Pull-up cerca GPIO, track ancho 0.3mm
- **I2C/SPI**: Paralelas, ground guard entre líneas
- **PWM Motor**: Tracks anchas, ground shield adyacente

### 3. Disipación térmica
- **LM7805**: Pad TO-220 con 4 vias térmicas a GND pour bottom
- **AMS1117**: Pad SOT-223 con vias térmicas
- **MX1508**: Pad expuesto conectado a VCC_MOTOR polygon + vias a GND

### 4. Conector ESP32
- Headers 2×19 alineados borde PCB
- Señales GPIO ruteadas perpendicular al conector
- Minimiza longitud stubs

## Silkscreen (F.SilkS / B.SilkS)

=== "Top Silkscreen"
    ![Top Silk](images/pcb/base_esp32-F_Silkscreen.png)

=== "Bottom Silkscreen"
    ![Bottom Silk](images/pcb/base_esp32-B_Silkscreen.png)

**Elementos clave:**
- ✅ Referencias componentes (R1, C1, U1, J1...)
- ✅ Polaridad LED, diodos, electrolíticos
- ✅ Pin 1 en conectores e ICs
- ✅ Etiquetas funcionales: "5V", "3.3V", "GND", "MOTOR", "SERVO"
- ✅ **NUEVO v0.4**: Nombres reguladores junto a footprint
- ✅ Logo Xizuth + QR code repositorio
- ✅ Versión PCB (v0.4) y fecha

## Solder Mask & Paste

=== "Top Mask"
    ![Top Mask](images/pcb/base_esp32-F_Mask.png)

=== "Bottom Mask"
    ![Bottom Mask](images/pcb/base_esp32-B_Mask.png)

=== "Top Paste"
    ![Top Paste](images/pcb/base_esp32-F_Paste.png)

=== "Bottom Paste"
    ![Bottom Paste](images/pcb/base_esp32-B_Paste.png)

## Edge Cuts

![Edge Cuts](images/pcb/base_esp32-Edge_Cuts.svg)

- Rectángulo 100×80mm
- Esquinas redondeadas R=2mm
- 4 cutouts para mounting holes M3
- Sin cutouts internos

## Fabricación - Archivos Gerber

Ver [Fabricación > Archivos Gerber](../fabricacion/gerber.md) para lista completa y visualizador.

## Validación DRC

```bash
# Ejecutar DRC en KiCad 8/9
# Inspect > Design Rule Checker
# Run DRC
# Errors: 0 | Warnings: 0 (esperado)
```

**Checks críticos:**
- [ ] Clearance violations
- [ ] Track width vs current
- [ ] Via count thermal relief
- [ ] Unconnected nets
- [ ] Footprint courtyard overlaps
- [ ] Silk over pads
- [ ] Missing thermal reliefs

## Exportaciones disponibles

| Formato | Ubicación | Uso |
|---------|-----------|-----|
| **KiCad PCB** | `base_esp32.kicad_pcb` | Edición |
| **Gerber ZIP** | `gerber/base_esp32_v4.zip` | Fabricación |
| **STEP 3D** | `step/base_esp32.step` | Mechanical CAD |
| **SVG Layers** | `pcb/*.svg` | Documentación |
| **PNG Renders** | `3D/*.png` | Visualización |
| **BOM** | `fabricacion/bom.csv` | Compras |
| **Pick&Place** | `fabricacion/pnp.csv` | Assembly SMT |