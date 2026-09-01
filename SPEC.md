# SPEC-001: Leonardo Krell / Portal de Calificaciones

## 1. Contexto y Propósito

En el IES 9-018, estudiantes y docentes necesitan un lugar centralizado para ver y gestionar calificaciones de forma clara y organizada.

**Problema:** Actualmente no hay un sistema digital que permita:
- Docentes cargar calificaciones de forma ágil
- Estudiantes ver sus notas en tiempo real
- Validación automática de datos

**Solución:** Construir un portal web donde docentes cargan calificaciones y estudiantes las consultan, con autenticación y permisos específicos por rol.

---

## 2. Requerimientos Funcionales

### 2.1 Autenticación

- [ ] **RF-01:** El sistema autentica usuarios con email y contraseña
- [ ] **RF-02:** El sistema asigna roles automáticamente: `estudiante`, `docente`
- [ ] **RF-03:** El sistema mantiene la sesión del usuario durante la navegación

### 2.2 Funcionalidades del Estudiante

- [ ] **RF-04:** Un estudiante puede ver todas sus calificaciones de todas las materias
- [ ] **RF-05:** El estudiante ve la calificación agrupada por materia, con nombre del docente
- [ ] **RF-06:** El estudiante ve un promedio general de todas las materias
- [ ] **RF-07:** El estudiante NO puede ver calificaciones de otros estudiantes
- [ ] **RF-08:** El estudiante NO puede editar, eliminar ni cargar calificaciones

### 2.3 Funcionalidades del Docente

- [ ] **RF-09:** Un docente puede ver todas las materias que dicta
- [ ] **RF-10:** El docente puede cargar calificaciones para una materia específica (formulario o CSV)
- [ ] **RF-11:** El docente puede ver la lista de estudiantes de una materia con sus calificaciones
- [ ] **RF-12:** El docente puede editar/actualizar calificaciones de sus propias materias
- [ ] **RF-13:** El docente NO puede ver/editar materias de otros docentes
- [ ] **RF-14:** El docente NO puede ver calificaciones de otras materias

### 2.4 Validaciones y Negocio

- [ ] **RF-15:** El sistema valida que las calificaciones estén entre 0 y 10
- [ ] **RF-16:** El sistema rechaza calificaciones no numéricas o vacías
- [ ] **RF-17:** El sistema NO permite que dos estudiantes tengan dos calificaciones para la misma materia

---

## 3. Non-Goals (Límites del Alcance)

Lo que explícitamente NO se construirá en esta etapa:

- **NG-01:** No hay panel de coordinador ni auditoría de cambios
- **NG-02:** No hay notificaciones por email/SMS
- **NG-03:** No hay gráficos ni análisis estadísticos
- **NG-04:** No hay asistencia ni presencia de estudiantes
- **NG-05:** No hay exportación de reportes en Excel o PDF
- **NG-06:** No hay integración con sistemas externos
- **NG-07:** No hay recuperación de contraseña automática
- **NG-08:** No hay gestión de usuarios (crear/eliminar/editar perfiles)

---

## 4. Stack Tecnológico y Restricciones

### Backend
- **Lenguaje:** Python
- **Framework:** Django

### Frontend
- **Tecnología:** HTML5 + CSS

### Base de Datos
- **Desarrollo:** SQLite

---

## 5. Contratos de Datos / Tipos

### Entidades Principales

```python
# ==================== USUARIO ====================
class Usuario:
    id: int
    email: str (único, validado)
    password: str (hasheada)
    nombre_completo: str
    rol: "estudiante" | "docente"
    activo: bool (default True)
    fecha_creacion: datetime

# ==================== MATERIA ====================
class Materia:
    id: int
    codigo: str (único, ej: "MAT-101")
    nombre: str (ej: "Matemática II")
    descripcion: str (opcional)
    docente: FK(Usuario, rol=docente)
    cuatrimestre: int (1 o 2, opcional)
    año: int (ej: 2026)
    activa: bool (default True)
    fecha_creacion: datetime

# ==================== INSCRIPCION ====================
class Inscripcion:
    id: int
    estudiante: FK(Usuario, rol=estudiante)
    materia: FK(Materia)
    fecha_inscripcion: datetime
    # Constraint: (estudiante, materia) es única

# ==================== CALIFICACION ====================
class Calificacion:
    id: int
    estudiante: FK(Usuario, rol=estudiante)
    materia: FK(Materia)
    nota: float (0-10, validado)
    cargada_por: FK(Usuario, rol=docente)  # quién cargó
    fecha_carga: datetime
    fecha_ultima_modificacion: datetime (nullable)
    observaciones: str (max 500, opcional)
    # Constraint: (estudiante, materia) es única
