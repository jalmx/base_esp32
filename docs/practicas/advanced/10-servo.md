# Práctica 10 - Control Servo

## Objetivo
Control preciso de servo: posiciones, barridos, sincronismo con potenciómetro, secuencias grabadas.

## Nivel
**Advanced** | Duración: ~3 horas

## Hardware
- Servo estándar 5V (SG90, MG90S, MG996R)
- Header 3 pines J_SERVO: Signal(GPIO18), VCC(+5V), GND
- **Alimentar desde +5V Rail** (no 3.3V)
- Potenciómetro GPIO34 para control manual

## Librería ESP32Servo
```bash
# Arduino Library Manager → "ESP32Servo" by Kevin Harrington
#include <ESP32Servo.h>
```

## Control básico
```cpp
#include <ESP32Servo.h>

Servo myservo;
#define SERVO_PIN 18

void setup() {
  myservo.setPeriodHertz(50);      // 50Hz estándar
  myservo.attach(SERVO_PIN, 500, 2500);  // min/max µs
}

void loop() {
  // 0° a 180°
  for(int pos=0; pos<=180; pos++) {
    myservo.write(pos);
    delay(15);
  }
  for(int pos=180; pos>=0; pos--) {
    myservo.write(pos);
    delay(15);
  }
}
```

## Control por microsegundos (precisión)
```cpp
void servoWriteUs(int us) {  // 500-2500 µs
  myservo.writeMicroseconds(us);
}

// Mapear ángulo a µs (calibrar por servo)
int angleToUs(int angle) {
  // SG90 típico: 500-2400
  return map(angle, 0, 180, 500, 2400);
}
```

## Control por Potenciómetro
```cpp
#define POT_PIN 34

void loop() {
  int pot = analogRead(POT_PIN);        // 0-4095
  int angle = map(pot, 0, 4095, 0, 180);
  myservo.write(angle);
  
  Serial.printf("Pot: %d  Angle: %d\n", pot, angle);
  delay(20);
}
```

## Múltiples Servos (hasta 16)
```cpp
Servo servos[4];
int pins[4] = {18, 19, 21, 22};

void setup() {
  for(int i=0; i<4; i++) {
    servos[i].setPeriodHertz(50);
    servos[i].attach(pins[i]);
  }
}

void setAll(int angle) {
  for(int i=0; i<4; i++) servos[i].write(angle);
}

void wave() {
  for(int i=0; i<4; i++) {
    servos[i].write(90);
    delay(100);
  }
  for(int i=3; i>=0; i--) {
    servos[i].write(0);
    delay(100);
  }
}
```

## Secuencia grabada (memorizar posiciones)
```cpp
#define MAX_POS 50
int positions[MAX_POS];
int posCount = 0;
bool recording = false;

void recordPosition() {
  if(posCount < MAX_POS) {
    positions[posCount++] = myservo.read();
    Serial.printf("Grabado: %d\n", positions[posCount-1]);
  }
}

void playSequence() {
  for(int i=0; i<posCount; i++) {
    myservo.write(positions[i]);
    delay(500);
  }
}

// Botón para grabar (GPIO25 - compartir con buzzer)
void checkRecordButton() {
  static bool lastBtn = HIGH;
  bool btn = digitalRead(25);
  if(lastBtn == HIGH && btn == LOW) {
    recording = !recording;
    if(recording) { posCount = 0; Serial.println("REC ON"); }
    else { Serial.println("REC OFF - Posiciones: " + String(posCount)); }
  }
  lastBtn = btn;
}
```

