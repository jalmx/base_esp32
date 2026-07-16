# Visión general del proyecto

## ¿Qué es Base ESP32?

**Base ESP32** es una placa de desarrollo (shield/baseboard) diseñada específicamente para el módulo **ESP32 DevKit V4** (ESP32-WROOM-32E). Su objetivo es proporcionar una plataforma completa y lista para usar que permita:

- **Aprender** programación de sistemas embebidos e IoT
- **Prototipar** rápidamente proyectos con sensores y actuadores
- **Enseñar** conceptos de electrónica y firmware en entornos educativos
- **Desarrollar** proof-of-concepts sin necesidad de breadboards

## Público objetivo

| Perfil | Uso |
|--------|-----|
| **Estudiantes** | Prácticas de laboratorio, TFG, proyectos finales |
| **Makers** | Prototipado rápido, domótica, robótica ligera |
| **Docentes** | Material didáctico para asignaturas de IoT/embebidos |
| **Ingenieros** | Validación de conceptos, testing de firmware |

## Especificaciones rápidas

| Parámetro | Valor |
|-----------|-------|
| **Dimensiones PCB** | 100mm × 80mm (aprox) |
| **Capas** | 2 capas (FR-4) |
| **Alimentación entrada** | 7-12V DC (jack) / 5V USB |
| **Corriente máxima 5V** | 1.5A (LM7805) |
| **Corriente máxima 3.3V** | 800mA (AMS1117) |
| **Temperatura operación** | -40°C a +85°C |

## Versiones de hardware

| Versión | Fecha | Cambios principales |
|---------|-------|---------------------|
| **v0.1** | Nov 2024 | Diseño inicial, componentes básicos |
| **v0.2** | Ene 2025 | Añadido monitor corriente, ajustes silkscreen |
| **v0.3** | Abr 2025 | Cambio driver motor a MX1508, LED RGB |
| **v0.4** | Jul 2026 | Optimización layout, nuevos footprints |

> **Nota**: Ver [Historia y versiones](historia.md) para changelog detallado.

## Repositorio y archivos

```
base_esp32/
├── *.kicad_sch          # Esquemático KiCad
├── *.kicad_pcb          # Layout PCB KiCad
├── *.kicad_pro          # Proyecto KiCad
├── gerber/              # Archivos de fabricación
├── 3D/                  # Renders 3D
├── pcb/                 # Exportaciones capas PCB
├── assets/              # Imágenes, pinouts, logos
├── datasheets/          # Datasheets componentes
├── models/              # Modelos 3D STEP
├── docs/                # Esta documentación
└── README.md            # README principal
```

## Comunidad y soporte

- **Issues**: [GitHub Issues](https://github.com/Xizuth/base_esp32/issues)
- **Discusiones**: [GitHub Discussions](https://github.com/Xizuth/base_esp32/discussions)
- **Email**: jalm_x@hotmail.com (Josef Leyva)