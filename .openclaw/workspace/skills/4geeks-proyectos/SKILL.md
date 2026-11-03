# 4geeks-proyectos

Consultar la lista de proyectos (cohorts/syllabi) asignados en 4Geeks Academy con su estado actual.

## ¿Qué hace?

Obtiene el perfil del alumno desde la API de Breathecode y extrae la lista completa de cohorts con:
- Nombre del syllabus/proyecto
- Estado educacional: `ACTIVE`, `GRADUATED`, `NOT_COMPLETING`
- Estado financiero: `UP_TO_DATE`

## Input requerido

Ninguno. Usa la variable de entorno `FOURGEEKS_TOKEN`.

## Output

Lista de proyectos con formato:

```
📌 Data pipelines | Estado: ACTIVE | Financiero: UP_TO_DATE
✅ Working with AI coding agents | Estado: GRADUATED | Financiero: UP_TO_DATE
⏸️ AI Engineering Introduction | Estado: NOT_COMPLETING | Financiero: UP_TO_DATE
```

## Endpoint utilizado

```
GET https://breathecode.herokuapp.com/v1/admissions/user/me
Authorization: Token ***
```

Los proyectos se leen de `cohorts[].cohort.syllabus_version.name` y el estado de `cohorts[].educational_status`.

## Dependencias

- Variable de entorno `FOURGEEKS_TOKEN`
- `curl`
- `python3`