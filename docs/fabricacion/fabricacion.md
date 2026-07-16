# Fabricación - Cómo fabricar y pedir PCBs

## Opciones de fabricación

| Proveedor | Precio 5 uds | Envío ESP | Tiempo total | Calidad | SMT opcional |
|-----------|--------------|-----------|--------------|---------|--------------|
| **JLCPCB** | ~$2 + $8 envío | 5-10 días | 2-3 sem | Buena | Sí ($8+comp) |
| **PCBWay** | ~$5 + $10 envío | 5-12 días | 2-3 sem | Muy buena | Sí |
| **AISLER** | ~€15 + €8 | 3-5 días | 1-2 sem | Excelente | No |
| **OSH Park** | ~$15 (3 uds) | 2-3 sem | 3-4 sem | Muy buena | No |
| **Eurocircuits** | ~€40 | 5-7 días | 1-2 sem | Profesional | Sí |

> **Recomendación**: JLCPCB para prototipos (precio/calidad), PCBWay para calidad, AISLER para urgencia UE.

---

## Paso a paso: JLCPCB (Recomendado)

### 1. Preparar archivos
```bash
# En KiCad 8/9:
# File → Fabrication Outputs → Gerbers (.gbr)
#   ✓ Plot: F.Cu, B.Cu, F.SilkS, B.SilkS, F.Mask, B.Mask, F.Paste, B.Paste, Edge.Cuts
#   ✓ Use Protel filename extensions
#   ✓ Generate Drill Files: PTH + NPTH, Excellon, mm, decimal format
# File → Fabrication Outputs → BOM / Footprint Position (para SMT)
```

### 2. Subir a JLCPCB
1. Ir a https://jlcpcb.com/quote
2. **Add Gerber file** → Arrastrar `base_esp32_v4.zip`
3. Revisar preview (visor integrado)
4. Configurar:
   ```
   PCB Qty: 5
   Layers: 2
   Thickness: 1.6mm
   Min Track/Space: 0.15/0.15mm
   Min Hole: 0.3mm
   Surface Finish: HASL (Lead Free)
   Solder Mask: Green
   Silkscreen: White
   Via Process: Tented
   Remove Order Number: Yes
   ```

### 3. (Opcional) Ensamblaje SMT
1. Pestaña **SMT Assembly** → **Add SMT**
2. Subir `bom.csv` y `pnp.csv`
3. Revisar componentes:
   - **Confirm** para componentes en librería JLC (Basic/Extended)
   - **Omit** para componentes no disponibles (ESP32 socket, MX1508, conectores THT)
4. Seleccionar **Top side only** (Bottom solo pads THT)

### 4. Pagar y esperar
- Pago: PayPal, tarjeta, transferencia
- Tracking: Email + web
- Tiempo: 2-5 días fabricación + 5-10 días envío estándar

---

## Paso a paso: PCBWay

1. https://www.pcbway.com/orderonline.aspx
2. **Quick-order PCB** → Upload Gerber
3. Mismos parámetros que JLCPCB
4. **Assembly** → Upload BOM + Centroid
5. PCBWay revisa manualmente → Quote en 2-24h
6. Confirmar y pagar

---

## Pedido solo PCB + compra componentes aparte

### Lista compras (LCSC / Mouser / DigiKey)
Ver [BOM completa](../ensamblaje/componentes.md)

**Tip**: Pedir componentes en **LCSC** junto con PCB JLCPCB → Envío combinado, ahorras flete.

### LCSC + JLCPCB (Workflow integrado)
1. En JLCPCB quote → **Parts** → **Add from LCSC**
2. Buscar cada componente por LCSC Part Number
3. **Add to cart** → Se añade a pedido PCB
4. **Combined shipping** → Un solo envío

---

## Fabricación casera (No recomendado para v0.4)

| Método | Viabilidad | Comentario |
|--------|------------|------------|
| **Toner transfer** | ❌ | 2 capas, vias, SMD fino = muy difícil |
| **Fotolitografía** | ⚠️ | Requiere equipo, químicos, experiencia |
| **CNC milling** | ⚠️ | Vías manuales, aislamiento tracks difícil |
| **PCB prototipo FR-1** | ❌ | No aguanta calor LM7805, pistas finas |

> **Conclusión**: Para PCB 2 capas con SMD 0805, SOP-8, vias, pistas 6mil → **Fabricación profesional obligatoria**.

---

## Certificaciones y normativa

| Aspecto | Estándar | Nota |
|---------|----------|------|
| **RoHS** | 2011/65/EU | HASL-LF, componentes sin plomo |
| **REACH** | 1907/2006 | Sin SVHC en materiales |
| **UL 94V-0** | FR-4 | Autoextinguible |
| **IPC-A-600** | Clase 2 | Calidad estándar comercial |
| **IPC-6012** | Tipo 2 | Rigidez, espesor cobre |

---

## Control de calidad recepción

Al recibir PCBs:
```
1. □ Cantidad correcta (5 + 1-2 spare)
2. □ Dimensiones: 100×80mm ±0.2mm
3. □ Espesor: 1.6mm ±0.1mm
4. □ Bordes: Suaves, sin rebabas, esquinas R2mm
5. □ Agujeros: Mounting M3 (3.2mm), PTH 0.3-1.0mm limpios
6. □ Cobre: Sin rasguños, tracks continuos
7. □ Máscara: Uniforme, sin burbujas, pads expuestos
8. □ Silkscreen: Legible, alineado, sin sobre pads
9. □ Vías: Tentadas (cubiertas mask) o abiertas según spec
9. □ Marcado: UL, fabricante, fecha (lado B)
10. □ Test eléctrico: Continuidad nets críticos (opcional flying probe)
```

---

## Archivos para siguiente revisión (v0.5)

Mantén en repo:
```
fabricacion/
├── gerber_v0.4.zip          # Gerbers v0.4
├── gerber_v0.4_sources/     # Carpeta descomprimida
├── bom_v0.4.csv             # BOM JLCPCB
├── pnp_v0.4.csv             # Pick&Place
├── stackup_v0.4.png         # Captura stackup KiCad
├── drc_report_v0.4.txt      # Reporte DRC
└── quotes/
    ├── jlcpcb_v0.4.pdf
    ├── pcbway_v0.4.pdf
    └── aisler_v0.4.pdf
```

---

## Coste total estimado (v0.4, 5 uds)

| Concepto | JLCPCB | PCBWay | AISLER |
|----------|--------|--------|--------|
| PCB 5 uds | $2.00 | $5.00 | €15.00 |
| Envío estándar | $8.00 | $10.00 | €8.00 |
| **Solo PCB** | **$10.00** | **$15.00** | **€23.00** |
| SMT Básico (10 comp) | +$8.00 | +$15.00 | N/A |
| Componentes LCSC | ~$12.00 | ~$12.00 | ~€15.00 |
| **Total completo** | **~$30** | **~$42** | **~€38** |

*Precios 2024, IVA no incluido, envío España peninsular*