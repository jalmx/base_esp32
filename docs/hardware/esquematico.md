# Esquemático

## Visión general

El esquemático está organizado en **hojas jerárquicas** por bloques funcionales:

```
base_esp32.kicad_sch
├── Portada (Title Block)
├── Hoja principal: Conexiones ESP32
├── Power_5V: LM7805 + entrada DC/USB
├── Power_3V3: AMS1117 3.3V
├── Power_Monitor: A9013 current sense
├── Motor_Driver: MX1508
├── Sensors: DS18B20, LM35, LDR
├── Actuators: Buzzer, LEDs, Servo, RGB
├── Inputs: DIP, Push, SPDT, Pot
├── Connectors: Headers, Terminal, Servo
└── Mounting: 4× Mounting holes
```

## Hoja principal: ESP32 DevKit V4

### Socket pinout (J1-J2, headers 2×19)

| Pin ESP32 | Función Base | Componente conectado |
|-----------|--------------|----------------------|
| **GPIO0** | Boot/LED | LED D1 (pull-up) |
| **GPIO2** | LED Built-in / RGB | LED D2 + RGB WS2812B |
| **GPIO4** | User LED | LED D3 |
| **GPIO5** | User LED | LED D4 |
| **GPIO12** | Motor IN1 | MX1508 IN1 |
| **GPIO13** | Motor IN2 | MX1508 IN2 |
| **GPIO14** | Motor IN3 | MX1508 IN3 |
| **GPIO15** | Motor IN4 / ADC2_CH3 | MX1508 IN4 + Pot |
| **GPIO16** | Buzzer | Buzzer pasivo |
| **GPIO17** | DS18B20 | 1-Wire temp sensor |
| **GPIO18** | Servo PWM | Conector servo |
| **GPIO19** | I2C SCL | Header expansion |
| **GPIO21** | I2C SDA | Header expansion |
| **GPIO22** | SPI MISO | Header expansion |
| **GPIO23** | SPI MOSI | Header expansion |
| **GPIO25** | ADC2_CH8 / DAC1 | LM35 / User |
| **GPIO26** | ADC2_CH9 / DAC2 | LDR / User |
| **GPIO27** | ADC2_CH7 | User |
| **GPIO32** | ADC1_CH4 | User |
| **GPIO33** | ADC1_CH5 | User |
| **GPIO34** | ADC1_CH6 (input only) | Potenciómetro |
| **GPIO35** | ADC1_CH7 (input only) | User |
| **GPIO36** | ADC1_CH0 (input only) | LM35 |
| **GPIO39** | ADC1_CH3 (input only) | LDR |

> **Nota**: GPIO6-11 conectados a flash interno (no usar). GPIO34-39 solo entrada.

## Hoja Power_5V: LM7805

```
VIN (7-12V) ──┬──┬──► Jack DC (J_PWR)
              │  └──► USB 5V (via diodo Schottky)
              ▼
         ┌─────────┐
         │ LM7805  │──► +5V Rail
         │ TO-220  │     │
         └────┬────┘     ├─► Motor Driver (MX1508 VCC)
              │          ├─► Servo VCC
              │          ├─► Terminal Block (Motor Power)
              │          └─► A9013 Sense Input
              ▼
           GND
```

**Componentes clave:**
- **C1**: 470µF electrolítico (bulk input)
- **C2**: 100nF cerámico (decoupling input)
- **C3**: 100µF electrolítico (bulk output)
- **C4**: 100nF cerámico (decoupling output)
- **D1**: Diodo Schottky 1N5819 (protección USB/Jack)
- **D2**: LED Power 5V + R (1kΩ)
- **JP1**: Jumper selección fuente (Jack vs USB)

## Hoja Power_3V3: AMS1117

```
+5V Rail ──┬──► AMS1117-3.3 ──► +3.3V Rail
           │        │
           │        ├─► C5: 100µF (output bulk)
           │        └─► C6: 100nF (decoupling)
           │
           └──► LED 3.3V + R (1kΩ)
```

**Salida**: 800mA max (suficiente para ESP32 + sensores)

## Hoja Power_Monitor: A9013

```
+5V Rail ──► A9013 (Hall Effect) ──► GPIO34 (ADC1_CH6)
                    │
                    └─► Sensitivity: 400mV/A
                        Offset: 2.5V @ 0A
                        Range: ±5A
```

