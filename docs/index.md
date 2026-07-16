# Base ESP32 DevKit V4

!!! info "Placa de entrenamiento para IoT y sistemas embebidos"
    Una placa base diseñada para facilitar el aprendizaje y prototipado con el módulo **ESP32 DevKit V4**. Incluye reguladores de voltaje, drivers de motor, sensores, actuadores y múltiples conectores para expansión.

## Características principales

| Categoría | Detalles |
|-----------|----------|
| **MCU** | ESP32 DevKit V4 (ESP32-WROOM-32E) |
| **Alimentación** | 5V (LM7805) + 3.3V (AMS1117) con protección |
| **Motor** | Driver MX1508 dual H-bridge (2 canales) |
| **Sensores** | DS18B20 (temp), LM35 (temp), LDR (luz) |
| **Actuadores** | Buzzer, 4 LEDs, conector servo |
| **Entradas** | 4× DIP switch, pulsador, SPDT, potenciómetro |
| **Conectores** | 40+ pines header, terminal tornillo, servo |
| **Monitoreo** | Sensor de corriente A9013 |

## Vistas de la placa

=== "3D Superior"
    ![3D Top](images/3D/base_esp32_top.png)

=== "3D Inferior"
    ![3D Bottom](images/3D/base_esp32_botton.png)

=== "3D Lateral"
    ![3D Side](images/3D/base_esp32_l.png)

=== "Render Raytraced"
    ![3D Ray](images/3D/base_esp32_top_ray.png)

## Silkscreen PCB

=== "Frontal"
    ![Silkscreen Top](images/pcb/base_esp32-F_Silkscreen.png)

=== "Trasera"
    ![Silkscreen Bottom](images/pcb/base_esp32-B_Silkscreen.png)

## Esquemático

![Esquemático](images/fritzing_out/schematic.svg)

## Inicio rápido

```bash
# Clonar repositorio
git clone https://github.com/Xizuth/base_esp32.git
cd base_esp32

# Instalar dependencias de documentación
pip install -r requirements.txt

# Ver documentación local
mkdocs serve
```

## Enlaces útiles

- [Documentación completa](https://xizuth.github.io/base_esp32)
- [Repositorio GitHub](https://github.com/Xizuth/base_esp32)
- [Reportar issue](https://github.com/Xizuth/base_esp32/issues)
- [Datasheet ESP32](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wrover-32e_datasheet_en.pdf)

## Licencia

Este proyecto está bajo licencia **MIT**. Ver [LICENSE.md](../LICENSE.md) para detalles.