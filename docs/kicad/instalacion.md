# KiCad - Instalación y configuración

## Instalación

### Windows
```bash
# Chocolatey
choco install kicad

# O instalador oficial
# https://kicad.org/download/windows/
```

### Linux (Ubuntu/Debian)
```bash
# PPA oficial (versión estable)
sudo add-apt-repository ppa:kicad/kicad-8.0-releases
sudo apt update
sudo apt install kicad

# Flatpak (última versión)
flatpak install flathub org.kicad.KiCad
```

### macOS
```bash
# Homebrew
brew install --cask kicad

# O descargar .dmg desde kicad.org
```

### Verificar versión
```bash
kicad --version
# Requerido: 8.0+ (para este proyecto)
# Recomendado: 9.0 (actual estable 2024)
```

---

## Estructura del proyecto KiCad

```
base_esp32/
├── base_esp32.kicad_pro        # Configuración proyecto
├── base_esp32.kicad_sch        # Esquemático
├── base_esp32.kicad_pcb        # PCB Layout
├── base_esp32.kicad_prl        # Reglas locales (DRC, net classes)
├── sym-lib-table               # Tabla librerías símbolos (global)
├── fp-lib-table                # Tabla librerías footprints (global)
├── ESP32_devkit_v1.kicad_sym   # Símbolo custom ESP32 DevKit
├── MX1508.kicad_sym            # Símbolo custom MX1508
├── logo_x.kicad_mod            # Footprint logo Xizuth
├── qr.kicad_mod                # Footprint QR code
└── my_symbols.pretty/          # Librería footprints local
```

---

## Abrir proyecto

1. **KiCad → File → Open Project** → `base_esp32.kicad_pro`
2. Se abren: **Project Manager**, **Schematic Editor**, **PCB Editor**

### Ventanas principales
| Editor | Atajo | Uso |
|--------|-------|-----|
| Schematic | `Ctrl+E` | Esquemático |
| PCB | `Ctrl+P` | Layout PCB |
| Symbol Editor | - | Crear/editar símbolos |
| Footprint Editor | - | Crear/editar footprints |
| 3D Viewer | `Alt+3` | Vista 3D PCB |

---

## Librerías personalizadas

### Símbolos (`.kicad_sym`)

**ESP32-DevKitC-V4** (`assets/ESP32-DEVKIT-V1/ESP32-DevKitC.kicad_sym`):
- 38 pines (2×19)
- Unidades separadas: Power, GPIO, ADC, Comm, Strapping
- Campos: Reference, Value, Footprint, Datasheet, ki_keywords

**MX1508** (`MX1508.kicad_sym`):
- 8 pines SOP-8
- Unidades: Power, Logic, H-Bridge A, H-Bridge B
- Pin thermal pad marcado

### Footprints (`.kicad_mod`)

**Logo Xizuth** (`logo_footprint/logo_x.kicad_mod`):
- Gráficos vectoriales en silkscreen
- Posición: Esquina PCB

**QR Code** (`logo_footprint/qr.kicad_mod`):
- Enlaza a GitHub repo
- Generado: `qrencode -o qr.svg "https://github.com/Xizuth/base_esp32"`

### Añadir al proyecto
```
Preferences → Manage Symbol Libraries → Project Specific Libraries → +
  → Nickname: "project_symbols" → Path: "${KIPRJMOD}/ESP32-DevKitC.kicad_sym"

Preferences → Manage Footprint Libraries → Project Specific Libraries → +
  → Nickname: "project_footprints" → Path: "${KIPRJMOD}/my_symbols.pretty"
```

---

## Modificar el diseño

### 1. Cambiar componente (ej. Driver motor)

**Esquemático:**
1. Abrir `base_esp32.kicad_sch`
2. Seleccionar `U_MOT` (MX1508) → `Delete`
3. `A` (Add Symbol) → Buscar "L293D" o nuevo driver
4. Colocar → Conectar pines (wires `W`)
5. Anotar: `Tools → Annotation → Annotate Schematic`

**PCB:**
1. Abrir `base_esp32.kicad_pcb`
2. `Tools → Update PCB from Schematic` (`F8`)
   - ✓ Re-link footprints to schematic symbols
   - ✓ Update net classes
   - ✓ Delete footprints with no symbols
3. Nuevo footprint aparece en "Unplaced" → Colocar → Routear

### 2. Añadir conector (ej. Qwiic I2C)

**Esquemático:**
1. `A` → "Connector:Conn_01x04" → Colocar cerca I2C
2. Conectar: SDA→GPIO21, SCL→GPIO19, 3.3V, GND
3. Valor: "QWIIC" → Referencia: "J_QWIIC"

**PCB:**
1. `F8` → Update PCB
2. Footerprint: "Connector_PinHeader_2.54mm:PinHeader_1x04_P2.54mm_Vertical"
3. Colocar borde PCB → Routear pistas 0.2mm

