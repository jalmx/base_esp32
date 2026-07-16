# Práctica 08 - Switches y Entradas

## Objetivo
Leer DIP Switch (4-bit), SPDT y Push Button para configurar modos de operación.

## Nivel
**Intermediate** | Duración: ~2.5 horas

## Hardware
| Entrada | GPIO | Tipo | Pull |
|---------|------|------|------|
| DIP-1 | 27 | Digital | Interno ↑ |
| DIP-2 | 32 | Digital | Interno ↑ |
| DIP-3 | 33 | Digital | Interno ↑ |
| DIP-4 | 35 | Digital | **Externo 10kΩ ↑** |
| Push Button | 25 | Digital | Interno ↑ |
| SPDT | 26 | Digital | Externo ↑/↓ |

> **Nota**: GPIO34,35,36,39 son INPUT ONLY. GPIO35 requiere pull-up externo.

## Código - DIP Switch (4 bits → 0-15)
```cpp
#define DIP1 27
#define DIP2 32
#define DIP3 33
#define DIP4 35

uint8_t readDIP() {
  uint8_t val = 0;
  val |= (!digitalRead(DIP1)) << 0;
  val |= (!digitalRead(DIP2)) << 1;
  val |= (!digitalRead(DIP3)) << 2;
  val |= (!digitalRead(DIP4)) << 3;
  return val;  // 0x00 a 0x0F
}
```

## Código - SPDT (3 posiciones)
```cpp
#define SPDT_PIN 26

// Con pull-up interno + switch a GND (2 posiciones)
bool readSPDT() {
  return digitalRead(SPDT_PIN);  // true=Pos1, false=Pos2
}

// Para 3 posiciones real (ON-OFF-ON) usar 2 pines
```

## Código - Push Button con debounce
```cpp
#define BTN_PIN 25

bool readButton() {
  static uint32_t lastPress = 0;
  static bool lastState = HIGH;
  bool current = digitalRead(BTN_PIN);
  
  if(current != lastState && millis() - lastPress > 50) {
    lastPress = millis();
    lastState = current;
    return !current;  // true = presionado
  }
  return false;
}
```

