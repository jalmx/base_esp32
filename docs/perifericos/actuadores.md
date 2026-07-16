# Actuadores

## Motor DC - MX1508

### Hardware
```
MX1508 Dual H-Bridge
┌─────────────────────────────────────────────────────────────┐
│  GPIO12 ──► IN1 ──┐                                        │
│  GPIO13 ──► IN2 ──┤       Canal A (Motor 1)                │
│                   ├──── OUT1 ──► MOT_A+                     │
│                   ├──── OUT2 ──► MOT_A-                     │
│  GPIO14 ──► IN3 ──┤                                        │
│  GPIO15 ──► IN4 ──┤       Canal B (Motor 2)                │
│                   ├──── OUT3 ──► MOT_B+                     │
│                   ├──── OUT4 ──► MOT_B-                     │
│                      VM ──► +5V Rail (desde LM7805)        │
│                      VCC ──► +5V Rail (lógica)             │
│                      GND ──► GND                            │
└─────────────────────────────────────────────────────────────┘

Terminal Block (J_TERM):
Pin 1: MOT_A+  │  Pin 2: MOT_A-
Pin 3: MOT_B+  │  Pin 4: MOT_B-
```

### Control básico (un motor)
```cpp
#define MOT_A_IN1 12
#define MOT_A_IN2 13

void motorA(int speed) {  // -255 a 255
  if(speed > 0) {
    analogWrite(MOT_A_IN1, speed);
    analogWrite(MOT_A_IN2, 0);
  } else if(speed < 0) {
    analogWrite(MOT_A_IN1, 0);
    analogWrite(MOT_A_IN2, -speed);
  } else {
    analogWrite(MOT_A_IN1, 0);
    analogWrite(MOT_A_IN2, 0);  // Coast
  }
  // Para brake: analogWrite ambos = 255 (o 128 para brake suave)
}

void setup() {
  pinMode(MOT_A_IN1, OUTPUT);
  pinMode(MOT_A_IN2, OUTPUT);
  // PWM setup
  ledcAttachPin(MOT_A_IN1, 0);
  ledcAttachPin(MOT_A_IN2, 1);
  ledcSetup(0, 20000, 8);  // 20kHz, 8-bit
  ledcSetup(1, 20000, 8);
}

void loop() {
  motorA(200);  delay(2000);  // Forward
  motorA(0);    delay(500);   // Stop
  motorA(-200); delay(2000);  // Reverse
  motorA(0);    delay(500);
}
```

### Control dos motores (diferencial)
```cpp
#define MOT_A_IN1 12
#define MOT_A_IN2 13
#define MOT_B_IN3 14
#define MOT_B_IN4 15

// Canales PWM: 0,1,2,3
void setMotor(int motor, int speed) {  // motor: 0=A, 1=B
  int in1 = motor ? 2 : 0;
  int in2 = motor ? 3 : 1;
  
  if(speed > 0) { ledcWrite(in1, speed); ledcWrite(in2, 0); }
  else if(speed < 0) { ledcWrite(in1, 0); ledcWrite(in2, -speed); }
  else { ledcWrite(in1, 0); ledcWrite(in2, 0); }
}

void moveRobot(int left, int right) {  // -255 a 255
  setMotor(0, left);
  setMotor(1, right);
}

// Movimientos básicos
void forward(int s) { moveRobot(s, s); }
void backward(int s) { moveRobot(-s, -s); }
void turnLeft(int s) { moveRobot(-s, s); }
void turnRight(int s) { moveRobot(s, -s); }
void stopRobot() { moveRobot(0, 0); }
```

### PWM Frequency - Importante!
```cpp
// MX1508 soporta hasta ~20-50kHz
// ESP32: 20kHz buen compromiso (audible whisper vs eficiencia)

#define PWM_FREQ 20000  // 20 kHz - fuera rango auditivo humano
#define PWM_RES 8       // 8-bit = 0-255

ledcSetup(channel, PWM_FREQ, PWM_RES);
```

