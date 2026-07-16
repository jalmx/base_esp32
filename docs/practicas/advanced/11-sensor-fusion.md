# Práctica 11 - Fusión de Sensores

## Objetivo
Combinar múltiples sensores (temperatura, luz, corriente) para detectar estados del sistema y tomar decisiones inteligentes.

## Nivel
**Advanced** | Duración: ~4 horas

## Hardware
| Sensor | GPIO | Práctica previa |
|--------|------|-----------------|
| DS18B20 | 17 | 05-Temperature |
| LM35 | 36 | 05-Temperature |
| LDR | 39 | 03-Analog |
| A9013 | 34* | 09-Current |
| Potenciómetro | 34* | 03-Analog |
| DIP Switch | 27,32,33,35 | 08-Switches |

*GPIO34 compartido A9013 + Pot

## Concepto: State Machine
```cpp
enum SystemState {
  STATE_SLEEP,      // Corriente < 20mA, sin actividad
  STATE_IDLE,       // ESP32 activo, sin carga
  STATE_ACTIVE,     // Sensores leyendo, WiFi
  STATE_MOTOR_LOW,  // Motor PWM < 50%
  STATE_MOTOR_HIGH, // Motor PWM > 50%
  STATE_OVERLOAD,   // Corriente > 2A
  STATE_OVERTEMP,   // Temp > 60°C
  STATE_LOW_LIGHT   // LDR < umbral
};
```

## Código - Fusionador
```cpp
/*
  Sensor Fusion - Detección estados sistema
  Base ESP32 DevKit V4
*/

#include <OneWire.h>
#include <DallasTemperature.h>

// Pines
#define DS18B20_PIN 17
#define LM35_PIN 36
#define LDR_PIN 39
#define IMON_PIN 34
#define POT_PIN 34
#define DIP1 27
#define DIP2 32
#define DIP3 33
#define DIP4 35

// Sensores
OneWire oneWire(DS18B20_PIN);
DallasTemperature ds18b20(&oneWire);
DeviceAddress dsAddr;

// Filtros
float tempDS = 0, tempLM = 0, light = 0, current = 0;
const float ALPHA = 0.1;  // Filtro pasa-bajos

// Umbrales
const float TEMP_HIGH = 50.0;
const float LIGHT_LOW = 0.5;  // V
const float CURR_OVERLOAD = 2.0;
const float CURR_MOTOR_LOW = 0.5;
const float CURR_MOTOR_HIGH = 1.0;

enum SystemState {
  SLEEP, IDLE, ACTIVE, MOTOR_LOW, MOTOR_HIGH, OVERLOAD, OVERTEMP, LOW_LIGHT
};

SystemState currentState = IDLE;
SystemState prevState = IDLE;

void setup() {
  Serial.begin(115200);
  
  // ADC
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
  
  // DS18B20
  ds18b20.begin();
  ds18b20.getAddress(dsAddr, 0);
  ds18b20.setResolution(dsAddr, 12);
  
  // DIP switches
  pinMode(DIP1, INPUT_PULLUP);
  pinMode(DIP2, INPUT_PULLUP);
  pinMode(DIP3, INPUT_PULLUP);
  pinMode(DIP4, INPUT_PULLUP);
  
  Serial.println("Sensor Fusion iniciado");
  Serial.println("Estado\tTempDS\tTempLM\tLight\tCurr\tDIP");
}

void loop() {
  // 1. Leer sensores (con filtro)
  readSensors();
  
  // 2. Detectar estado
  SystemState newState = detectState();
  
  // 3. Transición de estado
  if(newState != currentState) {
    onStateChange(currentState, newState);
    prevState = currentState;
    currentState = newState;
  }
  
  // 4. Acciones por estado
  executeStateActions(currentState);
  
  // 5. Log
  logData();
  
  delay(200);
}

void readSensors() {
  // DS18B20
  ds18b20.requestTemperatures();
  float t = ds18b20.getTempC(dsAddr);
  if(t != DEVICE_DISCONNECTED_C) tempDS = ALPHA * t + (1-ALPHA) * tempDS;
  
  // LM35 (promedio 16 muestras)
  long sum = 0;
  for(int i=0; i<16; i++) sum += analogRead(LM35_PIN);
  float v = (sum/16.0) * 3.3 / 4095.0;
  float t2 = v * 100.0;
  tempLM = ALPHA * t2 + (1-ALPHA) * tempLM;
  
  // LDR
  int ldrRaw = analogRead(LDR_PIN);
  float ldrV = ldrRaw * 3.3 / 4095.0;
  light = ALPHA * ldrV + (1-ALPHA) * light;
  
  // Corriente (solo si pot no usado - ver nota)
  int imonRaw = analogRead(IMON_PIN);
  float imonV = imonRaw * 3.3 / 4095.0;
  float i = (imonV - 1.65) / 0.4;  // Calibrar offset/sens
  current = ALPHA * i + (1-ALPHA) * current;
}

SystemState detectState() {
  // Prioridad: seguridad > operación > reposo
  
  if(current > CURR_OVERLOAD) return OVERLOAD;
  if(tempDS > TEMP_HIGH || tempLM > TEMP_HIGH) return OVERTEMP;
  if(light < LIGHT_LOW) return LOW_LIGHT;
  
  if(current > CURR_MOTOR_HIGH) return MOTOR_HIGH;
  if(current > CURR_MOTOR_LOW) return MOTOR_LOW;
  
  // Leer DIP para modo
  uint8_t dip = (!digitalRead(DIP1)<<0) | (!digitalRead(DIP2)<<1) | 
                (!digitalRead(DIP3)<<2) | (!digitalRead(DIP4)<<3);
  
  if(dip == 0) return SLEEP;
  if(dip & 0x01) return ACTIVE;  // Modo activo
  
  return IDLE;
}

void onStateChange(SystemState from, SystemState to) {
  Serial.printf(">>> TRANSICIÓN: %s → %s\n", stateName(from), stateName(to));
  
  // Acciones de transición
  switch(to) {
    case OVERLOAD:
      // Apagar motor, sonar buzzer
      break;
    case OVERTEMP:
      // Alerta térmica
      break;
    case SLEEP:
      // Preparar deep sleep
      break;
  }
}

void executeStateActions(SystemState s) {
  switch(s) {
    case ACTIVE:
      // Leer sensores rápido, enviar datos
      break;
    case MOTOR_LOW:
    case MOTOR_HIGH:
      // Monitoreo motor activo
      break;
    case SLEEP:
      // Deep sleep si DIP=0 sostenido
      break;
  }
}

void logData() {
  static uint32_t lastLog = 0;
  if(millis() - lastLog > 1000) {
    lastLog = millis();
    uint8_t dip = readDIP();
    Serial.printf("%s\t%.1f\t%.1f\t%.2f\t%.2f\t0x%X\n",
                  stateName(currentState), tempDS, tempLM, light, current, dip);
  }
}

uint8_t readDIP() {
  return (!digitalRead(DIP1)<<0) | (!digitalRead(DIP2)<<1) |
         (!digitalRead(DIP3)<<2) | (!digitalRead(DIP4)<<3);
}

const char* stateName(SystemState s) {
  static const char* names[] = {
    "SLEEP", "IDLE", "ACTIVE", "MOTOR_LOW", 
    "MOTOR_HIGH", "OVERLOAD", "OVERTEMP", "LOW_LIGHT"
  };
  return names[s];
}
```

