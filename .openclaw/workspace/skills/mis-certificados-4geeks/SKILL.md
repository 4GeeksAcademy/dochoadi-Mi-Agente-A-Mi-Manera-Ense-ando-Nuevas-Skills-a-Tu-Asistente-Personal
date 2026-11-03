# mis-certificados-4geeks

Consultar los certificados obtenidos en 4Geeks Academy.

## ¿Qué hace?

Llama a la API de Breathecode (`/v1/certificate/me`) con el token `FOURGEEKS_TOKEN` y devuelve, por cada certificado:
- Nombre del curso (especialidad)
- Academia (4Geeks Madrid)
- Estado (PERSISTED = generado)
- Fecha de emisión

## Input requerido

Ninguno. Usa la variable de entorno `FOURGEEKS_TOKEN`.

Para activar la skill, pregúntale a Kai:
- "enséñame mis certificados de 4Geeks"
- "qué certificados tengo en 4Geeks"

## Output

Lista de certificados con formato:

```
🎓 Basic personal assistants with Openclaw
   🏛 4Geeks Madrid
   📄 Estado: PERSISTED (generado)
   📅 Emitido: 28 oct 2026
   🔗 PDF: https://certificate.4geeks.com/pdf/27daed3b...
```

## Endpoint utilizado

```
GET https://breathecode.herokuapp.com/v1/certificate/me
Authorization: Token ${FOURGEEKS_TOKEN}
```

Datos extraídos de `specialty.name`, `academy.name`, `status`, `issued_at`, `pdf_url`.

## Dependencias

- Variable de entorno `FOURGEEKS_TOKEN`
- `curl`
- `python3`