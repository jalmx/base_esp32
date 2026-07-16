# Entradas

## Resumen

| Entrada | Tipo | GPIO | Pull | Activo | Notas |
|---------|------|------|------|--------|-------|
| **DIP Switch ×4** | Digital | GPIO27,32,33,35 | Interno ↑ | LOW | Configuración modo |
| **Push Button** | Digital | GPIO25 | Interno ↑ | LOW | Usuario / Reset sw |
| **SPDT Switch** | Digital | GPIO26 | Externo | HIGH/LOW | 3 posiciones |
| **Potenciómetro** | Analógico | GPIO34 (ADC1_CH6) | - | 0-3.3V | 10kΩ lineal |

---

## DIP Switch 4 posiciones (SW_DIP)

### Hardware
```
GPIO27 ──► SW1-1 ──► GND
GPIO32 ──► SW1-2 ──► GND
GPIO33 ──► SW1-3 ──► GND
GPIO35 ──► SW1-4 ──► GND

Pull-up: Interno ESP32 (GPIO_PULLUP_ENABLE)
```

### Lectura
```cpp
#define DIP1 27
#define DIP2 32
#define DIP3 33
#define DIP4 35

void setup() {
  pinMode(DIP1, INPUT_PULLUP);
  pinMode(DIP2, INPUT_PULLUP);
  pinMode(DIP3, INPUT_PULLUP);
  pinMode(DIP4, INPUT_PULLUP);
  // GPIO34,35,36,39 son INPUT_ONLY - no tienen pull-up interno
  // Para GPIO35: usar pull-up EXTERNO 10kΩ a 3.3V si necesario
}

uint8_t readDIP() {
  uint8_t val = 0;
  val |= (!digitalRead(DIP1)) << 0;  // Bit 0
  val |= (!digitalRead(DIP2)) << 1;  // Bit 1
  val |= (!digitalRead(DIP3)) << 2;  // Bit 2
  val |= (!digitalRead(DIP4)) << 3;  // Bit 3
  return val;  // 0x00 a 0x0F
}

// Uso: seleccionar modo programa
void loop() {
  uint8_t mode = readDIP();
  switch(mode) {
    case 0x01: modoBlink(); break;
    case 0x02: modoPWM(); break;
    case 0x03: modoSensor(); break;
    case 0x0F: modoTest(); break;
    default: modoDefault(); break;
  }
}
```

### Aplicaciones típicas
| Valor DIP (binario) | Hex | Función sugerida |
|---------------------|-----|------------------|
| 0000 | 0x0 | Modo normal / Usuario |
| 0001 | 0x1 | Blink LED test |
| 0010 | 0x2 | PWM Demo |
| 0011 | 0x3 | Leer sensores |
| 0100 | 0x4 | Motor test |
| 0101 | 0x5 | Servo test |
| 0110 | 0x6 | Buzzer melodía |
| 0111 | 0x7 | Comunicación serial |
| 1000 | 0x8 | Deep sleep test |
| 1111 | 0xF | Factory test completo |

---

## Push Button (SW_PUSH)

### Hardware
```
GPIO25 ──► SW_PUSH ──► GND
           │
           └── Pull-up interno (INPUT_PULLUP)
```

**¡Comparte GPIO25 con BUZZER!**
- No usar buzzer y botón simultáneamente
- O diseñar firmware que alterne

### Lectura con debounce
```cpp
#define BTN_PIN 25

// Debounce simple
bool readButton() {
  static uint32_t lastPress = 0;
  static bool lastState = HIGH;
  bool current = digitalRead(BTN_PIN);
  
  if(current != lastState) {
    if(millis() - lastPress > 50) {  // 50ms debounce
      lastPress = millis();
      lastState = current;
      return !current;  // true = presionado (LOW)
    }
  }
  return false;
}

// Detectar flanco (press event)
bool buttonPressed() {
  static bool lastReading = HIGH;
  bool current = digitalRead(BTN_PIN);
  bool pressed = (lastReading == HIGH && current == LOW);
  lastReading = current;
  return pressed;
}
```

### Interrupción (wake from sleep)
```cpp
#define BTN_PIN 25

void IRAM_ATTR buttonISR() {
  // Mínimo código en ISR
  buttonFlag = true;
}

void setup() {
  pinMode(BTN_PIN, INPUT_PULLUP);
  attachInterrupt(BTN_PIN, buttonISR, FALLING);
  
  // Configurar wake-up desde deep sleep
  esp_sleep_enable_ext0_wakeup(GPIO_NUM_25, 0); // LOW = wake
}

void loop() {
  if(buttonFlag) {
    buttonFlag = false;
    Serial.println("Botón presionado!");
    // Acción...
  }
  
  // Deep sleep ejemplo
  if(shouldSleep) {
    esp_deep_sleep_start();
  }
}
```

---

## SPDT Switch (SW_SPDT)

