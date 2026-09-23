# 📊 Portal de Calificaciones Académicas

> **Proyecto de Arquitectura y Diseño de Interfaces · IES 9-018 · 2026**  
> Tecnicatura Superior en Desarrollo de Software · Prof. Paulo Alvarez

Sistema web para que docentes carguen calificaciones y estudiantes consulten sus notas en tiempo real.

---

## 🎯 Objetivo

Construir una plataforma simple pero bien arquitecturada donde:
- **Docentes** cargan y gestionen calificaciones de sus materias
- **Estudiantes** ven sus notas y calculan promedios
- **Sistema** valida datos y asegura permisos por rol

---

## 📖 Documentación Principal

| Documento | Descripción |
|-----------|-------------|
| [SPEC.md](./SPEC.md) | Especificación formal v2: requisitos funcionales, non-goals, contratos de datos y restricciones arquitectónicas |
| [ADR-001: Stack tecnológico](./docs/adr/ADR-001-stack-tecnologico.md) | Decisión: Django + Python + SQLite con justificación y alternativas descartadas |
| [ADR-002: Estilo arquitectónico](./docs/adr/ADR-002-estilo-arquitectonico.md) | Decisión: Monolito modular con Django. Alternativas: microservicios, serverless, hexagonal |
| [ADR-003: Persistencia](./docs/adr/ADR-003-persistencia.md) | Decisión: SQLite + Django ORM. Alternativas: PostgreSQL, MongoDB, archivos |
| [C4 Contexto](./docs/arquitectura/C4-contexto.md) | Diagrama C4 Nivel 1: sistema como caja negra con actores y sistemas externos |
| [C4 Contenedores](./docs/arquitectura/C4-contenedores.md) | Diagrama C4 Nivel 2: aplicaciones, bases de datos y protocolos internos |
| [.opencoderules](./.opencoderules) | Arnés IA v2: reglas de código, estructura, seguridad y coherencia arquitectónica |

---

## 🚀 Setup y Ejecución

### Requisitos previos
- Python 3.10+
- pip (gestor de paquetes)
- Git

### Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/IES9018/LeonardoKrell_PortalDeCalificaciones.git
cd LeonardoKrell_PortalDeCalificaciones

# 2. Crear entorno virtual
python -m venv venv

# 3. Activar entorno virtual
# En Linux/macOS:
source venv/bin/activate
# En Windows:
venv\Scripts\activate

# 4. Instalar dependencias
pip install django

# 5. Ejecutar migraciones (una vez tengas models.py)
python manage.py migrate

# 6. Crear superusuario (administrador)
python manage.py createsuperuser

# 7. Correr servidor de desarrollo
python manage.py runserver
```

El servidor estará disponible en `http://localhost:8000`

---

## 📁 Estructura del Proyecto

```
LeonardoKrell_PortalDeCalificaciones/
├── SPEC.md                          ← Especificación del sistema
├── README.md                        ← Este archivo
├── .opencoderules                   ← Arnés de IA y estándares
├── manage.py                        ← Punto de entrada Django
├── requirements.txt                 ← Dependencias Python (crear después)
│
├── docs/
│   └── adr/
│       ├── ADR-001-stack-tecnologico.md
│       └── ADR-template.md
│
├── portal_calificaciones/           ← Configuración principal del proyecto Django
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
│
├── usuarios/                        ← App Django: autenticación y usuarios
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   ├── urls.py
│   ├── tests.py
│   ├── admin.py
│   └── templates/
│
└── calificaciones/                  ← App Django: gestión de calificaciones
    ├── models.py
    ├── views.py
    ├── forms.py
    ├── urls.py
    ├── tests.py
    ├── decorators.py
    ├── permissions.py
    └── templates/
```

---

## ✅ Requerimientos Funcionales (RF)

| ID | Descripción | Estado |
|----|-------------|--------|
| RF-01 | El sistema autentica usuarios con email y contraseña | Pendiente |
| RF-02 | El sistema asigna roles automáticamente | Pendiente |
| RF-03 | El sistema mantiene la sesión del usuario | Pendiente |
| RF-04 | Estudiante ve todas sus calificaciones | Pendiente |
| RF-05 | Calificaciones agrupadas por materia | Pendiente |
| RF-06 | Promedio general visible | Pendiente |
| RF-07 | Estudiante NO ve calificaciones de otros | Pendiente |
| RF-08 | Estudiante NO puede editar calificaciones | Pendiente |
| RF-09 | Docente ve sus materias | Pendiente |
| RF-10 | Docente carga calificaciones | Pendiente |
| RF-11 | Docente ve lista de estudiantes | Pendiente |
| RF-12 | Docente edita calificaciones propias | Pendiente |
| RF-13 | Docente NO ve materias ajenas | Pendiente |
| RF-14 | Docente NO ve calificaciones ajenas | Pendiente |
| RF-15 | Validación: notas 0-10 | Pendiente |
| RF-16 | Validación: notas numéricas | Pendiente |
| RF-17 | Constraint: una nota por (estudiante, materia) | Pendiente |

Ver detalles completos en [SPEC.md](./SPEC.md)

---

## 🔒 Seguridad

Este proyecto implementa:
- ✅ Autenticación con Django
- ✅ Control de acceso por rol (RBAC)
- ✅ Validación en servidor (no solo frontend)
- ✅ ORM seguro contra inyección SQL
- ✅ CSRF protection en formularios

Ver detalles en [.opencoderules](./.opencoderules) → Sección 5

---

## 🧪 Testing

Ejecutar tests:
```bash
python manage.py test --verbosity=2
```

Objetivo: ≥70% cobertura en vistas críticas

---

## 🛠️ Decisiones Arquitectónicas

| ADR | Decisión |
|-----|----------|
| [ADR-001](./docs/adr/ADR-001-stack-tecnologico.md) | Stack: Django + Python + SQLite + Bootstrap 5 |
| [ADR-002](./docs/adr/ADR-002-estilo-arquitectonico.md) | Estilo: Monolito modular con Django |
| [ADR-003](./docs/adr/ADR-003-persistencia.md) | Persistencia: SQLite + Django ORM |

Las restricciones resultantes están registradas en [SPEC.md](./SPEC.md) → Sección 4 (Restricciones Arquitectónicas).

---

## 📋 Checklist TP1

- [x] Repo en `IES9018` con nomenclatura correcta
- [x] SPEC.md completo con RF-01 a RF-17
- [x] ADR-001 con alternativas descartadas
- [x] .opencoderules con reglas propias
- [x] README.md con setup y links
- [x] Estructura `docs/adr/` completa

---

## 👤 Autor

**Leonardo Krell**  
Tecnicatura Superior en Desarrollo de Software · IES 9-018 · 2026

---

## 📞 Contacto y Ayuda

- **Dudas sobre el SPEC:** Ver [SPEC.md](./SPEC.md)
- **Dudas sobre arquitectura:** Ver [ADR-001](./docs/adr/ADR-001-stack-tecnologico.md)
- **Dudas sobre estándares:** Ver [.opencoderules](./.opencoderules)
- **Issues con la cátedra:** Abrir Issue en [proyecto-adi-2026](https://github.com/IES9018/proyecto-adi-2026)

---

**Estado del Proyecto:** TP2 completado · Próximo: TP3 (Sprint 2)
