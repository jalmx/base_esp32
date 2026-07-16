# Historia y versiones

## Changelog

### v0.4 (Julio 2026) - Versión actual
- Optimización de layout PCB para mejor integridad de señal
- Nuevos footprints personalizados para logo y QR
- Actualización libreria símbolos ESP32 DevKit V4
- Mejora en routing de alimentación (pistas más anchas)
- Añadido conector servo estándar (3 pines)
- Corrección silkscreen: etiquetas reguladores 3.3V/5V

### v0.3 (Abril 2025)
- **Cambio driver motor**: L293D → MX1508 (más compacto, mejor eficiencia)
- Añadido **LED RGB** (WS2812B compatible) en GPIO2
- Reconfiguración LEDs: primeros 3 a pull-up, inversión para compatibilidad RGB
- Inversión terminal de tornillos (mejor acceso cables)
- Añadido potenciómetro 10k en ADC1_CH0 (GPIO36)

### v0.2 (Enero 2025)
- Añadido **monitor de corriente A9013** en alimentación principal
- Mejoras en disipación térmica LM7805 (pistas cobre expandidas)
- Etiquetas silkscreen para reguladores 3.3V y 5V
- Añadidos test points en rails de alimentación

### v0.1 (Noviembre 2024) - Diseño inicial
- Esquemático base con ESP32 DevKit V4
- Reguladores LM7805 (5V) y AMS1117 (3.3V)
- Driver motor L293D (THT)
- Conectores: headers ESP32, terminal tornillo, servo
- LEDs indicadores: Power, 3.3V, 5V, User
- Buzzer pasivo en GPIO25
- Switches: DIP×4, Push, SPDT
- Sensores: DS18B20, LM35, LDR
- Potenciómetro 10k

## Próximos cambios planificados

> Fuente: [`to_changes.md`](../to_changes.md)

- [ ] Colocar nombre junto a footprint reguladores 3.3V
- [ ] Cambiar driver a L293D (THT y SMD dual footprint)
- [ ] Añadir LED RGB direccionable
- [ ] Invertir primeros 3 LEDs (pull-up) para no interferir con RGB
- [ ] Invertir terminal de tornillos (mejor routing)

## Roadmap futuro

| Feature | Prioridad | Estimado |
|---------|-----------|----------|
| Footprint dual L293D/MX1508 | Alta | v0.5 |
| LED RGB WS2812B | Alta | v0.5 |
| Conector Qwiic/STEMMA QT | Media | v0.6 |
| MicroSD slot | Media | v0.7 |
| Versión 4 capas PCB | Baja | v1.0 |