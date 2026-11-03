# 4geeks-pendientes

Listar los proyectos o tareas pendientes de entregar en 4Geeks Academy.

## ¿Qué hace?

Filtra los cohorts del alumno cuyo `educational_status` sea `ACTIVE` — esto indica que el proyecto está en curso y aún no se ha completado/entregado.

## Input requerido

Ninguno. Usa la variable de entorno `FOURGEEKS_TOKEN`.

## Output

Lista de proyectos pendientes con formato:

```
📋 Trabajo pendiente (ACTIVE):
  - Data pipelines
  - Asynchronous processing and offloading
  - Command Line, Git and Github
  - ...
```

## Endpoint utilizado

```
GET https://breathecode.herokuapp.com/v1/admissions/user/me
Authorization: Token ***
```

Filtro aplicado: `cohorts[?educational_status == "ACTIVE"].cohort.syllabus_version.name`

## Dependencias

- Variable de entorno `FOURGEEKS_TOKEN`
- `curl`
- `python3`