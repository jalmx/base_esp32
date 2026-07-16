# Reguladores de voltaje

## LM7805CT - Regulador 5V

### Datos técnicos

| Parámetro | Valor |
|-----------|-------|
| **Tipo** | Regulador lineal positivo (LDO no) |
| **Voltaje salida** | 5.0V fijo |
| **Corriente máxima** | 1.5A |
| **Voltaje entrada** | 7-35V (recomendado 7-12V) |
| **Dropout voltage** | ~2.0V @ 1A |
| **Regulación línea** | 10mV típico |
| **Regulación carga** | 50mV típico |
| **Corriente quiescente** | 4-8mA |
| **Protecciones** | Térmica, cortocircuito, SOA |
| **Encapsulado** | TO-220-3 (vertical) |
| **Temperatura** | 0-125°C (junction) |

### Circuito en la Base ESP32

```
           ┌──────────────┐
VIN (7-12V)─►│ IN     OUT │──► +5V Rail
           │  LM7805CT   │
           │     GND     │
           └──────┬──────┘
                  │
                  ▼
              GND Plane
```

**Componentes asociados:**
- **C_IN1**: 470µF 25V Electrolítico (bulk, cerca pin IN)
- **C_IN2**: 100nF 0805 Cerámico (decoupling HF, pegado a pin IN)
- **C_OUT1**: 100µF 10V Electrolítico (bulk, cerca pin OUT)
- **C_OUT2**: 100nF 0805 Cerámico (decoupling HF, pegado a pin OUT)
- **D_PROT**: 1N5819 Schottky (pin OUT → IN, protección descarga C_OUT)
- **LED_5V**: LED 3mm Rojo + 1kΩ (indicador visual)

### Disipación térmica

**Cálculo potencia disipada:**
```
P_D = (V_IN - V_OUT) × I_OUT + V_IN × I_Q
```

| Escenario | V_IN | I_OUT | P_D | ΔT (θ_JA=25°C/W) |
|-----------|------|-------|-----|------------------|
| **Typical** | 9V | 300mA | 1.2W | 30°C |
| **Worst case** | 12V | 800mA | 5.6W | 140°C ❌ |
| **USB only** | 5V | 500mA | 0W (dropout) | - |

**Soluciones implementadas:**
1. **4 vias térmicas** bajo pad GND del TO-220 → GND pour bottom
2. **Copper pour** +5V en top layer actúa como heatsink lateral
3. **Recomendación**: Vin ≤ 9V para uso continuo, disipador externo si >500mA sostenido

### Selección de capacitores

| Posición | Valor | Tipo | Por qué |
|----------|-------|------|---------|
| Input Bulk | 470µF | Electrolítico 25V | Reservorio energía, ripple bajo |
| Input HF | 100nF | X7R 0805 25V | Filtro ruido alta frecuencia |
| Output Bulk | 100µF | Electrolítico 10V | Estabilidad, respuesta transitoria |
| Output HF | 100nF | X7R 0805 10V | Ruido switching, ESP32 TX bursts |

> **Nota**: LM7805 requiere capacitor salida ≥ 0.1µF para estabilidad. 100µF + 100nF da margen amplio.

---

## AMS1117-3.3 - Regulador 3.3V

### Datos técnicos

| Parámetro | Valor |
|-----------|-------|
| **Tipo** | Regulador lineal LDO (Low Dropout) |
| **Voltaje salida** | 3.3V fijo (versión adjustable existe) |
| **Corriente máxima** | 800mA (1A peak) |
| **Voltaje entrada** | 3.8-12V (max 15V) |
| **Dropout voltage** | 1.1V típico @ 800mA |
| **Regulación línea** | 0.2% típico |
| **Regulación carga** | 0.4% típico |
| **Corriente quiescente** | 5mA típico |
| **Protecciones** | Térmica, current limit |
| **Encapsulado** | SOT-223-4 (Tab = GND) |
| **Temperatura** | -40 a +125°C |

### Circuito en la Base ESP32

```
       +5V Rail ──►│ IN    OUT │──► +3.3V Rail
                   │ AMS1117  │
                   │  GND TAB │
                   └────┬─────┘
                        │
                        ▼
                    GND Plane (tab soldered)
```

**Componentes asociados:**
- **C_IN**: 10µF 0805 + 100nF 0805 (cerca pin IN)
- **C_OUT**: 100µF 6.3V Electrolítico + 100nF 0805 (cerca pin OUT)
- **LED_3V3**: LED 3mm Verde + 1kΩ

### Disipación térmica

| Escenario | V_IN | I_OUT | P_D | ΔT (θ_JA=40°C/W) |
|-----------|------|-------|-----|------------------|
| **Typical** | 5V | 200mA | 0.34W | 14°C ✅ |
| **WiFi TX** | 5V | 500mA | 0.85W | 34°C ✅ |
| **Max** | 5V | 800mA | 1.36W | 54°C ⚠️ |

**Gestión térmica SOT-223:**
- **Tab (pin 2)** = GND, soldado a **GND pour** top + bottom
- **4 vias térmicas** bajo tab → GND plane bottom
- Copper pour +3.3V en top actúa como spreader lateral
- Suficiente para 500mA continuo sin heatsink externo

### ¿Por qué AMS1117 y no otro LDO?

| LDO | I_max | Dropout @500mA | Costo | Disponibilidad |
|-----|-------|----------------|-------|----------------|
| **AMS1117-3.3** | 800mA | 1.1V | $0.15 | Muy alta (LCSC/JLC) |
| LM1117-3.3 | 800mA | 1.2V | $0.20 | Alta |
| MCP1700-3.3 | 250mA | 1.7V | $0.30 | Media |
| AP2112-3.3 | 600mA | 0.9V | $0.25 | Media |
| TPS7A47 | 1A | 0.3V | $1.50 | Baja |

**Ganador**: AMS1117 por relación costo/rendimiento/disponibilidad en fabricantes chinos.

---

## Comparativa y selección

```
                    LM7805 (5V)          AMS1117 (3.3V)
                    ┌─────────────┐      ┌─────────────┐
Topología           │ Lineal std  │      │ LDO         │
Dropout             │ ~2V         │      │ ~1.1V       │
Eficiencia @9V→5V   │ 55%         │      │ N/A         │
Eficiencia @5V→3.3V │ N/A         │      │ 66%         │
Corriente max       │ 1.5A        │      │ 800mA       │
Costo unitario      │ $0.10       │      │ $0.15       │
Encapsulado         │ TO-220 THT  │      │ SOT-223 SMD │
Disipación          │ Fácil (THT) │      │ Requiere PCB│
Ruido salida        │ Bajo        │      │ Muy bajo    │
```

## Datasheets

- [LM7805 Datasheet (TI)](https://www.ti.com/lit/ds/symlink/lm7805.pdf)
- [AMS1117 Datasheet (Advanced Monolithic)](https://www.diodes.com/assets/Datasheets/AMS1117.pdf)
- [MX1508 Datasheet](../datasheets/MX1515H_C5119046.pdf)