# Visión general del hardware

## Arquitectura del sistema

```
┌─────────────────────────────────────────────────────────────┐
│                      ESP32 DevKit V4                        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  GPIO   │  │  ADC    │  │  PWM    │  │  Comm   │        │
│  │  0-39   │  │  18 ch  │  │  16 ch  │  │ I2C/SPI │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
└───────┼────────────┼────────────┼────────────┼──────────────┘
        │            │            │            │
  ┌─────┴─────┐ ┌────┴────┐ ┌─────┴─────┐ ┌────┴────┐
  │  Power    │ │ Sensors │ │ Actuators │ │  I/O    │
  │  5V/3.3V  │ │ DS18B20 │ │ MX1508    │ │ Switches│
  │  Monitor  │ │ LM35    │ │ Servo     │ │ Potenci │
  │  A9013    │ │ LDR     │ │ Buzzer    │ │ LEDs    │
  └───────────┘ └─────────┘ └───────────┘ └─────────┘
```

## Vista 3D de la placa

=== "Superior"
    ![Top](images/3D/base_esp32_top.png)

=== "Inferior"
    ![Bottom](images/3D/base_esp32_botton.png)

=== "Lateral Izquierda"
    ![Left](images/3D/base_esp32_l.png)

=== "Lateral Derecha"
    ![Right](images/3D/base_esp32_r.png)

=== "Render Superior"
    ![Ray Top](images/3D/base_esp32_top_ray.png)

=== "Render Inferior"
    ![Ray Bottom](images/3D/base_esp32_botton_ray.png)

## Bloques funcionales

| Bloque | Componentes clave | Página |
|--------|-------------------|--------|
| **Alimentación** | LM7805, AMS1117, A9013, jack DC, USB | [Alimentación](../alimentacion/overview.md) |
| **MCU** | ESP32 DevKit V4 socket | [Pinout](pinout.md) |
| **Motor DC** | MX1508 (2 canales H-bridge) | [Actuadores](../perifericos/actuadores.md) |
| **Sensores** | DS18B20, LM35, LDR | [Sensores](../perifericos/sensores.md) |
| **Actuadores** | Buzzer, 4×LED, Servo, LED RGB | [Actuadores](../perifericos/actuadores.md) |
| **Entradas** | DIP×4, Push, SPDT, Pot 10k | [Entradas](../perifericos/entradas.md) |
| **Conectividad** | Headers ESP32, Terminal tornillo, Servo | [Pinout](pinout.md) |

## Esquemático completo

![Esquemático](images/fritzing_out/schematic.svg)

## PCB Layout

=== "Capa Superior (F.Cu)"
    ![Top Copper](images/pcb/base_esp32-F_Cu.png)

=== "Capa Inferior (B.Cu)"
    ![Bottom Copper](images/pcb/base_esp32-B_Cu.png)

=== "Silkscreen Superior"
    ![Top Silk](images/pcb/base_esp32-F_Silkscreen.png)

=== "Silkscreen Inferior"
    ![Bottom Silk](images/pcb/base_esp32-B_Silkscreen.png)

=== "Máscara Superior"
    ![Top Mask](images/pcb/base_esp32-F_Mask.png)

=== "Máscara Inferior"
    ![Bottom Mask](images/pcb/base_esp32-B_Mask.png)

## Decisiones de diseño clave

1. **Alimentación dual**: 5V para motor/servo, 3.3V para ESP32 y sensores
2. **Monitor corriente**: A9013 en rail principal para protección
3. **Driver MX1508**: SMD, compacto, 1.5A/canal, PWM hasta 20kHz
4. **Socket ESP32**: Permite cambio de módulo, reduce riesgo soldadura
5. **Headers estándar**: Pitch 2.54mm compatible breadboard/dupont
6. **Terminal tornillo**: Conexión robusta para motor/alimentación externa
7. **Test points**: Verificación rails de alimentación sin soldar