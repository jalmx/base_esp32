# LEDs

## Resumen

| LED | Referencia | GPIO | Color | Resistencia | Función |
|-----|------------|------|-------|-------------|---------|
| **D1** | LED_USR1 | GPIO0 | Rojo | 330Ω | Usuario / Boot indicator |
| **D2** | LED_USR2 | GPIO2 | Azul | 330Ω | Built-in / RGB Data |
| **D3** | LED_USR3 | GPIO4 | Verde | 330Ω | Usuario |
| **D4** | LED_USR4 | GPIO5 | Amarillo | 330Ω | Usuario |
| **RGB** | LED_RGB | GPIO2 | WS2812B | - | Direccionable (comparte GPIO2) |
| **PWR5V** | LED_5V | - | Rojo | 1kΩ | Indicador rail 5V |
| **PWR3V3** | LED_3V3 | - | Verde | 1kΩ | Indicador rail 3.3V |

---

## LEDs Usuario (D1-D4)

### Hardware
```
GPIOx ──► [330Ω] ──► Ánodo LED ──► Cátodo ──► GND

GPIO0 ──► D1 (Rojo)
GPIO2 ──► D2 (Azul) ──┐
GPIO4 ──► D3 (Verde)  │
GPIO5 ──► D4 (Amarillo)│
                      └── Comparten GPIO2 con RGB
```

### Control básico
```cpp
#define LED_D1 0
#define LED_D2 2
#define LED_D3 4
#define LED_D4 5

void setup() {
  pinMode(LED_D1, OUTPUT);
  pinMode(LED_D2, OUTPUT);
  pinMode(LED_D3, OUTPUT);
  pinMode(LED_D4, OUTPUT);
}

void loop() {
  // Secuencia
  digitalWrite(LED_D1, HIGH); delay(200);
  digitalWrite(LED_D2, HIGH); delay(200);
  digitalWrite(LED_D3, HIGH); delay(200);
  digitalWrite(LED_D4, HIGH); delay(200);
  
  digitalWrite(LED_D1, LOW); delay(200);
  digitalWrite(LED_D2, LOW); delay(200);
  digitalWrite(LED_D3, LOW); delay(200);
  digitalWrite(LED_D4, LOW); delay(200);
}
```

### PWM (brillo)
```cpp
#define PWM_FREQ 5000
#define PWM_RES 8
#define CH_D1 0
#define CH_D2 1
#define CH_D3 2
#define CH_D4 3

void setupPWM() {
  ledcAttachPin(LED_D1, CH_D1);
  ledcAttachPin(LED_D2, CH_D2);
  ledcAttachPin(LED_D3, CH_D3);
  ledcAttachPin(LED_D4, CH_D4);
  
  ledcSetup(CH_D1, PWM_FREQ, PWM_RES);
  ledcSetup(CH_D2, PWM_FREQ, PWM_RES);
  ledcSetup(CH_D3, PWM_FREQ, PWM_RES);
  ledcSetup(CH_D4, PWM_FREQ, PWM_RES);
}

void setLED(uint8_t ch, uint8_t brightness) {  // 0-255
  ledcWrite(ch, brightness);
}

// Fade in/out
void fadeLED(uint8_t ch) {
  for(int i=0; i<=255; i++) { setLED(ch, i); delay(5); }
  for(int i=255; i>=0; i--) { setLED(ch, i); delay(5); }
}
```

### Patrones comunes
```cpp
// Knight Rider (coche fantástico)
void knightRider() {
  uint8_t leds[4] = {CH_D1, CH_D2, CH_D3, CH_D4};
  for(int dir=0; dir<2; dir++) {
    for(int i=0; i<4; i++) {
      int idx = dir ? 3-i : i;
      setLED(leds[idx], 255);
      delay(100);
      setLED(leds[idx], 0);
    }
  }
}

// Binario counter
void binaryCounter() {
  for(int i=0; i<16; i++) {
    setLED(CH_D1, (i>>0)&1 ? 255:0);
    setLED(CH_D2, (i>>1)&1 ? 255:0);
    setLED(CH_D3, (i>>2)&1 ? 255:0);
    setLED(CH_D4, (i>>3)&1 ? 255:0);
    delay(500);
  }
}
```

---

## LED RGB WS2812B (LED_RGB)

### Hardware
```
GPIO2 ──► DIN (WS2812B)
3.3V ───► VDD
GND ────► VSS

⚠️ COMPARTE GPIO2 con LED_D2 (Azul)
   - No usar ambos simultáneamente
   - Desconectar LED_D2 físicamente si usar RGB
   - O usar GPIO2 solo para uno
```

