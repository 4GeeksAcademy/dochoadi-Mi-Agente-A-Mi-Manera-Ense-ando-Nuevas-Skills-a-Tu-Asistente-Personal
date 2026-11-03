# 4geeks-progreso

Consultar el porcentaje global de avance en el curso de 4Geeks Academy.

## ¿Qué hace?

Obtiene el perfil del alumno y calcula estadísticas globales sobre los cohorts:
- Total de proyectos (cohorts)
- Completados (GRADUATED) con porcentaje
- En curso (ACTIVE) con porcentaje
- No completados (NOT_COMPLETING) con porcentaje

## Input requerido

Ninguno. Usa la variable de entorno `FOURGEEKS_TOKEN`.

## Output

Resumen de progreso con formato:

```
📊 Progreso global en 4Geeks Academy
   Total proyectos: 32
   ✅ Completados: 4 (12%)
   📋 En curso: 27 (84%)
   ⏸️ No completados: 1 (3%)
```

## Endpoint utilizado

```
GET https://breathecode.herokuapp.com/v1/admissions/user/me
Authorization: Token ***
```

Cálculo: se evalúa `educational_status` de cada cohort y se agrupa por categoría.

## Dependencias

- Variable de entorno `FOURGEEKS_TOKEN`
- `curl`
- `python3`