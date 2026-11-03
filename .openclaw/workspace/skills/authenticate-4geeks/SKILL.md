# authenticate-4geeks

Verificar si el token de estudiante almacenado en `FOURGEEKS_TOKEN` es válido y la sesión está activa en 4Geeks Academy.

## ¿Qué hace?

Realiza una llamada GET a la API de Breathecode (`/v1/admissions/user/me`) con el token de estudiante y devuelve:
- Nombre completo del estudiante
- Email asociado
- Estado de los cohorts activos
- Fecha de alta

## Input requerido

Ninguno. La skill lee automáticamente la variable de entorno `FOURGEEKS_TOKEN` definida en el sistema.

## Output

- ✅ **Token válido** → `first_name` + `last_name` del alumno + resumen de cohorts activos
- ❌ **Token inválido/expirado** → mensaje de error con el código HTTP devuelto por la API

## Endpoint utilizado

```
GET https://breathecode.herokuapp.com/v1/admissions/user/me
Authorization: Token ${FOURGEEKS_TOKEN}
```

## Dependencias

- Variable de entorno `FOURGEEKS_TOKEN` configurada en el sistema
- Entrada `authenticate-4geeks` en `skills.entries` de `openclaw.json` (ya existe)
- `curl` disponible en el sistema (preinstalado)