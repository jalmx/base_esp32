# Monitoreo de corriente - A9013

## Descripción

El sensor **A9013** es un sensor de efecto Hall lineal que mide corriente alterna o continua sin contacto galvánico. En la Base ESP32 se usa para monitorizar la corriente total consumida desde la rail **+5V** (antes de entrar al MX1508 y servo).

## Hardware

### Conexión
```
+5V Rail (desde LM7805) ──►│ A9013 (IP+) 
                            │
                         IP- (a MX1508 VM, Servo VCC, Terminal Block)
                            │
                        GND ──► Pin GND A9013
                            │
                        VCC (3.3V) ──► Pin VCC A9013
                            │
                        VOUT ──► GPIO34 (ADC1_CH6) ──► ESP32
```

### Pinout A9013 (SIP-3)
| Pin | Nombre | Función |
|-----|--------|---------|
| 1 | VCC | Alimentación 3.3V - 5V |
| 2 | GND | Tierra |
| 3 | VOUT | Salida analógica proporcional a corriente |

### Características eléctricas
| Parámetro | Valor |
|-----------|-------|
| **Sensibilidad** | 400 mV/A (típico) |
| **Offset (0A)** | VCC/2 = 1.65V @ 3.3V |
| **Rango lineal** | ±5A (hasta ±8A saturación) |
| **Voltaje suministro** | 3.3V - 5V |
| **Corriente quiescente** | ~10mA |
| **Ancho de banda** | 50 kHz |
| **Tiempo respuesta** | < 5 µs |
| **Aislamiento** | 2.1 kV RMS |

---

## Fórmula de conversión

### Teórica
```cpp
// VCC = 3.3V
const float VCC = 3.3;
const float SENSITIVITY = 0.400;  // 400 mV/A = 0.4 V/A
const float OFFSET = VCC / 2.0;   // 1.65V

float readCurrent() {
  int raw = analogRead(IMON_PIN);
  float voltage = raw * VCC / 4095.0;  // 12-bit ADC
  float current = (voltage - OFFSET) / SENSITIVITY;  // Amperios
  return current;  // Positivo = consumo, Negativo = inyección (raro)
}
```

### Calibrada (recomendada)
```cpp
// Calibrar con carga conocida (ej. resistor 10Ω en 5V = 500mA)
const float CAL_OFFSET = 1.642;   // Medido en 0A
const float CAL_SENS = 0.398;     // Medido con carga conocida

float readCurrentCalibrated() {
  int raw = analogRead(IMON_PIN);
  float voltage = raw * 3.3 / 4095.0;
  float current = (voltage - CAL_OFFSET) / CAL_SENS;
  return current;
}

// Promediado para reducir ruido
float readCurrentAvg(uint8_t samples = 32) {
  float sum = 0;
  for(int i=0; i<samples; i++) {
    sum += readCurrentCalibrated();
    delayMicroseconds(100);
  }
  return sum / samples;
}
```

---

## Procedimiento calibración

1. **Medir offset (0A)**:
   - Desconectar carga (MX1508 VM, Servo)
   - Medir VOUT con multímetro → `CAL_OFFSET`
   - Debe ser ~1.65V (VCC/2)

2. **Medir sensibilidad**:
   - Conectar carga conocida: Resistencia 10Ω 5W entre +5V y GND
   - Corriente teórica: I = 5V / 10Ω = 0.5A
   - Medir VOUT → `V_load`
   - `CAL_SENS = (V_load - CAL_OFFSET) / 0.5`

3. **Verificar linealidad**:
   - Probar con 250mA (20Ω), 1A (5Ω), 2A (2.5Ω)
   - Ajustar si no lineal

---

## Ejemplo completo

```cpp
#define IMON_PIN 34  // ADC1_CH6 (comparte con Potenciómetro!)

const float VCC = 3.3;
const float CAL_OFFSET = 1.642;  // Tu valor calibrado
const float CAL_SENS = 0.398;    // Tu valor calibrado

// Filtro pasa-bajos simple
float currentFiltered = 0;
const float ALPHA = 0.1;  // 0.01-0.2

float readCurrent() {
  int raw = analogRead(IMON_PIN);
  float voltage = raw * VCC / 4095.0;
  float current = (voltage - CAL_OFFSET) / CAL_SENS;
  
  // Low-pass filter
  currentFiltered = ALPHA * current + (1 - ALPHA) * currentFiltered;
  return currentFiltered;
}

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
  
  // Inicializar filtro
  for(int i=0; i<100; i++) readCurrent();
  delay(100);
}

void loop() {
  float current = readCurrent();
  float power = current * 5.0;  // Potencia aprox en 5V rail
  
  Serial.printf("Corriente: %.3f A  Potencia: %.2f W\n", current, power);
  
  // Alerta sobrecorriente
  if(current > 1.5) {
    Serial.println("⚠ SOBRECORRIENTE!");
    // Apagar motor, sonar buzzer, etc.
  }
  
  delay(500);
}
```

---

## Compartición GPIO34 (¡Importante!)

**GPIO34** se usa para **DOS señales analógicas**:
1. **IMON_OUT** (A9013) - Monitor corriente
2. **POTenciómetro** - Entrada usuario

### Multiplexación por hardware (no implementada en v0.4)
```
Opción A: MUX analógico (74HC4051) - 1 GPIO → 8 canales
Opción B: Switch analógico (DG408) - Control digital
Opción C: Leer secuencialmente (desconectar uno)
```

### Workaround v0.4 (Software)
```cpp
// NO se puede leer ambas simultáneamente
// Solución: Leer una, luego la otra (si no críticas tiempo real)

float readCurrentOrPot(bool readCurrent) {
  // En v0.4 están en paralelo - leer una afecta a la otra
  // SOLO FUNCIONA si una está desconectada (high-Z)
  // Potenciómetro SIEMPRE conectado → Corriente NO fiable
  
  // RECOMENDACIÓN: Usar solo UNA función a la vez
  // O modificar HW: cortar pista pot, añadir jumper
  return readCurrent ? readCurrent() : readPot();
}
```

> **Nota diseño v0.5**: Separar IMON a GPIO35 (ADC1_CH7) o usar MUX analógico.

---

## Detección de estados

```cpp
enum PowerState {
  SLEEP,      // < 20mA
  IDLE,       // 20-100mA (ESP32 solo)
  ACTIVE,     // 100-500mA (WiFi, sensores)
  MOTOR_LOW,  // 500mA-1A (Motor PWM bajo)
  MOTOR_HIGH, // 1-2A (Motor carga)
  OVERLOAD    // > 2A
};

PowerState detectState(float current) {
  if(current < 0.02) return SLEEP;
  if(current < 0.1) return IDLE;
  if(current < 0.5) return ACTIVE;
  if(current < 1.0) return MOTOR_LOW;
  if(current < 2.0) return MOTOR_HIGH;
  return OVERLOAD;
}
```

---

## Prácticas sugeridas

| Práctica | Descripción | Nivel |
|----------|-------------|-------|
| **09-Current Monitor** | Leer corriente, mostrar OLED/Serial, alerta | Advanced |
| **11-Sensor Fusion** | Corriente + Temp motor + Velocidad | Advanced |
| **Energy Meter** | Integrar corriente → mWh consumidos | Advanced |

---

## Datasheet

- [A9013 Datasheet](https://www.allegromicro.com/en/products/sense/current-sensor/integrated/a9013)
- Alternativas: ACS712 (más común, 5V only), INA219 (I2C, alta precisión, bajo voltaje)