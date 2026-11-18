# XB-Uno

> Base ESP32 DevKit V4

Plataforma modular de desarrollo de hardware embebido y IoT basada en el microcontrolador ESP32 (DevKit V4) que concentra periféricos de entrada, salida, potencia y comunicación bajo una topología de interconexión no estática. A diferencia de las tarjetas entrenadoras convencionales con conexiones rígidas, este diseño incorpora bancos de DIP switches y arreglos de jumpers de aislamiento en todas sus secciones: fuentes de alimentación con bypass, entradas digitales y analógicas seleccionables, indicadores LED, puente H para motores DC, zumbador, zócalos para sensores removibles y puertos de expansión para teclado matricial, I2C, SPI y UART. De esta forma, es posible desconectar cualquier elemento integrado para liberar los pines GPIO del ESP32, interactuar con hardware externo o instrumentar las líneas con osciloscopio y analizador lógico durante la depuración de firmware.

* **Gestión de energía:** Bornera de entrada (`J8`) conectada a un regulador de 5 V (`LM7805`) con puente de *bypass*, acompañada de dos reguladores de 3.3 V (`AMS1117`: uno exclusivo para la placa/ESP32 y otro para periféricos externos) y rieles de distribución de 5 V, 3.3 V y GND.
* **Entradas digitales:** Dos pulsadores físicos (`INPUT_D1`, `INPUT_D2`) y un *DIP switch* de 4 posiciones (`SW6`) con cabezales de acceso para los pines de entrada del ESP32 (GPIO 34, 35, 36 y 39).
* **Entradas analógicas seleccionables:** Dos canales analógicos (`AI_1`, `AI_2`) configurables mediante interruptores selectores (`SW8`, `SW9`) para alternar entre dos potenciómetros (`RV1`, `RV2`), una fotorresistencia (`LDR`) y un sensor de temperatura analógico (`LM35`).
* **Salidas digitales y actuadores:** Cinco indicadores LED con *jumpers* de aislamiento individual (GPIO 33, 15, 32, 2 y 4), un zumbador piezoeléctrico (`BZ1`), cabezal para servomotor y un controlador puente H dual (`MX1508`) para motores DC.
* **Sensores removibles:** Zócalos intercambiables con *jumpers* de desconexión para sensor ultrasónico (`HC-SR04`), sensor de temperatura y humedad (`DHT11`) y sensor digital de temperatura 1-Wire (`DS18B20`).
* **Comunicación y expansión:** Puertos dedicados (macho y hembra) para buses **I2C**, **SPI** y **UART (TX/RX)**, además de un puerto de expansión de 8 líneas aislado por dos *DIP switches* (`SW1`, `SW2`) orientado a teclados matriciales o E/S de propósito general.

## Características principales

| Categoría        | Detalles                                     |
|------------------|----------------------------------------------|
| **MCU**          | ESP32 DevKit V4 (ESP32-WROOM-32E)            |
| **Alimentación** | 5V (LM7805) + 3.3V (AMS1117) con protección  |
| **Motor**        | Driver MX1508 dual H-bridge (2 canales)      |
| **Sensores**     | DS18B20 (temp), LM35 (temp), LDR (luz)       |
| **Actuadores**   | Buzzer, 4 LEDs, conector servo               |
| **Entradas**     | 4× DIP switch, pulsador, SPDT, potenciómetro |
| **Conectores**   | 40+ pines header, terminal tornillo, servo   |

## Vistas de la placa

### 3D Superior

![3D Top](images/3D/base_esp32_top.png)

### 3D Inferior

![3D Bottom](images/3D/base_esp32_botton.png)

### 3D Lateral

![3D Side](images/3D/base_esp32_l.png)

### Render Raytraced

![3D Ray](images/3D/base_esp32_top_ray.png)

## Silkscreen PCB

### Frontal

![Silkscreen Top](images/pcb/base_esp32-F_Silkscreen.png)

### Trasera
![Silkscreen Bottom](images/pcb/base_esp32-B_Silkscreen.png)

## Esquemático

![Esquemático](images/fritzing_out/kicad_svg_plots/board_render.png)

## Inicio rápido

```bash
# Clonar repositorio
git clone https://github.com/jalmx/base_esp32.git
cd base_esp32

# Instalar dependencias de documentación
pip install -r requirements.txt

# Ver documentación local
mkdocs serve
```

## Enlaces útiles

- [Documentación completa](https://www.alejandro-leyva.com/base_esp32/)
- [Repositorio GitHub](https://github.com/jalmx/base_esp32)
- [Reportar issue](https://github.com/jalmx/base_esp32/issues)
- [Datasheet ESP32](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wrover-32e_datasheet_en.pdf)

## Licencia

Este proyecto está bajo licencia **CERN-OHL-S-2.0**. Ver [LICENSE.md](../LICENSE.md) para detalles.