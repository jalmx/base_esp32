# KiCad - Modificar el diseño

## Flujo de trabajo típico

```
1. Esquemático → 2. PCB → 3. DRC → 4. Fabricación
     ↑              ↑
     └──────────────┘ (Iterar)
```

---

## Cambios comunes

### 1. Cambiar valor componente (ej. R 1k → 4.7k)

**Esquemático:**
1. Click derecho en R → `E` (Properties)
2. Cambiar `Value`: `1k` → `4.7k`
3. `OK`

**PCB:** *No requiere cambio* (footprint mismo)

---

### 2. Cambiar componente por otro equivalente (ej. LM7805 → L7805CV)

**Esquemático:**
1. Click en U_5V → `E` → Change Symbol
3. Buscar "L7805" / "LM7805" → Seleccionar
4. Verificar pines coinciden (IN/GND/OUT)
5. `OK`

**PCB:**
1. `F8` (Update PCB from Schematic)
2. `Update` → Footprint debe ser mismo (TO-220)
3. Si footprint distinto: `Delete` viejo → colocar nuevo → `X` (route)

---

### 3. Añadir nuevo componente (ej. Sensor BMP280 I2C)

**Esquemático:**
1. `A` → Buscar "BMP280" (o genérico "Sensor_I2C")
2. Colocar cerca J_I2C
3. Conectar: SDA→GPIO21, SCL→GPIO19, 3.3V, GND
4. `Value`: "BMP280" → `Reference`: "U_BMP"
5. Anotar: `Tools → Annotation → Annotate`

**PCB:**
1. `F8` → Update PCB
2. Footprint aparece en "Unplaced" (lista derecha)
3. Click → `M` (Move) → Colocar cerca J_I2C
3. `X` (Route) → Conectar pistas 0.2mm
4. `B` → Regenerar GND pour

---

### 4. Cambiar footprint (ej. 0805 → 0603)

**Esquemático:**
1. Click componente → `E` → `Footprint` → Browse
2. Filtrar "0603" → Seleccionar `Resistor_SMD:R_0603_1608Metric`
3. `OK`

**PCB:**
1. `F8` → Update
2. KiCad detecta cambio → Pregunta "Replace footprint?"
3. `Update` → Componente se mueve a "Unplaced"
4. Colocar nuevo → Routear

---

### 5. Mover conector / añadir header

**Esquemático:**
1. Añadir símbolo `Conn_01x04` → Conectar nets
3. Valor: "J_I2C_EXT" → Ref: "J_I2C_EXT"

**PCB:**
1. `F8` → Update
2. Footprint: `Connector_PinHeader_2.54mm:PinHeader_1x04_P2.54mm_Vertical`
3. Colocar en borde PCB (Edge.Cuts margin 5mm)
4. Routear: `X` → pistas 0.2mm → `V` (via si cruza)

---

## Edición de pistas y zonas

### Ampliar pista potencia
```
1. Click pista → `E` (Properties)
2. Net Class: "Power_5V" (0.5mm) → OK
3. O: `Ctrl+E` (Global Edit) → Tracks → Width → 0.5mm → Filter: Net="+5V"
```

### Añadir via stitching GND
```
1. Route track GND → `V` (add via) → continua en B.Cu
2. O: `Place → Add Filled Zone` → GND → dibuja área → `B` (fill)
```

### Modificar zone GND pour
```
1. Click zone outline → `E`
3. Net: GND → Layer: B.Cu (o F.Cu)
4. Clearance: 0.5mm → Min width: 0.2mm
5. Thermal relief: 0.25mm gap, 4 spokes
6. `B` (Fill zones) para regenerar
```

---

## Cambios estructurales

### Añadir capa (4 capas → requiere PCB nuevo)
```
File → Board Setup → Board Stackup
  Layers: 4 → Add In1.Cu, In2.Cu
  Materials: FR4, 1.6mm total
  Impedance controlled: configure diff pairs
```
> **Nota**: Cambio mayor, requiere re-routeo completo

### Cambiar tamaño PCB
```
1. Edge.Cuts layer activo
2. Click línea borde → `E` → Edit coordinates
3. O: `Ctrl+M` (Move Exact) → ΔX, ΔY
4. `B` → Regenerar zones
5. Revisar DRC (mounting holes, connectores)
```

### Añadir mounting hole
```
PCB Editor:
  Place → Footprint → "MountingHole" → "MountingHole_3.2mm_M3"
  Colocar 4 esquinas (5mm margen)
  Properties → Net: "GND" (opcional, plated)
```

---

## Validación post-cambios

### Checklist obligatorio
```
□ DRC: 0 Errors, 0 Warnings (except courtyards opcional)
□ Netlist: Esquemático ↔ PCB coinciden (F8 → no cambios)
□ ERC: Esquemático sin errores (Inspect → ERC)
□ BOM: Genera sin warnings (Tools → Generate BOM)
□ 3D Viewer: No colisiones mecánicas (Alt+3)
□ Gerbers: Visualizar en gerber-viewer (online)
```

### Test eléctrico mental
```
□ Power nets: +5V, +3V3, GND separados, sin shorts
□ MCU: Todos GPIOs usados tienen net asignada
□ Strapping pins: GPIO0,2,12,15 no forzados a nivel incorrecto
□ Decoupling: 100nF cerca cada IC VCC/GND
□ High current: Pistas motor ≥1mm, vias stitching
□ Analog: ADC traces lejos de PWM/Clock
□ Mounting: Holes no tocan pistas (clearance 1mm)
```

---

## Git workflow para cambios

```bash
# 1. Rama para cambio
git checkout -b feat/nuevo-sensor-bmp280

# 2. Editar en KiCad, commit frecuente
git add base_esp32.kicad_sch base_esp32.kicad_pcb
git commit -m "feat: add BMP280 I2C sensor
- Added U_BMP symbol connected to I2C bus
- Footprint: LGA-8 2x2mm
- Routed on top layer near J_I2C"

# 3. Generar fabricación
# File → Fabrication Outputs → Gerbers → gerber_v0.5.zip

# 4. Commit artefacts (opcional, o en release)
git add fabricacion/gerber_v0.5.zip fabricacion/bom_v0.5.csv
git commit -m "chore: fab files v0.5"

# 5. Push y PR
git push origin feat/nuevo-sensor-bmp280
# Crear PR → Review → Merge → Tag v0.5
```

---

## Backup y recuperación

### Backup automático (KiCad)
```
Preferences → Preferences → Common → Auto-save
  ✓ Create backup files
  ✓ Auto-save interval: 5 min
  ✓ Max backup files: 10
```

### Backup manual (antes de cambios grandes)
```bash
cp base_esp32.kicad_sch base_esp32.kicad_sch.backup
cp base_esp32.kicad_pcb base_esp32.kicad_pcb.backup
cp base_esp32.kicad_pro base_esp32.kicad_pro.backup
```

### Restaurar
```bash
# Si corrupto:
cp base_esp32.kicad_sch.backup base_esp32.kicad_sch
# O desde git:
git restore base_esp32.kicad_sch
```

---

## Migración versiones KiCad

| De → A | Acción |
|--------|--------|
| 6 → 7 | Abrir → Guardar (auto-migra) |
| 7 → 8 | Abrir → Guardar (auto-migra) |
| 8 → 9 | Abrir → Guardar (auto-migra) |
| 5 → 6+ | Exportar netlist → Importar en nuevo |

> **Siempre**: Backup antes de abrir en versión mayor