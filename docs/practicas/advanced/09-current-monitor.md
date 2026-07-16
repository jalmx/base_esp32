# Práctica 09 - Monitoreo de Corriente

## Objetivo
Medir consumo total con sensor Hall A9013, calibrar, filtrar, detectar estados y alertar sobrecorriente.

## Nivel
**Advanced** | Duración: ~4 horas

## Hardware
- A9013 (U_IMON) - Sensor Hall 400mV/A
- Salida → GPIO34 (ADC1_CH6) **¡Compartido con Potenciómetro!**
- Alimentación: 3.3V Rail
- Mide corriente en rail +5V (antes de MX1508/Servo)

## Teoría A9013
| Parámetro | Valor |
|-----------|-------|
| Sensibilidad | 400 mV/A |
| Offset (0A) | VCC/2 = 1.65V @ 3.3V |
| Rango lineal | ±5A |
| Ancho banda | 50 kHz |

**Fórmula**: `I = (Vout - Voffset) / Sensitivity`

## Calibración (¡OBLIGATORIA!)
```cpp
// 1. Medir offset sin carga (0A)
float calibrateOffset() {
  long sum = 0;
  for(int i=0; i<1000; i++) {
    sum += analogRead(34);
    delayMicroseconds(100);
  }
  return (sum / 1000.0) * 3.3 / 4095.0;  // ~1.65V esperado
}

// 2. Medir sensibilidad con carga conocida
// Conectar R=10Ω 5W entre +5V y GND → I = 0.5A
float calibrateSens(float offset) {
  float v_load = readAvgVoltage();  // Con carga 0.5A
  return (v_load - offset) / 0.5;   // V/A
}
```

## Código completo
```cpp
/*
  Monitor Corriente - A9013 Hall Effect
  Base ESP32 DevKit V4
*/

#define IMON_PIN 34  // ADC1_CH6 (¡Compartido con Pot!)

struct CurrentCal {
  float offset = 1.642;  // Tu valor calibrado
  float sens = 0.398;    // Tu valor calibrado (V/A)
} cal;

// Filtro pasa-bajos
float iFiltered = 0;
const float ALPHA = 0.1;

float readCurrent() {
  int raw = analogRead(IMON_PIN);
  float voltage = raw * 3.3 / 4095.0;
  float current = (voltage - cal.offset) / cal.sens;
  
  // Low-pass filter
  iFiltered = ALPHA * current + (1 - ALPHA) * iFiltered;
  return iFiltered;
}

// Promediado para estabilidad
float readCurrentAvg(uint8_t n = 32) {
  float sum = 0;
  for(int i=0; i<n; i++) {
    sum += readCurrent();
    delayMicroseconds(100);
  }
  return sum / n;
}

enum PowerState { SLEEP, IDLE, ACTIVE, MOTOR_LOW, MOTOR_HIGH, OVERLOAD };

PowerState detectState(float current) {
  if(current < 0.02) return SLEEP;
  if(current < 0.10) return IDLE;
  if(current < 0.50) return ACTIVE;
  if(current < 1.00) return MOTOR_LOW;
  if(current < 2.00) return MOTOR_HIGH;
  return OVERLOAD;
}

const char* stateName(PowerState s) {
  switch(s) {
    case SLEEP: return "SLEEP";
    case IDLE: return "IDLE";
    case ACTIVE: return "ACTIVE";
    case MOTOR_LOW: return "MOTOR_LOW";
    case MOTOR_HIGH: return "MOTOR_HIGH";
    case OVERLOAD: return "OVERLOAD";
  }
  return "UNKNOWN";
}

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
  
  // Inicializar filtro
  for(int i=0; i<100; i++) readCurrent();
  delay(100);
  
  Serial.println("Monitor Corriente A9013");
  Serial.println("Offset: " + String(cal.offset, 3) + "V");
  Serial.println("Sens: " + String(cal.sens, 3) + " V/A");
  Serial.println("Tiempo(s)\tCorriente(A)\tPotencia(W)\tEstado");
}

void loop() {
  float current = readCurrentAvg(16);
  float power = current * 5.0;  // Aprox @ 5V rail
  PowerState state = detectState(current);
  
  Serial.printf("%lu\t\t%.3f\t\t%.2f\t\t%s\n", 
                millis()/1000, current, power, stateName(state));
  
  // Alerta sobrecorriente
  if(state == OVERLOAD) {
    Serial.println("⚠⚠⚠ SOBRECORRIENTE > 2A! ⚠⚠⚠");
    // tone(BUZZER, 1000, 100);  // Si buzzer libre
  }
  
  delay(500);
}
```

## ⚠️ Compartición GPIO34 (Potenciómetro)
En v0.4, GPIO34 recibe **AMBAS** señales:
- A9013 VOUT (corriente)
- Potenciómetro wiper (usuario)

**No se pueden leer simultáneamente correctamente.**

### Workaround v0.4
```cpp
// Solo funciona si UNA está desconectada (high-Z)
// Potenciómetro SIEMPRE conectado → Corriente NO fiable

// Opción A: Usar solo potenciómetro (corriente no disponible)
// Opción B: Cortar pista pot, añadir jumper para seleccionar
// Opción B: v0.5 → Mover IMON a GPIO35 o usar MUX analógico
```

## Verificación
1. Calibrar offset (sin carga) → ~1.65V
2. Calibrar sensibilidad (carga 0.5A conocida)
3. Subir código con valores calibrados
4. **Serial muestra corriente, potencia, estado** ✅
5. Conectar motor → Ver MOTOR_LOW/HIGH ✅
6. Sobrecorriente → Alerta ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Mostrar en OLED/I2C (externa) |
| 🟡 Medio | Energy meter: Integrar `mWh = Σ(I×V×dt)` |
| 🟡 Medio | Log circular en RAM (últimos 1000 samples) |
| 🟡 Medio | Perfil motor: Corriente vs PWM duty → curva |
| 🔴 Difícil | Protección: Apagar motor si I > 1.5A por >1s |
| 🔴 Difícil | Detección stall: dI/dt brusco → motor atascado |

### Reto: Energy Meter
```cpp
float energy_mWh = 0;
uint32_t lastTime = 0;

void loop() {
  float current = readCurrentAvg();
  float voltage = 5.0;  // Asumido
  uint32_t now = millis();
  float dt_h = (now - lastTime) / 3600000.0;  // horas
  energy_mWh += current * voltage * dt_h * 1000;  // mWh
  lastTime = now;
  
  Serial.printf("Energía: %.2f mWh\n", energy_mWh);
}
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| Offset no 1.65V | VCC no 3.3V / Ruido | Verificar 3.3V rail, promediar más |
| Sensibilidad errónea | Carga calibración inexacta | Usar multímetro en serie para verificar |
| Lecturas saltan | Ruido ADC / Compartición GPIO34 | Promediar 32+, filtro pasa-bajos |
| Corriente negativa | Offset mal calibrado | Recalibrar offset en 0A real |
| No detecta motor | Ruido > señal | Aumentar ALPHA, más muestras |

## Referencias
- [A9013 Datasheet](https://www.allegromicro.com/en/products/sense/current-sensor/integrated/a9013)
- [ACS712 Alternative](https://www.allegromicro.com/en/products/sense/current-sensor/acs712)
- [INA219 I2C Alternative](https://www.ti.com/product/INA219)