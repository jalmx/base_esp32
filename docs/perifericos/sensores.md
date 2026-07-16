# Sensores

## Resumen

| Sensor | Tipo | GPIO | Interfaz | Rango | Precisión |
|--------|------|------|----------|-------|-----------|
| **DS18B20** | Temperatura | GPIO17 | 1-Wire | -55 a +125°C | ±0.5°C |
| **LM35DZ** | Temperatura | GPIO36 (ADC1_CH0) | Analógico | 0 a +100°C | ±0.5°C |
| **LDR (GL5528)** | Luz | GPIO39 (ADC1_CH3) | Analógico | 1-100k lux (relativo) | - |

---

## DS18B20 - Sensor Temperatura Digital 1-Wire

### Hardware
```
GPIO17 ──► Data (Pin 2 DS18B20)
         │
         ├──► 4.7kΩ Pull-up ──► 3.3V
         │
3.3V ────┼──► VDD (Pin 3)  [Modo normal]
         │
GND ─────┼──► GND  (Pin 1)
         │
         └──► [Opcional] Parasite mode: VDD a GND
```

**Ubicación en PCB**: Header 3 pines cerca GPIO17 (J_DS18B20)
- Permite sonda waterproof por cables
- O montaje directo TO-92 en PCB

### Librerías
```cpp
// Arduino Library Manager: "OneWire" + "DallasTemperature"
#include <OneWire.h>
#include <DallasTemperature.h>

#define ONE_WIRE_BUS 17

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

void setup() {
  Serial.begin(115200);
  sensors.begin();
  Serial.print("Dispositivos encontrados: ");
  Serial.println(sensors.getDeviceCount());
}

void loop() {
  sensors.requestTemperatures();
  float tempC = sensors.getTempCByIndex(0);
  
  if(tempC != DEVICE_DISCONNECTED_C) {
    Serial.print("Temp: ");
    Serial.print(tempC);
    Serial.println(" °C");
  } else {
    Serial.println("Error: Sensor no conectado");
  }
  delay(1000);
}
```

### Múltiples sensores (1-Wire bus)
```cpp
// Cada DS18B20 tiene dirección única de 64 bits
DeviceAddress sensor1, sensor2;

void setup() {
  sensors.begin();
  if(sensors.getAddress(sensor1, 0)) {
    Serial.print("Sensor 1: ");
    printAddress(sensor1);
  }
  if(sensors.getAddress(sensor2, 1)) {
    Serial.print("Sensor 2: ");
    printAddress(sensor2);
  }
  // Configurar resolución (9-12 bits)
  sensors.setResolution(sensor1, 12); // 0.0625°C, 750ms
  sensors.setResolution(sensor2, 12);
}

void printAddress(DeviceAddress addr) {
  for(uint8_t i=0; i<8; i++) {
    if(addr[i] < 16) Serial.print("0");
    Serial.print(addr[i], HEX);
  }
  Serial.println();
}
```

### Modo Parasite Power (2 hilos)
```cpp
// Conectar VDD del sensor a GND (pin 3 a pin 1)
// Requiere pull-up fuerte (2.2k-4.7k) y tiempo de conversión mayor
sensors.setWaitForConversion(true); // Bloqueante
// O no bloqueante:
sensors.requestTemperatures();
delay(750); // 12-bit = 750ms max
```

---

## LM35DZ - Sensor Temperatura Analógico

### Hardware
```
GPIO36 (ADC1_CH0) ──► Vout (Pin 2 LM35)
3.3V ───────────────► Vcc (Pin 1)
GND ────────────────► GND (Pin 3)

Opcional: C 100nF Vout-GND para estabilidad
```

**Características:**
- **Salida**: 10.0 mV/°C (lineal)
- **Rango**: 0°C a +100°C (LM35DZ)
- **Offset**: 0V @ 0°C (sin offset negativo)
- **Corriente**: 60µA típico

### Conversión
```cpp
#define LM35_PIN 36  // ADC1_CH0

void setup() {
  Serial.begin(115200);
  analogReadResolution(12); // 0-4095
  // ADC attenuation para 0-3.3V rango completo
  analogSetAttenuation(ADC_11db); 
}

float readLM35() {
  int raw = analogRead(LM35_PIN);
  // 3.3V / 4095 = 0.805 mV/LSB
  float voltage_mV = raw * 3300.0 / 4095.0;
  // 10 mV/°C
  float tempC = voltage_mV / 10.0;
  return tempC;
}

void loop() {
  float temp = readLM35();
  Serial.printf("LM35: %.1f°C (raw=%d)\n", temp, analogRead(LM35_PIN));
  delay(1000);
}
```

