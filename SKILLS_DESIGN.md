# SKILLS_DESIGN.md

Diseño de las skills personalizadas de Kai para el proyecto "Mi Agente a Mi Manera: Enseñando Nuevas Skills a Tu Asistente Personal". Cada skill responde a las tres preguntas de diseño antes de implementarse. Ninguna configura una API, flujo OAuth o servicio externo nuevo — todas usan las conexiones ya existentes (Zapier: Google Docs, Calendar, Gmail, Drive, Tasks, GitHub; Telegram).

---

## Skill 1 — Diario de aprendizaje

**¿Qué hace esta skill?**
Le cuento a Kai qué aprendí hoy (unos pocos puntos) y añade una entrada estructurada a un Google Doc que uso como diario de aprendizaje — útil para la propia formación de 4Geeks y para mi objetivo de pasar a Director de CS en SaaS/ciberseguridad.

**¿Qué input necesita el agente?**
Unos pocos puntos en lenguaje natural sobre lo aprendido hoy (p. ej. de la formación de IA, de mis proyectos personales, o de mi trabajo). Ya sabe por `USER.md` mi perfil y objetivo profesional, y por `TOOLS.md` la cuenta de Google Docs a usar.

**¿Cómo es un buen output?**
Una entrada nueva, fechada y bien formateada, añadida al final del Google Doc de diario (si no existe, lo crea la primera vez). Se sabe que funcionó si la entrada aparece en el Doc con fecha real y contenido real, no genérico.

> **Nota:** la skill originalmente diseñada aquí (`resumen-github-diario`) se descartó — ver `ERRORS.md`, sección "GitHub, Google Drive y Google Tasks no están conectados vía Zapier": requería GitHub, que no estaba conectado, y el enunciado prohíbe configurar una conexión nueva como parte de este proyecto.

---

## Skill 2 — Triaje de bandeja de entrada → Google Tasks

**¿Qué hace esta skill?**
Revisa mis correos no leídos en Gmail, identifica cuáles requieren una acción de mi parte, y crea una Google Task por cada uno con una descripción breve de qué hay que hacer.

**¿Qué input necesita el agente?**
Un mensaje de activación ("triaje mi bandeja", "revisa mis correos pendientes"). Ya sabe por `TOOLS.md` qué cuenta de Gmail usar (`dodochoadiaz@gmail.com`) y que crear tareas internas (a diferencia de enviar algo a un tercero) no requiere confirmación previa por `AGENTS.md`, ya que no es una acción saliente visible para terceros.

**¿Cómo es un buen output?**
N tareas nuevas en Google Tasks, cada una con el asunto/remitente del correo y una frase de qué acción requiere. Al terminar, Kai confirma por chat cuántas tareas creó y un resumen de una línea por cada una. Se sabe que funcionó si las tareas aparecen en Google Tasks y corresponden a correos reales no leídos.

---

## Skill 3 — Borrador de email con mi voz

**¿Qué hace esta skill?**
A partir de una idea breve mía (destinatario + qué quiero decir), redacta un borrador de email ya con mi firma y mi tono directo, listo para revisar antes de enviar.

**¿Qué input necesita el agente?**
Destinatario y una idea breve del contenido (en lenguaje natural, no un email ya escrito). Ya sabe por `TOOLS.md` la firma por defecto ("Daniel Ochoa", sin cargo) y por `USER.md` el tono (directo, sin frases de relleno, sin halagos vacíos).

**¿Cómo es un buen output?**
Un borrador guardado en Gmail (Drafts) o el texto completo propuesto en el chat, pendiente de mi confirmación explícita antes de enviarse — nunca se envía solo, por la regla de `AGENTS.md`/`TOOLS.md` de confirmar toda acción saliente. Se sabe que funcionó si el borrador necesita mínimas correcciones mías y respeta el tono y la firma definidos.

---

## Skill 4 — Plan de semana → Doc + eventos de Calendar

**¿Qué hace esta skill?**
A partir de mis objetivos y compromisos de la semana, escribe un plan semanal priorizado, lo guarda como un Google Doc nuevo, y crea los eventos clave en Google Calendar.

**¿Qué input necesita el agente?**
Un listado de objetivos/compromisos de la semana en lenguaje natural (p. ej. "esta semana: cerrar el QBR de un cliente, avanzar en el build de iOS de Padel School Manager, entrevista de trabajo el jueves"). Ya sabe por `TOOLS.md` el calendario por defecto a usar y que debe confirmar antes de crear eventos si hay ambigüedad en fecha/hora.

**¿Cómo es un buen output?**
Un Google Doc nuevo con el plan priorizado (bloques por día o por prioridad) y los eventos correspondientes creados en Calendar, con Kai confirmando por chat qué documento y qué eventos creó (con enlaces/horas). Se sabe que funcionó si el Doc y los eventos existen y reflejan fielmente lo que pedí, no una plantilla genérica.
