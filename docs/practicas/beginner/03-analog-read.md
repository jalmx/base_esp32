# Práctica 03 - Lectura Analógica

## Objetivo
Leer el potenciómetro (GPIO34) y mostrar valor/voltaje por Serial.

## Nivel
**Beginner** | Duración: ~1.5 horas

## Hardware
- Potenciómetro 10kΩ → GPIO34 (ADC1_CH6)
- **GPIO34 = INPUT ONLY** (no output, no pull-up/down interno)

## Teoría
- **ADC**: Conversor Analógico-Digital - Convierte voltaje a número
- **ESP32 ADC**: 2 ADCs (ADC1: 8 canales, ADC2: 10 canales) - 12-bit (0-4095)
- **Atenuación**: Rango de voltaje medible
  - `ADC_0db`: 0-1.1V
  - `ADC_2_5db`: 0-1.5V
  - `ADC_6db`: 0-2.2V
  - `ADC_11db`: 0-3.3V (recomendado para 3.3V)
- **No linealidad**: Precisión peor en extremos (0-100mV, 3.0-3.3V)
- **Ruido**: ~10-20 LSB típico → Promediar lecturas

## Código paso a paso

### 1. Configurar ADC
```cpp
#define POT_PIN 34  // ADC1_CH6

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);        // 12-bit = 0-4095
  analogSetAttenuation(ADC_11db);  // Rango 0-3.3V
}
```

### 2. Lectura básica
```cpp
void loop() {
  int raw = analogRead(POT_PIN);
  float voltage = raw * 3.3 / 4095.0;
  
  Serial.printf("Raw: %4d  Voltage: %.3f V\n", raw, voltage);
  delay(100);
}
```

### 3. Filtrado (promedio móvil)
```cpp
#define SAMPLES 16

int readFiltered() {
  long sum = 0;
  for(int i = 0; i < SAMPLES; i++) {
    sum += analogRead(POT_PIN);
    delayMicroseconds(100);
  }
  return sum / SAMPLES;
}
```

### 4. Código completo
```cpp
/*
  Lectura Analógica - Potenciómetro
  Base ESP32 DevKit V4
*/

#define POT_PIN 34
#define SAMPLES 32

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
  Serial.println("ADC Potenciómetro iniciado");
  Serial.println("Raw\tVolts\t%");
}

int readFiltered() {
  long sum = 0;
  for(int i = 0; i < SAMPLES; i++) {
    sum += analogRead(POT_PIN);
    delayMicroseconds(50);
  }
  return sum / SAMPLES;
}

void loop() {
  int raw = readFiltered();
  float voltage = raw * 3.3 / 4095.0;
  float percent = (raw / 4095.0) * 100.0;
  
  Serial.printf("%d\t%.3f\t%.1f%%\n", raw, voltage, percent);
  delay(200);
}
```

## Verificación
1. Subir código
2. Abrir **Serial Monitor** (115200 baud)
3. **Girar potenciómetro** → Valores cambian 0-4095 ✅
4. Abrir **Serial Plotter** → Gráfica en tiempo real ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Mostrar barra ASCII: `[=====-----]` proporcional |
| 🟢 Fácil | Convertir a grados servo (0-180°) con `map()` |
| 🟡 Medio | Detectar cambio significativo (umbral) y solo imprimir entonces |
| 🟡 Medio | Calcular RPM si potenciómetro en eje motor (contar vueltas) |
| 🔴 Difícil | Calibración 2-puntos: medir 0V y 3.3V reales, corregir ganancia/offset |

### Reto: Barra ASCII
```cpp
void printBar(int value, int maxVal = 4095, int width = 40) {
  int filled = map(value, 0, maxVal, 0, width);
  Serial.print("[");
  for(int i=0; i<width; i++) {
    Serial.print(i < filled ? "=" : "-");
  }
  Serial.print("] ");
}
```

### Reto: Calibración 2-puntos
```cpp
// Medir con multímetro:
// V_min_real = 0.02V (raw_min = 25)
// V_max_real = 3.28V (raw_max = 4070)

float readCalibrated() {
  int raw = readFiltered();
  // y = m*x + b
  float m = (3.28 - 0.02) / (4070 - 25);
  float b = 0.02 - m * 25;
  return m * raw + b;
}
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| Valor fijo 0 o 4095 | Pin mal conectado | Verificar potenciómetro, continuidad |
| Lecturas ruidosas | Sin filtro | Promediar 16-64 muestras |
| No llega a 0 o 4095 | No linealidad ADC | Normal en extremos, calibrar |
| `analogSetAttenuation` error | Core antiguo | Actualizar ESP32 Arduino Core |
| GPIO34 no lee | Configurado OUTPUT | GPIO34 es INPUT ONLY - no usar pinMode |

## Referencias
- [ESP32 ADC Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/adc.html)
- [ADC Calibration](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/adc_calibration.html)
- [ADC Non-linearity](https://github.com/espressif/arduino-esp32/issues/1533)