## Fusión de temperaturas (DS18B20 + LM35)
```cpp
// Promedio ponderado por confianza
float fusedTemp() {
  float w1 = 0.7;  // DS18B20 más preciso
  float w2 = 0.3;  // LM35 más rápido
  
  // Si uno falla, usar el otro
  bool dsOK = (tempDS > -50 && tempDS < 125);
  bool lmOK = (tempLM > 0 && tempLM < 100);
  
  if(dsOK && lmOK) return w1*tempDS + w2*tempLM;
  if(dsOK) return tempDS;
  if(lmOK) return tempLM;
  return NAN;
}
```

## Detección anomalías (Simple)
```cpp
// Detectar cambios bruscos (derivada)
float prevTemp = 0, prevCurr = 0;
bool anomalyDetected() {
  float dTemp = abs(tempDS - prevTemp);
  float dCurr = abs(current - prevCurr);
  prevTemp = tempDS;
  prevCurr = current;
  
  return (dTemp > 5.0) || (dCurr > 1.0);  // Cambio >5°C o >1A en 200ms
}
```

## Verificación
1. Subir código
2. Serial Monitor → Ver tabla estados
3. **Calentar DS18B20** → OVERTEMP ✅
4. **Conectar motor** → MOTOR_LOW/HIGH ✅
5. **Tapar LDR** → LOW_LIGHT ✅
6. **DIP switches** → Cambian modo ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟡 Medio | Filtro Kalman simple para temperatura |
| 🟡 Medio | Machine Learning: Clasificar estados con Edge Impulse |
| 🔴 Difícil | Predictivo: "Motor se atascó" por dI/dt |
| 🔴 Difícil | Auto-calibración umbrales al inicio |
| 🔴 Difícil | Data Logger circular + timestamp + export CSV |

## Referencias
- [Sensor Fusion Tutorial](https://www.analog.com/en/analog-dialogue/articles/sensor-fusion.html)
- [Kalman Filter Arduino](https://github.com/denyssene/SimpleKalmanFilter)
- [Edge Impulse for ESP32](https://docs.edgeimpulse.com/docs/deployment/arduino-library)