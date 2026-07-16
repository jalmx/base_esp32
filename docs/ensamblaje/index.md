# Ensamblaje - Visión general

## Herramientas necesarias

| Herramienta | Especificación | Imprescindible |
|-------------|----------------|----------------|
| **Soldador** | Punta fina 0.5-1mm, 40-60W control temp | ✅ |
| **Estaño** | Sn63/Pb37 o SAC305, 0.5-0.8mm, flux core | ✅ |
| **Flux** | Pasta o líquido (no-corrosivo) | ✅ |
| **Pinzas** | Puntas finas ESD-safe | ✅ |
| **Lupa/Visor** | 5-10x o microscopio USB | ✅ |
| **Multímetro** | Continuidad, voltaje, resistencia | ✅ |
| **Desoldador** | Bomba o malla desoldar | Recomendado |
| **Alicates corte** | Micro-corte flush | Recomendado |
| **Third hand** | Con lupa integrada | Útil |

## Componentes - Qué necesitas

Ver [Lista completa (BOM)](../hardware/componentes.md) con referencias y proveedores.

**Kits por nivel:**
| Nivel | Incluye | Coste aprox |
|-------|---------|-------------|
| **Básico** | Solo pasivos + headers + reguladores | ~$8 |
| **Completo** | Básico + sensores + actuadores + ESP32 | ~$25 |
| **Full** | Completo + MX1508 + WS2812B + servo | ~$30 |

## Orden de soldadura recomendado

```
1. Componentes SMD pequeños (0805, 0603, SOD-123, SOT-223)
   ├─ Resistencias 0805/0603
   ├─ Capacitores 0805
   ├─ Diodos SOD-123 (1N5819)
   └─ ICs SMD: AMS1117 (SOT-223), MX1508 (SOP-8)

2. Componentes THT bajos
   ├─ Headers hembra 2x19 (socket ESP32)
   ├─ Headers macho 2x19, 3p, 4p, 6p, 10p
   ├─ Terminal block 4p
   ├─ Jack DC 2.1mm
   ├─ USB Micro-B
   ├─ Switches DIP, Push, SPDT
   ├─ Potenciómetro 10k
   ├─ Buzzer
   ├─ LEDs 3mm (×4)
   └─ DS18B20 (TO-92) / LM35 (TO-92)

3. Componentes THT altos / gruesos
   ├─ LM7805 TO-220 (con disipador)
   ├─ Capacitores electrolíticos (470µF, 100µF)
   └─ Conector servo 3p

4. Módulo ESP32 DevKit V4
   └─ Insertar en socket hembra (ÚLTIMO)

5. Verificación y test
```

## Técnicas de soldadura

### SMD (0805, 0603, SOT-223, SOP-8)
1. **Flux** en pads
2. **Estañar un pad** (esquina)
3. **Posicionar componente** con pinzas
4. **Soldar pin esquina** (sujeta componente)
4. **Soldar resto pines** (arrastrar estaño)
5. **Inspeccionar** lupa: sin puentes, buena mojado

### THT (Through-Hole)
1. **Insertar componente** (doblar patillas si necesario)
2. **Voltear PCB**, apoyar en third-hand
3. **Calentar pad + pata** 2-3s
4. **Aplicar estaño** en unión (no en punta)
5. **Retirar estaño, luego soldador**
6. **Recortar patillas** flush cutters

### ICs sensibles (ESD)
- **Pulsera antiestática** o tocar metal antes
- **No tocar pines** directamente
- **Socket hembra** para ESP32 (no soldar directo)

## Puntos críticos PCB v0.4

| Componente | Cuidado |
|------------|---------|
| **MX1508 (U_MOT)** | Pad térmico expuesto → Soldar a polygon VCC_MOTOR + vias a GND |
| **AMS1117 (U_3V3)** | SOT-223 - Pin 2 (GND/Tab) soldar bien a GND pour |
| **LM7805 (U_5V)** | TO-220 - Disipador + pasta térmica + 4 vias térmicas |
| **Socket ESP32 (J1, J2)** | 2x19 pines - Alinear perfecto antes de soldar |
| **A9013 (U_IMON)** | SIP-3 - Orientación correcta (marca en silkscreen) |
| **WS2812B (LED_RGB)** | 5050 SMD - Orientación (esquina cortada = GND) |
| **DIP Switch** | 4 posiciones - Pin 1 marcado en silkscreen |

