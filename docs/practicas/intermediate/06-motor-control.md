# Práctica 06 - Control Motor DC

## Objetivo
Controlar motor DC con MX1508: velocidad (PWM), dirección, freno.

## Nivel
**Intermediate** | Duración: ~2.5 horas

## Hardware
- Motor DC 3-6V (ej: TT Motor amarillo)
- MX1508 (U_MOT) - Dual H-Bridge
- Alimentación: +5V Rail (desde LM7805)
- Terminal Block: MOT_A+, MOT_A-, MOT_B+, MOT_B-
- GPIO12,13 (Motor A) | GPIO14,15 (Motor B)

## Teoría MX1508
| IN1 | IN2 | OUT1 | OUT2 | Estado |
|-----|-----|------|------|--------|
| 0 | 0 | Z | Z | **Coast** (libre) |
| 1 | 0 | H | L | **Forward** |
| 0 | 1 | L | H | **Reverse** |
| 1 | 1 | L | L | **Brake** (freno) |

- PWM en IN1/IN2 (o IN3/IN4) controla velocidad
- Frecuencia recomendada: **20 kHz** (fuera rango auditivo)
- Corriente máx: 1.5A continuo, 2.5A pico

## Librería simple (sin dependencias)
```cpp
class MX1508Motor {
  uint8_t in1, in2, ch1, ch2;
  int speed = 0;
  
public:
  MX1508Motor(uint8_t _in1, uint8_t _in2, uint8_t _ch1, uint8_t _ch2)
    : in1(_in1), in2(_in2), ch1(_ch1), ch2(_ch2) {}
  
  void begin(int freq = 20000, int res = 8) {
    ledcSetup(ch1, freq, res);
    ledcSetup(ch2, freq, res);
    ledcAttachPin(in1, ch1);
    ledcAttachPin(in2, ch2);
    stop();
  }
  
  void setSpeed(int s) {  // -255 a 255
    speed = constrain(s, -255, 255);
    if(speed > 0) {
      ledcWrite(ch1, speed);
      ledcWrite(ch2, 0);
    } else if(speed < 0) {
      ledcWrite(ch1, 0);
      ledcWrite(ch2, -speed);
    } else {
      ledcWrite(ch1, 0);
      ledcWrite(ch2, 0);  // Coast
    }
  }
  
  void brake() {
    ledcWrite(ch1, 255);
    ledcWrite(ch2, 255);
  }
  
  void stop() { setSpeed(0); }
  int getSpeed() { return speed; }
};
```

## Código - Motor único
```cpp
/*
  Motor DC - Control PWM
  Base ESP32 DevKit V4
*/

MX1508Motor motorA(12, 13, 0, 1);  // IN1, IN2, CH1, CH2

void setup() {
  Serial.begin(115200);
  motorA.begin();
  Serial.println("Motor A listo");
}

void loop() {
  // Acelerar
  for(int s=0; s<=255; s+=5) {
    motorA.setSpeed(s);
    Serial.printf("Speed: %d\n", s);
    delay(50);
  }
  delay(1000);
  
  // Frenar
  motorA.brake();
  delay(500);
  
  // Reversa
  for(int s=0; s>=-255; s-=5) {
    motorA.setSpeed(s);
    delay(50);
  }
  delay(1000);
  
  motorA.stop();
  delay(2000);
}
```

## Código - Robot diferencial (2 motores)
```cpp
/*
  Robot 2WD - Control diferencial
  Base ESP32 DevKit V4
*/

MX1508Motor leftMotor(12, 13, 0, 1);   // Motor A
MX1508Motor rightMotor(14, 15, 2, 3);  // Motor B

void setup() {
  Serial.begin(115200);
  leftMotor.begin();
  rightMotor.begin();
  Serial.println("Robot 2WD listo");
}

void move(int left, int right) {
  leftMotor.setSpeed(left);
  rightMotor.setSpeed(right);
}

void forward(int s)  { move(s, s); }
void backward(int s) { move(-s, -s); }
void left(int s)     { move(-s, s); }
void right(int s)    { move(s, -s); }
void stopAll()       { move(0, 0); }

void loop() {
  forward(200);  delay(2000);
  stopAll();     delay(500);
  right(150);    delay(1000);  // Giro 90° aprox
  stopAll();     delay(500);
  forward(200);  delay(1000);
  left(150);     delay(1000);
  stopAll();     delay(2000);
}
```

