# Práctica 04 - Interrupciones

## Objetivo
Detectar pulsación del botón (GPIO25) usando interrupción, sin polling.

## Nivel
**Beginner** | Duración: ~2 horas

## Hardware
- Push Button SW_PUSH → GPIO25
- Pull-up interno activado
- **¡Comparte GPIO25 con BUZZER!** No usar ambos a la vez.

## Teoría
- **Polling**: Leer pin en loop() constantemente - gasta CPU
- **Interrupción**: Hardware avisa a CPU cuando pin cambia - eficiente
- **ISR** (Interrupt Service Routine): Función que se ejecuta al dispararse
- **IRAM_ATTR**: Atributo para poner ISR en RAM (rápido, obligatorio en ESP32)
- **Flancos**: `RISING` (LOW→HIGH), `FALLING` (HIGH→LOW), `CHANGE` (ambos)
- **Debounce**: Rebote mecánico del botón (ms) - filtrar por software o hardware

## Código paso a paso

### 1. Configurar interrupción
```cpp
#define BTN_PIN 25

volatile bool buttonPressed = false;  // volatile = visible desde ISR
volatile uint32_t lastPress = 0;

void IRAM_ATTR buttonISR() {
  uint32_t now = millis();
  if(now - lastPress > 50) {  // Debounce 50ms
    buttonPressed = true;
    lastPress = now;
  }
}

void setup() {
  pinMode(BTN_PIN, INPUT_PULLUP);
  attachInterrupt(BTN_PIN, buttonISR, FALLING);
  Serial.begin(115200);
}
```

### 2. Loop principal
```cpp
void loop() {
  if(buttonPressed) {
    buttonPressed = false;  // Limpiar flag
    Serial.println("¡Botón presionado!");
    // Acción: toggle LED, cambiar modo, etc.
  }
  
  // Resto del código no bloqueado
}
```

### 3. Código completo con LED
```cpp
/*
  Interrupciones - Botón con debounce
  Base ESP32 DevKit V4
*/

#define BTN_PIN 25
#define LED_PIN 4  // LED D3

volatile bool buttonFlag = false;
volatile uint32_t lastPress = 0;
bool ledState = false;

void IRAM_ATTR buttonISR() {
  uint32_t now = millis();
  if(now - lastPress > 50) {
    buttonFlag = true;
    lastPress = now;
  }
}

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BTN_PIN, INPUT_PULLUP);
  attachInterrupt(BTN_PIN, buttonISR, FALLING);
  Serial.begin(115200);
  Serial.println("Interrupción botón lista");
}

void loop() {
  if(buttonFlag) {
    buttonFlag = false;
    ledState = !ledState;
    digitalWrite(LED_PIN, ledState);
    Serial.printf("Botón -> LED %s\n", ledState ? "ON" : "OFF");
  }
  
  // Simular otra tarea
  static uint32_t lastBlink = 0;
  if(millis() - lastBlink > 2000) {
    lastBlink = millis();
    Serial.println("Loop vivo...");
  }
}
```

## Verificación
1. Subir código
2. Serial Monitor 115200
3. **Presionar botón** → "¡Botón presionado!" + LED D3 toggle ✅
4. "Loop vivo..." cada 2s confirma no bloqueo ✅

## Retos

| Nivel | Reto |
|-------|------|
| 🟢 Fácil | Contar pulsaciones, mostrar contador |
| 🟢 Fácil | Diferenciar pulsación corta vs larga (>1s) |
| 🟡 Medio | Máquina de estados: 1 click=LED1, 2 clicks=LED2, largo=apagar todo |
| 🟡 Medio | Usar interrupción para wake-up desde Deep Sleep |
| 🔴 Difícil | Rotary encoder (2 pines interrupción) - detectar dirección |

### Reto: Click corto vs largo
```cpp
volatile uint32_t pressTime = 0;
volatile bool pressed = false;

void IRAM_ATTR btnISR() {
  bool state = digitalRead(BTN_PIN);
  if(!state) {  // FALLING - press
    pressTime = millis();
    pressed = true;
  } else if(pressed) {  // RISING - release
    uint32_t duration = millis() - pressTime;
    if(duration < 500) shortPressFlag = true;
    else longPressFlag = true;
    pressed = false;
  }
}

// En setup: attachInterrupt(BTN_PIN, btnISR, CHANGE);
```

### Reto: Wake from Deep Sleep
```cpp
#define BTN_PIN 25

void setup() {
  Serial.begin(115200);
  pinMode(BTN_PIN, INPUT_PULLUP);
  
  // Configurar wake-up por GPIO25 LOW
  esp_sleep_enable_ext0_wakeup(GPIO_NUM_25, 0);
  
  Serial.println("Entrando en deep sleep...");
  Serial.println("Presiona botón para despertar");
  delay(100);
  esp_deep_sleep_start();
}

void loop() {}  // Nunca llega aquí
```

## Errores comunes
| Error | Causa | Solución |
|-------|-------|----------|
| ISR no se ejecuta | Falta `IRAM_ATTR` | Añadir `void IRAM_ATTR isr()` |
| Reinicio aleatorio (Guru Meditation) | ISR muy larga / Serial.print en ISR | ISR mínima, solo flags |
| Múltiples disparos | Rebote (debounce) | `millis()` diff > 50ms en ISR |
| Variable no cambia en loop | Falta `volatile` | `volatile bool flag` |
| `attachInterrupt` error | Pin no soporta interrupción | GPIO0,2,4,12-15,25-27,32-39 OK |
| Conflicto buzzer | GPIO25 compartido | No usar buzzer y botón simultáneo |

## Referencias
- [ESP32 Interrupts](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/intr_alloc.html)
- [Arduino attachInterrupt](https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/)
- [Deep Sleep Wakeup](https://randomnerdtutorials.com/esp32-deep-sleep-arduino-ide-wake-up-sources/)