### 3. Cambiar regulador (ej. AMS1117 → LM1117)

**Esquemático:**
1. Cambiar Value: "AMS1117" → "LM1117-3.3"
2. Verificar footprint compatible (SOT-223)
3. Si distinto: `E` (Properties) → Footprint → Seleccionar correcto

**PCB:**
1. `F8` → Update
2. Si footprint cambió: `Delete` viejo → Colocar nuevo → Routear

---

## Reglas de diseño (Design Rules)

### Net Classes (Clases de red)
```
Design Rules → Net Classes:
  Default:        Clearance 0.15mm, Track 0.2mm, Via 0.3/0.6mm
  Power_5V:       Clearance 0.2mm,  Track 0.5mm,  Via 0.5/1.0mm
  Power_3V3:      Clearance 0.2mm,  Track 0.4mm,  Via 0.5/1.0mm
  Motor_Power:    Clearance 0.3mm,  Track 1.0mm,  Via 0.5/1.0mm
  High_Speed:     Clearance 0.2mm,  Track 0.2mm,  Via 0.3/0.6mm, Diff 100Ω
```

### Reglas personalizadas (`.kicad_prl`)
- Via sizes por net class
- Track widths por net class
- Clearances custom (ej. HV > 2mm)
- Min annular ring, min microvia

---

## DRC (Design Rule Check)

```bash
# En PCB Editor:
Inspect → Design Rule Checker → Run DRC

# Errores típicos a cero:
✓ Clearance violations: 0
✓ Track width violations: 0
✓ Via size violations: 0
✓ Annular ring violations: 0
✓ Unconnected items: 0 (except mounting holes)
✓ Footprint courtyard overlaps: 0
✓ Silkscreen over pads: 0
✓ Edge cuts: 1 closed contour
```

---

## Generar archivos fabricación

### Gerbers
```
File → Fabrication Outputs → Gerbers (.gbr)
  Layers:
    ☑ F.Cu, B.Cu
    ☑ F.SilkS, B.SilkS
    ☑ F.Mask, B.Mask
    ☑ F.Paste, B.Paste (para stencil)
    ☑ Edge.Cuts
  Options:
    ☑ Use Protel filename extensions
    ☑ Generate drill files (separate)
    ☑ Drill origin: Absolute
    ☑ Drill units: Millimeters
    ☑ Drill format: Excellon (decimal)
  Output: gerber/
```

### Drill Files
```
Generate Drill Files:
  ☑ PTH (Plated)
  ☑ NPTH (Non-plated)
  Format: Excellon
  Units: Millimeters
  Zeros format: Decimal
  Map: Gerber + Drill map
```

### BOM
```
Tools → Generate BOM
  Plugin: bom_csv_grouped_by_value
  Output: fabricacion/bom.csv
```

### Pick & Place
```
File → Fabrication Outputs → Footprint Position (.csv)
  ☑ Only SMD
  ☑ Units: mm
  ☑ Format: JLCPCB / PCBWay
  Output: fabricacion/pnp.csv
```

### STEP 3D
```
File → Export → STEP
  ☑ Include: All layers
  ☑ Origin: PCB center
  Output: step/base_esp32.step
```

---

## Atajos útiles

| Acción | Esquemático | PCB |
|--------|-------------|-----|
| Zoom fit | `Home` | `Home` |
| Zoom selection | `Ctrl+Home` | `Ctrl+Home` |
| Grid 0.1/0.5/1mm | `N` / `Shift+N` | `N` / `Shift+N` |
| Add wire | `W` | - |
| Add symbol | `A` | - |
| Add footprint | - | `O` (Footprint) |
| Route track | - | `X` |
| Via while routing | - | `V` |
| Switch layer | - | `PgUp`/`PgDn` |
| Flip selection | - | `F` |
| Rotate 90° | `R` | `R` |
| Move exactly | `Ctrl+M` | `Ctrl+M` |
| Duplicate | `Ctrl+D` | `Ctrl+D` |
| Delete | `Del` | `Del` |
| Undo/Redo | `Ctrl+Z`/`Ctrl+Y` | `Ctrl+Z`/`Ctrl+Y` |
| DRC | - | `F11` |
| Update PCB from Sch | - | `F8` |
| 3D Viewer | - | `Alt+3` |

---

## Buenas prácticas

1. **Commit menudo**: Cada cambio lógico = commit
2. **Net labels**: Usar labels jerárquicos (`GPIO12`, `+5V`, `I2C_SDA`)
3. **Net classes**: Asignar al crear nets nuevos
4. **Polygons**: `B` para regenerar GND pours tras cambios
5. **DRC frecuente**: Cada sesión de routing
6. **3D Viewer**: Verificar clearances mecánicos (connectors, tall components)
7. **Documentación**: Comentar cambios en `to_changes.md`

---