## Código completo - Máquina de estados
```cpp
/*
  Switches - Modos de operación
  Base ESP32 DevKit V4
*/

#define DIP1 27
#define DIP2 32
#define DIP3 33
#define DIP4 35
#define BTN_PIN 25
#define SPDT_PIN 26
#define LED_D1 0
#define LED_D3 4
#define LED_D4 5

enum Mode {
  MODE_BLINK   = 0x1,
  MODE_PWM     = 0x2,
  MODE_SENSOR  = 0x3,
  MODE_MOTOR   = 0x4,
  MODE_SERVO   = 0x5,
  MODE_BUZZER  = 0x6,
  MODE_COMMS   = 0x7,
  MODE_TEST    = 0xF
};

Mode currentMode = MODE_BLINK;
uint32_t lastModeCheck = 0;

void setup() {
  pinMode(DIP1, INPUT_PULLUP);
  pinMode(DIP2, INPUT_PULLUP);
  pinMode(DIP3, INPUT_PULLUP);
  pinMode(DIP4, INPUT_PULLUP);  // Requiere pull-up externo 10k
  pinMode(BTN_PIN, INPUT_PULLUP);
  pinMode(SPDT_PIN, INPUT_PULLUP);
  
  pinMode(LED_D1, OUTPUT);
  pinMode(LED_D3, OUTPUT);
  pinMode(LED_D4, OUTPUT);
  
  Serial.begin(115200);
  Serial.println("Switches - Selector de modos");
}

uint8_t readDIP() {
  uint8_t v = 0;
  v |= (!digitalRead(DIP1)) << 0;
  v |= (!digitalRead(DIP2)) << 1;
  v |= (!digitalRead(DIP3)) << 2;
  v |= (!digitalRead(DIP4)) << 3;
  return v;
}

void loop() {
  // Leer DIP cada 500ms
  if(millis() - lastModeCheck > 500) {
    lastModeCheck = millis();
    uint8_t dip = readDIP();
    if(dip != currentMode) {
      currentMode = (Mode)dip;
      Serial.printf("Modo cambiado: 0x%02X\n", currentMode);
    }
  }
  
  // Ejecutar modo actual
  switch(currentMode) {
    case MODE_BLINK:
      digitalWrite(LED_D1, millis() % 1000 < 500);
      break;
    case MODE_PWM: {
      static int duty = 0, dir = 1;
      duty += dir * 5;
      if(duty >= 255 || duty <= 0) dir = -dir;
      analogWrite(LED_D3, duty);
      delay(10);
      break;
    }
    case MODE_SENSOR:
      // Leer sensores (ver práctica 05)
      break;
    case MODE_MOTOR:
      // Control motor (ver práctica 06)
      break;
    case MODE_TEST:
      // Test todos los LEDs
      digitalWrite(LED_D1, HIGH);
      digitalWrite(LED_D3, HIGH);
      digitalWrite(LED_D4, HIGH);
      break;
    default:
      digitalWrite(LED_D1, LOW);
      digitalWrite(LED_D3, LOW);
      digitalWrite(LED_D4, LOW);
  }
  
  // Botón: toggle LED D4
  static bool btnLast = HIGH;
  bool btnNow = digitalRead(BTN_PIN);
  if(btnLast == HIGH && btnNow == LOW) {
    digitalWrite(LED_D4, !digitalRead(LED_D4));
    Serial.println("Botón: Toggle LED D4");
  }
  btnLast = btnNow;
  
  // SPDT: Mostrar estado
  static bool spdtLast = HIGH;
  bool spdtNow = digitalRead(SPDT_PIN);
  if(spdtLast != spdtNow) {
    Serial.printf("SPDT: %s\n", spdtNow ? "Pos1 (HIGH)" : "Pos2 (LOW)");
  }
  spdtLast = spdtNow;
  
  delay(10);
}
```

## Verificación
1. Subir código
2. Serial Monitor 115200
3. **Mover DIP switches** → "Modo cambiado: 0xXX" ✅
4. **Presionar botón** → LED D4 toggle ✅
5. **Mover SPDT** → "SPDT: Pos1/Pos2" ✅
6. Modo 0xF (1111) → Todos LEDs ON ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Añadir más modos (hasta 16) |
| 🟢 Fácil | Mostrar modo en binario con 4 LEDs |
| 🟡 Medio | Guardar modo en EEPROM/Preferences (persistente) |
| 🟡 Medio | Menú navegable: Botón=cambiar, SPDT=seleccionar |
| 🔴 Difícil | Configuración WiFi por DIP (SSID/Password predefinidos) |
| 🔴 Difícil | State machine completa con transiciones válidas |

### Reto: Modo persistente
```cpp
#include <Preferences.h>

Preferences prefs;

void saveMode(Mode m) {
  prefs.begin("config", false);
  prefs.putUChar("mode", m);
  prefs.end();
}

Mode loadMode() {
  prefs.begin("config", true);
  Mode m = (Mode)prefs.getUChar("mode", MODE_BLINK);
  prefs.end();
  return m;
}
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| DIP-4 (GPIO35) siempre HIGH | Falta pull-up externo | Añadir 10kΩ 3.3V-GPIO35 |
| Lectura errática | Ruido / cables largos | Pull-up fuertes, capacitor 100nF |
| Botón rebota | Sin debounce | 50ms en software o 100nF hardware |
| SPDT estado medio (float) | Switch 3 posiciones | Usar ON-ON o leer solo extremos |
| Modo no cambia | DIP mal soldado | Verificar continuidad GPIO-GND |

## Referencias
- [ESP32 GPIO](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/gpio.html)
- [Preferences Library](https://github.com/espressif/arduino-esp32/tree/master/libraries/Preferences)