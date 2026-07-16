# Práctica 07 - Buzzer

## Objetivo
Generar tonos, melodías y alarmas con buzzer pasivo en GPIO25.

## Nivel
**Intermediate** | Duración: ~2 horas

## Hardware
- Buzzer pasivo (piezoeléctrico) → GPIO25
- **¡Comparte GPIO25 con Push Button!** No usar simultáneamente.

## Teoría
- **Buzzer pasivo**: Requiere señal AC (PWM/tonos), no DC
- **Frecuencia resonante**: ~2.7 kHz (máximo volumen)
- **tone()**: Función Arduino, usa timer hardware
- **PWM manual**: `ledc` para control total

## Método 1: tone() - Simple
```cpp
#define BUZZER 25

void setup() {
  // tone() configura timer automáticamente
}

void loop() {
  tone(BUZZER, 440, 500);  // 440Hz (La4) 500ms
  delay(600);
  tone(BUZZER, 523, 500);  // 523Hz (Do5)
  delay(600);
  noTone(BUZZER);
  delay(1000);
}
```

## Método 2: PWM (ledc) - Control total
```cpp
#define BUZZER 25
#define CH 5

void tonePWM(int freq, int duration) {
  if(freq == 0) { ledcWrite(CH, 0); return; }
  ledcSetup(CH, freq, 8);
  ledcWrite(CH, 128);  // 50% duty
  delay(duration);
  ledcWrite(CH, 0);
}

void setup() {
  ledcAttachPin(BUZZER, CH);
}

void loop() {
  tonePWM(440, 300);
  tonePWM(523, 300);
  tonePWM(659, 300);
  delay(1000);
}
```

## Método 3: Melodía completa (array notas)
```cpp
#include "pitches.h"

#define BUZZER 25

// Star Wars - Imperial March (fragmento)
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
    tone(BUZZER, melody[i], durations[i] * 0.9);
    delay(durations[i]);
  }
  noTone(BUZZER);
}
```

## Código completo
```cpp
/*
  Buzzer - Tonos y melodías
  Base ESP32 DevKit V4
*/

#define BUZZER 25

// Definiciones básicas (pitches.h simplificado)
#define NOTE_C4  262
#define NOTE_D4  294
#define NOTE_E4  330
#define NOTE_F4  349
#define NOTE_G4  392
#define NOTE_A4  440
#define NOTE_B4  494
#define NOTE_C5  523

int scale[] = {NOTE_C4, NOTE_D4, NOTE_E4, NOTE_F4, NOTE_G4, NOTE_A4, NOTE_B4, NOTE_C5};

void playTone(int freq, int dur) {
  if(freq > 0) tone(BUZZER, freq, dur * 0.9);
  delay(dur);
}

void setup() {
  Serial.begin(115200);
  Serial.println("Buzzer listo");
}

void loop() {
  Serial.println("Escala ascendente");
  for(int n : scale) playTone(n, 200);
  
  delay(500);
  
  Serial.println("Escala descendente");
  for(int i=7; i>=0; i--) playTone(scale[i], 200);
  
  delay(1000);
}
```

## Verificación
1. Subir código (¡sin botón presionado!)
2. **Se escuchan tonos claros** ✅
3. Serial muestra mensajes ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Tocar "Happy Birthday" o "Jingle Bells" |
| 🟢 Fácil | Alarma: 3 beeps rápidos + pausa |
| 🟡 Medio | Theremin: Potenciómetro (GPIO34) → Frecuencia |
| 🟡 Medio | Morse: Codificar/decodificar SOS |
| 🔴 Difícil | Polifonía simple: 2 timers = 2 voces |
| 🔴 Difícil | DTMF: Tonos teléfono (filas+columnas) |

### Reto: Theremin
```cpp
#define POT_PIN 34

void loop() {
  int pot = analogRead(POT_PIN);  // 0-4095
  int freq = map(pot, 0, 4095, 100, 5000);
  tone(BUZZER, freq);
  delay(10);
}
```

### Reto: Código Morse
```cpp
const char* morse[] = {
  ".-", "-...", "-.-.", "-..", ".", "..-.", "--.", "....", "..",
  ".---", "-.-", ".-..", "--", "-.", "---", ".--.", "--.-", ".-.",
  "...", "-", "..-", "...-", ".--", "-..-", "-.--", "--.."
};

void sendMorse(const char* msg) {
  for(const char* c = msg; *c; c++) {
    if(*c == ' ') { delay(700); continue; }
    int idx = toupper(*c) - 'A';
    if(idx < 0 || idx > 25) continue;
    
    for(const char* s = morse[idx]; *s; s++) {
      tone(BUZZER, 800, *s == '.' ? 100 : 300);
      delay(*s == '.' ? 150 : 350);
    }
    delay(300);  // espacio entre letras
  }
}
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| No suena | Pin mal / Buzzer activo | Verificar GPIO25, buzzer pasivo |
| Suena muy bajo | Duty cycle bajo | `ledcWrite(CH, 128)` = 50% |
| `tone()` interfiere con PWM | Timers compartidos | Usar canales PWM distintos |
| Pitido constante | `noTone()` no llamado | Siempre llamar `noTone()` al acabar |
| Conflicto botón | GPIO25 compartido | No usar botón y buzzer juntos |

## Referencias
- [Arduino tone()](https://www.arduino.cc/reference/en/language/functions/advanced-io/tone/)
- [Buzzer Pasivo](https://components101.com/sites/default/files/component_pin/Piezo-Buzzer.pdf)
- [RTTTL Format](https://en.wikipedia.org/wiki/Ring_Tone_Transfer_Language)