## Proyectos KiCad

### Configuración del proyecto ESP32

```
base_esp32/
├── base_esp32.kicad_pro        # Configuración proyecto
├── base_esp32.kicad_sch        # Esquemático
├── base_esp32.kicad_pcb        # PCB Layout
├── base_esp32.kicad_prl        # Reglas locales (DRC, net classes)
├── sym-lib-table               # Tabla librerías símbolos (global)
├── fp-lib-table                # Tabla librerías footprints (global)
├── ESP32_devkit_v1.kicad_sym   # Símbolo custom ESP32 DevKit
├── MX1508.kicad_sym            # Símbolo custom MX1508
├── logo_x.kicad_mod            # Footprint logo Xizuth
├── qr.kicad_mod                # Footprint QR code
└── my_symbols.pretty/          # Librería footprints local
```

### Configuración de librerías

**Biblioteca específica del proyecto (subdirectorio):**

```bash
mkdir -p kicad_libs/symbols
mkdir -p kicad_libs/footprints

# Añadir en KiCad
Preferences → Manage Symbol Libraries → + → Project Specific Libraries
  Nickname: "my_symbols"
  Path: "${KIPRJMOD}/kicad_libs/symbols"

Preferences → Manage Footprint Libraries → + → Project Specific Libraries
  Nickname: "my_footprints"
  Path: "${KIPRJMOD}/kicad_libs/footprints"
```

**Librería compartida (submódulo git):**

```bash
git submodule add https://github.com/tu-usuario/kicad-common-lib.git kicad_libs/project
```

### Archivos de configuración KiCad

| Archivo | Propósito |
|---------|----------|
| `kicad.rc` | Config global (colores, atajos) |
| `common.sch` | Símbolos compartidos entre esquemas |
| `common.fp` | Footprints compartidos |

---

## Atajos de teclado esenciales

| Acción | Atajo (Linux/Windows) | Atajo (macOS) |
|--------|-----------------------|---------------|
| Nuevo esquema | `Ctrl+N` | `Cmd+N` |
| Abrir esquema | `Ctrl+O` | `Cmd+O` |
| Guardar | `Ctrl+S` | `Cmd+S` |
| Insertar símbolo | `A` (Add) | `A` (Add) |
| Rotar | `R` | `R` |
| Mover | `M` | `M` |
| Escalar | `S` | `S` |
| Eliminar | `Del` | `Del` |
| Conectar pistas | `X` (Track) | `X` (Track) |
| Añadir vía | `V` | `V` |
| Ajustar Board | `Ctrl+B` | `Cmd+B` |
| Design Rule Check | `Ctrl+Shift+C` | `Cmd+Shift+C` |
| 3D Viewer | `Alt+3` | `Ctrl+3` |

---

## Resolución de problemas

### Error: "Failed to load symbol file"
```bash
# Ajustar PATH usando variables de entorno
export KISYSMOD="${KIPRJMOD}/kicad_libs"
```

### Error: "Footprint not found"
```bash
# Repetir con los cuatro entresijos del haystack
 ls -la ${KIPRJMOD}/*

# Ejecutar "Update PCB from Schematic"
F8 → Update
castear
```

### Error: "Symbol library empty"
```bash
# Importar símbolos originales
Preferences → Manage Symbol Libraries → + → Archivo --> choose CSV/JSON
```

### Error: "La capa está bloqueada"
```bash
Design → Layer Settings → desbloquear F.SilkS, F.Cu
```

---

## Administración de librerías

### Exportar librerías del proyecto
```bash
#!/bin/bash

PROJECT_ROOT="/media/data/Projects/base_esp32"
BACKUP_DIR="${PROJECT_ROOT}/backups/kicad"
mkdir -p "${BACKUP_DIR}"

# Exportar símbolos
kicad-cli -p "${PROJECT_ROOT}/base_esp32.kicad_prj" -s "${PROJECT_ROOT}/kicad_libs/symbols" -e "${BACKUP_DIR}/symbols_${DATE}.lib"

# Exportar footprints
kicad-cli -p "${PROJECT_ROOT}/base_esp32.kicad_prj" -f "${PROJECT_ROOT}/kicad_libs/footprints" -e "${BACKUP_DIR}/footprints_${DATE}.mod"

# Exportar esquema y PCB en formato netlist
cd "${PROJECT_ROOT}"
/kicad-cli/sch_export -o "${BACKUP_DIR}/export_${DATE}.net" base_esp32
```

### Importar librerías a nuevo proyecto
```bash
#!/bin/bash

# Desde una librería compartida en otra máquina
cd nuevo_proyecto
cp "${SOURCE}/kicad_libs/symbols/*.sym" "${KIPRJMOD}/"
cp "${SOURCE}/kicad_libs/footprints/*.mod" "${KIPRJMOD}/"

# Actualizar KiCad
kicad --expert -p nuevo
```