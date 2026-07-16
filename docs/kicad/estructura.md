# KiCad - Estructura del proyecto

## Archivos principales

```
base_esp32/
├── base_esp32.kicad_pro      # Configuración proyecto
├── base_esp32.kicad_sch      # Esquemático
├── base_esp32.kicad_pcb      # PCB Layout
├── base_esp32.kicad_prl      # Reglas locales (DRC)
├── sym-lib-table             # Tabla librerías símbolos
├── fp-lib-table              # Tabla librerías footprints
├── fp-info-cache             # Cache footprints (auto)
├── ESP32_devkit_v1.bak       # Backup librería símbolos
├── ESP32_devkit_v1.kicad_sym # Librería símbolos ESP32
├── MX1508.kicad_sym          # Símbolo MX1508
├── MX1508.kicad_mod          # Footprint MX1508
└── logo_footprint/           # Footprints branding
    ├── logo_x.kicad_mod
    └── qr.kicad_mod
```

---

## `base_esp32.kicad_pro` (Configuración)

```json
{
  "board": {
    "design_settings": {
      "defaults": {
        "board_outline_line_width": 0.15,
        "copper_line_width": 0.2,
        "copper_text_size": 1.5,
        "copper_text_thickness": 0.3,
        "courtyard_line_width": 0.05,
        "dimension_precision": 4,
        "dimension_units": 3,
        "edge_cut_line_width": 0.15,
        "fabrication_line_width": 0.15,
        "fabrication_text_size": 1,
        "fabrication_text_thickness": 0.15,
        "other_line_width": 0.15,
        "other_text_size": 1,
        "other_text_thickness": 0.15,
        "pads_to_mask_clearance": 0.051,
        "pcb_text_size": 1.5,
        "pcb_text_thickness": 0.3,
        "silk_line_width": 0.15,
        "silk_text_size": 1,
        "silk_text_thickness": 0.15,
        "solder_mask_clearance": 0.051,
        "solder_mask_min_width": 0.25,
        "zone_45_only": false,
        "zone_clearance": 0.508,
        "zone_line_width": 0.15
      },
      "diff_pair_dimensions": [],
      "drc_exclusions": [],
      "net_classes": [
        {"name": "Default", "bus_width": 12, "clearance": 0.15, "diff_pair_gap": 0.25, "diff_pair_via_gap": 0.25, "diff_pair_width": 0.2, "line_style": 0, "microvia_diameter": 0.3, "microvia_drill": 0.15, "pcb_color": "rgba(0, 0, 0, 0.000)", "schematic_color": "rgba(0, 0, 0, 0.000)", "track_width": 0.2, "via_diameter": 0.6, "via_drill": 0.3, "wire_width": 0.2},
        {"name": "Power_5V", "bus_width": 12, "clearance": 0.2, "diff_pair_gap": 0.25, "diff_pair_via_gap": 0.25, "diff_pair_width": 0.2, "line_style": 0, "microvia_diameter": 0.3, "microvia_drill": 0.15, "pcb_color": "rgba(255, 0, 0, 0.000)", "schematic_color": "rgba(255, 0, 0, 0.000)", "track_width": 0.5, "via_diameter": 1.0, "via_drill": 0.5, "wire_width": 0.2},
        {"name": "Power_3V3", "bus_width": 12, "clearance": 0.2, "diff_pair_gap": 0.25, "diff_pair_via_gap": 0.25, "diff_pair_width": 0.2, "line_style": 0, "microvia_diameter": 0.3, "microvia_drill": 0.15, "pcb_color": "rgba(0, 255, 0, 0.000)", "schematic_color": "rgba(0, 255, 0, 0.000)", "track_width": 0.4, "via_diameter": 1.0, "via_drill": 0.5, "wire_width": 0.2},
        {"name": "Motor_Power", "bus_width": 12, "clearance": 0.3, "diff_pair_gap": 0.25, "diff_pair_via_gap": 0.25, "diff_pair_width": 0.2, "line_style": 0, "microvia_diameter": 0.3, "microvia_drill": 0.15, "pcb_color": "rgba(255, 128, 0, 0.000)", "schematic_color": "rgba(255, 128, 0, 0.000)", "track_width": 1.0, "via_diameter": 1.0, "via_drill": 0.5, "wire_width": 0.2}
      ],
      "rule_area": 100000000,
      "rule_severity": {
        "clearance": "error",
        "copper_edge_clearance": "error",
        "courtyards_overlap": "warning",
        "diff_pair_gap_out_of_range": "error",
        "diff_pair_unbalanced_length": "error",
        "drill_out_of_range": "error",
        "duplicate_footprints": "warning",
        "extra_footprint": "warning",
        "footprint_courtyard_overlap": "warning",
        "footprint_type_mismatch": "warning",
        "hole_clearance": "error",
        "hole_near_hole": "error",
        "invalid_outline": "error",
        "item_on_disabled_layer": "error",
        "items_not_allowed": "error",
        "length_out_of_range": "error",
        "malformed_courtyard": "error",
        "microvia_drill_out_of_range": "error",
        "missing_courtyard": "ignore",
        "missing_footprint": "error",
        "net_conflict": "error",
        "no_connect_connected": "warning",
        "npth_inside_courtyard": "warning",
        "padstack_zero_drill": "error",
        "silkscreen_over_copper": "warning",
        "skew_out_of_range": "error",
        "through_hole_pad_without_hole": "error",
        "too_many_vias": "warning",
        "track_width": "error",
        "unconnected_items": "error",
        "unresolved_variable": "error",
        "via_dangling": "warning",
        "zone_has_no_outline": "warning"
      },
      "technology": 0,
      "track_widths": [0.2, 0.25, 0.3, 0.4, 0.5, 0.6, 0.8, 1.0, 1.2, 1.5, 2.0],
      "via_diameters": [0.6, 0.8, 1.0, 1.2, 1.5],
      "via_drills": [0.3, 0.4, 0.5, 0.6],
      "zone_clearances": [0.508, 0.254, 0.2, 0.15]
    },
    "layer_presets": [],
    "layers": {
      "0": {"name": "F.Cu", "type": 0, "visible": true},
      "31": {"name": "B.Cu", "type": 0, "visible": true},
      "32": {"name": "B.Adhes", "type": 1, "visible": false},
      "33": {"name": "F.Adhes", "type": 1, "visible": false},
      "34": {"name": "B.Paste", "type": 2, "visible": true},
      "35": {"name": "F.Paste", "type": 2, "visible": true},
      "36": {"name": "B.SilkS", "type": 3, "visible": true},
      "37": {"name": "F.SilkS", "type": 3, "visible": true},
      "38": {"name": "B.Mask", "type": 4, "visible": true},
      "39": {"name": "F.Mask", "type": 4, "visible": true},
      "40": {"name": "Dwgs.User", "type": 5, "visible": true},
      "41": {"name": "Cmts.User", "type": 5, "visible": true},
      "42": {"name": "Eco1.User", "type": 5, "visible": false},
      "43": {"name": "Eco2.User", "type": 5, "visible": false},
      "44": {"name": "Edge.Cuts", "type": 6, "visible": true},
      "45": {"name": "Margin", "type": 7, "visible": false},
      "46": {"name": "B.CrtYd", "type": 8, "visible": false},
      "47": {"name": "F.CrtYd", "type": 8, "visible": false},
      "48": {"name": "B.Fab", "type": 9, "visible": false},
      "49": {"name": "F.Fab", "type": 9, "visible": false},
      "50": {"name": "In1.Cu", "type": 0, "visible": false},
      "51": {"name": "In2.Cu", "type": 0, "visible": false}
    },
    "physical_stackup": {
      "copper_finish": "HASL",
      "copper_finishes": ["HASL", "ENIG", "OSP", "Immersion Tin", "Immersion Silver"],
      "dielectric_constraints": [],
      "board_thickness": 1.6,
      "board_thicknesses": [0.8, 1.0, 1.2, 1.6, 2.0, 2.4],
      "copper_thickness": 35,
      "copper_thicknesses": [17, 35, 70, 105],
      "material": "FR4",
      "materials": ["FR4", "FR4 High Tg", "Rogers", "Aluminum"],
      "solder_mask": "Green",
      "solder_masks": ["Green", "Red", "Blue", "Black", "White", "Yellow", "Matte Green", "Matte Black"],
      "silkscreen": "White",
      "silkscreens": ["White", "Black", "Yellow"],
      "finished_copper_thickness": 42,
      "finished_copper_thicknesses": [35, 42, 70, 105],
      "prepreg_thicknesses": [],
      "core_thicknesses": []
    }
  },
  "libraries": {
    "symbol_lib_table": [],
    "footprint_lib_table": []
  },
  "metadata": {
    "version": 1,
    "generator": "KiCad Schematic Editor",
    "generator_version": "9.0"
  },
  "project": {
    "files": [],
    "meta": {
      "filename": "base_esp32.kicad_pro",
      "version": 1
    },
    "name": "base_esp32"
  },
  "schematic": {
    "annotation": {
      "annotate_start_num": 1,
      "direction": 0,
      "hierarchical_prefix_separator": "/",
      "hierarchical_sheet_prefix": "1",
      "use_hierarchical_reference": true
    },
    "drawing": {
      "default_text_size": 1.27,
      "inter_sheet_ref_prefix": "#",
      "inter_sheet_ref_suffix": "",
      "inter_sheet_ref_size": 1.27,
      "label_size_ratio": 0.375,
      "pin_symbol_size": 1.27,
      "text_offset": 0.0
    },
    "legacy_lib_order": false,
    "net_name_format": "",
    "ngspice": {
      "fix_sim_value": false,
      "model_mode": 1,
      "use_ngspice": false
    },
    "page_settings": [
      {"page": "1", "width": 297, "height": 210, "left_margin": 20, "right_margin": 10, "top_margin": 10, "bottom_margin": 10, "orientation": "L"}
    ],
    "text_variables": {}
  }
}
```