**Fórmula conversión:**
```cpp
float current = (adc_voltage - 2.5) / 0.4;  // Amperios
```

## Hoja Motor_Driver: MX1508

```
GPIO12 ──► IN1 ──┐
GPIO13 ──► IN2 ──┤   MX1508 (Dual H-Bridge)
GPIO14 ──► IN3 ──┤   │ OUT1+OUT2 = Motor A
GPIO15 ──► IN4 ──┘   │ OUT3+OUT4 = Motor B
                     ▼
              +5V Rail (VM)
              GND
              Terminal Block: MOT_A+, MOT_A-, MOT_B+, MOT_B-
```

**Especificaciones MX1508:**
- Voltaje: 2-10V
- Corriente: 1.5A cont / 2.5A pico por canal
- PWM: Hasta 20kHz
- Protección: Térmica, under-voltage

## Hoja Sensors

### DS18B20 (1-Wire)
```
GPIO17 ──► Data (pull-up 4.7kΩ a 3.3V)
VDD ──► 3.3V
GND ──► GND
```
**Dirección**: Parasite power o VDD mode (configurable por jumper)

### LM35 (Analógico)
```
GPIO36 (ADC1_CH0) ──► Vout
VDD ──► 3.3V
GND ──► GND
```
**Fórmula**: `Temp(°C) = ADC_mV / 10`

### LDR + Divisor tensión
```
GPIO39 (ADC1_CH3) ──► Punto medio
              ┌─────┴─────┐
              ▼           ▼
           LDR          10kΩ
              │           │
             3.3V        GND
```

## Hoja Actuators

### Buzzer pasivo
```
GPIO16 ──► Buzzer (+)
GND ─────► Buzzer (-)
```
**Frecuencia**: Control por PWM/tone()

### LEDs Usuario (×4)
```
GPIO0  ──► LED D1 (R=330Ω) ──► GND  (Boot indicator)
GPIO2  ──► LED D2 (R=330Ω) ──► GND  (Built-in / RGB)
GPIO4  ──► LED D3 (R=330Ω) ──► GND
GPIO5  ──► LED D4 (R=330Ω) ──► GND
```

### LED RGB (WS2812B)
```
GPIO2 ──► DIN (data)
3.3V ───► VDD
GND ────► VSS
```
**Nota**: Comparte GPIO2 con LED D2 - usar uno a la vez

### Conector Servo (3 pines estándar)
```
GPIO18 ──► Signal
5V  ─────► VCC (from 5V rail)
GND ─────► GND
```

## Hoja Inputs

### DIP Switch ×4 (SW1)
```
GPIO27 ──► SW1-1
GPIO32 ──► SW1-2
GPIO33 ──► SW1-3
GPIO35 ──► SW1-4
Común ───► GND (pull-up interno ESP32)
```

### Push Button (SW2)
```
GPIO25 ──► SW2 (momentary)
Común ───► GND (pull-up interno)
```

### SPDT Switch (SW3)
```
GPIO26 ──► SW3-COM
        ├──► SW3-NO (3.3V via R pull-up)
        └──► SW3-NC (GND)
```

### Potenciómetro 10k (RV1)
```
GPIO34 (ADC1_CH6) ──► Wiper
3.3V ────────────────► End 1
GND ────────────────► End 2
```

## Hoja Connectors

### Headers ESP32 (J1, J2 - 2×19)
- Pitch 2.54mm
- Macho en PCB, Hembra en DevKit
- Todas las señales GPIO, Power, GND

### Terminal Block 2×2 (J_TERM)
```
Pin 1: Motor A+
Pin 2: Motor A-
Pin 3: Motor B+
Pin 4: Motor B-
```
**Rating**: 10A, 250V, wire 12-24 AWG

### Conector Servo (J_SERVO)
- 3 pines 2.54mm (Signal, VCC, GND)
- Compatible estándar Futaba/JR

### Mounting Holes (×4)
- Diámetro 3.2mm (M3)
- Plated para conexión GND opcional
- Esquinas PCB (100×80mm)

## Referencias cruzadas

- [PCB Layout](pcb.md) - Implementación física
- [Componentes](componentes.md) - BOM completa
- [Pinout ESP32](pinout.md) - Mapeo detallado
- [Alimentación](../alimentacion/overview.md) - Análisis power tree