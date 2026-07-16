# Lista de materiales (BOM)

## Resumen por categorías

| Categoría | Componentes | Coste estimado |
|-----------|-------------|----------------|
| **MCU & Socket** | 2 | ~$4.50 |
| **Reguladores** | 5 | ~$2.00 |
| **Driver Motor** | 3 | ~$1.50 |
| **Sensores** | 6 | ~$3.00 |
| **Actuadores** | 8 | ~$2.50 |
| **Entradas** | 7 | ~$1.50 |
| **Conectores** | 12 | ~$3.00 |
| **Pasivos (R/C/D)** | 35+ | ~$2.00 |
| **Mecánicos** | 4 | ~$0.50 |
| **TOTAL** | ~82 | **~$20.50** |

> Precios orientativos (cantidades 1-10, proveedores chinos). PCB no incluido.

## BOM Detallada

### MCU y Socket

| Ref | Cantidad | Valor/PN | Footprint | Descripción | Datasheet |
|-----|----------|----------|-----------|-------------|-----------|
| U_ESP32 | 1 | ESP32-DevKitC-V4 | ESP32-DevKitC (custom) | Módulo ESP32-WROOM-32E | [PDF](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wrover-32e_datasheet_en.pdf) |
| J_ESP32 | 2 | Header 2×19 F | Header_2x19_F | Socket hembra para DevKit | - |

### Reguladores de voltaje

| Ref | Cantidad | Valor/PN | Footprint | Descripción | Datasheet |
|-----|----------|----------|-----------|-------------|-----------|
| U_5V | 1 | LM7805CT | TO-220-3_Vertical | Regulador lineal 5V 1.5A | [PDF](../datasheets/LM7805.pdf) |
| U_3V3 | 1 | AMS1117-3.3 | SOT-223 | Regulador lineal 3.3V 800mA | [PDF](../datasheets/AMS1117.pdf) |
| D_PWR | 1 | 1N5819 | SOD-123 | Diodo Schottky 1A 40V (protección) | - |
| LED_5V | 1 | LED 3mm Red | LED_THT_3mm | Indicador 5V | - |
| LED_3V3 | 1 | LED 3mm Green | LED_THT_3mm | Indicador 3.3V | - |
| R_LED5V | 1 | 1kΩ | R_Axial_0207 | Limitación LED 5V | - |
| R_LED3V3 | 1 | 1kΩ | R_Axial_0207 | Limitación LED 3.3V | - |
| C_IN_5V | 1 | 470µF 25V | Cap_THT_D8 | Bulk input LM7805 | - |
| C_IN_5V_S | 1 | 100nF | C_0805 | Decoupling input | - |
| C_OUT_5V | 1 | 100µF 10V | Cap_THT_D6 | Bulk output 5V | - |
| C_OUT_5V_S | 1 | 100nF | C_0805 | Decoupling output | - |
| C_OUT_3V3 | 1 | 100µF 6.3V | Cap_THT_D6 | Bulk output 3.3V | - |
| C_OUT_3V3_S | 1 | 100nF | C_0805 | Decoupling output | - |
| JP_PWR | 1 | Jumper 2p | Jumper_2p | Selección Jack/USB | - |

### Monitor de corriente

| Ref | Cantidad | Valor/PN | Footprint | Descripción | Datasheet |
|-----|----------|----------|-----------|-------------|-----------|
| U_IMON | 1 | A9013 | SIP-3 | Sensor Hall 400mV/A | - |
| C_IMON | 1 | 100nF | C_0805 | Decoupling VCC | - |
| R_IMON | 1 | 10kΩ | R_0805 | Pull-down salida | - |

### Driver Motor MX1508

