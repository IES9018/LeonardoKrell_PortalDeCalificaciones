# INSTRUCTIONS.md — Guía para Agentes de IA

## 📌 Contexto del Proyecto
Este repositorio aloja el **Portal de Calificaciones Académicas** (`LeonardoKrell_PortalDeCalificaciones`), desarrollado en el marco de la materia *Arquitectura y Diseño de Interfaces* (IES 9-018).
El sistema permite a los docentes cargar y gestionar calificaciones y a los estudiantes consultar su desempeño académico y promedios en tiempo real.

## 🛠 Stack Tecnológico
- **Lenguaje:** Python 3.10+
- **Framework Web:** Django
- **Frontend:** HTML5 + CSS3 + Bootstrap 5
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
  ```
- **Ejecutar tests:**
  ```bash
  python manage.py test --verbosity=2
  ```
- **Levantar servidor de desarrollo:**
  ```bash
  python manage.py runserver
  ```

## 📁 Estructura Esperada
- `portal_calificaciones/` — Configuración del proyecto Django
- `usuarios/` — App de autenticación y gestión de usuarios
- `calificaciones/` — App de carga y consulta de calificaciones
- `docs/adr/` — Decisiones arquitectónicas registradas

## ⚠️ Restricciones
- No implementar funcionalidad que no esté en un RF del SPEC.md
- No agregar dependencias nuevas sin ADR previo
- No desactivar protecciones de seguridad (CSRF, validaciones, etc.)
- Todo output de IA debe ser revisado y entendido antes de commits
