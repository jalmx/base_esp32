# Ensamblaje - Paso a paso

## Sesión 1: SMD (30-45 min)

### 1.1 Preparación
- [ ] PCB limpia (alcohol IPA)
- [ ] Flux aplicado en pads SMD
- [ ] Componentes SMD organizados en cinta/bandeja
- [ ] Soldador 350°C (leaded) / 380°C (lead-free)

### 1.2 Resistencias 0805/0603
```
Orden sugerido (por zona PCB):
1. R_LED_USR (x4) - 330Ω cerca LEDs
2. R_LED_PWR (x2) - 1kΩ cerca LEDs power
3. R_IMON - 10kΩ cerca A9013
4. R_PU_DIP (x4) - 10kΩ cerca DIP switch
5. R_LDR - 10kΩ cerca LDR
6. R_DS18B20 - 4.7kΩ cerca DS18B20
7. Resto 0805 (decoupling cerca ICs)
```

### 1.3 Capacitores 0805
```
1. C_DEC_* (100nF) - Cerca de cada IC (prioridad)
2. C_MOT_VCC, C_RGB, C_LDR, C_IMON
3. Resto decoupling
```

### 1.4 Diodos SOD-123 (1N5819 x3)
```
Marcas en silkscreen: cátodo = raya
D_PWR: Jack → LM7805 (orientación: raya hacia LM7805)
D_PROT: USB → ORing (raya hacia ORing)
D_DISC: LM7805 OUT → IN (raya hacia IN)
```

### 1.5 ICs SMD
```
1. U_3V3 (AMS1117 SOT-223)
   - Pin 2 (Tab grande) = GND → Soldar bien a GND pour
   
2. U_MOT (MX1508 SOP-8)
   - Pad central térmico → Soldar a VCC_MOTOR polygon + vias GND
   - Pin 1 = marca punto/muesca
   
3. U_IMON (A9013 SIP-3)
   - 3 pines en línea
   - Marca en silkscreen indica pin 1 (VCC)
```

### 1.6 LED RGB WS2812B (5050)
```
- Esquina cortada = GND (pin 4)
- Orientar según silkscreen (marca esquina)
- Flux generoso en 4 pads
- Soldar 2 esquinas opuestas primero
```

---

## Sesión 2: THT Bajo perfil (45-60 min)

### 2.1 Socket ESP32 (J1, J2 - 2x19 hembra)
```
CRÍTICO: Alineación perfecta

1. Insertar socket en breadboard (2 breadboards paralelos)
2. Colocar PCB encima (pads atraviesan socket)
3. Presionar firmemente - socket plano contra PCB
4. Soldar 1 pin esquina → Verificar perpendicular
5. Soldar pin opuesto → Verificar
6. Soldar resto (38 pines total)
7. Continuidad: cada pin socket → pad PCB
```

### 2.2 Headers macho (2x19, 3p, 4p, 6p, 10p)
```
Mismo truco: Header en breadboard → PCB encima → Soldar
- J_ESP32_A (2x19) - ya en socket hembra
- J_ESP32_B (2x19) - ya en socket hembra  
- J_SERVO (3p)
- J_I2C (4p)
- J_SPI (6p)
- J_GPIO (10p)
```

### 2.3 Switches
```
SW_DIP (4 pos): Pin 1 marcado → hacia GPIO27
SW_PUSH (6x6): 4 patas, 2 pares internos
SW_SPDT: 3 patas, pin central = COM
```

### 2.4 Potenciómetro
```
RV_POT (B10K): 3 patas, cuerpo redondo
- Pata central = wiper → GPIO34
- Laterales = 3.3V y GND
```

### 2.5 Buzzer
```
BZ1: 2 patas, sin polaridad
- Pata + hacia GPIO25 (marca + en silkscreen)
```

### 2.6 LEDs 3mm (D1-D4)
```
Polaridad: Pata larga = Ánodo (+) → GPIO
           Pata corta / lado plano = Cátodo (-) → GND via R
D1 (Rojo) → GPIO0
D2 (Azul) → GPIO2  
D3 (Verde) → GPIO4
D4 (Amarillo) → GPIO5
```

