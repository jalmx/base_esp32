# Pinout ESP32 DevKit V4

## Pinout visual

![ESP32 DevKit V4 Pinout](images/assets/esp32_devkitC_v4_pinlayout.png)

## Mapeo Base ESP32 → ESP32 DevKit V4

| Base ESP32 Pin | GPIO | Función Base | Tipo | Notas |
|----------------|------|--------------|------|-------|
| **J1-1** | GPIO36 (VP) | ADC1_CH0 | Input only | LM35 Temp |
| **J1-2** | GPIO39 (VN) | ADC1_CH3 | Input only | LDR |
| **J1-3** | GPIO34 | ADC1_CH6 | Input only | Potenciómetro |
| **J1-4** | GPIO35 | ADC1_CH7 | Input only | SW_DIP-4 |
| **J1-5** | GPIO32 | ADC1_CH4 / Touch | I/O | SW_DIP-2, LED_USR? |
| **J1-6** | GPIO33 | ADC1_CH5 / Touch | I/O | SW_DIP-3 |
| **J1-7** | GPIO25 | ADC2_CH8 / DAC1 | I/O | Buzzer |
| **J1-8** | GPIO26 | ADC2_CH9 / DAC2 | I/O | SW_SPDT, LDR alt |
| **J1-9** | GPIO27 | ADC2_CH7 / Touch | I/O | SW_DIP-1 |
| **J1-10** | GPIO14 | ADC2_CH6 / Touch | I/O | Motor IN3 |
| **J1-11** | GPIO12 | ADC2_CH5 / Touch | I/O | Motor IN1 (boot strapping!) |
| **J1-12** | GPIO13 | ADC2_CH4 / Touch | I/O | Motor IN2 |
| **J1-13** | GPIO15 | ADC2_CH3 / Touch | I/O | Motor IN4, Pot alt (strap!) |
| **J1-14** | GPIO2 | ADC2_CH2 / Touch | I/O | LED Built-in / RGB DIN |
| **J1-15** | GPIO0 | ADC2_CH1 / Touch | I/O | LED D1 / Boot (strap!) |
| **J1-16** | GPIO4 | ADC2_CH0 / Touch | I/O | LED D3 |
| **J1-17** | GPIO16 | - | I/O | (Libre) |
| **J1-18** | GPIO17 | - | I/O | DS18B20 1-Wire |
| **J1-19** | GPIO5 | - | I/O | LED D4 |
| **J1-20** | GPIO18 | - | I/O | Servo PWM |
| **J1-21** | GPIO19 | - | I/O | I2C SCL / SPI MISO |
| **J1-22** | GPIO21 | - | I/O | I2C SDA |
| **J1-23** | GPIO22 | - | I/O | Libre / SPI MISO alt |
| **J1-24** | GPIO23 | - | I/O | SPI MOSI |
| **J1-25** | GND | - | Power | Ground |
| **J1-26** | VIN | - | Power | 5V Input (from LM7805) |
| **J1-27** | 3V3 | - | Power | 3.3V Output (from AMS1117) |
| **J1-28** | EN | - | Input | Enable (reset) |
| **J1-29** | GPIO3 (RX0) | UART0_RX | I/O | Serial Debug |
| **J1-30** | GPIO1 (TX0) | UART0_TX | I/O | Serial Debug |
| **J1-31** | GPIO18 | - | I/O | (Duplicate J1-20) |
| **J1-32** | GPIO19 | - | I/O | (Duplicate J1-21) |
| **J1-33** | GPIO21 | - | I/O | (Duplicate J1-22) |
| **J1-34** | GPIO22 | - | I/O | (Duplicate J1-23) |
| **J1-35** | GPIO23 | - | I/O | (Duplicate J1-24) |
| **J1-36** | GND | - | Power | Ground |
| **J1-37** | VIN | - | Power | 5V Input |
| **J1-38** | 3V3 | - | Power | 3.3V Output |

## Strapping Pins (¡Cuidado al arranque!)

| GPIO | Estado por defecto | Función strap | Uso en Base | Precaución |
|------|-------------------|---------------|-------------|------------|
| **GPIO0** | Pull-up | Boot mode | LED D1 | **LOW = Download boot** - LED pull-up OK |
| **GPIO2** | Pull-down | Boot/SDIO | LED Built-in / RGB | **HIGH = SDIO boot** - Pull-down interno OK |
| **GPIO12** | Pull-down | Flash voltage | Motor IN1 | **HIGH = 1.8V flash** - Motor driver pull-down OK |
| **GPIO15** | Pull-up | Boot logging | Motor IN4 / Pot | **LOW = Silent boot** - Pot divider cuidado |