| Ref | Cantidad | Valor/PN | Footprint | Descripción | Datasheet |
|-----|----------|----------|-----------|-------------|-----------|
| U_MOT | 1 | MX1508 | MX1508_SOP8 (custom) | Dual H-Bridge 1.5A | [PDF](../datasheets/MX1515H_C5119046.pdf) |
| C_MOT_VM | 1 | 100µF 16V | Cap_THT_D6 | Bulk VM motor | - |
| C_MOT_VM_S | 1 | 100nF | C_0805 | Decoupling VM | - |
| C_MOT_VCC | 1 | 10µF 10V | C_0805 | Decoupling VCC logic | - |
| D_MOT_FLY | 4 | 1N5819 | SOD-123 | Flyback diodes (integradas en MX1508) | - |

### Sensores

| Ref | Cantidad | Valor/PN | Footprint | Descripción | Datasheet |
|-----|----------|----------|-----------|-------------|-----------|
| U_TEMP1 | 1 | DS18B20 | TO-92 / Waterproof probe | Temp digital 1-Wire | [PDF](https://datasheets.maximintegrated.com/en/ds/DS18B20.pdf) |
| U_TEMP2 | 1 | LM35DZ | TO-92 | Temp analógica 10mV/°C | [PDF](https://www.ti.com/lit/ds/symlink/lm35.pdf) |
| R_DS18B20 | 1 | 4.7kΩ | R_Axial_0207 | Pull-up 1-Wire | - |
| U_LDR | 1 | GL5528 / genérico | LDR_5mm | Fotoresistor | - |
| R_LDR | 1 | 10kΩ | R_Axial_0207 | Divisor tensión LDR | - |
| C_LDR | 1 | 100nF | C_0805 | Filtro ruido ADC | - |

### Actuadores

| Ref | Cantidad | Valor/PN | Footprint | Descripción | Datasheet |
|-----|----------|----------|-----------|-------------|-----------|
| BZ1 | 1 | Buzzer pasivo 5V | Buzzer_THT_12mm | Buzzer piezoeléctrico | [PDF](../datasheets/SS-12D1305-G2.PDF) |
| LED_USR1 | 1 | LED 3mm Red | LED_THT_3mm | Usuario GPIO0 | - |
| LED_USR2 | 1 | LED 3mm Blue | LED_THT_3mm | Usuario GPIO2/Built-in | - |
| LED_USR3 | 1 | LED 3mm Green | LED_THT_3mm | Usuario GPIO4 | - |
| LED_USR4 | 1 | LED 3mm Yellow | LED_THT_3mm | Usuario GPIO5 | - |
| LED_RGB | 1 | WS2812B 5050 | LED_SMD_5050 | LED RGB direccionable | - |
| R_LED_USR | 4 | 330Ω | R_Axial_0207 | Limitación LEDs usuario | - |
| C_RGB | 1 | 100nF | C_0805 | Decoupling WS2812B | - |
| J_SERVO | 1 | Header 3p M | Header_3p_M | Conector servo estándar | - |

### Entradas

| Ref | Cantidad | Valor/PN | Footprint | Descripción | Datasheet |
|-----|----------|----------|-----------|-------------|-----------|
| SW_DIP | 1 | DIP Switch 4pos | SW_DIP_4 | 4 switches configurables | [PDF](../datasheets/SS12D00.pdf) |
| SW_PUSH | 1 | Push button 6×6mm | SW_Push_6x6 | Botón momentáneo | - |
| SW_SPDT | 1 | SPDT Slide Switch | SS-12D13 | Switch deslizante 3 pos | [PDF](../datasheets/SS-12D1305-G2.PDF) |
| RV_POT | 1 | Potenciómetro 10k | Pot_THT_B10K | Potenciómetro lineal | [PDF](../datasheets/POTENTIOMETER_Trim_THT.step) |
| R_PU_DIP | 4 | 10kΩ | R_0805 | Pull-up DIP (opcional, usar interno) | - |

### Conectores

| Ref | Cantidad | Valor/PN | Footprint | Descripción |
|-----|----------|----------|-----------|-------------|
| J_PWR | 1 | Jack DC 2.1mm | PJ-002AH | Entrada 7-12V |
| J_USB | 1 | USB Micro-B | USB_Micro_B | Solo 5V (via diodo) |
| J_TERM | 1 | Terminal Block 4p | Terminal_Block_4p | Motor power out |
| J_ESP32_A | 1 | Header 2×19 M | Header_2x19_M | Socket ESP32 parte 1 |
| J_ESP32_B | 1 | Header 2×19 M | Header_2x19_M | Socket ESP32 parte 2 |
| J_SERVO | 1 | Header 3p M | Header_3p_M | Servo signal/VCC/GND |
| J_I2C | 1 | Header 4p M | Header_4p_M | Expansión I2C |
| J_SPI | 1 | Header 6p M | Header_6p_M | Expansión SPI |
| J_GPIO | 1 | Header 10p M | Header_10p_M | GPIO extra |
| MH1-MH4 | 4 | Mounting Hole M3 | MountingHole_3.2mm | Tornillos M3 |

### Pasivos generales (resumen)

| Tipo | Cantidad | Valores típicos |
|------|----------|-----------------|
| **Resistencias 0805** | 20+ | 10k, 4.7k, 1k, 330Ω, 100Ω |
| **Resistencias Axial** | 10+ | 1k, 330Ω, 4.7k, 10k |
| **Capacitores 0805** | 15+ | 100nF, 10µF, 1µF |
| **Capacitores THT** | 8+ | 470µF, 100µF, 10µF |
| **Diodos SOD-123** | 5+ | 1N5819 |

## Footprints personalizados

| Nombre | Ubicación | Descripción |
|--------|-----------|-------------|
| `ESP32-DevKitC` | `assets/ESP32-DEVKIT-V1/ESP32-DEVKIT-V1.mod` | Socket 2×19 para DevKit V4 |
| `MX1508_SOP8` | `MX1508.kicad_mod` | Driver motor SOP-8 con pad térmico |
| `Logo_X` | `logo_footprint/logo_x.kicad_mod` | Logo Xizuth en silkscreen |
| `QR_Code` | `logo_footprint/qr.kicad_mod` | QR código GitHub |

## Símbolos personalizados

| Nombre | Ubicación | Descripción |
|--------|-----------|-------------|
| `ESP32-DevKitC` | `assets/ESP32-DEVKIT-V1/ESP32-DevKitC.kicad_sym` | Símbolo socket DevKit |
| `MX1508` | `MX1508.kicad_sym` | Símbolo driver motor |

## Notas de montaje

1. **Orden recomendado**: Pasivos SMD → ICs SMD → THT bajos → THT altos → Conectores → Módulo ESP32
2. **LM7805**: Montar con disipador o usar pista cobre como heatsink (4 vias térmicas)
3. **MX1508**: Soldar pad térmico expuesto a polygon VCC_MOTOR + vias a GND
4. **ESP32**: Insertar al final, verificar orientación (antena hacia borde PCB)
5. **DS18B20**: Versión TO-92 en PCB o sonda waterproof por cables
6. **Terminal Block**: Apriete moderado, verificar continuidad
7. **Mounting holes**: Usar spacers M3 nylon para evitar cortos

## Proveedores sugeridos

| Componente | LCSC | Mouser | DigiKey | AliExpress |
|------------|------|--------|---------|------------|
| ESP32 DevKit V4 | ✅ | ✅ | ✅ | ✅ |
| LM7805CT | ✅ | ✅ | ✅ | ✅ |
| AMS1117-3.3 | ✅ | ✅ | ✅ | ✅ |
| MX1508 | ✅ | ❌ | ❌ | ✅ |
| DS18B20 | ✅ | ✅ | ✅ | ✅ |
| LM35DZ | ✅ | ✅ | ✅ | ✅ |
| WS2812B | ✅ | ✅ | ✅ | ✅ |
| Conectores/THT | ✅ | ✅ | ✅ | ✅ |

## Archivos BOM exportables

- `fabricacion/bom.csv` - CSV para JLCPCB/PCBWay
- `fabricacion/bom.xlsx` - Excel con columnas extra
- `fabricacion/pnp.csv` - Pick&Place para SMT