### Corriente y protección
- **Límite interno**: ~2.5A pico por canal
- **Térmico**: Shutdown @ 150°C
- **Medir corriente**: Usar A9013 en rail +5V (ver [Monitoreo](../perifericos/monitoreo.md))

---

## Servo - Control directo

### Hardware
```
GPIO18 ──► Signal (Naranja/Blanco)
+5V Rail ──► VCC (Rojo)  ← Desde LM7805 (terminal block)
GND ─────► GND (Marrón/Negro)

Header 3 pines estándar (pitch 2.54mm)
Compatible: SG90, MG90S, MG996R, etc.
```

### Control con ESP32Servo
```cpp
#include <ESP32Servo.h>

Servo myservo;
#define SERVO_PIN 18

void setup() {
  myservo.setPeriodHertz(50);  // 50Hz estándar
  myservo.attach(SERVO_PIN, 500, 2500);  // min/max µs
}

void loop() {
  // Barrido
  for(int pos=0; pos<=180; pos++) {
    myservo.write(pos);
    delay(15);
  }
  for(int pos=180; pos>=0; pos--) {
    myservo.write(pos);
    delay(15);
  }
  
  // Posiciones específicas
  myservo.write(0);   delay(1000);
  myservo.write(90);  delay(1000);
  myservo.write(180); delay(1000);
}
```

### Control sin librería (PWM directo)
```cpp
#define SERVO_PIN 18
#define PWM_CH 4

void setup() {
  ledcAttachPin(SERVO_PIN, PWM_CH);
  ledcSetup(PWM_CH, 50, 16);  // 50Hz, 16-bit (0-65535)
}

void servoWrite(int angle) {  // 0-180
  // 0.5ms = 0°, 2.5ms = 180° @ 50Hz (periodo 20ms)
  // duty = (pulse_ms / 20ms) * 65535
  int pulse_us = map(angle, 0, 180, 500, 2500);
  uint32_t duty = (pulse_us * 65535) / 20000;
  ledcWrite(PWM_CH, duty);
}
```

### Múltiples servos
```cpp
// ESP32 soporta hasta 16 canales PWM independientes
Servo servo[4];
int pins[4] = {18, 19, 21, 22};  // GPIOs disponibles

void setup() {
  for(int i=0; i<4; i++) {
    servo[i].setPeriodHertz(50);
    servo[i].attach(pins[i]);
  }
}
```

---

## Buzzer
Ver [Buzzer](../perifericos/buzzer.md) - GPIO25

---

## LEDs
Ver [LEDs](../perifericos/leds.md) - GPIO0,2,4,5 + RGB en GPIO2

---

## Resumen pines actuadores

| Función | GPIO | PWM Channel | Notas |
|---------|------|-------------|-------|
| Motor A IN1 | 12 | 0 | PWM 20kHz |
| Motor A IN2 | 13 | 1 | PWM 20kHz |
| Motor B IN3 | 14 | 2 | PWM 20kHz |
| Motor B IN4 | 15 | 3 | PWM 20kHz |
| Servo | 18 | 4 | 50Hz |
| Buzzer | 25 | 5 | tone()/PWM |
| LED D1 | 0 | 6 | PWM opcional |
| LED D2 | 2 | 7 | Compartido RGB |
| LED D3 | 4 | 8 | PWM opcional |
| LED D4 | 5 | 9 | PWM opcional |
| RGB | 2 | - | FastLED/NeoPixel (GPIO bit-bang) |

> **Total canales PWM usados**: 10 de 16 disponibles (ESP32)

---

## Prácticas sugeridas

| Práctica | Actuadores | Nivel |
|----------|------------|-------|
| **06-Motor Control** | Motor DC velocidad/dirección | Intermediate |
| **10-Servo** | Servo barrido/posiciones | Intermediate |
| **07-Buzzer** | Buzzer melodías | Beginner |
| **12-Communication** | Motor + Servo coordinados | Advanced |
| **Robot Chasis** | 2 Motores diferencial + Servo sensor | Advanced |