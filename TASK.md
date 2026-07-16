# Tareas pendientes - Documentación Base ESP32

## Completado

- [x] Instalar mkdocs-material 9.7.6
- [x] Crear `mkdocs.yml` (Material theme, nav completa, extensions)
- [x] Crear symlinks imágenes (`docs/images/`)
- [x] Crear estructura directorios (`proyecto/`, `hardware/`, `alimentacion/`, `perifericos/`, `practicas/`, `ensamblaje/`, `fabricacion/`, `kicad/`, `recursos/`, `contribuir/`, `about/`)
- [x] **docs/index.md** - Página principal
- [x] **docs/proyecto/** - overview.md, historia.md, licencia.md
- [x] **docs/hardware/** - overview.md, esquematico.md, pcb.md, componentes.md, pinout.md
- [x] **docs/alimentacion/** - overview.md, reguladores.md, proteccion.md
- [x] **docs/perifericos/** - leds.md, buzzer.md, sensores.md, actuadores.md, entradas.md, monitoreo.md
- [x] **docs/practicas/** - index.md + 12 prácticas (beginner 01-04, intermediate 05-08, advanced 09-12)
- [x] **docs/ensamblaje/** - index.md, componentes.md, soldadura.md, paso-a-paso.md, verificacion.md
- [x] **docs/fabricacion/** - gerber.md, fabricacion.md, fritzing.md
- [x] **docs/kicad/** - instalacion.md, estructura.md, modificacion.md, librerias.md

---

## Pendiente

### Páginas de docs faltantes

- [ ] **docs/recursos/datasheets.md** - Tabla con datasheets de cada componente, enlaces oficiales
- [ ] **docs/recursos/enlaces.md** - Enlaces útiles: tutoriales, herramientas, comunidades ESP32
- [ ] **docs/recursos/tools.md** - Herramientas: IDE, multímetro, osciloscopio, programadores
- [ ] **docs/contribuir/guia.md** - Guía de contribución: fork, branch naming, PRs, convenciones commits
- [ ] **docs/about/creditos.md** - Créditos, autores, licencias de dependencias, logos

### Archivos raíz

- [ ] **Readme.md** - Reescribir completamente:
  - Badges (License, Build Status, Spanish)
  - Descripción en español + inglés
  - Sección Features (componentes, periféricos, prácticas)
  - Imágenes destacadas (3D top, schematic)
  - Quick Start (cómo clonar, abrir en KiCad, fabricar)
  - Link a documentación (`https://xizuth.github.io/base_esp32/`)
  - Dev section (mkdocs build, preview)
  - Tabla de contenidos

- [ ] **.gitignore** - Reescribir con reglas completas:
  - KiCad: `*.kicad_sch-bak`, `*.kicad_pcb-bak`, `fp-info-cache`, `*-backups/`, `*.bak`, `*.lck`, `*.tmp`, `_autosave-*`
  - Python: `venv/`, `__pycache__/`, `*.pyc`, `.env`
  - MkDocs: `site/`
  - OS: `.DS_Store`, `Thumbs.db`, `Desktop.ini`
  - IDEs: `.vscode/`, `.idea/`, `*.swp`, `*.swo`, `*~`
  - Build: `*.o`, `*.elf`, `*.bin`, `build/`

### Verificación

- [ ] Ejecutar `mkdocs build --strict -d /tmp/mkdocs-test` para verificar que no hay errores de compilación