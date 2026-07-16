# Práctica 05 - Sensores de Temperatura

## Objetivo
Leer DS18B20 (1-Wire) y LM35 (Analógico), comparar precisión, mostrar por Serial/OLED.

## Nivel
**Intermediate** | Duración: ~3 horas

## Hardware
| Sensor | Tipo | GPIO | Librería |
|--------|------|------|----------|
| **DS18B20** | Digital 1-Wire | GPIO17 | OneWire + DallasTemperature |
| **LM35DZ** | Analógico | GPIO36 (ADC1_CH0) | Solo ADC |

## Preparación librerías
```bash
# Arduino IDE → Library Manager → Buscar e instalar:
"OneWire" by Paul Stoffregen
"DallasTemperature" by Miles Burton
```

## Código - DS18B20
```cpp
#include <OneWire.h>
#include <DallasTemperature.h>

#define ONE_WIRE_BUS 17

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

DeviceAddress sensorAddr;

void setup() {
  Serial.begin(115200);
  sensors.begin();
  
  if(!sensors.getAddress(sensorAddr, 0)) {
    Serial.println("¡No se encontró DS18B20!");
    while(1);
  }
  
  Serial.print("Dirección: ");
  for(uint8_t i=0; i<8; i++) {
    if(sensorAddr[i] < 16) Serial.print("0");
    Serial.print(sensorAddr[i], HEX);
  }
  Serial.println();
  
  sensors.setResolution(sensorAddr, 12);  // 9-12 bits
}

float readDS18B20() {
  sensors.requestTemperatures();
  return sensors.getTempC(sensorAddr);
}
```

## Código - LM35
```cpp
#define LM35_PIN 36  // ADC1_CH0 (INPUT ONLY)

float readLM35() {
  int raw = analogRead(LM35_PIN);
  float voltage = raw * 3.3 / 4095.0;  // mV
  return voltage * 100.0;  // 10mV/°C
}

// Con promedio
float readLM35Avg(uint8_t samples = 32) {
  long sum = 0;
  for(int i=0; i<samples; i++) {
    sum += analogRead(LM35_PIN);
    delayMicroseconds(100);
  }
  return (sum / (float)samples) * 3300.0 / 4095.0 / 10.0;
}
```

## Código completo - Comparativo
```cpp
/*
  Temperatura Dual - DS18B20 vs LM35
  Base ESP32 DevKit V4
*/

#include <OneWire.h>
#include <DallasTemperature.h>

#define DS18B20_PIN 17
#define LM35_PIN    36

OneWire oneWire(DS18B20_PIN);
DallasTemperature ds18b20(&oneWire);
DeviceAddress dsAddr;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
  
  ds18b20.begin();
  if(!ds18b20.getAddress(dsAddr, 0)) {
    Serial.println("ERROR: DS18B20 no detectado");
  } else {
    ds18b20.setResolution(dsAddr, 12);
    Serial.println("DS18B20 OK");
  }
  Serial.println("Tiempo\tDS18B20\tLM35\tDiff");
}

float readDS() {
  ds18b20.requestTemperatures();
  return ds18b20.getTempC(dsAddr);
}

float readLM35() {
  long sum = 0;
  for(int i=0; i<32; i++) sum += analogRead(LM35_PIN);
  return (sum / 32.0) * 3300.0 / 4095.0 / 10.0;
}

void loop() {
  float t1 = readDS();
  float t2 = readLM35();
  float diff = t1 - t2;
  
  Serial.printf("%lu\t%.2f\t%.2f\t%.2f\n", 
                millis()/1000, t1, t2, diff);
  delay(2000);  // DS18B20 12-bit = 750ms máx
}
```

## Verificación
1. Subir código
2. Serial Monitor → Ver dos temperaturas
3. **Calentar con dedos** el DS18B20 y LM35 → Ambos suben ✅
4. Diferencia típica: ±0.5°C (DS18B20 más preciso) ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Mostrar en °F también: `F = C * 9/5 + 32` |
| 🟢 Fácil | Alarma si temp > 30°C → Buzzer + LED |
| 🟡 Medio | Múltiples DS18B20 en bus 1-Wire (buscar direcciones) |
| 🟡 Medio | Data Logger: Guardar en SD/CSV cada minuto |
| 🔴 Difícil | Control PID: Mantener temperatura con resistor calefactor (externo) |
| 🔴 Difícil | Compensar LM35 con calibración 2-puntos vs DS18B20 referencia |

### Reto: Múltiples DS18B20
```cpp
void discoverSensors() {
  DeviceAddress addr;
  int count = 0;
  while(ds18b20.getAddress(addr, count)) {
    Serial.print("Sensor "); Serial.print(count);
    Serial.print(": ");
    printAddress(addr);
    count++;
  }
  Serial.printf("Total: %d sensores\n", count);
}
```

### Reto: Termostato simple
```cpp
#define SETPOINT 25.0
#define HYSTERESIS 0.5

void thermostat() {
  float temp = readDS();
  static bool heating = false;
  
  if(temp < SETPOINT - HYSTERESIS) heating = true;
  if(temp > SETPOINT + HYSTERESIS) heating = false;
  
  digitalWrite(RELAY_PIN, heating);  // Relevador externo
}
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| `DEVICE_DISCONNECTED` | Cableado / Pull-up | Verificar 4.7kΩ Data-3.3V |
| `-127°C` | Resolución no completada | `delay(750)` tras request |
| LM35 estable pero erróneo | No linealidad ADC | Calibrar 2-puntos, promediar |
| DS18B20 no responde | Parasite power mal | Conectar VDD a 3.3V (no GND) |
| Lecturas saltan | Ruido alimentación | Cap 100nF cerca sensor, promedio |

## Referencias
- [DS18B20 Datasheet](https://datasheets.maximintegrated.com/en/ds/DS18B20.pdf)
- [LM35 Datasheet](https://www.ti.com/lit/ds/symlink/lm35.pdf)
- [OneWire Library](https://www.pjrc.com/teensy/td_libs_OneWire.html)
- [DallasTemperature](https://github.com/milesburton/Arduino-Temperature-Control-Library)