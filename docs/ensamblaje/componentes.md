# Ensamblaje - Lista de componentes (BOM)

## Dónde comprar

| Proveedor | Ventajas | Envío a España |
|-----------|----------|----------------|
| **LCSC / JLCPCB** | Precios bajos, stock real, integra con JLCPCB | 5-10 días |
| **Mouser / DigiKey** | Stock garantizado, datasheets, rápido | 1-3 días |
| **Amazon / AliExpress** | Barato, envío rápido (Amazon) | 1-7 días |
| **Tiendas locales** | Soporte, sin aduanas | Inmediato |

> **Tip**: Haz pedido único en LCSC + JLCPCB para PCBs + componentes (ensamblaje opcional)

---

## BOM por categorías

### 1. MCU y Socket

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| U_ESP32 | 1 | ESP32 DevKit V4 (WROOM-32E) | - | ESP32-DevKitC (custom) | C132755 | 868-ESP32-DEVKITC-V4 |
| J1, J2 | 2 | Header hembra 2x19 | 2.54mm | Header_2x19_F | C110754 | 649-68000-136 |

### 2. Reguladores y protección

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| U_5V | 1 | Regulador lineal 5V 1.5A | LM7805CT | TO-220-3_Vertical | C17332 | 511-LM7805CT |
| U_3V3 | 1 | Regulador LDO 3.3V 800mA | AMS1117-3.3 | SOT-223-4 | C2882143 | 511-AMS1117-3.3 |
| D_PWR | 1 | Diodo Schottky 1A 40V | 1N5819 | SOD-123 | C23272 | 511-1N5819 |
| D_PROT | 1 | Diodo Schottky 1A 40V | 1N5819 | SOD-123 | C23272 | 511-1N5819 |
| D_DISC | 1 | Diodo Schottky 1A 40V | 1N5819 | SOD-123 | C23272 | 511-1N5819 |
| F_PTC | 0 | *Opcional v0.5* PTC 1A | 1206/2920 | 1206/2920 | C114542 | 652-MF-R1000 |

### 3. Driver Motor

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| U_MOT | 1 | Dual H-Bridge 1.5A | MX1508 / MX1515 | SOP-8 (custom) | C2882143 | - |

### 4. Monitor Corriente

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| U_IMON | 1 | Sensor Hall 400mV/A | A9013 | SIP-3 | C2882143 | 620-A9013 |

### 5. Sensores

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| U_TEMP1 | 1 | Temp digital 1-Wire | DS18B20 | TO-92 / Waterproof | C17829 | 700-DS18B20 |
| U_TEMP2 | 1 | Temp analógica 10mV/°C | LM35DZ | TO-92 | C2882143 | 511-LM35DZ |
| R_DS18B20 | 1 | Resistencia pull-up | 4.7kΩ | 0805 / Axial | C17434 | 660-ERJ-6ENF4701V |
| U_LDR | 1 | Fotoresistor | GL5528 / 5537 | 5mm THT | C17434 | - |
| R_LDR | 1 | Resistencia divisor | 10kΩ | 0805 / Axial | C17434 | 660-ERJ-6ENF1002V |

### 6. Actuadores

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| BZ1 | 1 | Buzzer pasivo 5V | 2.7kHz | 12mm THT | C17434 | 660-PKLS-1212E4001-R1 |
| LED_D1 | 1 | LED 3mm Rojo | 620nm | LED_THT_3mm | C17434 | 604-1080511000 |
| LED_D2 | 1 | LED 3mm Azul | 470nm | LED_THT_3mm | C17434 | 604-1080511000 |
| LED_D3 | 1 | LED 3mm Verde | 525nm | LED_THT_3mm | C17434 | 604-1080511000 |
| LED_D4 | 1 | LED 3mm Amarillo | 590nm | LED_THT_3mm | C17434 | 604-1080511000 |
| LED_RGB | 1 | LED RGB WS2812B | 5050 | LED_SMD_5050 | C2882143 | 604-1811 |
| R_LED1-4 | 4 | Resistencia limitación | 330Ω | 0805 / Axial | C17434 | 660-ERJ-6ENF3300V |
| R_LED5V | 1 | Resistencia LED 5V | 1kΩ | 0805 / Axial | C17434 | 660-ERJ-6ENF1001V |
| R_LED3V3 | 1 | Resistencia LED 3.3V | 1kΩ | 0805 / Axial | C17434 | 660-ERJ-6ENF1001V |

### 7. Entradas

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| SW_DIP | 1 | DIP Switch 4 posiciones | - | SW_DIP_4 | C17434 | 611-CTS-206-4 |
| SW_PUSH | 1 | Pulsador 6x6mm | - | SW_Push_6x6 | C17434 | 611-B3F-1000 |
| SW_SPDT | 1 | Switch deslizante 3 pos | ON-OFF-ON | SS-12D13 | C17434 | 611-SS-12D13-G2 |
| RV_POT | 1 | Potenciómetro 10k lineal | 10kΩ | Pot_THT_B10K | C17434 | 611-PTV09A-4020F |

### 8. Conectores