> ⚠️ **Regla de oro**: No conectar cargas que fuercen estado contrario al strap en estos pines durante reset.

## Pines solo entrada (Input Only)

| GPIO | ADC Canal | Uso en Base |
|------|-----------|-------------|
| **GPIO34** | ADC1_CH6 | Potenciómetro |
| **GPIO35** | ADC1_CH7 | SW_DIP-4 |
| **GPIO36** | ADC1_CH0 | LM35 |
| **GPIO39** | ADC1_CH3 | LDR |

*No tienen pull-up/pull-down interno. Usar resistencias externas si necesario.*

## Pines con funciones especiales

| GPIO | Función especial | Uso Base |
|------|------------------|----------|
| **GPIO25** | DAC1 | Buzzer (PWM/tone) |
| **GPIO26** | DAC2 | Libre / SW_SPDT |
| **GPIO17** | - | DS18B20 (1-Wire) |
| **GPIO18** | - | Servo (PWM 50Hz) |
| **GPIO19** | SPI0_MISO / I2C | I2C SCL |
| **GPIO21** | I2C_SDA | I2C SDA |
| **GPIO22** | SPI0_MISO | Libre |
| **GPIO23** | SPI0_MOSI | Libre |

## Periféricos disponibles (no usados en Base)

| Periférico | Pines disponibles | Notas |
|------------|-------------------|-------|
| **UART1** | GPIO9/10 (flash) / GPIO16/17 | GPIO16/17 libres |
| **UART2** | GPIO16/17 | GPIO16 libre, GPIO17 = DS18B20 |
| **I2C** | Cualquier GPIO | GPIO19/21 usados, otros libres |
| **SPI** | Cualquier GPIO (VSPI/HSPI) | GPIO18/19/21/22/23 parcialmente usados |
| **PWM** | 16 canales (cualquier GPIO) | 4 usados motor, 1 servo, 1 buzzer |
| **ADC2** | GPIO0,2,4,12-15,25-27 | **No usable con WiFi activo** |
| **Touch** | GPIO0,2,4,12-15,25-27,32-33 | 10 canales disponibles |

## Headers de expansión en Base

### J_I2C (4 pines)
| Pin | Señal |
|-----|-------|
| 1 | 3.3V |
| 2 | GND |
| 3 | SDA (GPIO21) |
| 4 | SCL (GPIO19) |

### J_SPI (6 pines)
| Pin | Señal |
|-----|-------|
| 1 | 3.3V |
| 2 | GND |
| 3 | MOSI (GPIO23) |
| 4 | MISO (GPIO19/22) |
| 5 | SCK (GPIO18) |
| 6 | CS (GPIO5/22) |

### J_GPIO (10 pines)
| Pin | GPIO | Nota |
|-----|------|------|
| 1 | 3.3V | |
| 2 | GND | |
| 3 | GPIO16 | Libre |
| 4 | GPIO17 | DS18B20 |
| 5 | GPIO22 | Libre |
| 6 | GPIO23 | Libre |
| 7 | GPIO25 | Buzzer |
| 8 | GPIO26 | SW_SPDT |
| 9 | GPIO27 | SW_DIP-1 |
| 10 | GPIO32 | SW_DIP-2 |

## Referencia rápida Programación

```cpp
// Definiciones pines Base ESP32
#define PIN_LED_D1       0   // Boot LED
#define PIN_LED_BUILTIN  2   // Built-in / RGB
#define PIN_LED_D3       4
#define PIN_LED_D4       5
#define PIN_MOTOR_IN1    12
#define PIN_MOTOR_IN2    13
#define PIN_MOTOR_IN3    14
#define PIN_MOTOR_IN4    15
#define PIN_BUZZER       25
#define PIN_DS18B20      17
#define PIN_SERVO        18
#define PIN_I2C_SDA      21
#define PIN_I2C_SCL      19
#define PIN_SW_DIP1      27
#define PIN_SW_DIP2      32
#define PIN_SW_DIP3      33
#define PIN_SW_DIP4      35
#define PIN_SW_PUSH      25  // Comparte con Buzzer!
#define PIN_SW_SPDT      26
#define PIN_POT          34
#define PIN_LM35         36
#define PIN_LDR          39
#define PIN_IMON         34  // Comparte con Pot!

// ADC2 pins (no usar con WiFi)
#define ADC2_PINS {0, 2, 4, 12, 13, 14, 15, 25, 26, 27}
```

## Recursos adicionales

- [Espressif ESP32 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
- [ESP32 Technical Reference Manual](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf)
- [Pinout interactivo](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wrover-32e_datasheet_en.pdf)