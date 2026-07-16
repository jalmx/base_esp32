# Práctica 01 - Hello World LED

## Objetivo
Hacer parpadear el LED D1 (GPIO0) - Primer programa en ESP32.

## Nivel
**Beginner** | Duración: ~1 hora

## Hardware
- LED D1 (Rojo) → GPIO0
- Resistencia 330Ω integrada en PCB

## Teoría
- **GPIO**: General Purpose Input/Output - Pines programables
- **OUTPUT**: Modo salida - El pin proporciona 3.3V (HIGH) o 0V (LOW)
- **LED**: Diodo emisor luz - Polaridad: Ánodo (+) a GPIO, Cátodo (-) a GND
- **Resistencia**: Limita corriente a ~4mA (seguro para GPIO)

## Código paso a paso

### 1. Esqueleto básico
```cpp
void setup() {
  // Configuración inicial - se ejecuta una vez
}

void loop() {
  // Código principal - se repite infinitamente
}
```

### 2. Configurar pin como salida
```cpp
#define LED_PIN 0  // GPIO0 = LED D1

void setup() {
  pinMode(LED_PIN, OUTPUT);  // Configurar GPIO0 como salida
}
```

### 3. Parpadear (Blink)
```cpp
void loop() {
  digitalWrite(LED_PIN, HIGH);  // Encender (3.3V)
  delay(1000);                  // Esperar 1000ms = 1s
  digitalWrite(LED_PIN, LOW);   // Apagar (0V)
  delay(1000);                  // Esperar 1s
}
```

### 4. Código completo
```cpp
/*
  Hello World - Blink LED D1
  Base ESP32 DevKit V4
*/

#define LED_PIN 0

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(1000);
  digitalWrite(LED_PIN, LOW);
  delay(1000);
}
```

## Verificación
1. Conectar ESP32 DevKit a Base
2. Conectar USB al PC
3. Seleccionar placa: **ESP32 Dev Module**
4. Puerto COM correcto
5. Subir código (botón →)
6. **LED D1 parpadea 1s ON / 1s OFF** ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Cambiar velocidad: 500ms, 200ms, 50ms |
| 🟡 Medio | Parpadear SOS: `... --- ...` (corto=200ms, largo=600ms) |
| 🔴 Difícil | Usar `millis()` en vez de `delay()` (no bloqueante) |

### Reto millis() (no bloqueante)
```cpp
#define LED_PIN 0
const unsigned long INTERVAL = 1000;
unsigned long previousMillis = 0;
bool ledState = false;

void loop() {
  unsigned long current = millis();
  if(current - previousMillis >= INTERVAL) {
    previousMillis = current;
    ledState = !ledState;
    digitalWrite(LED_PIN, ledState);
  }
  // Aquí puedes hacer otras cosas mientras parpadea
}
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| LED no enciende | Placa incorrecta | Tools → Board → ESP32 Dev Module |
| Error upload | Boot mode | Mantener BOOT, presionar EN, soltar BOOT |
| Parpadea muy rápido | Delay en µs | `delay(1000)` = ms, no µs |
| LED tenue | Pin INPUT | `pinMode(OUTPUT)` obligatorio |

## Referencias
- [ESP32 Arduino Core](https://github.com/espressif/arduino-esp32)
- [pinMode Reference](https://www.arduino.cc/reference/en/language/functions/digital-io/pinmode/)
- [digitalWrite Reference](https://www.arduino.cc/reference/en/language/functions/digital-io/digitalwrite/)