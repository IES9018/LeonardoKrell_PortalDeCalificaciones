# ADR-003: Estrategia de Persistencia

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-09-22 |
| **Decisor** | Leonardo Krell |
| **ADR relacionado** | ADR-001 (Stack Tecnológico), ADR-002 (Estilo Arquitectónico) |

## Contexto

El Portal de Calificaciones necesita persistir datos con las siguientes características (definidas en SPEC.md Sección 5):

### Modelo de datos

| Entidad | Campos clave | Relaciones | Restricciones |
|---|---|---|---|
| `Usuario` | id, email (unique), password (hashed), nombre_completo, rol, activo, fecha_creacion | — | Email único, rol estudiante/docente |
| `Materia` | id, codigo (unique), nombre, descripcion, docente FK, cuatrimestre, ano, activa, fecha_creacion | FK → Usuario (docente) | Código único |
| `Inscripcion` | id, estudiante FK, materia FK, fecha_inscripcion | FK → Usuario, FK → Materia | Unique en (estudiante + materia) |
| `Calificacion` | id, estudiante FK, materia FK, nota (0-10), cargada_por FK, fecha_carga, fecha_ultima_modificacion, observaciones | FK → Usuario, FK → Materia, FK → Usuario (docente) | Unique en (estudiante + materia), nota 0-10 |

### Volumen estimado

- ~200 usuarios (estudiantes + docentes)
- ~50 materias por cuatrimestre
- ~2000 inscripciones
- ~2000 calificaciones
- **Total: ~4250 registros** (volumen muy bajo)

### Requisitos de integridad

- Relaciones N-M con constraints de unicidad
- Validación de rangos (nota 0-10)
- Integridad referencial (FK)
- Transaccionalidad básica (carga de calificaciones)

## Decisión

**SQLite 3** como base de datos de desarrollo y producción inicial, accedido exclusivamente vía **Django ORM**.

### Justificación basada en el modelo de datos real

| Requisito del modelo | SQLite + Django ORM | Cumple |
|---|---|---|
| Email único en Usuario | `unique=True` en campo | ✅ |
| FK Usuario → Materia (docente) | `ForeignKey(Usuario)` | ✅ |
| Unique constraint (estudiante + materia) | `unique_together = ('estudiante', 'materia')` | ✅ |
| Nota 0-10 | `validators=[MinValueValidator(0), MaxValueValidator(10)]` | ✅ |
| Relaciones N-M | Django ORM las resuelve con tablas intermedias | ✅ |
| Migraciones | `makemigrations` / `migrate` | ✅ |
| Admin panel | `admin.site.register()` automático | ✅ |

### Criterios de la decisión

| Criterio | Puntuación | Justificación |
|---|---|---|
| Volumen de datos (~4250 registros) | ⭐⭐⭐⭐⭐ | SQLite maneja millones de registros, sobra capacidad |
| Setup sin configuración | ⭐⭐⭐⭐⭐ | Archivo `.db` autocontenido, sin servidor externo |
| Integración con Django ORM | ⭐⭐⭐⭐⭐ | Django tiene backend SQLite integrado, cero config |
| Portabilidad | ⭐⭐⭐⭐⭐ | Archivo único, fácil de copiar, respaldar, versionar |
| Integridad referencial | ⭐⭐⭐⭐⭐ | SQLite soporta FK (desde 3.6.19), Django lo habilita |
| Costo | ⭐⭐⭐⭐⭐ | Gratuito, sin licencias, sin infraestructura |
| Deployment | ⭐⭐⭐⭐⭐ | Sin dependencias externas, funciona en cualquier SO |

## Alternativas Descartadas

### Alternativa 1: PostgreSQL

**Descripción:** Base de datos relacional robusta, estándar en producción para aplicaciones Django.

| Criterio | Evaluación | Razón de rechazo |
|---|---|---|
| Funcionalidad SQL | ⭐⭐⭐⭐⭐ | Más funciones que SQLite (JSON, full-text search, etc.) |
| Setup | ❌ Requiere instalación y configuración | Necesita usuario, contraseña, permisos, puerto — overhead innecesario para ~4250 registros |
| Deployment | ❌ Servicio externo | En la facultad no se garantiza disponibilidad de PostgreSQL |
| Rendimiento | ⚠️ Excesivo | Para ~4250 registros, la diferencia de rendimiento es imperceptible |
| Portabilidad | ⚠️ Menor | Requiere dump/restore para mover entre máquinas |
| Costo | ⚠️ Variables | Hosting gratuito limitado, self-hosted requiere infraestructura |
| Adecuación al problema | ❌ Sobredimensionado | El volumen y complejidad no justifican un RDBMS pesado |

