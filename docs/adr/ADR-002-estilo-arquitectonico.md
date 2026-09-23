# ADR-002: Estilo Arquitectónico

| Campo | Valor |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 2026-09-22 |
| **Decisor** | Leonardo Krell |
| **ADR relacionado** | ADR-001 (Stack Tecnológico) |

## Contexto

El Portal de Calificaciones es un sistema web académico con las siguientes características:

- **Equipo:** 1 desarrollador (estudiante)
- **Alcance:** MVP funcional para el instituto IES 9-018
- **Usuarios finales:** ~100-200 (estudiantes y docentes)
- **Deadline:** TP2 (29 sep), TP3 (Sprint 2), TP4 (API), TP5 (Mobile), TP6 (CI/CD)
- **Complejidad de datos:** 4 entidades con relaciones N-M simples
- **Integraciones:** Ninguna (Non-Goal en SPEC v2)

Se necesita un estilo arquitectónico que permita desarrollo ágil, mantenimiento predecible y escalabilidad suficiente para el contexto académico.

## Decisión

**Monolito modular** con Django como framework principal.

El sistema se organiza como una única aplicación desplegable (`portal_calificaciones`) con módulos internos separados por dominio (`usuarios/`, `calificaciones/`), compartiendo:

- Una única base de datos SQLite
- Un único proceso de servidor web (Django dev server / Gunicorn)
- Un único repositorio de código
- Un pipeline de despliegue simple

### Criterios de la decisión

| Criterio | Puntuación | Justificación |
|---|---|---|
| Tamaño del equipo (1 persona) | ⭐⭐⭐⭐⭐ | Un solo codebase, sin coordinación entre servicios |
| Velocidad de desarrollo | ⭐⭐⭐⭐⭐ | Scaffold → migrations → admin en minutos (ADR-001) |
| Complejidad operativa | ⭐⭐⭐⭐⭐ | Un solo `manage.py runserver`, una DB, un deploy |
| Costo de infraestructura | ⭐⭐⭐⭐⭐ | Servidor único, SQLite sin configuración externa |
| Deadline PP3 (Sprint 2) | ⭐⭐⭐⭐⭐ | Diagramas C4 ya listos, código rápido de iterar |
| Mantenibilidad | ⭐⭐⭐⭐ | Modular dentro de Django, separación por apps |

## Alternativas Descartadas

### Alternativa 1: Microservicios

**Descripción:** Separar autenticación, calificaciones e inscripciones en servicios independientes comunicados vía HTTP o mensajería.

| Criterio | Evaluación | Razón de rechazo |
|---|---|---|
| Tamaño del equipo (1 persona) | ❌ Alto overhead | Coordinar 3+ repos, 3+ deploys, 3+ bases de datos siendo 1 solo desarrollador |
| Velocidad de desarrollo | ❌ Lenta | Configurar Docker, red, discovery de servicios consume tiempo que no hay |
| Complejidad operativa | ❌ Alta | Monitoreo, logs distribuidos, tolerancia a fallos — sin justificación para MVP |
| Costo | ❌ Alto | Múltiples instancias, posible necesidad de Kubernetes o ECS |
| Deadline PP3 | ❌ Riesgo alto | La complejidad operativa retrasa el desarrollo funcional |
| Adecuación al problema | ❌ Excesivo | 4 entidades simples no justifican la separación |

**Veredicto:** Descartado. La complejidad operativa y el overhead de coordinación no se justifican para un equipo de 1 persona y un sistema de 4 entidades.

### Alternativa 2: Serverless (AWS Lambda / Cloud Functions)

**Descripción:** Funciones independientes por endpoint, base de datos serverless (DynamoDB o Aurora Serverless), API Gateway como punto de entrada.

| Criterio | Evaluación | Razón de rechazo |
|---|---|---|
| Tamaño del equipo (1 persona) | ❌ Curva de aprendizaje | AWS/GCP requiere conocimiento específico, no es el stack del curso |
| Velocidad de desarrollo | ❌ Lenta | Configuración de IAM, permisos, cold starts, debugging complejo |
| Complejidad operativa | ❌ Media-Alta | Observabilidad distribuida, costos variables, límites de ejecución |
| Costo | ⚠️ Variables | Free tier puede funcionar, pero costos impredecibles con tráfico |
| Deadline PP3 | ❌ Riesgo alto | Tiempo invertido en infraestructura vs. funcionalidad |
| Adecuación al problema | ⚠️ Viable pero innecesario | El sistema no tiene tráfico variable ni eventos asíncronos |

**Veredicto:** Descartado. El modelo serverless introduce complejidad de infraestructura y dependencia de proveedor cloud que no se justifica para un sistema CRUD monolítico con tráfico predecible.

### Alternativa 3: Arquitectura Hexagonal (Ports & Adapters)

**Descripción:** Separar la lógica de negocio del framework (Django) usando puertos y adaptadores, permitiendo cambiar de framework o DB sin afectar el dominio.

| Criterio | Evaluación | Razón de rechazo |
|---|---|---|
| Tamaño del equipo (1 persona) | ⚠️ Overhead innecesario | Más capas de abstracción para un solo desarrollador |
| Velocidad de desarrollo | ⚠️ Moderada | Más archivos, más interfaces, más boilerplate |
| Complejidad operativa | ✅ Baja | Misma que monolito, pero con más código interno |
| Flexibilidad | ⭐⭐⭐⭐⭐ | Permite cambiar de framework o DB fácilmente |
| Deadline PP3 | ⚠️ Riesgo medio | La abstracción consume tiempo que podría usarse en funcionalidad |
| Adecuación al problema | ⚠️ Excesiva | No se anticipa cambio de framework ni DB en este proyecto |

**Veredicto:** Descartado. Si bien offers flexibilidad máxima, la complejidad adicionada no se justifica dado que el stack (Django + SQLite) está definido en ADR-001 y no se prevé cambiar.

## Consecuencias

### Positivas

- **Desarrollo rápido:** Scaffold, migraciones y admin panel de Django permiten prototipar en minutos
- **Deploy simple:** Un solo comando `manage.py runserver` o Gunicorn, una base de datos SQLite
- **Dependencias mínimas:** Solo Python + Django + Bootstrap, sin orquestación de servicios
- **Trazabilidad clara:** Cada app Django (`usuarios/`, `calificaciones/`) mapea directamente a un dominio de la SPEC
- **Escalabilidad suficiente:** Para ~200 usuarios, un monolito Django es más que suficiente
- **Facilidad de debugging:** Todo en un proceso, logs centralizados, breakpoints funcionan sin configuración

### Negativas / Riesgos

- **Punto único de fallo:** Si el proceso Django cae, todo el sistema cae (aceptable para contexto académico)
- **Acoplamiento entre módulos:** Las apps de Django comparten el mismo proceso; un error en `usuarios/` puede afectar `calificaciones/` (mitigado con buenas prácticas de separación)
- **Escalabilidad limitada:** Si el sistema creciera a miles de usuarios concurrentes, sería necesario escalar horizontalmente (no es el caso actual)
- **Despliegue atómico:** Cualquier cambio requiere redeploy completo del monolito (aceptable para equipo de 1 persona)

## Referencias

- ADR-001: Stack Tecnológico (aceptado)
- SPEC v2: Sección 4 (Tech Stack), Non-Goals
- [The Monolith vs. Microservices Debate](https://martinfowler.com/bliki/MonolithFirst.html) — Fowler recomienda empezar con monolito
