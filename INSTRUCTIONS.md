# INSTRUCTIONS.md — Guía para Agentes de IA

## 📌 Contexto del Proyecto
Este repositorio aloja el **Portal de Calificaciones Académicas** (`LeonardoKrell_PortalDeCalificaciones`), desarrollado en el marco de la materia *Arquitectura y Diseño de Interfaces* (IES 9-018).
El sistema permite a los docentes cargar y gestionar calificaciones y a los estudiantes consultar su desempeño académico y promedios en tiempo real.

## 🛠 Stack Tecnológico
- **Lenguaje:** Python 3.10+
- **Framework Web:** Django
- **Base de Datos:** SQLite
- Para conocer la justificación del stack y las alternativas evaluadas, consultar `docs/adr/ADR-001-stack-tecnologico.md`.

## 📐 Reglas y Contratos Obligatorios
Cualquier modificación o generación de código realizada por la IA debe alinearse con los siguientes documentos:
1. **`SPEC.md`**: Especificación declarativa con las reglas del sistema y requerimientos funcionales (RF-01 al RF-17).
2. **`.opencoderules`**: Estándares de código, arquitectura, estructura del proyecto y reglas de seguridad (RBAC, CSRF, validación en servidor y ORM).
3. **`docs/adr/`**: Registro de decisiones de arquitectura.

## 🚀 Comandos de Entorno y Verificación
- **Ejecutar migraciones:**
  ```bash
  python manage.py migrate
