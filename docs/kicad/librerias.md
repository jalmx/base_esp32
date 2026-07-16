# KiCad - Librerías personalizadas

## Librerías del proyecto

### 1. `ESP32-DevKitC.kicad_sym` - Símbolo ESP32 DevKit V4

**Ubicación:** `assets/ESP32-DEVKIT-V1/ESP32-DevKitC.kicad_sym`

**Estructura:**
```
ESP32-DevKitC (Root)
├── Unit A: Power (Pins 1-6, 25-28, 37-38)
│   VIN, 3V3, GND, EN, VP, VN
├── Unit B: GPIO 0-17
│   GPIO0-17 (Strapping notes)
├── Unit C: GPIO 18-39
│   GPIO18-39 (ADC, Touch, Input-only)
├── Unit D: ADC / Analog
│   ADC1_CH0-7, ADC2_CH0-9
├── Unit E: Communication
│   SPI, I2C, UART, I2S, SDIO
└── Unit F: Special / Strapping
    BOOT, Strapping pins highlighted
```

**Propiedades clave:**
- `Reference`: "U"
- `Value`: "ESP32-DevKitC-V4"
- `Footprint`: "ESP32-DevKitC:ESP32-DevKitC"
- `Datasheet`: "https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wrover-32e_datasheet_en.pdf"
- `ki_keywords`: "esp32 devkit wifi bluetooth mcu"
- `ki_fp_filters`: "ESP32*"

**Pines strapping marcados** (propiedad `Description`):
- GPIO0: "Boot mode select (LOW=Download)"
- GPIO2: "Boot mode (HIGH=SDIO), MTDI"
- GPIO12: "Flash voltage (LOW=3.3V, HIGH=1.8V)"
- GPIO15: "Boot logging (HIGH=Silent)"

---

### 2. `MX1508.kicad_sym` - Símbolo Driver Motor

**Ubicación:** `MX1508.kicad_sym`

**Unidades:**
```
Unit 1: Power
  VM (Pin 8) - Motor supply
  VCC (Pin 1) - Logic supply
  GND (Pin 4,5) - Ground (2 pines)

Unit 2: Channel A (H-Bridge 1)
  IN1 (Pin 2) - Input A1
  IN2 (Pin 3) - Input A2
  OUT1 (Pin 7) - Output A1
  OUT2 (Pin 6) - Output A2

Unit 3: Channel B (H-Bridge 2)
  IN3 (Pin 9) - Input B1
  IN4 (Pin 10) - Input B2
  OUT3 (Pin 13) - Output B1
  OUT4 (Pin 12) - Output B2

Unit 4: Thermal
  EPAD (Pad central) - Thermal pad (GND/VM)
```

**Footprint asignado:** `MX1508_SOP8` (custom)

---

### 3. `MX1508.kicad_mod` - Footprint MX1508

**Ubicación:** `MX1508.kicad_mod`

**Características:**
- SOP-8 estándar + pad térmico central expuesto
- Pads: 0.65mm pitch, 0.4×0.6mm
- Thermal pad: 3.2×2.2mm (centrado)
- Courtyard: 6.5×5.5mm
- 3D model: `mx1508.step` (opcional)

**Zona térmica PCB:**
```
En PCB: Polygon en F.Cu (VCC_MOTOR) + B.Cu (GND)
  Vías térmicas: 0.3mm drill, 0.6mm pad
  Grid: 1mm bajo pad térmico
  Conectado a: GND pour (Bottom) + VCC_MOTOR pour (Top)
```

---

### 4. `logo_x.kicad_mod` - Logo Xizuth

**Ubicación:** `logo_footprint/logo_x.kicad_mod`

**Contenido:**
- Gráficos vectoriales en `F.SilkS` (Top Silkscreen)
- Dimensiones: ~15×15mm
- Origen: Centro logo
- Layer: `F.SilkS` (visible en PCB)

**Uso:**
1. PCB Editor → `O` (Add Footprint) → "logo_x"
2. Colocar esquina PCB (ej. superior derecha)
3. `F` (Flip) si necesita en Bottom

---

### 5. `qr.kicad_mod` - QR Code GitHub

**Ubicación:** `logo_footprint/qr.kicad_mod`

**Generación:**
```bash
# Generar QR SVG
qrencode -o qr.svg -s 10 -l M "https://github.com/Xizuth/base_esp32"

# Convertir a footprint KiCad
# Opcional: Script python svg2mod.py
```

**Contenido:**
- Módulos en `F.SilkS` (puntos QR)
- Versión compacta 21×21 módulos
- Tamaño: ~10×10mm
- Quiet zone: 4 módulos

---

## Añadir librería al proyecto

### Símbolos (`.kicad_sym`)
```
Preferences → Manage Symbol Libraries
  → Project Specific Libraries tab
  → + (Add)
  → Nickname: "project_symbols"
  → Library Path: "${KIPRJMOD}/ESP32-DevKitC.kicad_sym"
  → Plugin: "KiCad"
  → OK
```

