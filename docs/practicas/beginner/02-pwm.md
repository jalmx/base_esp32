# Práctica 02 - PWM LED

## Objetivo
Controlar brillo del LED D3 (GPIO4) usando PWM (Pulse Width Modulation).

## Nivel
**Beginner** | Duración: ~1.5 horas

## Hardware
- LED D3 (Verde) → GPIO4
- Resistencia 330Ω en PCB

## Teoría
- **PWM**: Modulación por ancho de pulso - Simula voltaje variable con señal digital
- **Duty Cycle**: Porcentaje tiempo HIGH en un período (0-100%)
- **Frecuencia**: Ciclos por segundo (Hz). >1kHz evita parpadeo visible
- **Resolución**: Bits de precisión (8-bit = 0-255, 10-bit = 0-1023, etc.)
- **ESP32**: 16 canales PWM independientes (LEDC), cualquier GPIO

## Código paso a paso

### 1. Configurar PWM (LEDC)
```cpp
#define LED_PIN 4

// Parámetros PWM
const int PWM_FREQ = 5000;    // 5 kHz (inaudible, sin flicker)
const int PWM_CHANNEL = 0;    // Canal 0-15
const int PWM_RES = 8;        // 8-bit = 0-255

void setup() {
  ledcSetup(PWM_CHANNEL, PWM_FREQ, PWM_RES);
  ledcAttachPin(LED_PIN, PWM_CHANNEL);
}
```

### 2. Fade in / Fade out
```cpp
void loop() {
  // Fade in (0 → 255)
  for(int duty = 0; duty <= 255; duty++) {
    ledcWrite(PWM_CHANNEL, duty);
    delay(5);
  }
  
  // Fade out (255 → 0)
  for(int duty = 255; duty >= 0; duty--) {
    ledcWrite(PWM_CHANNEL, duty);
    delay(5);
  }
}
```

### 3. Código completo
```cpp
/*
  PWM LED - Control brillo LED D3
  Base ESP32 DevKit V4
*/

#define LED_PIN 4

const int PWM_FREQ = 5000;
const int PWM_CHANNEL = 0;
const int PWM_RES = 8;

void setup() {
  ledcSetup(PWM_CHANNEL, PWM_FREQ, PWM_RES);
  ledcAttachPin(LED_PIN, PWM_CHANNEL);
  Serial.begin(115200);
  Serial.println("PWM LED iniciado");
}

void loop() {
  // Subida
  for(int duty = 0; duty <= 255; duty++) {
    ledcWrite(PWM_CHANNEL, duty);
    delay(3);
  }
  delay(500);
  
  // Bajada
  for(int duty = 255; duty >= 0; duty--) {
    ledcWrite(PWM_CHANNEL, duty);
    delay(3);
  }
  delay(500);
}
```

## Verificación
1. Subir código
2. **LED D3 aumenta y disminuye brillo suavemente** ✅
3. Abrir Serial Monitor → ver mensajes

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Cambiar frecuencia a 1000Hz, 20000Hz - notar diferencia |
| 🟢 Fácil | Cambiar resolución a 10-bit (0-1023) - ajustar bucles |
| 🟡 Medio | Controlar 2 LEDs (D3 y D4) con fase opuesta (uno sube, otro baja) |
| 🟡 Medio | Leer potenciómetro (GPIO34) y mapear a brillo LED |
| 🔴 Difícil | "Respiración" suave: `sin()` wave en vez de rampas lineales |

### Reto: Potenciómetro → PWM
```cpp
#define POT_PIN 34
#define LED_PIN 4

void setup() {
  ledcSetup(0, 5000, 8);
  ledcAttachPin(LED_PIN, 0);
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
}

void loop() {
  int pot = analogRead(POT_PIN);          // 0-4095
  int brightness = map(pot, 0, 4095, 0, 255);  // Mapear a 8-bit
  ledcWrite(0, brightness);
  delay(10);
}
```

### Reto: Onda sinusoidal (respiración)
```cpp
void loop() {
  for(int i = 0; i < 360; i++) {
    float rad = i * DEG_TO_RAD;
    // sin() va -1 a 1 → mapear a 0-255
    int duty = (sin(rad) + 1) * 127.5;
    ledcWrite(0, duty);
    delay(5);
  }
}
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| LED no cambia brillo | Pin no PWM | Verificar `ledcAttachPin()` correcto |
| Parpadea a bajo brillo | Frecuencia baja | Usar ≥ 1000Hz (5000Hz recomendado) |
| `ledcSetup` error | Canal ocupado | Usar canal distinto (0-15) |
| Solo ON/OFF | Resolución 1-bit | `PWM_RES = 8` (o más) |

## Referencias
- [LEDC API](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/ledc.html)
- [PWM Tutorial](https://randomnerdtutorials.com/esp32-pwm-arduino-ide/)
- [ESP32 Technical Reference - LEDC](https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf)