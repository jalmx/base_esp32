# Fabricación - Archivos Gerber

## Archivos generados (v0.4)

```
gerber/
├── base_esp32_v4.zip           # Paquete completo para JLCPCB/PCBWay
├── base_esp32-F_Cu.gtl         # Top Copper
├── base_esp32-B_Cu.gbl         # Bottom Copper
├── base_esp32-F_Silkscreen.gto # Top Silkscreen
├── base_esp32-B_Silkscreen.gbo # Bottom Silkscreen
├── base_esp32-F_Mask.gts       # Top Solder Mask
├── base_esp32-B_Mask.gbs       # Bottom Solder Mask
├── base_esp32-F_Paste.gtp      # Top Paste (stencil)
├── base_esp32-B_Paste.gbp      # Bottom Paste (stencil)
├── base_esp32-Edge_Cuts.gm1    # Board Outline
├── base_esp32-PTH.drl          # Plated holes (vias, THT)
├── base_esp32-PTH-drl_map.gbr  # Drill map PTH
├── base_esp32-NPTH.drl         # Non-plated holes (mounting)
├── base_esp32-NPTH-drl_map.gbr # Drill map NPTH
```

## Especificaciones PCB

| Parámetro | Valor | Estándar |
|-----------|-------|----------|
| **Dimensiones** | 100mm × 80mm | - |
| **Capas** | 2 | FR-4 |
| **Espesor** | 1.6mm | ±10% |
| **Cobre** | 35µm (1oz) | Interior/exterior |
| **Track min** | 0.15mm (6mil) | 0.15mm estándar |
| **Via min** | 0.3mm / 0.6mm pad | 0.3/0.6mm |
| **Clearance** | 0.15mm (6mil) | 0.15mm |
| **Hole size** | 0.3-3.2mm | PTH/NPTH |
| **Silkscreen** | Blanco | Top + Bottom |
| **Solder Mask** | Verde | Top + Bottom |
| **Finish** | HASL sin plomo / ENIG | HASL-LF estándar |
| **UL** | 94V-0 | FR-4 |

## Capas incluidas en Gerber

| Archivo | Capa KiCad | Descripción |
|---------|------------|-------------|
| `*_F_Cu.gtl` | F.Cu | Pistas y pads Top |
| `*_B_Cu.gbl` | B.Cu | Pistas y pads Bottom (GND pour) |
| `*_F_Silkscreen.gto` | F.SilkS | Referencias, polaridad, logos Top |
| `*_B_Silkscreen.gbo` | B.SilkS | Referencias Bottom |
| `*_F_Mask.gts` | F.Mask | Aberturas pads Top |
| `*_B_Mask.gbs` | B.Mask | Aberturas pads Bottom |
| `*_F_Paste.gtp` | F.Paste | Aperturas stencil Top (SMD) |
| `*_B_Paste.gbp` | B.Paste | Aperturas stencil Bottom |
| `*_Edge_Cuts.gm1` | Edge.Cuts | Contorno PCB + mounting holes |
| `*_PTH.drl` | - | Taladros plated (vias, THT pads) |
| `*_NPTH.drl` | - | Taladros non-plated (mounting holes) |

## Visualizador online

Sube `base_esp32_v4.zip` a:
- **JLCPCB**: https://jlcpcb.com/quote (arrastrar zip)
- **PCBWay**: https://www.pcbway.com/orderonline.aspx
- **Gerber Viewer**: https://gerber-viewer.easyeda.com/ o https://www.ucamco.com/en/gerber-viewer

## Pedido típico (JLCPCB)

```
Quantity: 5 (mínimo)
Layers: 2
Dimensions: 100×80mm
PCB Thickness: 1.6mm
Min Track/Space: 0.15/0.15mm
Min Hole: 0.3mm
Surface Finish: HASL (Lead Free)
Solder Mask: Green
Silkscreen: White
Via Process: Tented
Remove Order Number: Yes
```

**Coste aprox**: $2-5 por 5 unidades + envío

## Stencil (opcional, para SMD)

