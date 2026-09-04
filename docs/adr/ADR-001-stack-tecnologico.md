# ADR-001: Selección del Stack Tecnológico

**Nombre:** Leonardo Krell
**Fecha:** 2026-09-01  
**Estado:** Aceptado  
 
## Contexto

El Portal de Calificaciones Académicas necesita una arquitectura técnica que permita:
- Autenticación de usuarios con roles diferenciados (docente, estudiante)
- Gestión de datos con relaciones complejas (estudiante → materias → calificaciones)
- Validación automática de datos (notas 0-10, unicidad estudiante-materia)
- Desarrollo ágil con documentación automática

Se necesita un stack que sea simple para MVP pero extensible para agregar features sin refactorizar base.

## Decisión

**Stack elegido:**

| Componente | Tecnología |
|-----------|-----------|
| Backend | Python + Django |
| Frontend | HTML5 |
| Base de datos | SQLite3 |
| ORM | Django ORM nativo |
| Autenticación | Django Sessions |
| Testing | pytest + pytest-django |

**Razones principales:**

1. **Django ORM:** Maneja relaciones complejas (N-M) con una línea. Validaciones automáticas en modelos.
2. **Admin panel automático:** Django genera CRUD sin código extra (útil para coordinador futuro).
3. **Validaciones nativas:** `validators` en modelos + formularios reutilizables. No requiere capas de validación manual.
4. **Comunidad Python:** Los estudiantes conocen Python. Hay documentación abundante.
5. **Bootstrap:** Framework responsive, desarrollo rápido sin complejidad de React/Vue.
6. **Portabilidad BD:** Cambiar SQLite → PostgreSQL es UNA línea en settings.py.
7. **Escalabilidad:** Django REST Framework en TP4 agrega API sin cambiar models/tests.

## Alternativas Descartadas

### Alternativa A: FastAPI + React

**Características:** Backend async (FastAPI), frontend con componentes (React), PostgreSQL.

**Por qué NO:**

- **ORM:** FastAPI requiere SQLAlchemy extra. Django ORM es nativo, más simple.
- **Setup:** FastAPI + React requiere Node.js, npm, webpack, build process. Django: `pip install` y listo.
- **Validaciones:** Pydantic en FastAPI es manual. Django `validators` son automáticos en modelos y forms.
- **Admin panel:** FastAPI no genera admin. Django genera CRUD sin código.
- **Async complexity:** Async/await en FastAPI es complejo para estudiantes acostumbrados a código síncrono Django.
- **React overhead:** CRUD simple (tablas, formularios) no necesita componentes. Bootstrap alcanza.
- **Curva aprendizaje:** 2 lenguajes (Python + JavaScript). Django: 1 lenguaje, 1 paradigma.

**Cuándo sería mejor:** Si el proyecto requiere dashboard interactivo con gráficos en tiempo real (no es el caso).

---

### Alternativa B: Node.js + Express + EJS

**Características:** Backend lightweight (Express), templating (EJS), MongoDB/PostgreSQL.

**Por qué NO:**

- **ORM:** Express requiere Prisma/TypeORM (menos maduro que Django ORM). Más setup.
- **Type safety:** JavaScript débilmente tipado. TypeScript suma complejidad.
- **Validaciones:** Joi/Yup requieren dependencias extra. Django: validators built-in.
- **Admin panel:** Express NO genera admin. Django lo hace automáticamente.
- **Comunidad Python:** Estudiantes conocen Python. Node.js es cambio innecesario.
- **Dependencias frágiles:** Ecosistema Node es grande y versiones se deprecan rápido.

**Cuándo sería mejor:** Si ya se dominaba Node.js y era más rápido.

## Consecuencias

### ✅ Positivas

- **Desarrollo ágil:** Scaffold automático (modelos → migrations → admin panel en minutos).
- **Validaciones nativas:** Range 0-10, unique_together se validan en modelo. No requiere lógica extra en vistas.
- **Seguridad:** CSRF protection, password hashing, SQL injection prevention son automáticos.
- **Testeo:** pytest + pytest-django con fixtures reutilizables.
- **Escalabilidad BD:** Cambiar SQLite → PostgreSQL es 1 línea en settings.
- **Escalabilidad API:** Django REST Framework genera Swagger automático. No requiere reescribir modelos.

### ⚠️ Negativas / Riesgos

- **Performance Python:** Python es más lento que Node.js. Mitigación: Para MVP < 1000 usuarios no es issue.
- **Frontend básico:** Bootstrap es funcional pero no "premium".
- **Deployment Windows:** Python en Windows puede ser frágil.
