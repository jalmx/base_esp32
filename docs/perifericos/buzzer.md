# Buzzer

## Especificaciones

| Parámetro | Valor |
|-----------|-------|
| **Tipo** | Buzzer pasivo (piezoeléctrico) |
| **Modelo** | SS-12D1305-G2 (o equivalente) |
| **Voltaje** | 3-5V (drive directo GPIO) |
| **Corriente** | ~15mA @ 3.3V |
| **Frecuencia resonante** | ~2.7 kHz |
| **SPL** | 85dB @ 10cm |
| **Encapsulado** | THT 12mm, pitch 7.5mm |
| **Pines** | 2 pines (sin polaridad) |

## Conexión en la Base

```
GPIO25 (DAC1) ──► Buzzer Pin 1
GND ─────────────► Buzzer Pin 2
```

**Nota**: GPIO25 también es DAC1 y ADC2_CH8. Compartido con sensor LM35 (ADC).

## Teoría: Buzzer pasivo vs activo

| Tipo | Drive | Frecuencia | Control |
|------|-------|------------|---------|
| **Activo** | DC (HIGH/LOW) | Fija interna | Solo ON/OFF |
| **Pasivo** | AC (PWM/tonos) | Variable | Control total frecuencia |

**El nuestro es PASIVO** → Requiere señal alterna (PWM, tone(), DAC)

## Programación

### Método 1: `tone()` - Arduino core (Recomendado)
```cpp
#define BUZZER_PIN 25

void setup() {
  // tone() usa hardware timer, no bloquea
}

void loop() {
  // Nota única
  tone(BUZZER_PIN, 440, 500); // 440Hz (La4) por 500ms
  delay(600);
  
  // Melodía simple
  int melody[] = {262, 294, 330, 349, 392, 440, 494, 523}; // Do Re Mi Fa Sol La Si Do
  for(int n : melody) {
    tone(BUZZER_PIN, n, 200);
    delay(250);
  }
  noTone(BUZZER_PIN);
  delay(1000);
}
```

### Método 2: PWM con `ledc` (Más control)
```cpp
#define BUZZER_PIN 25
#define PWM_CHANNEL 0

void playTone(uint16_t freq, uint16_t duration_ms) {
  if(freq == 0) {
    ledcWrite(PWM_CHANNEL, 0);
    return;
  }
  ledcSetup(PWM_CHANNEL, freq, 8); // 8-bit resolution
  ledcWrite(PWM_CHANNEL, 128);     // 50% duty cycle
  delay(duration_ms);
  ledcWrite(PWM_CHANNEL, 0);
}

void setup() {
  ledcAttachPin(BUZZER_PIN, PWM_CHANNEL);
}

void loop() {
  playTone(440, 300);  // La4
  playTone(523, 300);  // Do5
  playTone(659, 300);  // Mi5
  delay(1000);
}
```

### Método 3: DAC (Waveform arbitraria)
```cpp
#define BUZZER_PIN 25  // DAC1

void playSineWave(uint16_t freq, uint16_t duration_ms) {
  const uint16_t samples = 100;
  uint32_t period_us = 1000000 / freq;
  uint32_t start = micros();
  
  while(micros() - start < duration_ms * 1000) {
    for(int i=0; i<samples; i++) {
      // Seno: 0-255 centrado en 127
      uint8_t val = 127 + 127 * sin(2 * PI * i / samples);
      dacWrite(BUZZER_PIN, val);
      delayMicroseconds(period_us / samples);
    }
  }
  dacWrite(BUZZER_PIN, 0);
}
```

### Melodía completa (RTTTL / notas)
```cpp
#include "pitches.h"  // Definiciones NOTE_C4, NOTE_D4, etc.

#define BUZZER_PIN 25

// Melodía: Star Wars Imperial March (simplificada)
int melody[] = {
  NOTE_G3, NOTE_G3, NOTE_G3, NOTE_DS3, NOTE_AS3,
  NOTE_G3, NOTE_DS3, NOTE_AS3, NOTE_G3,
  NOTE_D4, NOTE_D4, NOTE_D4, NOTE_DS4, NOTE_AS3,
  NOTE_FS3, NOTE_DS3, NOTE_AS3, NOTE_G3
};

int durations[] = {
  500, 500, 500, 350, 150,
  500, 350, 150, 1000,
  500, 500, 500, 350, 150,
  500, 350, 150, 1000
};

void playMelody() {
  for(int i=0; i<sizeof(melody)/sizeof(int); i++) {
    tone(BUZZER_PIN, melody[i], durations[i] * 0.9);
    delay(durations[i]);
  }
  noTone(BUZZER_PIN);
}
```

## Prácticas sugeridas

| Práctica | Descripción | Nivel |
|----------|-------------|-------|
| **01-Beep** | Beep simple 1s on/off | Beginner |
| **02-Scale** | Tocar escala mayor | Beginner |
| **03-Melody** | Tocar canción completa | Intermediate |
| **04-Alarm** | Alarma con sensor (temp > 30°C) | Intermediate |
| **05-Morse** | Código morse por buzzer | Intermediate |
| **06-DTMF** | Tonos teléfono (DTMF) | Advanced |
| **07-Synthesis** | Síntesis aditiva simple | Advanced |

## Consideraciones de hardware

### Driver transistor (opcional, para más volumen)
```
GPIO25 ──[1kΩ]──► Base (BC547/2N2222)
          Colector ──► Buzzer ──► 5V
          Emisor ──────► GND
```
- Permite usar 5V rail → más volumen
- Aísla GPIO de corriente inductiva buzzer
- **No implementado en v0.4** (drive directo GPIO suficiente)

### Filtrado EMI
- Buzzer piezoeléctrico = carga capacitiva ~10-20nF
- No genera EMI significativa
- Si se usa transistor: añadir diodo flyback en colector

## Datasheet

- [SS-12D1305-G2 Datasheet](../datasheets/SS-12D1305-G2.PDF)
- [Buzzer Pasivo Genérico](https://components101.com/sites/default/files/component_pin/Piezo-Buzzer.pdf)