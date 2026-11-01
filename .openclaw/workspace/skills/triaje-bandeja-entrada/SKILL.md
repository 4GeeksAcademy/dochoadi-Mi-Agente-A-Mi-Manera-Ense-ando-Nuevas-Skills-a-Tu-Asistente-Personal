---
name: triaje-bandeja-entrada
description: Revisa los correos no leídos de Gmail, identifica cuáles requieren una acción, y crea una Google Task por cada uno con una descripción breve. Actívala cuando pida "triaje mi bandeja", "revisa mis correos pendientes" o expresiones equivalentes.
---

# Skill: Triaje de bandeja de entrada

## Descripción
Revisa mis correos no leídos en Gmail, identifica los que requieren una acción de mi parte, y crea una Google Task por cada uno con una descripción breve de qué hay que hacer.

## Cuándo usar
Cuando pida "triaje mi bandeja", "revisa mis correos pendientes", "qué emails tengo que atender" o active esta skill explícitamente.

## Prerrequisitos
- Gmail y Google Tasks disponibles vía Zapier (cuenta `dodochoadiaz@gmail.com`, ver `TOOLS.md`).

## Procedimiento
1. Lista los correos no leídos de la bandeja de entrada.
2. Para cada uno, decide si requiere una acción mía (responder, revisar, decidir algo) o es informativo/spam/notificación automática.
3. Para los que requieren acción, crea una Google Task con: asunto breve (remitente + tema) y una frase de qué hay que hacer.
4. No marca los correos como leídos ni los archiva — solo crea las tareas.
5. Confirma por chat cuántas tareas creó, con una línea por tarea.

## Salida esperada
N tareas nuevas visibles en Google Tasks, correspondientes a correos reales no leídos, cada una con una descripción clara de la acción requerida. El agente confirma el resultado por chat.

## Casos especiales
- Si no hay correos no leídos que requieran acción, informa "Sin correos pendientes de acción" y no crea tareas vacías.
- Si un correo es ambiguo sobre si requiere acción, créale la tarea de todas formas (mejor una tarea de más que perder algo importante) y dilo explícitamente en el resumen.
- Crear tareas en mi propia lista no es una acción saliente hacia terceros, así que no requiere confirmación previa según `AGENTS.md`/`TOOLS.md` — pero nunca responde ni reenvía ningún correo por su cuenta.
