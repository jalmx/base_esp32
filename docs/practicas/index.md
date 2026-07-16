# Índice de Prácticas

## Estructura de niveles

| Nivel | Prácticas | Duración estimada | Prerrequisitos |
|-------|-----------|-------------------|----------------|
| **Beginner** | 4 | 2-3 horas cada una | Arduino IDE, C básico |
| **Intermediate** | 4 | 3-4 horas cada una | Beginner completado, PWM, ADC |
| **Advanced** | 4 | 4-6 horas cada una | Intermediate, librerías, timers |

---

## Beginner - Fundamentos

### [01 - Hello World LED](./beginner/01-hello-world.md)
**Objetivo**: Parpadear LED D1 (GPIO0) - Primer programa
- `pinMode`, `digitalWrite`, `delay`
- Estructura `setup()` / `loop()`
- Subir código al ESP32

### [02 - PWM LED](./beginner/02-pwm.md)
**Objetivo**: Controlar brillo LED D3 (GPIO4) con PWM
- `ledcSetup`, `ledcAttachPin`, `ledcWrite`
- Resolución y frecuencia PWM
- Fade in/out suave

### [03 - Lectura Analógica](./beginner/03-analog-read.md)
**Objetivo**: Leer potenciómetro (GPIO34) y mostrar por Serial
- `analogRead`, resolución 12-bit
- Atenuación ADC (`ADC_11db`)
- Conversión a voltaje, mapeo a rangos

### [04 - Interrupciones](./beginner/04-interrupts.md)
**Objetivo**: Detectar pulsación botón (GPIO25) sin polling
- `attachInterrupt`, `IRAM_ATTR`
- Debounce por hardware/software
- `INPUT_PULLUP`, flancos `FALLING`/`RISING`

---

## Intermediate - Periféricos integrados

### [05 - Temperatura](./intermediate/05-temperature.md)
**Objetivo**: Leer DS18B20 (1-Wire) y LM35 (Analógico), comparar
- Librerías `OneWire` + `DallasTemperature`
- ADC ruido y filtrado
- Conversión unidades, calibración

### [06 - Motor DC](./intermediate/06-motor-control.md)
**Objetivo**: Controlar motor DC bidireccional con MX1508
- H-Bridge teoría, tabla de verdad
- PWM 20kHz para motor
- Rampa aceleración, freno, dirección

### [07 - Buzzer](./intermediate/07-buzzer.md)
**Objetivo**: Generar tonos, melodías, alarmas con buzzer
- `tone()` / `noTone()`
- PWM manual con `ledc`
- Array notas, duraciones, melodía completa

### [08 - Switches](./intermediate/08-switches.md)
**Objetivo**: Leer DIP switch (4-bit), SPDT, configurar modos
- Lectura paralela 4 bits → byte
- Máquina de estados por modo
- SPDT 3 posiciones (evitar float)

---

## Advanced - Integración y sistemas

### [09 - Monitoreo Corriente](./advanced/09-current-monitor.md)
**Objetivo**: Medir consumo total con A9013, alertas, logging
- Sensor Hall efecto, calibración offset/sensibilidad
- Filtrado pasa-bajos, detección estados
- Integración energía (mWh), alarma sobrecorriente

### [10 - Servo](./advanced/10-servo.md)
**Objetivo**: Control preciso servo, secuencias, sincronismo
- Librería `ESP32Servo`, PWM 50Hz 16-bit
- Múltiples servos, movimiento coordinado
- Control por potenciómetro, secuencia grabada

### [11 - Fusión de Sensores](./advanced/11-sensor-fusion.md)
**Objetivo**: Combinar temperatura, luz, corriente → decisiones
- Fusión multi-sensor (promedio, votación, Kalman simple)
- Machine state detection (IDLE, ACTIVE, OVERLOAD)
- Data logging circular buffer + timestamp

### [12 - Comunicación](./advanced/12-communication.md)
**Objetivo**: UART, I2C, SPI, WiFi, Bluetooth
- Serial debugging, protocolos serie
- I2C scan, leer sensor externo (BMP280, etc)
- WiFi AP/Station, HTTP server, OTA update

---

## Metodología común

### Estructura cada práctica
```markdown
# Título
**Nivel**: Beginner/Intermediate/Advanced
**Duración**: ~X horas
**Objetivo**: Qué se aprende
**Hardware**: Pines, componentes usados
**Prerrequisitos**: Prácticas previas necesarias

## Teoría (15-30 min)
Conceptos clave explicados brevemente

## Hardware (5 min)
Conexiones, esquemático simplificado

## Código paso a paso (60-120 min)
1. Esqueleto básico
2. Función principal
3. Mejoras iterativas

## Retos / Extensiones
- Fácil: Cambiar parámetros
- Medio: Añadir funcionalidad
- Difícil: Integrar con otra práctica

## Verificación
Cómo saber si funciona correctamente

## Errores comunes
Troubleshooting típico

## Referencias
Links a datasheets, librerías, docs
```

### Entrega sugerida
- Código comentado en `.ino`
- Foto/video funcionando
- Captura Serial Plotter / Terminal
- Respuesta a preguntas de reflexión

---

## Progreso sugerido (curso 12 semanas)

| Semana | Práctica | Tema principal |
|--------|----------|----------------|
| 1 | 01 | Setup, IDE, Blink |
| 2 | 02 | PWM, analogWrite |
| 3 | 03 | ADC, Serial, potenciómetro |
| 4 | 04 | Interrupciones, botones |
| 5 | 05 | Sensores temp (1-Wire + Analog) |
| 6 | 06 | Motor DC, H-Bridge, PWM alto |
| 7 | 07 | Buzzer, tone(), arrays |
| 8 | 08 | Switches, máquinas estado |
| 9 | 09 | Corriente, Hall effect, calibración |
| 10 | 10 | Servo, PWM 50Hz, 16-bit |
| 11 | 11 | Fusión sensores, state machine |
| 12 | 12 | Comunicación, proyecto final |

---

## Recursos por práctica

Cada práctica tiene:
- `codigo/basico.ino` - Versión mínima funcional
- `codigo/completo.ino` - Con mejoras y retos
- `esquematico.pdf` - Conexiones relevantes
- `checklist.md` - Verificación paso a paso

---

## Evaluación sugerida

| Criterio | Peso |
|----------|------|
| Funcionamiento correcto | 40% |
| Código limpio, comentado | 20% |
| Resolución retos propuestos | 20% |
| Documentación / explicación | 10% |
| Creatividad / extensión | 10% |