---

## `sym-lib-table` (Librerías símbolos)

```
(sym_lib_table
  (lib (name "project_symbols") (type "KiCad") (uri "${KIPRJMOD}/ESP32-DevKitC.kicad_sym") (options "") (descr "Project symbols"))
  (lib (name "MX1508") (type "KiCad") (uri "${KIPRJMOD}/MX1508.kicad_sym") (options "") (descr "MX1508 Motor Driver"))
  (lib (name "power") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/power.kicad_sym") (options "") (descr "Power symbols"))
  (lib (name "Device") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Device.kicad_sym") (options "") (descr "Passive devices"))
  (lib (name "Connector") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Connector.kicad_sym") (options "") (descr "Connectors"))
  (lib (name "Regulator") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Regulator.kicad_sym") (options "") (descr "Voltage regulators"))
  (lib (name "Sensor") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Sensor.kicad_sym") (options "") (descr "Sensors"))
  (lib (name "Switch") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Switch.kicad_sym") (options "") (descr "Switches"))
  (lib (name "LED") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/LED.kicad_sym") (options "") (descr "LEDs"))
  (lib (name "Buzzer") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Buzzer.kicad_sym") (options "") (descr "Buzzers"))
  (lib (name "Motor") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Motor.kicad_sym") (options "") (descr "Motor drivers"))
  (lib (name "MCU_ESP32") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/MCU_ESP32.kicad_sym") (options "") (descr "ESP32 MCU"))
  (lib (name "Interface_USB") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Interface_USB.kicad_sym") (options "") (descr "USB interfaces"))
  (lib (name "Amplifier") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Amplifier.kicad_sym") (options "") (descr "Amplifiers"))
  (lib (name "Opto") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Opto.kicad_sym") (options "") (descr "Optocouplers"))
  (lib (name "Transistor") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Transistor.kicad_sym") (options "") (descr "Transistors"))
  (lib (name "Diode") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Diode.kicad_sym") (options "") (descr "Diodes"))
  (lib (name "Crystal") (type "KiCad") (uri "${KICAD_SYMBOL_DIR}/Crystal.kicad_sym") (options "") (descr "Crystals"))
```