### Calibración
```cpp
// Medir con termómetro de referencia a 25°C
// Ajustar factor si hay desviación
float tempC = (voltage_mV * CAL_FACTOR) / 10.0;
// CAL_FACTOR = 25.0 / temp_measured_at_25C
```

### Limitaciones ESP32 ADC
- **No linealidad** en extremos (0-100mV, 3.0-3.3V)
- **Ruido** ~10-20 LSB típico
- **Solución**: Promediar 16-64 muestras
```cpp
float readLM35Avg(uint8_t samples = 32) {
  long sum = 0;
  for(int i=0; i<samples; i++) {
    sum += analogRead(LM35_PIN);
    delayMicroseconds(100);
  }
  float avg = sum / (float)samples;
  return (avg * 3300.0 / 4095.0) / 10.0;
}
```

---

## LDR (GL5528) - Sensor Luz

### Hardware
```
GPIO39 (ADC1_CH3) ──►───┬───► LDR
                        │
                        ▼
                     10kΩ
                        │
                       GND

3.3V ──────────────────► LDR (otro pin)
```

**Divisor de tensión:**
- LDR resistencia: ~10kΩ (luz) a >1MΩ (oscuridad)
- R_fija: 10kΩ → Rango ~0.15V (luz) a ~3.3V (oscuro)
- **Invertido**: Más luz = menos voltaje

### Programación
```cpp
#define LDR_PIN 39  // ADC1_CH3 (Input only)

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
}

void loop() {
  int raw = analogRead(LDR_PIN);
  float voltage = raw * 3300.0 / 4095.0;
  
  // Estimación lux (aproximada, calibrar por sensor)
  // R_ldr = 10k * (3.3/V - 1)
  float r_ldr = 10000.0 * (3.3 / voltage - 1.0);
  // Lux ≈ 500 / R_ldr(kΩ)  (muy aproximado)
  float lux = 500000.0 / r_ldr;
  
  Serial.printf("LDR: raw=%d V=%.2f R=%.0fΩ lux≈%.0f\n", 
                raw, voltage/1000.0, r_ldr, lux);
  delay(500);
}
```

### Umbral día/noche
```cpp
#define LDR_THRESHOLD 2000  // Ajustar experimentalmente

bool isDark() {
  return analogRead(LDR_PIN) > LDR_THRESHOLD;
}

void loop() {
  if(isDark()) {
    digitalWrite(LED_D3, HIGH); // Encender luz
  } else {
    digitalWrite(LED_D3, LOW);
  }
}
```

---

## Comparativa sensores temperatura

| Característica | DS18B20 | LM35DZ |
|----------------|---------|--------|
| **Interfaz** | Digital (1-Wire) | Analógico |
| **Pines** | 3 (o 2 parasite) | 3 |
| **Precisión** | ±0.5°C | ±0.5°C (calibrado) |
| **Resolución** | 9-12 bits (0.0625°C) | Limitada por ADC (12-bit = 0.8mV = 0.08°C) |
| **Velocidad** | 94-750ms | <1ms (muestreo ADC) |
| **Cableado** | Bus compartido (múltiples) | 1 pin ADC por sensor |
| **Costo** | ~$1.50 | ~$0.50 |
| **Ruido** | Inmune (digital) | Sensible a ruido ADC |
| **Alimentación** | 3.3V/5V | 3.3V/5V |

---

## Prácticas sugeridas

| Práctica | Sensores | Nivel |
|----------|----------|-------|
| **05-Temperatura** | DS18B20 + LM35 comparativo | Intermediate |
| **06-Termostato** | DS18B20 + Relay/Buzzer | Intermediate |
| **07-Luz Auto** | LDR + LED | Beginner |
| **08-Data Logger** | DS18B20 + SD/Serial | Intermediate |
| **11-Sensor Fusion** | DS18B20 + LM35 + LDR | Advanced |

---

## Datasheets

- [DS18B20](https://datasheets.maximintegrated.com/en/ds/DS18B20.pdf)
- [LM35](https://www.ti.com/lit/ds/symlink/lm35.pdf)
- [GL5528 LDR](https://www.gl5528.com/datasheet.pdf)