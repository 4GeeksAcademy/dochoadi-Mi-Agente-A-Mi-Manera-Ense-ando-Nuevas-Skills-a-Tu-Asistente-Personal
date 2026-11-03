# Mi Agente a Mi Manera: Enseñando Nuevas Skills a Tu Asistente Personal

Repositorio de documentación y entrega de los ejercicios de 4Geeks Academy sobre **OpenClaw**: despliegue de una instancia autoalojada en un VPS, conexión a un modelo de IA, ampliación con integraciones externas, y personalización del agente con skills propias. Este mismo tipo de repositorio (heredado del repo de clase original) se usa para documentar los ejercicios del curso relacionados, cada uno en su propia sección de `TRACKING.md`.

## Contenido de este repositorio
- [`CONTEXT.md`](./CONTEXT.md) — contexto de los ejercicios, alcance y criterios de evaluación
- [`TRACKING.md`](./TRACKING.md) — seguimiento del estado de cada actividad, por ejercicio
- [`ERRORS.md`](./ERRORS.md) — registro de errores encontrados y cómo se resolvieron
- [`SKILLS_DESIGN.md`](./SKILLS_DESIGN.md) — diseño de las skills del proyecto 3 (qué hace, qué input necesita, cómo es un buen output), commiteado antes de implementar
- [`SKILL_LOG.md`](./SKILL_LOG.md) — log de la conexión de OpenClaw con GitHub (`gh` CLI) y de las 6 skills creadas contra la API de 4Geeks Academy (proyecto 4): prompt, endpoint usado, prueba y resultado de cada una
- [`.openclaw/workspace/`](./.openclaw/workspace/) — el workspace real de OpenClaw (identidad, memoria, logs)
- [`openclaw-connection/`](./openclaw-connection/) — evidencias (capturas y notas) del ejercicio de conexión con Telegram/Zapier

## Ejercicios documentados aquí

### 1. Configurando tu Agente de IA Personal con OpenClaw
Instalación y configuración básica en el VPS, conexión a LiteLLM, personalización de identidad y respaldo del workspace en GitHub. Se decidió usar este mismo repositorio (asignado por el classroom) en vez de crear un repo nuevo `openclaw-<usuario>` — ver `TRACKING.md`.

### 2. Conecta tu Agente: Telegram, Google Docs y Calendario
Conexión del agente a Telegram y, vía el MCP de Zapier, a Google Docs y Google Calendar — ver sección correspondiente en `TRACKING.md`.

### 3. Mi Agente a Mi Manera: Enseñando Nuevas Skills a Tu Asistente Personal
Configuración de los cinco archivos de identidad del agente (`IDENTITY.md`, `SOUL.md`, `AGENTS.md`, `USER.md`, `TOOLS.md`) e implementación de al menos dos skills personalizadas de OpenClaw que usan las integraciones ya conectadas (Zapier/Telegram) — ver sección correspondiente en `TRACKING.md` y el detalle de diseño en `SKILLS_DESIGN.md`.

### 4. Conectar tu Agente de IA a la API de 4Geeks Academy
Conexión de OpenClaw a GitHub (identidad nativa de `gh` CLI, sin tocar el Control UI ni ninguna API/OAuth nueva) y a la API de 4Geeks Academy (Breathecode), e implementación de 6 skills nuevas de un único endpoint cada una (autenticación, proyectos, pendientes, progreso, calendario de eventos y certificados) — ver sección correspondiente en `TRACKING.md` y el detalle de cada skill en `SKILL_LOG.md`.