## Código completo
```cpp
/*
  Servo - Control avanzado
  Base ESP32 DevKit V4
*/

#include <ESP32Servo.h>

#define SERVO_PIN 18
#define POT_PIN 34
#define BTN_PIN 25

Servo servo;
int positions[50];
int posCount = 0;
bool recording = false;

void setup() {
  Serial.begin(115200);
  pinMode(BTN_PIN, INPUT_PULLUP);
  pinMode(POT_PIN, INPUT);
  
  servo.setPeriodHertz(50);
  servo.attach(SERVO_PIN, 500, 2500);
  
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
  
  Serial.println("Servo Control");
  Serial.println("POT: Control manual");
  Serial.println("BTN: Toggle REC");
}

void loop() {
  // Botón grabar
  static bool lastBtn = HIGH;
  bool btn = digitalRead(BTN_PIN);
  if(lastBtn == HIGH && btn == LOW) {
    recording = !recording;
    if(recording) { posCount = 0; Serial.println("● REC"); }
    else { Serial.println("■ STOP - " + String(posCount) + " pos"); }
    delay(200);  // debounce simple
  }
  lastBtn = btn;
  
  if(recording) {
    // Grabar posición actual cada 200ms
    static uint32_t lastRec = 0;
    if(millis() - lastRec > 200) {
      lastRec = millis();
      recordPosition();
    }
  } else if(posCount > 0) {
    // Reproducir secuencia
    playSequence();
    posCount = 0;  // Solo una vez
  } else {
    // Control por potenciómetro
    int pot = analogRead(POT_PIN);
    int angle = map(pot, 0, 4095, 0, 180);
    servo.write(angle);
    
    static uint32_t lastPrint = 0;
    if(millis() - lastPrint > 200) {
      lastPrint = millis();
      Serial.printf("Angle: %d\n", angle);
    }
  }
  
  delay(10);
}

void recordPosition() {
  if(posCount < 50) {
    positions[posCount++] = servo.read();
    Serial.printf("Pos[%d] = %d°\n", posCount-1, positions[posCount-1]);
  }
}

void playSequence() {
  Serial.println("▶ PLAY");
  for(int i=0; i<posCount; i++) {
    servo.write(positions[i]);
    delay(500);
  }
  Serial.println("■ END");
}
```

## Verificación
1. Conectar servo a J_SERVO
2. Subir código
3. **Potenciómetro mueve servo 0-180°** ✅
3. **Presionar botón** → Graba posiciones cada 200ms ✅
4. **Soltar botón** → Reproduce secuencia ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Añadir LED indicador REC (parpadea grabando) |
| 🟢 Fácil | Velocidad variable: 2º potenciómetro (externo) |
| 🟡 Medio | Easing: Movimiento suave (ease-in/out) |
| 🟡 Medio | Brazo robot 2DOF: 2 servos coordinados |
| 🔴 Difícil | Inverso cinemática: XY → ángulos (brazo 2 links) |
| 🔴 Difícil | Seguimiento objeto: Cámara + servo pan/tilt |

### Reto: Easing (suavizado)
```cpp
float easeInOut(float t) {  // t: 0-1
  return t < 0.5 ? 2*t*t : -1 + (4-2*t)*t;
}

void moveServoSmooth(int from, int to, int duration) {
  uint32_t start = millis();
  while(millis() - start < duration) {
    float t = (millis() - start) / (float)duration;
    float eased = easeInOut(t);
    int pos = from + (to - from) * eased;
    servo.write(pos);
    delay(10);
  }
  servo.write(to);
}
```

## Consideraciones potencia
- **Servo consume 500mA+** en arranque/stall
- **Capacitor 100-470µF** cerca conector (recomendado v0.5)
- **No mover servo + WiFi TX simultáneo** → Brownout riesgo
- **Máx 2-3 servos** desde +5V Rail (LM7805 1.5A límite)

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| Servo vibra/ruido | PWM frecuencia / resolución | `setPeriodHertz(50)`, 16-bit |
| No alcanza 0°/180° | Min/max µs incorrectos | Ajustar `attach(pin, min, max)` |
| Se reinicia al mover | Pico corriente > LM7805 | Cap 470µF en +5V, menos servos |
| Jitter posición | Ruido alimentación / ADC | Cap 100nF servo, promediar POT |
| Conflicto botón | GPIO25 compartido | No usar buzzer simultáneo |

## Referencias
- [ESP32Servo Library](https://github.com/madhephaestus/ESP32Servo)
- [SG90 Datasheet](https://www.electronicoscaldas.com/datasheet/SG90.pdf)
- [Servo Theory](https://learn.sparkfun.com/tutorials/servo-motor-control)