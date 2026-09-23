# C4 Contenedores — Portal de Calificaciones

> **Nivel 2 — Diagrama de Contenedores**
> Última actualización: 2026-09-22 | Ref: SPEC v2, ADR-001, ADR-002, ADR-003

## Descripción

Este diagrama desglosa el sistema en sus contenedores principales: la aplicación Django, la base de datos SQLite, y los componentes de frontend que interactúan a través del navegador.

## Diagrama

```mermaid
C4Container
    title Diagrama de Contenedores — Portal de Calificaciones

    Person(estudiante, "Estudiante", "Consulta calificaciones y promedios")
    Person(docente, "Docente", "Carga y edita calificaciones")

    System_Boundary(portal, "Portal de Calificaciones") {
        Container(django_app, "Aplicación Django", "Python 3.10+, Django 4.2+, Django ORM", "Backend web que maneja autenticación, lógica de negocio, validaciones y API interna")
        Container(html_css, "Frontend HTML/CSS", "HTML5, CSS3, Bootstrap 5", "Plantillas Jinja2 con estilos Bootstrap para formularios y vistas responsivas")
        Container(db, "Base de Datos", "SQLite 3", "Almacena usuarios, materias, inscripciones y calificaciones")
    }

    Rel(estudiante, html_css, "Visualiza en navegador", "HTTPS")
    Rel(docente, html_css, "Visualiza en navegador", "HTTPS")
    Rel(html_css, django_app, "Envía formularios y recibe vistas renderizadas", "HTTP")
    Rel(django_app, db, "Consulta y persiste datos", "Django ORM / SQLite")
    Rel(django_app, html_css, "Renderiza plantillas con datos", "Jinja2")
```

## Contenedores

### 1. Aplicación Django (`django_app`)

| Aspecto | Detalle |
|---|---|
| Tecnología | Python 3.10+, Django 4.2+ |
| Responsabilidad | Backend completo: autenticación (RF-01 a RF-03), lógica de negocio (RF-04 a RF-17), validaciones, control de acceso por roles |
| Módulos principales | `portal_calificaciones/` (core), `usuarios/` (autenticación y roles), `calificaciones/` (CRUD de notas) |
| Protocolo | HTTP interno, Sirve HTML renderizado |

### 2. Frontend HTML/CSS (`html_css`)

| Aspecto | Detalle |
|---|---|
| Tecnología | HTML5, CSS3, Bootstrap 5 |
| Responsabilidad | Interfaz de usuario responsiva: formularios de login, tablas de calificaciones, formularios de carga |
| Framework | Bootstrap 5 (CDN o local, según ADR) |
| Protocolo | HTTPS (entregado por Django) |

### 3. Base de Datos (`db`)

| Aspecto | Detalle |
|---|---|
| Tecnología | SQLite 3 (archivos `.db`) |
| Responsabilidad | Persistencia de datos: usuarios, materias, inscripciones, calificaciones |
| Acceso | Exclusivamente vía Django ORM (sin raw SQL, per ADR-001 y .opencoderules) |
| Migraciones | Django migrations para schema versionado |

## Protocolos y flujo

```mermaid
sequenceDiagram
    actor E as Estudiante/Docente
    participant B as Navegador
    participant D as Django App
    participant DB as SQLite

    E->>B: Accede al portal
    B->>D: POST /login (email + password)
    D->>DB: SELECT usuario WHERE email = ?
    DB-->>D: Usuario o error
    D-->>B: Redirect a dashboard o error

    E->>B: Solicita ver calificaciones
    B->>D: GET /calificaciones
    D->>DB: SELECT calificaciones WHERE estudiante_id = ?
    DB-->>D: Lista de calificaciones
    D-->>B: HTML renderizado con datos
```

## Restricciones de contenedores

- **Sin APIs externas:** No hay comunicación con servicios de terceros (Non-Goal)
- **Sin JavaScript frameworks:** Solo vanilla JS si se justifica en ADR futuro
- **SQLite en desarrollo:** PostgreSQL es viable en producción (ADR-001 menciona portabilidad)
- **Todo por HTTP:** No hay WebSocket ni conexiones persistentes

## Trazabilidad

| Elemento | Fuente |
|---|---|
| Django App | ADR-001 (stack tecnológico), SPEC.md Sección 4 |
| Frontend Bootstrap | SPEC.md Sección 4, ADR-001 |
| SQLite | ADR-001, ADR-003 |
| Django ORM | ADR-001, .opencoderules Sección 3 |
| Sin APIs externas | SPEC.md Non-Goals |
