---
name: plan-semana
description: A partir de mis objetivos y compromisos de la semana, escribe un plan priorizado en un Google Doc nuevo y crea los eventos clave en Google Calendar. Actívala cuando pida "plan de la semana", "organiza mi semana" o expresiones equivalentes.
---

# Skill: Plan de semana

## Descripción
A partir de mis objetivos y compromisos de la semana, escribe un plan semanal priorizado, lo guarda como un Google Doc nuevo, y crea los eventos clave en mi Google Calendar.

## Cuándo usar
Cuando pida "plan de la semana", "organiza mi semana", "dame un plan para esta semana" o active esta skill explícitamente.

## Prerrequisitos
- Google Docs y Google Calendar disponibles vía Zapier (cuenta `dodochoadiaz@gmail.com`, calendario por defecto, ver `TOOLS.md`).
- Un listado de objetivos/compromisos de la semana en lenguaje natural — si no lo doy, pregunta por mis prioridades antes de generar el plan (no las inventa).

## Procedimiento
1. Recoge mis objetivos y compromisos de la semana (del mensaje, o preguntando si no los he dado).
2. Organiza el plan por prioridad/urgencia, sin modificar el objetivo original de cada punto.
3. Crea un Google Doc nuevo con el plan semanal, estructurado por día o por prioridad.
4. Identifica qué puntos tienen fecha/hora concreta o la necesitan, y crea esos eventos en Calendar; si falta fecha/hora para alguno, pregunta antes de crear el evento.
5. Confirma por chat qué Doc creó (enlace) y qué eventos creó (con hora), en tono directo.

## Salida esperada
Un Google Doc nuevo con el plan priorizado y los eventos clave creados en Calendar, reflejando fielmente lo que pedí — no una plantilla genérica de "productividad". El agente confirma el resultado con enlaces/horas.

## Casos especiales
- Si doy menos de tres prioridades, trabaja solo con las que di — no completa la lista con tareas inventadas.
- Si hay ambigüedad de fecha/hora para un evento, pregunta antes de crearlo en vez de asumir un horario.
- Si ya existe un evento que choca con uno nuevo, lo señala en el resumen en vez de crear el conflicto en silencio.