### Hardware
```
        3.3V (via 10kΩ pull-up)
           │
           ▼
GPIO26 ───┼───► SW_SPDT Centro (COM)
           │
      ┌────┴────┐
      ▼         ▼
   NO (NC)   NC (NO)
      │         │
     GND       3.3V (via pull-down 10kΩ)
```

**Estados:**
| Posición | GPIO26 | Descripción |
|----------|--------|-------------|
| **Arriba** | HIGH (3.3V) | Conectado a 3.3V vía pull-down |
| **Centro** | FLOAT (evitar) | No conectado - ruido |
| **Abajo** | LOW (GND) | Conectado a GND |

### Implementación recomendada (evitar float)
```
Usar SPDT "ON-ON" (sin centro) o leer solo extremos:

        3.3V
         │
      10kΩ
         │
GPIO26 ───┼───► SW_SPDT Pin 1
         │
        SW_SPDT Pin 2 ───► GND
         │
        SW_SPDT Pin 3 ───► (no conectado)
```
Con ON-ON: Pos1=HIGH, Pos2=LOW, sin estado float.

### Lectura
```cpp
#define SPDT_PIN 26

// Con pull-up interno + switch a GND (configuración simple)
pinMode(SPDT_PIN, INPUT_PULLUP);
// Pos1 (switch abierto): HIGH
// Pos2 (switch a GND): LOW

bool readSPDT() {
  return digitalRead(SPDT_PIN);  // true = Pos1, false = Pos2
}
```

---

## Potenciómetro 10kΩ (RV_POT)

### Hardware
```
3.3V ────────► Pin 1 (Extremo)
              │
GPIO34 (ADC1_CH6) ──► Pin 2 (Cursor/Wiper)
              │
GND ─────────► Pin 3 (Extremo)
```

**GPIO34 = ADC1_CH6 = INPUT ONLY** (sin pull-up/down, sin output)

### Lectura
```cpp
#define POT_PIN 34  // ADC1_CH6

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);    // 0-4095
  analogSetAttenuation(ADC_11db); // Rango ~0-3.3V
}

void loop() {
  int raw = analogRead(POT_PIN);
  float voltage = raw * 3.3 / 4095.0;
  float percentage = (raw / 4095.0) * 100.0;
  
  Serial.printf("Pot: raw=%d V=%.2f %%%.1f\n", raw, voltage, percentage);
  delay(100);
}
```

### Filtrado (ruido ADC)
```cpp
// Promedio móvil simple
#define POT_SAMPLES 16
int potBuffer[POT_SAMPLES];
int potIndex = 0;
long potSum = 0;

int readPotFiltered() {
  potSum -= potBuffer[potIndex];
  potBuffer[potIndex] = analogRead(POT_PIN);
  potSum += potBuffer[potIndex];
  potIndex = (potIndex + 1) % POT_SAMPLES;
  return potSum / POT_SAMPLES;
}
```

### Mapeo a rangos útiles
```cpp
// Map a 0-255 (PWM)
int pwmVal = map(readPotFiltered(), 0, 4095, 0, 255);

// Map a 0-180 (Servo)
int servoAng = map(readPotFiltered(), 0, 4095, 0, 180);

// Map a frecuencia (Buzzer)
int freq = map(readPotFiltered(), 0, 4095, 100, 5000);

// Map a delay (Blink)
int delayMs = map(readPotFiltered(), 0, 4095, 50, 2000);
```

---

## Tabla completa GPIOs Entradas

| GPIO | Entrada | Tipo | Pull | ADC | Touch | Strapping |
|------|---------|------|------|-----|-------|-----------|
| 25 | Push Button / Buzzer | Digital/Out | ↑ | ADC2_CH8 | Sí | - |
| 26 | SPDT Switch | Digital | ↑/↓ | ADC2_CH9 | Sí | - |
| 27 | DIP-1 | Digital | ↑ | ADC2_CH7 | Sí | - |
| 32 | DIP-2 | Digital | ↑ | ADC1_CH4 | Sí | - |
| 33 | DIP-3 | Digital | ↑ | ADC1_CH5 | Sí | - |
| 34 | Potenciómetro | Analog | - | ADC1_CH6 | No | - |
| 35 | DIP-4 | Digital | Ext ↑ | ADC1_CH7 | No | - |

> **Nota**: GPIO34,35,36,39 son **INPUT ONLY**. No pueden ser output, no tienen pull-up/down interno.
> - GPIO35 (DIP-4): Requiere pull-up **externo** 10kΩ a 3.3V para lectura fiable
> - GPIO34 (Pot): No necesita pull, divisor de tensión del potenciómetro define voltaje

---

## Prácticas sugeridas

| Práctica | Entradas usadas | Nivel |
|----------|-----------------|-------|
| **03-Analog Read** | Potenciómetro | Beginner |
| **04-Interrupts** | Push Button | Beginner |
| **08-Switches** | DIP Switch + SPDT | Intermediate |
| **11-Sensor Fusion** | Todas + Sensores | Advanced |