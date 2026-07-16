# Ensamblaje - Técnicas de soldadura

## Soldadura SMD (0805, 0603, SOT-223, SOP-8)

### Preparación
```
1. Limpiar pads con alcohol isopropílico 99%
2. Aplicar FLUX (pasta o lápiz) en TODOS los pads
3. Pre-calentar placa 50-80°C (opcional, ayuda mojado)
```

### Pasos genéricos (Chip resistor/capacitor 0805/0603)
```
1. Estañar UN pad (preferiblemente esquina GND/negativo)
   - Punta limpia + poco estaño → pad
   
2. Posicionar componente con pinzas
   - Orientación correcta (polaridad si aplica)
   - Centrado en pads
   
3. Soldar pad estañado
   - Calentar pad + pata componente 2-3s
   - Retirar soldador, dejar enfriar 1s
   - Componente ya sujeto
   
4. Soldar pines restantes
   - Arrastrar estaño desde pad a pata
   - 1-2s por pin
   - No calentar excesivamente
   
5. Inspeccionar
   - Filete cóncavo, brillante, sin puentes
   - Componente plano contra PCB
```

### ICs SMD (SOT-223, SOP-8, etc.)
```
1. Flux generoso en todos los pads
2. Estañar pad esquina (pin 1 o GND)
3. Alinear IC (pin 1 = marca punto/muesca)
4. Soldar pin 1 → Sujeta IC
5. Soldar pin opuesto (esquina diagonal) → Alinea
6. Soldar resto pines (arrastrar estaño)
7. Revisar puentes entre pines adyacentes
   - Si hay puente: malla desoldar + flux + repasar
```

### Pad térmico expuesto (MX1508, AMS1117)
```
MX1508 (SOP-8 con pad central):
- Pad central = GND / térmico
- Debe soldar a polygon GND en bottom + vias
- Técnica: Estañar pad PCB → Flux → Posicionar IC → 
  Calentar desde abajo (placa caliente) o soldador grueso en pad
  + flux → Soldar pines laterales normal

AMS1117 (SOT-223, pin 2 = Tab = GND):
- Pin 2 grande = disipador
- Soldar bien a GND pour top + bottom + vias
- Calentar pin 2 más tiempo (masa térmica)
```

---

## Soldadura THT (Through-Hole)

### Componentes axiales (Resistencias, diodos, LEDs)
```
1. Doblar patillas a 90° (usar plantilla o alicates)
2. Insertar en PCB (cuerpo pegado a silkscreen)
3. Doblar ligeramente patillas hacia afuera (sujeta)
4. Voltear PCB
5. Calentar pad + pata 2-3s
6. Aplicar estaño en unión (no en punta)
7. Retirar estaño, luego soldador
8. Cortar patillas flush (1-2mm)
```

### Electrolíticos radiales
```
1. Identificar polaridad (raya negativa = cátodo)
2. Insertar (positivo = pata larga / marca +)
3. Mismo proceso THT
4. ¡No invertir! Explota / cortocircuito
```

### Conectores (Headers, Jack, USB, Terminal Block)
```
1. Insertar en PCB
2. Para headers: Usar breadboard o otro header como guía
   - Insertar header en breadboard → colocar PCB encima → soldar
   - Garantiza perpendicularidad y alineación
3. Jack DC / USB: Sujetar con cinta kapton o third-hand
4. Soldar 1 pin → revisar alineación → soldar resto
5. Terminal block: Soldar 1 pin → verificar orientación → resto
```

### ICs en Socket (ESP32 DevKit)
```
1. SOLDAR SOCKET HEMBRA (no el ESP32 directo)
2. Usar breadboard como guía para alinear 2x19
3. Soldar 1 pin esquina → revisar → soldar opuesto → resto
4. Verificar continuidad cada pin socket → pad PCB
5. INSERTAR ESP32 al final (orientación: USB hacia borde PCB)
```

---

## Perfiles de temperatura (Referencia)

### Estaño Sn63/Pb37 (Leaded - recomendado hobby)
| Fase | Temp | Tiempo |
|------|------|--------|
| Precalentamiento | 100-150°C | 60-120s |
| Activación flux | 150-180°C | 30-60s |
| Refusión (pico) | 215-225°C | 15-30s |
| Enfriamiento | - | 2-4°C/s |

### Estaño SAC305 (Lead-free - RoHS)
| Fase | Temp | Tiempo |
|------|------|--------|
| Precalentamiento | 150-180°C | 60-120s |
| Activación flux | 180-210°C | 30-60s |
| Refusión (pico) | 240-250°C | 20-40s |
| Enfriamiento | - | 2-4°C/s |

> **Con soldador manual**: No sigas perfil exacto. Calienta unión 2-3s, estaño fluye, retira. Si humea mucho = demasiado caliente.

---

## Defectos comunes y solución

| Defecto | Aspecto | Causa | Solución |
|---------|---------|-------|----------|
| **Puente (Short)** | Estaño une 2+ pads | Exceso estaño / pads cercanos | Malla desoldar + flux + repasar |
| **Soldadura fría** | Gris, rugoso, bola | Poca temp / movimiento al enfriar | Recalentar + flux + dejar quieto |
| **Poca mojado** | Estaño no cubre pad | Pad sucio / sin flux / temp baja | Limpiar + flux + temp adecuada |
| **Tombstoning** | Componente 0805 vertical | Desigual calentamiento pads | Pre-estañar ambos pads igual |
| **Pico/icicla** | Punta estaño afilada | Retirar soldador muy rápido | Retirar estaño, esperar 0.5s, retirar soldador |
| **Pad levantado** | Pad separado de PCB | Exceso calor / fuerza mecánica | Reparar con cable puente / jumper |
| **Componente quemado** | Negro, agrietado | Temp excesiva / tiempo largo | Reemplazar componente |

---

## Seguridad y buenas prácticas

| Regla | Por qué |
|-------|---------|
| **Ventilación / Extractor** | Humos flux + estaño tóxicos |
| **Gafas seguridad** | Proyecciones estaño caliente |
| **Pulsera antiestática** | ESD mata ICs (ESP32, AMS1117, MX1508) |
| **Punta limpia** | Estaño viejo = mala transferencia calor |
| **Estañar punta** | Previene oxidación, mejora mojado |
| **No tocar patas IC** | Aceite dedos = mala soldadura + ESD |
| **Temperatura controlada** | 350-380°C leaded / 380-420°C lead-free |
| **Apagar soldador** | Si >5min sin usar (ahorra punta) |

---

## Limpieza post-soldadura

```
Opcional pero recomendado para producción:

1. Cepillo suave + Alcohol isopropílico 99%
2. Frotar suavemente residuos flux
3. Secar aire comprimido / secador pelo (frío)
4. Inspección final lupa

No limpiar si:
- Flux "no-clean" (la mayoría modernos)
- Prueba rápida hobby
```