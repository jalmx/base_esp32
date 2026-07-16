# Ensamblaje - Verificación y Test

## Test automático: `test_all.ino`

Sube este sketch tras ensamblar para verificar todo:

```cpp
/*
  Test All - Verificación completa Base ESP32
  Sube tras ensamblar, abre Serial Monitor 115200
*/

#define LED_D1 0
#define LED_D2 2
#define LED_D3 4
#define LED_D4 5
#define MOT_A_IN1 12
#define MOT_A_IN2 13
#define MOT_B_IN3 14
#define MOT_B_IN4 15
#define BUZZER 25
#define DS18B20_PIN 17
#define SERVO_PIN 18
#define POT_PIN 34
#define LDR_PIN 39
#define LM35_PIN 36
#define DIP1 27
#define DIP2 32
#define DIP3 33
#define DIP4 35
#define BTN_PIN 25
#define SPDT_PIN 26

#include <OneWire.h>
#include <DallasTemperature.h>
#include <ESP32Servo.h>

OneWire oneWire(DS18B20_PIN);
DallasTemperature ds18b20(&oneWire);
DeviceAddress dsAddr;
Servo servo;

int testNum = 0;
int passed = 0, failed = 0;

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  // Pines
  pinMode(LED_D1, OUTPUT); pinMode(LED_D2, OUTPUT);
  pinMode(LED_D3, OUTPUT); pinMode(LED_D4, OUTPUT);
  pinMode(MOT_A_IN1, OUTPUT); pinMode(MOT_A_IN2, OUTPUT);
  pinMode(MOT_B_IN3, OUTPUT); pinMode(MOT_B_IN4, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  pinMode(DIP1, INPUT_PULLUP); pinMode(DIP2, INPUT_PULLUP);
  pinMode(DIP3, INPUT_PULLUP); pinMode(DIP4, INPUT_PULLUP);
  pinMode(BTN_PIN, INPUT_PULLUP);
  pinMode(SPDT_PIN, INPUT_PULLUP);
  
  analogReadResolution(12);
  analogSetAttenuation(ADC_11db);
  
  // DS18B20
  ds18b20.begin();
  ds18b20.getAddress(dsAddr, 0);
  ds18b20.setResolution(dsAddr, 12);
  
  // Servo
  servo.setPeriodHertz(50);
  servo.attach(SERVO_PIN, 500, 2500);
  
  // PWM Motor
  ledcAttachPin(MOT_A_IN1, 0); ledcAttachPin(MOT_A_IN2, 1);
  ledcAttachPin(MOT_B_IN3, 2); ledcAttachPin(MOT_B_IN4, 3);
  for(int i=0;i<4;i++) { ledcSetup(i, 20000, 8); ledcWrite(i, 0); }
  
  Serial.println("\n=== BASE ESP32 - TEST COMPLETO ===");
  Serial.println("Iniciando tests...\n");
  
  runAllTests();
  printSummary();
}

void loop() {
  // Blink LED D1 indicando fin
  digitalWrite(LED_D1, millis()%1000<500);
}

void runTest(String name, bool (*test)()) {
  testNum++;
  Serial.printf("[%02d] %s... ", testNum, name.c_str());
  bool ok = test();
  if(ok) { Serial.println("PASS"); passed++; }
  else { Serial.println("FAIL"); failed++; }
}

void runAllTests() {
  // 1. LEDs
  runTest("LED D1 (GPIO0)", testLED_D1);
  runTest("LED D2 (GPIO2)", testLED_D2);
  runTest("LED D3 (GPIO4)", testLED_D3);
  runTest("LED D4 (GPIO5)", testLED_D4);
  
  // 2. PWM LEDs
  runTest("PWM LED D3", testPWM_D3);
  runTest("PWM LED D4", testPWM_D4);
  
  // 3. Motor Driver
  runTest("Motor A Forward", testMotorA_Fwd);
  runTest("Motor A Reverse", testMotorA_Rev);
  runTest("Motor B Forward", testMotorB_Fwd);
  runTest("Motor B Reverse", testMotorB_Rev);
  
  // 4. Buzzer
  runTest("Buzzer Tone", testBuzzer);
  
  // 5. Sensores Temp
  runTest("DS18B20 Detect", testDS18B20);
  runTest("LM35 Read", testLM35);
  
  // 6. LDR
  runTest("LDR Read", testLDR);
  
  // 7. Potenciómetro
  runTest("Pot Read", testPot);
  
  // 8. Switches
  runTest("DIP Switch Read", testDIP);
  runTest("Push Button", testButton);
  runTest("SPDT Switch", testSPDT);
  
  // 9. Servo
  runTest("Servo Sweep", testServo);
  
  // 10. ADC channels
  runTest("ADC1 Channels", testADC);
  runTest("ADC2 Channels", testADC2);
  
  // 11. I2C Scan
  runTest("I2C Scan", testI2C);
}

void printSummary() {
  Serial.printf("\n=== RESUMEN ===\n");
  Serial.printf("Total: %d  Pass: %d  Fail: %d\n", testNum, passed, failed);
  if(failed == 0) Serial.println(">>> TODO OK <<<");
  else Serial.println(">>> REVISAR FALLOS <<<");
}

// === TESTS INDIVIDUALES ===

bool testLED_D1() {
  digitalWrite(LED_D1, HIGH); delay(50); 
  bool ok = digitalRead(LED_D1) == HIGH;
  digitalWrite(LED_D1, LOW);
  return ok;
}

bool testLED_D2() {
  digitalWrite(LED_D2, HIGH); delay(50);
  bool ok = digitalRead(LED_D2) == HIGH;
  digitalWrite(LED_D2, LOW);
  return ok;
}

bool testLED_D3() {
  digitalWrite(LED_D3, HIGH); delay(50);
  bool ok = digitalRead(LED_D3) == HIGH;
  digitalWrite(LED_D3, LOW);
  return ok;
}

bool testLED_D4() {
  digitalWrite(LED_D4, HIGH); delay(50);
  bool ok = digitalRead(LED_D4) == HIGH;
  digitalWrite(LED_D4, LOW);
  return ok;
}

bool testPWM_D3() {
  ledcAttachPin(LED_D3, 4); ledcSetup(4, 5000, 8);
  ledcWrite(4, 128); delay(50);
  int v = digitalRead(LED_D3);
  ledcWrite(4, 0); ledcDetachPin(LED_D3);
  return v == HIGH;  // PWM alto = HIGH en lectura digital
}

bool testPWM_D4() {
  ledcAttachPin(LED_D4, 5); ledcSetup(5, 5000, 8);
  ledcWrite(5, 128); delay(50);
  int v = digitalRead(LED_D4);
  ledcWrite(5, 0); ledcDetachPin(LED_D4);
  return v == HIGH;
}

bool testMotorA_Fwd() {
  ledcWrite(0, 100); ledcWrite(1, 0); delay(50);
  bool ok = (digitalRead(MOT_A_IN1)==HIGH && digitalRead(MOT_A_IN2)==LOW);
  ledcWrite(0, 0); ledcWrite(1, 0);
  return ok;
}

bool testMotorA_Rev() {
  ledcWrite(0, 0); ledcWrite(1, 100); delay(50);
  bool ok = (digitalRead(MOT_A_IN1)==LOW && digitalRead(MOT_A_IN2)==HIGH);
  ledcWrite(0, 0); ledcWrite(1, 0);
  return ok;
}

bool testMotorB_Fwd() {
  ledcWrite(2, 100); ledcWrite(3, 0); delay(50);
  bool ok = (digitalRead(MOT_B_IN3)==HIGH && digitalRead(MOT_B_IN4)==LOW);
  ledcWrite(2, 0); ledcWrite(3, 0);
  return ok;
}

bool testMotorB_Rev() {
  ledcWrite(2, 0); ledcWrite(3, 100); delay(50);
  bool ok = (digitalRead(MOT_B_IN3)==LOW && digitalRead(MOT_B_IN4)==HIGH);
  ledcWrite(2, 0); ledcWrite(3, 0);
  return ok;
}

bool testBuzzer() {
  tone(BUZZER, 1000, 50); delay(60);
  noTone(BUZZER);
  return true;  // No hay feedback eléctrico fácil
}

bool testDS18B20() {
  ds18b20.requestTemperatures();
  float t = ds18b20.getTempC(dsAddr);
  return (t != DEVICE_DISCONNECTED_C && t > -40 && t < 125);
}

bool testLM35() {
  long sum = 0;
  for(int i=0;i<16;i++) { sum += analogRead(LM35_PIN); delayMicroseconds(100); }
  float v = (sum/16.0) * 3.3 / 4095.0;
  float t = v * 100.0;
  return (t >= 0 && t <= 100);  // Rango razonable
}

bool testLDR() {
  int raw = analogRead(LDR_PIN);
  float v = raw * 3.3 / 4095.0;
  return (v >= 0.1 && v <= 3.2);  // No rail-to-rail
}

bool testPot() {
  int raw = analogRead(POT_PIN);
  return (raw >= 0 && raw <= 4095);
}

bool testDIP() {
  uint8_t v = (!digitalRead(DIP1)<<0) | (!digitalRead(DIP2)<<1) | 
              (!digitalRead(DIP3)<<2) | (!digitalRead(DIP4)<<3);
  return true;  // Solo lectura, siempre OK
}

bool testButton() {
  bool pressed = digitalRead(BTN_PIN) == LOW;
  return true;  // Solo lectura
}

bool testSPDT() {
  bool state = digitalRead(SPDT_PIN);
  return true;
}

bool testServo() {
  servo.write(0); delay(300);
  servo.write(90); delay(300);
  servo.write(180); delay(300);
  servo.write(90);
  return true;
}

bool testADC() {
  // ADC1: GPIO36,39,34,35,32,33
  int pins[] = {36, 39, 34, 35, 32, 33};
  for(int p : pins) {
    int r = analogRead(p);
    if(r < 0 || r > 4095) return false;
  }
  return true;
}

bool testADC2() {
  // ADC2: GPIO0,2,4,12,13,14,15,25,26,27
  // Nota: ADC2 no usable con WiFi activo
  int pins[] = {0, 2, 4, 12, 13, 14, 15, 25, 26, 27};
  for(int p : pins) {
    int r = analogRead(p);
    if(r < 0 || r > 4095) return false;
  }
  return true;
}

bool testI2C() {
  Wire.begin(21, 19);
  byte count = 0;
  for(byte addr=1; addr<127; addr++) {
    Wire.beginTransmission(addr);
    if(Wire.endTransmission() == 0) count++;
  }
  Wire.end();
  Serial.printf("    Dispositivos I2C: %d\n", count);
  return true;  // OK aunque 0
}
```