---

## `fp-lib-table` (Librerías footprints)

```
(fp_lib_table
  (lib (name "project_footprints") (type "KiCad") (uri "${KIPRJMOD}/my_symbols.pretty") (options "") (descr "Project footprints"))
  (lib (name "logo_footprint") (type "KiCad") (uri "${KIPRJMOD}/logo_footprint") (options "") (descr "Logo and QR footprints"))
  (lib (name "Connector") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Connector.pretty") (options "") (descr "Connectors"))
  (lib (name "Package_TO_SOT_THT") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Package_TO_SOT_THT.pretty") (options "") (descr "TO/SOT THT"))
  (lib (name "Package_TO_SOT_SMD") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Package_TO_SOT_SMD.pretty") (options "") (descr "TO/SOT SMD"))
  (lib (name "Package_DIP") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Package_DIP.pretty") (options "") (descr "DIP packages"))
  (lib (name "Package_SO") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Package_SO.pretty") (options "") (descr "SO packages"))
  (lib (name "Package_QFP") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Package_QFP.pretty") (options "") (descr "QFP packages"))
  (lib (name "Resistor_SMD") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Resistor_SMD.pretty") (options "") (descr "SMD Resistors"))
  (lib (name "Resistor_THT") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Resistor_THT.pretty") (options "") (descr "THT Resistors"))
  (lib (name "Capacitor_SMD") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Capacitor_SMD.pretty") (options "") (descr "SMD Capacitors"))
  (lib (name "Capacitor_THT") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Capacitor_THT.pretty") (options "") (descr "THT Capacitors"))
  (lib (name "LED_SMD") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/LED_SMD.pretty") (options "") (descr "SMD LEDs"))
  (lib (name "LED_THT") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/LED_THT.pretty") (options "") (descr "THT LEDs"))
  (lib (name "Diode_SMD") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Diode_SMD.pretty") (options "") (descr "SMD Diodes"))
  (lib (name "Diode_THT") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Diode_THT.pretty") (options "") (descr "THT Diodes"))
  (lib (name "Potentiometer") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Potentiometer.pretty") (options "") (descr "Potentiometers"))
  (lib (name "Switch") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Switch.pretty") (options "") (descr "Switches"))
  (lib (name "Buzzer") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Buzzer.pretty") (options "") (descr "Buzzers"))
  (lib (name "Terminal_Block") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Terminal_Block.pretty") (options "") (descr "Terminal blocks"))
  (lib (name "MountingHole") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/MountingHole.pretty") (options "") (descr "Mounting holes"))
  (lib (name "Mechanical") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Mechanical.pretty") (options "") (descr "Mechanical"))
  (lib (name "TestPoint") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/TestPoint.pretty") (options "") (descr "Test points"))
  (lib (name "RF_Module") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/RF_Module.pretty") (options "") (descr "RF Modules"))
  (lib (name "Housings_DIP") (type "KiCad") (uri "${KICAD_FOOTPRINT_DIR}/Housings_DIP.pretty") (options "") (descr "DIP housings"))
```