### Librería FastLED / Adafruit NeoPixel
```cpp
#include <FastLED.h>

#define RGB_PIN 2
#define NUM_LEDS 1
CRGB leds[NUM_LEDS];

void setup() {
  FastLED.addLeds<WS2812B, RGB_PIN, GRB>(leds, NUM_LEDS);
  FastLED.setBrightness(50);  // 0-255
}

void loop() {
  // Colores básicos
  leds[0] = CRGB::Red;   FastLED.show(); delay(500);
  leds[0] = CRGB::Green; FastLED.show(); delay(500);
  leds[0] = CRGB::Blue;  FastLED.show(); delay(500);
  
  // Arcoiris
  static uint8_t hue = 0;
  leds[0] = CHSV(hue++, 255, 255);
  FastLED.show();
  delay(20);
}
```

### HSV vs RGB
```cpp
// HSV más intuitivo para efectos
void rainbow() {
  for(int h=0; h<255; h++) {
    leds[0] = CHSV(h, 255, 255);  // Hue, Saturation, Value
    FastLED.show();
    delay(10);
  }
}

// Pulso
void pulse(uint8_t hue) {
  for(int v=0; v<=255; v++) {
    leds[0] = CHSV(hue, 255, v);
    FastLED.show();
    delay(2);
  }
  for(int v=255; v>=0; v--) {
    leds[0] = CHSV(hue, 255, v);
    FastLED.show();
    delay(2);
  }
}
```

---

## LEDs Potencia (PWR5V, PWR3V3)

### Hardware
```
+5V Rail ──► [1kΩ] ──► LED_5V (Rojo) ──► GND
+3.3V Rail ──► [1kΩ] ──► LED_3V3 (Verde) ──► GND
```
- **Siempre encendidos** si rail activo
- **Diagnóstico visual** inmediato
- No controlables por GPIO

---

## Prácticas sugeridas

| Práctica | LEDs | Nivel |
|----------|------|-------|
| **01-Hello World** | D1 blink | Beginner |
| **02-PWM** | D1-D4 fade | Beginner |
| **03-Binary Counter** | D1-D4 binario | Beginner |
| **04-Knight Rider** | D1-D4 secuencia | Beginner |
| **07-RGB** | WS2812B arcoiris | Intermediate |
| **08-Status Indicators** | D1-D4 + RGB estados sistema | Intermediate |

---

## Esquema conexiones

```
┌────────────────────────────────────────────────────────────┐
│                      ESP32 GPIO                              │
│  ┌──GPIO0 ──[330Ω]──►│>── D1 (Rojo)                         │
│  │                                                     │     │
│  ├──GPIO2 ──[330Ω]──►│>── D2 (Azul)                      │     │
│  │                └──────────────┐                        │     │
│  │                               ▼                        │     │
│  │                         ┌─────────────┐                │     │
│  │                         │  WS2812B    │                │     │
│  │                         │   (RGB)     │                │     │
│  │                         │  DIN=GPIO2  │                │     │
│  │                         └─────────────┘                │     │
│  ├──GPIO4 ──[330Ω]──►│>── D3 (Verde)                       │     │
│  └──GPIO5 ──[330Ω]──►│>── D4 (Amarillo)                    │     │
└────────────────────────────────────────────────────────────┘

Rail +5V ──[1kΩ]──►│>── LED_5V (Rojo) ──► GND
Rail +3.3V ──[1kΩ]──►│>── LED_3V3 (Verde) ──► GND
```

---

## Cálculo resistencias

| LED Color | Vf típico | IF máx | R para 3.3V @ 4mA | R para 3.3V @ 10mA |
|-----------|-----------|--------|-------------------|---------------------|
| **Rojo** | 1.8-2.2V | 20mA | 330Ω (4.5mA) | 150Ω (10mA) |
| **Verde** | 2.0-2.4V | 20mA | 330Ω (2.7mA) | 100Ω (9mA) |
| **Azul** | 2.8-3.2V | 20mA | 330Ω (0.3mA) ⚠️ | 47Ω (1mA) |
| **Amarillo** | 1.9-2.3V | 20mA | 330Ω (4.2mA) | 150Ω (9mA) |

> **Nota**: LED Azul en 3.3V con 330Ω apenas brilla (Vf ≈ 3V). En PCB v0.4 se usa 330Ω para todos por simplicidad BOM. Para brillo igualado, usar resistencias distintas por color.