---

## Verificación con Multímetro

### Antes de encender (PCB sola)
```
□ Continuidad GND: Todos GND pads conectados (beep)
□ Continuidad +5V: Jack DC → LM7805 IN → LM7805 OUT → MX1508 VM → Terminal + → Servo VCC
□ Continuidad +3.3V: AMS1117 OUT → ESP32 3V3 pins → Sensores VCC
□ NO continuidad: +5V ↔ GND, +3.3V ↔ GND, +5V ↔ +3.3V (OL = Open Loop)
□ Resistencia LM7805 IN-GND: >1kΩ (no corto)
□ Resistencia AMS1117 IN-GND: >1kΩ
□ Diodos 1N5813: Diodo test → ~0.3V forward, OL reverse
```

### Alimentado (USB o Jack 9V)
```
□ +5V Rail: 4.90 - 5.10V
□ +3.3V Rail: 3.25 - 3.35V
□ LM7805 Temp: <60°C (tocable)
□ AMS1117 Temp: <50°C
□ MX1508 Temp: <50°C (sin motor)
□ Corriente USB: 80-120mA (reposo)
□ Corriente Jack 9V: 50-100mA (reposo)
```

### Con ESP32 insertado + firmware test_all
```
□ Serial Monitor: 115200 baud, texto legible
□ Test LEDs: 4 parpadeos secuenciales
□ Test PWM: D3/D4 fade in/out
□ Test Motor: Terminal block MOT_A+/-, MOT_B+/- pulsos
□ Test Buzzer: Beep 1kHz 50ms
□ Test Temp: DS18B20 ~20-30°C, LM35 similar
□ Test LDR: Valor cambia tapando/exponiendo
□ Test Pot: Valor 0-4095 girando
□ Test DIP: Cambia valor moviendo switches
□ Test Button: "Button pressed" al pulsar
□ Test SPDT: Cambia estado moviendo
□ Test Servo: Barrido 0-180-0
□ Test I2C: "Dispositivos I2C: 0" (o >0 si hay externo)
□ Resumen final: "TODO OK" (0 FAIL)
```