---

## Hoja esquemática principal (Jerarquía)

```
base_esp32.kicad_sch
├── Portada (Title Block)
├── Main Sheet
│   ├── Power_Input (Jack DC, USB, ORing)
│   ├── Power_5V (LM7805)
│   ├── Power_3V3 (AMS1117)
│   ├── Power_Monitor (A9013)
│   ├── MCU_ESP32 (Socket 2x19)
│   ├── Motor_Driver (MX1508)
│   ├── Sensors (DS18B20, LM35, LDR)
│   ├── Actuators (Buzzer, LEDs, RGB, Servo)
│   ├── Inputs (DIP, Push, SPDT, Pot)
│   ├── Connectors (Headers, Term Block, Servo)
│   └── Mounting (4x M3)
```

---

## Net naming convention

| Patrón | Ejemplo | Uso |
|--------|---------|-----|
| `+5V` | `+5V` | Rail potencia 5V |
| `+3V3` | `+3V3` | Rail potencia 3.3V |
| `GND` | `GND` | Tierra |
| `GPIO<num>` | `GPIO12` | Pines GPIO directos |
| `<FUNC>_<num>` | `MOT_A_IN1` | Función específica |
| `ADC<num>` | `ADC1_CH0` | Canales ADC |
| `I2C_<SIG>` | `I2C_SDA` | Bus I2C |
| `SPI_<SIG>` | `SPI_MOSI` | Bus SPI |
| `UART_<SIG>` | `UART_TX` | Bus UART |

---

## Versionado KiCad

| Archivo | Formato | Compatibilidad |
|---------|---------|----------------|
| `.kicad_pro` | JSON | KiCad 7+ |
| `.kicad_sch` | S-expr | KiCad 6+ |
| `.kicad_pcb` | S-expr | KiCad 6+ |
| `.kicad_sym` | S-expr | KiCad 6+ |
| `.kicad_mod` | S-expr | KiCad 6+ |

> **Nota**: KiCad 9 (2024) usa mismo formato. Compatible hacia adelante.