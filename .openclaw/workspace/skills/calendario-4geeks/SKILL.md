# calendario-4geeks

Consultar los eventos de 4Geeks Academy de los próximos 7 días.

## ¿Qué hace?

Llama a la API de Breathecode (`/v1/events/me`) con el token `FOURGEEKS_TOKEN`, filtra los eventos cuya `starting_at` caiga en los próximos 7 días, y devuelve una lista clara con:
- Título del evento
- Fecha y hora (formato legible, hora local de Madrid)
- Enlace a la retransmisión en vivo si existe (`live_stream_url`)

## Input requerido

Ninguno. Usa la variable de entorno `FOURGEEKS_TOKEN`.

Para activar la skill, pregúntale a Kai:
- "¿qué eventos tengo en 4Geeks esta semana?"
- "mis próximos eventos de 4Geeks"

## Output

Lista de eventos con formato:

```
📅 Your Way Into the AI Jobs That Pay 62% More
   🕐 miércoles 4 nov 2026, 00:00 CET
   🔗 https://meet.google.com/odd-axjw-dcb

📅 El Agente Caza-Ofertas (DIA 2)
   🕐 miércoles 18 nov 2026, 19:30 CET
   🔗 https://www.youtube.com/watch?v=M9rSjoSN7lI
```

## Endpoint utilizado

```
GET https://breathecode.herokuapp.com/v1/events/me
Authorization: Token ${FOURGEEKS_TOKEN}
```

Filtro: `events[?starting_at >= now && starting_at <= now + 7 days]`

## Dependencias

- Variable de entorno `FOURGEEKS_TOKEN`
- `curl`
- `python3`