### Footprints (`.kicad_mod` en `.pretty`)
```
Preferences → Manage Footprint Libraries
  → Project Specific Libraries tab
  → + (Add)
  → Nickname: "project_footprints"
  → Library Path: "${KIPRJMOD}/my_symbols.pretty"
  → Plugin: "KiCad"
  → OK
```

> `${KIPRJMOD}` = variable = carpeta del proyecto `.kicad_pro`

---

## Crear nuevo símbolo

### Desde cero (Symbol Editor)
```
1. File → New Symbol Library → "my_lib.kicad_sym"
2. Create new symbol → Name: "Mi_IC"
3. Add pins (Pin tool `P`):
   - Number, Name, Type (Input/Output/Power/Passive)
   - Graphic style: Line/Inverted/Clock/Low level
4. Draw body (Rectangle `R`, Circle `C`, Arc `A`)
5. Add properties (Property tool `P`):
   Reference: "U"
   Value: "Mi_IC"
   Footprint: "Mi_IC_Footprint"
   Datasheet: "URL"
   ki_keywords: "palabras clave"
6. Save library → Add to project
```

### Desde datasheet (Import)
```
1. Symbol Editor → File → Import → "KiCad Symbol" / "Eagle" / "Altium"
2. Seleccionar archivo → Import
3. Revisar pines, asignar footprint
4. Guardar en librería proyecto
```

---

## Crear nuevo footprint

### Desde cero (Footprint Editor)
```
1. File → New Footprint Library → "my_lib.pretty"
2. New Footprint → Name: "Mi_Footprint"
3. Pads (Pad tool `P`):
   - SMD: Type SMD, Shape Rect/Round, Size X/Y
   - THT: Type Through-hole, Shape Circular/Oval, Drill size
   - Number: 1, 2, 3... (match symbol)
4. Courtyard (Draw → Polygon `Ctrl+Shift+P`):
   - Layer: F.CrtYd / B.CrtYd
   - Clearance IPC: 0.25mm (Nivel 1) / 0.5mm (Nivel 2)
5. Silkscreen (Layer F.SilkS):
   - Outline, Pin 1 marker, Reference designator
5. Properties:
   - Description: "Mi footprint description"
   - Tags: "palabras clave"
   - 3D Model: Add .step/.wrl (opcional)
6. Save → Add library to project
```

### Estándares IPC-7351 (Niveles)

| Nivel | Densidad | Clearance courtyard | Uso |
|-------|----------|---------------------|-----|
| **L (Least)** | Baja | 0.50mm | Prototipos, mano |
| **N (Nominal)** | Media | 0.25mm | **Producción estándar** |
| **M (Most)** | Alta | 0.10mm | Alta densidad, automático |

**Recomendado:** Nivel N (Nominal)

---

## 3D Models

### Formatos soportados
- `.step` / `.stp` (STEP AP214 - preferido)
- `.wrl` (VRML 2.0 - legacy)
- `.stl` (Solo visualización, sin colores)

### Añadir a footprint
```
Footprint Editor → Footprint Properties → 3D Models → +
  → File: "model.step"
  → Offset X/Y/Z: 0, 0, 0 (ajustar)
  → Rotation X/Y/Z: 0, 0, 0
  → Scale: 1, 1, 1
  → Opacity: 1.0
```

### Fuentes modelos 3D
| Fuente | URL | Licencia |
|--------|-----|----------|
| **KiCad 3D Models** | https://github.com/KiCad/kicad-packages3D | CC-BY-SA 4.0 |
| **SnapEDA** | https://www.snapeda.com/ | Gratis registro |
| **LCSC/JLCPCB** | https://lcsc.com/ | Gratis |
| **GrabCAD** | https://grabcad.com/ | Varias |
| **Fabricante** | Web del fabricante | Ver datasheet |

---

## Mejores prácticas librerías

| Práctica | Por qué |
|----------|---------|
| **Nombres descriptivos** | `Conn_01x04_P2.54mm_Vertical` vs `J1` |
| **Prefijos consistentes** | `Resistor_`, `Capacitor_`, `IC_`, `Conn_` |
| **Unidades métricas** | mm para todo (KiCad nativo mm) |
| **Origen en centro** | Para SMD: centro cuerpo; THT: pin 1 o centro |
| **Pin 1 marcado** | Círculo, muesca, chanfle en silkscreen |
| **Courtyard IPC** | Nivel N para producción |
| **3D model aligned** | Verificar en 3D Viewer (`Alt+3`) |
| **Versionar librerías** | Commit `.kicad_sym` y `.pretty` en repo |

---

## Compartir librerías

### Git submodule (recomendado)
```bash
git submodule add https://github.com/tu-usuario/kicad-libs.git libs/kicad-libs
# En proyecto:
Preferences → Libraries → ${KIPRJMOD}/../libs/kicad-libs/symbols
```

### Copia local (simple)
```bash
cp -r /ruta/a/librerias/* ./mis_libs/
# En KiCad: ${KIPRJMOD}/mis_libs/
```

### KiCad Library Convention (KLC)
Seguir [KLC](https://github.com/KiCad/kicad-library-conventions) para contribuir upstream:
- Nomenclatura, estructura, metadatos, 3D models
- Requerido para merge en librería oficial KiCad