```
Stencil Type: Framed / Frameless
Stencil Thickness: 0.12mm (recomendado 0805/SOP-8)
Stencil Size: 370×470mm (framed) / 150×150mm (frameless)
```

## BOM y Pick&Place (para ensamblaje SMT)

### `fabricacion/bom.csv`
```csv
Designator,Value,Footprint,LCSC Part,Qty
U_ESP32,ESP32-DevKitC-V4,ESP32-DevKitC,C132755,1
U_5V,LM7805CT,TO-220-3_Vertical,C17332,1
U_3V3,AMS1117-3.3,SOT-223-4,C2882143,1
U_MOT,MX1508,MX1508_SOP8,C2882143,1
U_IMON,A9013,SIP-3,C2882143,1
U_TEMP1,DS18B20,TO-92,C17829,1
U_TEMP2,LM35DZ,TO-92,C2882143,1
D_PWR,1N5819,SOD-123,C23272,1
D_PROT,1N5819,SOD-123,C23272,1
D_DISC,1N5819,SOD-123,C23272,1
LED_D1,LED 3mm Red,LED_THT_3mm,C17434,1
LED_D2,LED 3mm Blue,LED_THT_3mm,C17434,1
LED_D3,LED 3mm Green,LED_THT_3mm,C17434,1
LED_D4,LED 3mm Yellow,LED_THT_3mm,C17434,1
LED_RGB,WS2812B,LED_SMD_5050,C2882143,1
BZ1,Buzzer Passive 5V,Buzzer_THT_12mm,C17434,1
SW_DIP,DIP Switch 4pos,SW_DIP_4,C17434,1
SW_PUSH,Push Button 6x6,SW_Push_6x6,C17434,1
SW_SPDT,SPDT Slide,SS-12D13,C17434,1
RV_POT,Pot 10k,Pot_THT_B10K,C17434,1
J_PWR,Jack DC 2.1mm,PJ-002AH,C17434,1
J_USB,USB Micro-B,USB_Micro_B,C17434,1
J_TERM,Terminal Block 4p,Terminal_Block_4p,C17434,1
J_SERVO,Header 3p M,Header_3p_M,C17434,1
J_I2C,Header 4p M,Header_4p_M,C17434,1
J_SPI,Header 6p M,Header_6p_M,C17434,1
J_GPIO,Header 10p M,Header_10p_M,C17434,1
MH1-MH4,Mounting Hole M3,MountingHole_3.2mm,,4
```

### `fabricacion/pnp.csv` (Pick&Place)
```csv
Designator,Footprint,Mid X,Mid Y,Rotation,Layer
U_3V3,SOT-223-4,45.2,32.1,90,Top
U_MOT,MX1508_SOP8,67.8,45.3,0,Top
U_IMON,SIP-3,23.4,78.9,180,Top
LED_RGB,LED_SMD_5050,12.3,15.6,0,Top
D_PWR,SOD-123,5.5,85.2,90,Top
D_PROT,SOD-123,7.8,82.1,270,Top
D_DISC,SOD-123,48.2,35.4,0,Top
... (resto componentes SMD)
```

> Generar en KiCad: **File → Fabrication Outputs → BOM / Footprint Position (.csv)**

---

## Checklist pre-pedido

- [ ] DRC sin errores (KiCad: Inspect → DRC)
- [ ] Netlist coincide esquemático (Netlist → Verify)
- [ ] Footprints correctos (3D viewer revisar)
- [ ] Mounting holes M3 (4 esquinas)
- [ ] Edge cuts correcto (100×80mm, R2mm corners)
- [ ] Silkscreen: Referencias, Pin 1, Polaridad, Logos
- [ ] No silkscreen sobre pads
- [ ] Vías tienda bajo LM7805, MX1508, AMS1117
- [ ] Polygons GND +5V +3.3V regenerados (B key)
- [ ] Gerbers generados (Plot → Generate Drill Files)
- [ ] Gerbers verificados en visor online
- [ ] ZIP creado con todos archivos requeridos