**Veredicto:** Descartado. PostgreSQL es excelente para producción, pero el overhead de setup y la complejidad operativa no se justifican para un MVP con ~4250 registros.

**Nota:** La SPEC y ADR-001 ya planifican la migración a PostgreSQL si es necesario (cambiar `DATABASES` en `settings.py`). Esta decisión no cierra esa puerta.

### Alternativa 2: MongoDB (NoSQL)

**Descripción:** Base de datos documental (NoSQL) que almacena datos en formato BSON/JSON.

| Criterio | Evaluación | Razón de rechazo |
|---|---|---|
| Modelo de datos relacional | ❌ Inadecuado | Las entidades del Portal son inherentemente relacionales (FK, unique constraints) |
| Django ORM integration | ❌ Requiere `djongo` o `mongoengine` | Dependencias adicionales, menos maduras que el ORM nativo |
| Integridad referencial | ❌ No nativa | MongoDB no tiene FK constraints; la integridad debe manejarse en la aplicación |
| Unique constraints | ⚠️ Manuales | Se deben crear índices únicos, no son parte del schema declarativo |
| Volumen | ⚠️ Excesivo | MongoDB está diseñado para millones de documentos, no para ~4250 registros |
| Consistencia transaccional | ⚠️ Limitada | Transacciones multi-document requieren configuración especial |
| Adecuación al problema | ❌ Mala | El modelo relacional de la SPEC encaja perfectamente con SQL, no con documentos |

**Veredicto:** Descartado. MongoDB es inadecuado para un modelo de datos relacional con constraints de unicidad e integridad referencial. Requeriría abandonar el ORM nativo de Django y manejar la consistencia manualmente.

### Alternativa 3: Archivos JSON/CSV

**Descripción:** Persistir datos en archivos planos (JSON o CSV) en el sistema de archivos, accedidos directamente desde Python.

| Criterio | Evaluación | Razón de rechazo |
|---|---|---|
| Setup | ⭐⭐⭐⭐⭐ | Sin dependencias, archivos planos |
| Consultas | ❌ Muy limitadas | No hay JOINs, no hay índices, no hay UNIQUE constraints |
| Integridad referencial | ❌ Inexistente | No hay FK constraints; la integridad es 100% manual |
| Concurrencia | ❌ Problemática | Múltiples usuarios escribiendo al mismo archivo causan corrupción |
| Rendimiento en consultas | ❌ O(n) completo | Cada consulta lee todo el archivo; sin índices |
| Escalabilidad | ❌ Muy baja | Degrada rápidamente con el volumen |
| Adecuación al problema | ❌ Inadecuada | El modelo de datos requiere relaciones y constraints que los archivos planos no ofrecen |

**Veredicto:** Descartado. Los archivos planos no ofrecen integridad referencial, constraints ni consultas eficientes. Son inadecuados para un sistema con relaciones N-M y requisitos de consistencia.

## Consecuencias

### Positivas

- **Setup instantáneo:** `python manage.py migrate` crea la base de datos automáticamente
- **Desarrollo ágil:** Django admin panel permite inspeccionar y modificar datos sin herramientas externas
- **Migraciones:** El schema se versiona con el código (`makemigrations` genera archivos Python)
- **Testing:** Cada test crea una DB temporal, sin contaminar datos de desarrollo
- **Portabilidad:** El archivo `.db` se puede copiar, respaldar y compartir fácilmente
- **Migración futura:** Cambiar a PostgreSQL requiere modificar `settings.py` y ejecutar `migrate` — una línea de configuración (ADR-001)

### Negativas / Riesgos

- **Concurrencia de escritura:** SQLite bloquea el archivo completo durante escrituras. Con ~200 usuarios concurrentes el riesgo es bajo, pero si el tráfico creciera, habría que migrar a PostgreSQL
- **Sin replicación:** No hay backup automático ni replicación. Se debe respaldar manualmente el archivo `.db`
- **Límite de tamaño:** SQLite tiene un límite teórico de 140 TB, pero en la práctica se recomienda < 1 GB para mejor rendimiento (no es un riesgo con ~4250 registros)
- **Sin stored procedures:** Toda la lógica de negocio está en Python/Django, no en la DB (aceptable para este proyecto)

## Referencias

- ADR-001: Stack Tecnológico (SQLite como DB de desarrollo)
- SPEC v2: Sección 4 (Tech Stack), Sección 5 (Data Contracts)
- [SQLite vs PostgreSQL for Django](https://www.sqlite.org/whentouse.html) — SQLite recomienda para uso local/concurrento moderado
- [Django Database Backends](https://docs.djangoproject.com/en/4.2/ref/databases/) — Documentación oficial de Django