## Control por Potenciómetro
```cpp
#define POT_PIN 34

void loop() {
  int pot = analogRead(POT_PIN);          // 0-4095
  int speed = map(pot, 0, 4095, -255, 255);
  motorA.setSpeed(speed);
  
  Serial.printf("Pot: %d  Speed: %d\n", pot, speed);
  delay(20);
}
```

## Control Serial (Comandos)
```cpp
void loop() {
  if(Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    cmd.trim();
    
    if(cmd.startsWith("M")) {  // M200, M-150
      int speed = cmd.substring(1).toInt();
      motorA.setSpeed(constrain(speed, -255, 255));
      Serial.printf("Speed: %d\n", motorA.getSpeed());
    }
    else if(cmd == "B") motorA.brake();
    else if(cmd == "S") motorA.stop();
    else if(cmd == "?") Serial.println("Comandos: M<speed>, B, S, ?");
  }
}
```

## Verificación
1. Conectar motor a terminal block MOT_A+/-
2. Subir código
3. **Motor gira, cambia velocidad, direcciones** ✅
4. Serial Monitor → Comandos M200, M-100, B, S ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Rampa suave: acelerar/desacelerar con `millis()` no `delay()` |
| 🟡 Medio | Control PID velocidad (necesita encoder - externo) |
| 🟡 Medio | Seguidor línea: 2 sensores IR externos + 2 motores |
| 🟡 Medio | Evita obstáculos: Ultrasonido HC-SR04 + motores |
| 🔴 Difícil | Odometría: integrar encoder (externo) → posición XY |
| 🔴 Difícil | Balanceo: MPU6050 + 2 motores (robot inverso) |

### Reto: Rampa no bloqueante
```cpp
int targetSpeed = 0;
int currentSpeed = 0;
uint32_t lastRamp = 0;
const int RAMP_RATE = 5;  // unidades por ms

void setTargetSpeed(int s) { targetSpeed = constrain(s, -255, 255); }

void updateRamp() {
  if(millis() - lastRamp > 20) {
    lastRamp = millis();
    if(currentSpeed < targetSpeed) currentSpeed += RAMP_RATE;
    else if(currentSpeed > targetSpeed) currentSpeed -= RAMP_RATE;
    motorA.setSpeed(currentSpeed);
  }
}
```

## Medir consumo (A9013)
```cpp
// Ver práctica 09 - Monitoreo Corriente
float current = readCurrent();  // A
float power = current * 5.0;    // W aprox
Serial.printf("Motor: %.2fA  %.2fW\n", current, power);
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| Motor no gira | Alimentación insuficiente | Verificar +5V Rail, LM7805 1.5A |
| Motor vibra/no gira | PWM frecuencia muy alta | Bajear a 5-10kHz |
| Se reinicia al arrancar motor | Pico corriente > LM7805 | Cap 470µF + 100µF en +5V, PTC fuse |
| Un sentido no funciona | Cableado / IN pin | Verificar continuidad GPIO→MX1508 |
| Calienta mucho MX1508 | Corriente > 1.5A continuo | Reducir carga, añadir disipador |

## Referencias
- [MX1508 Datasheet](../datasheets/MX1515H_C5119046.pdf)
- [H-Bridge Theory](https://www.monolithicpower.com/en/what-is-an-h-bridge.html)
- [ESP32 LEDC PWM](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/ledc.html)