---

## Sesión 3: THT Alto / Potencia (30 min)

### 3.1 LM7805 TO-220 (U_5V)
```
1. Montar disipador ANTES de soldar
   - Tornillo M3 + tuerca + pasta térmica
   - Disipador hacia arriba (fuera PCB)
   
2. Insertar LM7805 (patilla central = GND = Tab)
3. Soldar 3 patas
4. Verificar: Tab → GND plane (continuidad)
```

### 3.2 Capacitores electrolíticos
```
C_IN_5V (470µF 25V): Cerca Jack DC → LM7805 IN
C_OUT_5V (100µF 10V): Cerca LM7805 OUT
C_OUT_3V3 (100µF 6.3V): Cerca AMS1117 OUT
C_MOT_VM (100µF 16V): Cerca MX1508 VM

¡POLARIDAD! Raya = Negativo = Corto
```

### 3.3 Conectores potencia
```
J_PWR (Jack DC 2.1mm): 3 patas, una central +, laterales GND
J_USB (Micro-B): 5 patas, soldar bien carcasa (GND/shield)
J_TERM (Terminal block 4p): Tornillos arriba, pines atraviesan
```

### 3.4 Sensores TO-92
```
DS18B20 (U_TEMP1): Cara plana → según silkscreen
LM35 (U_TEMP2): Cara plana → según silkscreen
- Dejar 5-10mm altura (no pegar a PCB) para aire
```

### 3.5 Mounting holes + Separadores
```
MH1-MH4: 4 agujeros M3
- Solo plated holes (sin componente)
- Usar para atornillar separadores M3x10mm nylon
- Separadores = pies + aislamiento mesa
```

---

## Sesión 4: Inserción ESP32 + Test (15 min)

### 4.1 Insertar ESP32 DevKit V4
```
ORIENTACIÓN: Conector USB hacia BORDE EXTERNO PCB
(Antenna WiFi hacia fuera)

1. Alinear pines ESP32 con sockets hembra
2. Presionar firmemente uniforme
3. Verificar: Sin pines doblados, todo insertado
```

### 4.2 Test rápido alimentación
```
1. Conectar USB a PC
   → LED 5V (Rojo) ON
   → LED 3.3V (Verde) ON
   → PC detecta puerto COM
   
2. Medir con multímetro:
   +5V Rail: 4.9-5.1V
   +3.3V Rail: 3.25-3.35V
   GND: 0V referencias
```

### 4.3 Subir firmware test
```
Archivo: test_all.ino (ver verificacion.md)

1. Arduino IDE → Board: ESP32 Dev Module
2. Port: COMx (el nuevo)
3. Upload
4. Serial Monitor 115200
5. Verificar cada test pasa
```

---

## Checklist final

### Visual
- [ ] Sin puentes estaño
- [ ] Sin soldaduras frías
- [ ] Componentes polarizados correctos
- [ ] ESP32 orientación correcta
- [ ] Disipador LM7805 puesto
- [ ] Separadores M3 atornillados

### Eléctrico
- [ ] GND ↔ +5V: OL
- [ ] GND ↔ +3.3V: OL
- [ ] +5V ↔ +3.3V: OL
- [ ] +5V Rail: 5V ±0.1V
- [ ] +3.3V Rail: 3.3V ±0.05V
- [ ] Corriente reposo +5V: <50mA
- [ ] Corriente reposo +3.3V: <80mA

### Funcional
- [ ] PC detecta ESP32
- [ ] Blink LED D1 funciona
- [ ] WiFi scan redes
- [ ] Test_all.ino: Todo PASS

---

## Tiempo estimado total

| Sesión | Tiempo | Dificultad |
|--------|--------|------------|
| 1. SMD | 30-45 min | Media |
| 2. THT Bajo | 45-60 min | Baja |
| 3. THT Potencia | 30 min | Media |
| 4. Test + ESP32 | 15 min | Baja |
| **TOTAL** | **2-2.5 horas** | - |

> **Principiante**: Duplicar tiempos. Hacer en 2-3 sesiones separadas.