---

## Debug común

| Síntoma | Causa probable | Verificación |
|---------|----------------|--------------|
| LED power OFF | LM7805 / AMS1117 mal soldado | Voltaje IN/OUT reguladores |
| ESP32 no detectado | Socket pines doblados / mal soldado | Continuidad cada pin socket→pad |
| Motor no gira | MX1508 VM sin alimentación | +5V en pin 8 MX1508 |
| DS18B20 -127°C | Pull-up 4.7k faltante / cableado | 4.7k Data-3.3V, continuidad |
| LM35 0V / 3.3V | ADC saturado / pin mal | analogRead raw, voltaje pin |
| LDR siempre 4095 | LDR abierto / GND faltante | Resistencia LDR multímetro |
| Pot siempre 0/4095 | Cursor roto / patas invertidas | Resistencia entre extremos |
| DIP siempre 1111 | Pull-up faltante GPIO35 | 10kΩ ext GPIO35-3.3V |
| Servo vibra/no mueve | PWM 50Hz mal / alimentación | ledcSetup 50Hz, +5V en servo |
| Buzzer mudo | GPIO25 conflicto / pin mal | tone() en pin correcto |

---

## Firmware de producción sugerido

Tras verificación, flashea tu aplicación real. Ejemplo estructura:

```cpp
// main.cpp
#include "config.h"
#include "wifi_mgr.h"
#include "sensor_mgr.h"
#include "motor_ctrl.h"
#include "web_server.h"

void setup() {
  Serial.begin(115200);
  hw_init();        // Pines, ADC, PWM
  sensors_init();   // DS18B20, LM35, LDR
  motor_init();     // MX1508
  wifi_init();      // STA + AP fallback
  webserver_init(); // API REST + OTA
}

void loop() {
  sensors_update();
  motor_update();
  webserver_handle();
  watchdog_feed();
}
```

Ver [Prácticas](../practicas/index.md) para ejemplos completos.