## Verificación post-soldadura

### 1. Inspección visual (Lupa 10x)
- [ ] Sin puentes de estaño (shorts)
- [ ] Sin soldaduras frías (grisáceas, rugosas)
- [ ] Todos los pines mojados (brillante, cóncavo)
- [ ] Componentes polarizados correctos (LED, Diode, Electro, IC)
- [ ] Sin patillas sueltas / components tombstoning

### 2. Continuidad / Cortos (Multímetro)
```
☐ GND ↔ +5V Rail (debe ser OL / >1MΩ)
☐ GND ↔ +3.3V Rail (debe ser OL)
☐ +5V ↔ +3.3V (debe ser OL)
☐ GND pins headers ↔ GND plane (0Ω)
☐ VIN Jack → LM7805 IN (0Ω)
☐ LM7805 OUT → +5V Rail (0Ω)
☐ AMS1117 OUT → +3.3V Rail (0Ω)
☐ ESP32 Socket GND pins → GND (0Ω)
☐ ESP32 Socket 3V3 pins → +3.3V (0Ω)
```

### 3. Pruebas alimentación (SIN ESP32 insertado)
```
☐ Conectar Jack 9-12V → LED 5V ON, LED 3.3V ON
☐ Medir +5V Rail: 4.9-5.1V
☐ Medir +3.3V Rail: 3.25-3.35V
☐ Conectar USB 5V (sin Jack) → LEDs ON
☐ Medir corrientes reposo: +5V < 50mA, +3.3V < 80mA
```

### 4. Test con ESP32
```
☐ Insertar ESP32 DevKit V4 en socket (orientación: USB hacia borde)
☐ Conectar USB a PC → PC detecta COM port
☐ Subir sketch "Blink" → LED D1 (GPIO0) parpadea
☐ Serial Monitor 115200 → "Hello ESP32"
☐ Test WiFi: Scan networks → Lista redes
```

### 5. Test periféricos (sketch test_all.ino)
```
☐ LED D1-D4: Blink sequence
☐ RGB: Rainbow cycle
☐ Buzzer: Tone sweep
☐ Botón: Serial "BTN Press"
☐ DIP: Leer 4 bits
☐ SPDT: Leer posición
☐ Potenciómetro: 0-4095 serial
☐ DS18B20: Temp °C
☐ LM35: Temp °C
☐ LDR: Valor luz
☐ Motor A/B: Forward/Reverse PWM
☐ Servo: Sweep 0-180
☐ A9013: Corriente reposo ~0A
```

## Solución problemas comunes

| Síntoma | Causa probable | Solución |
|---------|----------------|----------|
| LED 5V no enciende | LM7805 mal soldado / sin Vin | Revisar continuidad Jack→LM7805 IN, LM7805 OUT→+5V |
| LED 3.3V no enciende | AMS1117 mal soldado | Revisar pin 2 (Tab/GND), pin 3 (OUT) |
| ESP32 no detectado | Socket mal soldado / pines doblados | Revisar continuidad socket→pads, enderezar pines |
| Motor no gira | MX1508 pad térmico no soldado | Soldar pad central a polygon, verificar VM/VCC |
| Lecturas ADC ruidosas | Falta capacitor decoupling | Verificar 100nF cerca cada IC, 100nF en ADC inputs |
| Buzzer no suena | GPIO25 conflict / pin wrong | Verificar GPIO25 no usado por otra cosa |
| Servo vibra/no mueve | Alimentación 3.3V / PWM wrong | Verificar VCC=5V, PIN=18, 50Hz |

## Tips pro

1. **Flux is your friend** - Usa flux generosamente en SMD
2. **Solda todo SMD primero** - Antes de THT (acceso fácil)
3. **Socket ESP32** - Nunca sueldes el DevKit directo
4. **LM7805** - Montar con disipador ANTES de soldar (tornillo M3)
5. **Fotos** - Toma fotos a cada paso para referencia futura
6. **Test incremental** - Verifica cada rail antes de seguir