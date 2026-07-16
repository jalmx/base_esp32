# Visión general - Alimentación

## Árbol de potencia (Power Tree)

```
                    ┌─────────────────┐
                    │   ENTRADAS      │
                    │  ┌───────────┐  │
                    │  │ Jack DC   │  │  7-12V DC
                    │  │ 2.1mm     │──┤
                    │  └───────────┘  │
                    │        │        │
                    │  ┌───────────┐  │
                    │  │ USB Micro │  │  5V USB
                    │  │    -B     │──┤
                    │  └───────────┘  │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Diodo ORing    │  1N5819 Schottky
                    │  (Protección    │  Evita backfeed
                    │   polaridad)    │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   LM7805CT      │  Regulador lineal 5V
                    │   TO-220        │  1.5A max
                    │  + Disipador    │  Dropout ~2V
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
       ┌──────▼──────┐ ┌─────▼─────┐ ┌─────▼─────┐
       │  +5V Rail   │ │  A9013    │ │  MX1508   │
       │  (Main)     │ │  Current  │ │  VMotor   │
       │             │ │  Sense    │ │           │
       └──────┬──────┘ └─────┬─────┘ └─────┬─────┘
              │              │             │
       ┌──────▼──────┐       │       ┌─────▼─────┐
       │ AMS1117-3.3 │       │       │ Terminal  │
       │  3.3V 800mA │       │       │  Block    │
       └──────┬──────┘       │       │  (Motor   │
              │              │       │  Power)   │
       ┌──────▼──────┐       │       └───────────┘
       │  +3.3V Rail │       │
       │  (Logic)    │       │
       └─────────────┘       │
                             │
                    ┌────────▼────────┐
                    │     SERVO       │  5V directo
                    │   Connector     │  (desde +5V Rail)
                    └─────────────────┘
```

## Rails de alimentación

| Rail | Voltaje | Corriente max | Origen | Consumidores |
|------|---------|---------------|--------|--------------|
| **VIN** | 7-12V | 1.5A | Jack/USB | LM7805 input |
| **+5V** | 5.0V | 1.5A | LM7805 | MX1508, Servo, Terminal, A9013, LED 5V |
| **+3.3V** | 3.3V | 800mA | AMS1117 | ESP32, Sensores, LEDs 3.3V, Pull-ups |
| **VMOTOR** | 5.0V | 2A | +5V Rail | MX1508 VM pin (separado por track ancho) |

## Esquema de protecciones

```
VIN (7-12V)
    │
    ├─► D1 (1N5819) ──► LM7805 ──► +5V
    │                    │
    │                    ├─► C_bulk 100µF
    │                    ├─► C_dec 100nF
    │                    ├─► LED_5V + 1kΩ
    │                    ├─► A9013 VCC
    │                    ├─► MX1508 VM
    │                    ├─► Servo VCC
    │                    └─► Terminal Block
    │
    └─► D2 (1N5819) ──► USB 5V ──► (Diodo ORing con Jack)
```

## Cálculos térmicos

### LM7805 (TO-220)

| Parámetro | Valor |
|-----------|-------|
| V_in max | 12V |
| V_out | 5V |
| I_out max | 1.5A (datasheet) |
| I_typical | 500mA (ESP32 + sensores + LEDs) |
| P_dis = (Vin-Vout)×I | (12-5)×0.5 = **3.5W** |
| θ_JA (sin heatsink) | ~50°C/W |
| ΔT (sin heatsink) | 3.5×50 = **175°C** ❌ |
| θ_JA (con heatsink 10°C/W) | 3.5×10 = **35°C** ✅ |
| θ_JA (4 vias térmicas + pour) | ~25°C/W = **87.5°C** ⚠️ |

**Recomendación**: Usar disipador pequeño o limitar Vin a 9V max para uso continuo.

### AMS1117-3.3 (SOT-223)

| Parámetro | Valor |
|-----------|-------|
| V_in | 5V |
| V_out | 3.3V |
| I_out max | 800mA |
| I_typical | 200mA (ESP32 TX + sensores) |
| P_dis | (5-3.3)×0.2 = **0.34W** |
| θ_JA (SOT-223 + vias) | ~40°C/W |
| ΔT | 0.34×40 = **13.6°C** ✅ |

### MX1508

- Pad térmico expuesto soldado a polygon VCC_MOTOR + vias a GND
- Corriente motor típica: 500mA-1A
- Protección térmica interna (150°C shutdown)

## Secuencia de encendido

```
1. VIN aplicado (Jack o USB)
         │
         ▼
2. Diodo ORing conduce
         │
         ▼
3. LM7805 enciende → +5V sube
         │
         ├─► LED 5V enciende
         ├─► A9013 alimentado
         ├─► MX1508 VM alimentado
         └─► Servo VCC alimentado
         │
         ▼
4. AMS1117 enciende → +3.3V sube
         │
         ├─► LED 3.3V enciende
         ├─► ESP32 sale de reset (EN HIGH)
         └─► Sensores alimentados
         │
         ▼
5. ESP32 boot → firmware inicia
```

## Puntos de test (Test Points)

| TP | Net | Ubicación | Uso |
|----|-----|-----------|-----|
| TP1 | VIN | Cerca Jack | Verificar entrada |
| TP2 | +5V | Cerca LM7805 out | Verificar 5V |
| TP3 | +3.3V | Cerca AMS1117 out | Verificar 3.3V |
| TP4 | GND | Múltiples | Referencia |
| TP5 | IMON_OUT | Cerca A9013 | Medir corriente |
| TP6 | VMOTOR | Cerca MX1508 | Verificar motor power |

## Medidas típicas (multímetro)

| Punto | Esperado | Tolerancia |
|-------|----------|------------|
| Jack DC | 7-12V | ±5% |
| USB 5V | 4.75-5.25V | ±5% |
| +5V Rail | 4.9-5.1V | ±2% |
| +3.3V Rail | 3.25-3.35V | ±1.5% |
| Dropout LM7805 | ~2V | - |
| Dropout AMS1117 | ~1.1V | - |
| Corriente reposo (+5V) | ~50mA | - |
| Corriente reposo (+3.3V) | ~80mA | - |
| Corriente WiFi TX (+3.3V) | +150mA pico | - |

## Mejoras planificadas (v0.5)

Ver [Cambios planificados](../contribuir/guia.md#cambios-planificados)

- [ ] Añadir PTC/Polyfuse en entrada VIN
- [ ] TVS diodo en entrada (protección ESD/surge)
- [ ] LED indicador "Power Good" (AND de 5V y 3.3V)
- [ ] Jumper medición corriente serie en +5V/+3.3V