| Ref | Cant | Descripción | Valor | Footprint | LCSC | Mouser |
|-----|------|-------------|-------|-----------|------|--------|
| J_PWR | 1 | Jack DC 2.1mm | - | PJ-002AH | C17434 | 163-1637 |
| J_USB | 1 | USB Micro-B hembra | - | USB_Micro_B | C17434 | 475-1015-1 |
| J_TERM | 1 | Terminal block 4p | 5.08mm | Terminal_Block_4p | C17434 | 517-282836-4 |
| J_SERVO | 1 | Header 3p macho | 2.54mm | Header_3p_M | C17434 | 649-68000-136 |
| J_I2C | 1 | Header 4p macho | 2.54mm | Header_4p_M | C17434 | 649-68000-136 |
| J_SPI | 1 | Header 6p macho | 2.54mm | Header_6p_M | C17434 | 649-68000-136 |
| J_GPIO | 1 | Header 10p macho | 2.54mm | Header_10p_M | C17434 | 649-68000-136 |
| MH1-4 | 4 | Mounting hole M3 | 3.2mm | MountingHole_3.2mm | - | - |

### 9. Pasivos - Capacitores

| Ref | Cant | Valor | Tipo | Tensión | Footprint | LCSC |
|-----|------|-------|------|---------|-----------|------|
| C_IN_5V | 1 | 470µF | Electrolítico | 25V | Cap_THT_D8 | C17434 |
| C_IN_5V_S | 1 | 100nF | Cerámico X7R | 25V | 0805 | C17434 |
| C_OUT_5V | 1 | 100µF | Electrolítico | 10V | Cap_THT_D6 | C17434 |
| C_OUT_5V_S | 1 | 100nF | Cerámico X7R | 10V | 0805 | C17434 |
| C_OUT_3V3 | 1 | 100µF | Electrolítico | 6.3V | Cap_THT_D6 | C17434 |
| C_OUT_3V3_S | 1 | 100nF | Cerámico X7R | 6.3V | 0805 | C17434 |
| C_IMON | 1 | 100nF | Cerámico X7R | 6.3V | 0805 | C17434 |
| C_MOT_VM | 1 | 100µF | Electrolítico | 16V | Cap_THT_D6 | C17434 |
| C_MOT_VM_S | 1 | 100nF | Cerámico X7R | 16V | 0805 | C17434 |
| C_MOT_VCC | 1 | 10µF | Cerámico X5R | 10V | 0805 | C17434 |
| C_RGB | 1 | 100nF | Cerámico X7R | 6.3V | 0805 | C17434 |
| C_LDR | 1 | 100nF | Cerámico X7R | 6.3V | 0805 | C17434 |
| C_DEC_* | 10+ | 100nF | Cerámico X7R | 10-25V | 0805 | C17434 |

### 10. Pasivos - Resistencias

| Ref | Cant | Valor | Potencia | Tolerancia | Footprint | LCSC |
|-----|------|-------|----------|------------|-----------|------|
| R_IMON | 1 | 10kΩ | 1/8W | 1% | 0805 | C17434 |
| R_PU_DIP | 4 | 10kΩ | 1/8W | 1% | 0805 | C17434 |
| R_LED_USR | 4 | 330Ω | 1/8W | 1% | 0805 | C17434 |
| R_LED_PWR | 2 | 1kΩ | 1/8W | 1% | 0805 | C17434 |
| R_*_0805 | 20+ | 10k, 4.7k, 1k, 330, 100 | 1/8W | 1% | 0805 | C17434 |
| R_*_AXIAL | 10+ | 1k, 330, 4.7k, 10k | 1/4W | 5% | Axial_0207 | C17434 |

### 11. Mecánicos

| Ref | Cant | Descripción | Especificación |
|-----|------|-------------|----------------|
| - | 4 | Tornillo M3x10 | Acero / Nylon |
| - | 4 | Separador M3x10mm | Nylon (aislante) |
| - | 4 | Tuerca M3 | Acero / Nylon |
| - | 1 | Disipador TO-220 | Para LM7805 |
| - | 1 | Pasta térmica | Silicone / Cerámica |

---

## Resumen coste estimado (precios 2024, unidades sueltas)

| Categoría | Coste EUR |
|-----------|-----------|
| MCU + Socket | ~€5.50 |
| Reguladores + Diodos | ~€1.50 |
| Driver Motor | ~€0.80 |
| Monitor Corriente | ~€1.20 |
| Sensores | ~€3.00 |
| Actuadores | ~€2.50 |
| Entradas | ~€1.50 |
| Conectores | ~€2.00 |
| Pasivos (C+R) | ~€2.00 |
| Mecánicos | ~€1.00 |
| **TOTAL** | **~€21.00** |

*PCB no incluido (~$2-5 por 5 uds en JLCPCB)*

---

## Alternativas / Sustituciones

| Original | Alternativa | Notas |
|----------|-------------|-------|
| LM7805CT | LM7805TO-220 / L7805CV | Mismo pinout |
| AMS1117-3.3 | LM1117-3.3 / LD1117-3.3 | SOT-223 compatible |
| MX1508 | MX1515 / L9110S | Verificar pinout |
| DS18B20 | DS18B20+ / DS18B20-PAR | Versión parasitica = 2 hilos |
| LM35DZ | LM35CAZ / TMP36 | TMP36 = 500mV offset |
| GL5528 | GL5537 / VT90N2 | Similares |
| WS2812B | SK6812 / WS2812C | Compatibles NeoPixel |
| 1N5819 | SS14 / SB140 | Schottky 1A 40V |
| Terminal Block 5.08 | 3.5mm / 5.0mm | Verificar footprint |

---

## Archivos BOM para fabricación

- `fabricacion/bom.csv` - Formato JLCPCB/LCSC (LCSC Part, Designator, Quantity)
- `fabricacion/bom.xlsx` - Con columnas extra (proveedor alternativo, precio, stock)
- `fabricacion/pnp.csv` - Pick & Place para SMT (Mid X, Mid Y, Rotation, Layer)