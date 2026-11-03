# Contexto del ejercicio

## Objetivo
Desplegar y configurar una instancia funcional de OpenClaw en un VPS propio, conectarla a un modelo de IA mediante LiteLLM, y verificar que responde correctamente en el chat local — un asistente de IA autoalojado y privado, sin depender de plataformas de terceros.

## Alcance
- No es un proyecto de código: es una tarea de infraestructura y configuración.
- El resultado principal es un sistema funcionando, documentado con evidencia (capturas, configuración no sensible, historial de commits).

## Entorno
- VPS: Ubuntu 22.04.5 LTS
- Herramienta: OpenClaw `2026.10.7`
- Proveedor de modelo: LiteLLM (`https://llm.4geeks.ai/v1`)

## Criterios de evaluación
- OpenClaw instalado y accesible en el VPS.
- Proveedor LiteLLM configurado y conectado a un modelo funcional.
- Chat local devuelve respuesta válida.
- Repositorio `openclaw-<usuario>` existe y es accesible.
- `~/.openclaw/workspace` subido exitosamente a GitHub.
- `IDENTITY.md` presente con Name, Emoji y Greeting personalizados.
- Personalización hecha conversando con OpenClaw, no editando archivos a mano.
- SSH configurado para `git push` desde el VPS.
- Historial de commits muestra la subida del workspace.
- Sin archivos sensibles (API keys, tokens, credenciales) subidos.
- Conexión al VPS por SSH (no consola web/gráfica).
- Pasos seguidos en el orden de la guía de 4Geeks.

## Verificación final de seguridad antes de entregar
El workspace no debe contener: `openclaw.json`, archivos `.env`, API keys/tokens, ni credenciales. Si se sube algo sensible por error: eliminar el repo, limpiar el workspace, recrear el repo y contactar al instructor.

---

# Contexto — Mi Agente a Mi Manera: Enseñando Nuevas Skills a Tu Asistente Personal

## Objetivo
Pasar de un agente genérico a uno personalizado: rellenar los cinco archivos de configuración que definen quién es el agente y para quién trabaja (`IDENTITY.md`, `SOUL.md`, `AGENTS.md`, `USER.md`, `TOOLS.md`), y construir al menos dos skills personalizadas de OpenClaw que eliminen tareas repetitivas reales, usando las integraciones ya conectadas (Zapier: Google Docs, Calendar, Gmail, Drive, Tasks, GitHub; Telegram).

## Alcance
- No se configura ninguna API, flujo OAuth ni servicio externo nuevo — solo se usan las conexiones ya existentes.
- Las skills deben implementarse como skills reales de OpenClaw, no como prompts sueltos ejecutados a mano en terminal.
- Cada skill debe reflejar visiblemente la configuración de los cinco archivos (tono, estilo, contexto) y probarse con un input real y personal, no un placeholder.

## Qué debes hacer (checklist obligatorio)

**Configura los cinco archivos:**
- [ ] `IDENTITY.md` — nombre y símbolo del agente.
- [ ] `SOUL.md` — personalidad y estilo de trabajo específicos, no genéricos: cómo gestiona la incertidumbre, si pregunta o actúa primero, qué tono usa.
- [ ] `AGENTS.md` — límites inamovibles: al menos una regla de privacidad y una sobre cuándo parar y preguntar en vez de actuar.
- [ ] `USER.md` — contexto real: nombre, proyectos actuales, herramientas, objetivos.
- [ ] `TOOLS.md` — servicios accesibles vía Zapier, cuándo usar cada uno, valores por defecto (calendario, carpeta, firma, etc.).
- [ ] `openclaw doctor` ejecutado sin errores tras editar los cinco.

**Documenta tu diseño:**
- [ ] `SKILLS_DESIGN.md` en la raíz del repo, con las 3 preguntas de diseño respondidas para cada skill.
- [ ] Commit de `SKILLS_DESIGN.md` antes de escribir ninguna implementación de skill.

**Implementa tus skills:**
- [ ] Al menos 2 skills personalizadas, implementadas como skills de OpenClaw correctas.
- [ ] Cada skill refleja visiblemente la configuración de los cinco archivos.
- [ ] Al menos una skill produce output verificado en un servicio conectado a Zapier o por Telegram.
- [ ] Cada skill probada con un input real y personal.
- [ ] Ninguna skill requiere una nueva conexión/API/OAuth — si la requiere, no es válida para este proyecto.

## Criterios de evaluación
- Los cinco archivos de configuración contienen contenido específico y no genérico.
- `openclaw doctor` se ejecuta sin errores.
- `SKILLS_DESIGN.md` está commiteado antes de la implementación y responde las tres preguntas de diseño para cada skill.
- Al menos dos skills están implementadas como skills de OpenClaw correctas.
- Al menos una skill produce output verificado en un servicio conectado (Docs, Calendar, Gmail, Tasks, Drive o Telegram).
- No se ha configurado ninguna API externa ni flujo OAuth nuevo como parte del proyecto.
- Los outputs del agente reflejan visiblemente la configuración de los cinco archivos — tono, estilo y contexto coherentes con lo escrito allí.
- El estudiante puede explicar el razonamiento detrás de cada skill: por qué esa tarea, por qué ese input, por qué ese formato de output.

Nota del enunciado: un `SKILLS_DESIGN.md` bien razonado con una implementación parcial puntúa más alto que una skill que funciona sin ningún razonamiento de diseño documentado.

---

# Contexto — Conectar tu Agente de IA a la API de 4Geeks Academy

## Objetivo
Dar a OpenClaw acceso real, autenticado, a dos sistemas más: la API pública de 4Geeks Academy (Breathecode) y GitHub (para poder leer/escribir en este mismo repositorio), y usar esa conexión para crear 6 skills de un único endpoint cada una, conversando con el agente en vez de escribiendo las skills a mano.

Prompt original del estudiante a OpenClaw (registrado en `SKILL_LOG.md`):
> "Nuevo Objetivo - Conectar tu asistente de IA (OpenClaw) a la API de 4Geeks Academy - Creo que esto esta ya logrado. Por otro lado vamos a crear 4 Skills nuevas, de las cuales te dare un prompt por cada una de ellas, y hay que documentarlas dentro del fichero SKILL_LOG.md creado en el repositorio..."

## Alcance
- Dos conexiones nuevas en juego: la API de 4Geeks (token `FOURGEEKS_TOKEN` ya existente en el sistema, verificado funcional) y GitHub (identidad nueva, necesaria para que el propio agente pudiera escribir en el repo).
- Cada skill debe depender de **un único endpoint** de la API de 4Geeks (Breathecode), sin mezclar varias llamadas en una misma skill.
- Las skills se crean conversando con OpenClaw (un prompt por skill), no escribiendo el `SKILL.md` a mano.
- Cada skill se documenta en `SKILL_LOG.md`: prompt usado, endpoint, prueba real y resultado.

## Qué se hizo (checklist)

**Paso 1 — Conectar OpenClaw con GitHub:**
- [x] Instalar GitHub CLI (`gh`) en el VPS.
- [x] Autenticar `gh` con la cuenta `dochoadi` vía OAuth device-flow (scopes `repo`, `workflow`, `read:org`, `gist`).
- [x] Verificar permiso `push`/`admin` sobre este repo exacto.
- [x] Documentar el proceso completo en `SKILL_LOG.md` (incluyendo el comando inventado por Kai que no existía, y la vía real).

**Paso 2 — Verificar la conexión con la API de 4Geeks:**
- [x] Confirmar que `FOURGEEKS_TOKEN` es válido contra `GET /v1/admissions/user/me` (200 OK).

**Paso 3 — Crear 4 skills de un único endpoint (Skills 1-4):**
- [x] Skill 1 `authenticate-4geeks` — valida el token/sesión del estudiante.
- [x] Skill 2 `4geeks-proyectos` — lista los cohorts/proyectos y su estado.
- [x] Skill 3 `4geeks-pendientes` — filtra los proyectos en curso (`ACTIVE`).
- [x] Skill 4 `4geeks-progreso` — calcula el porcentaje global de avance.

**Paso 4 — Crear 2 skills extendidas adicionales (Skills 5-6):**
- [x] Skill 5 `calendario-4geeks` — próximos eventos (`GET /v1/events/me`), filtrados a 7 días.
- [x] Skill 6 `mis-certificados-4geeks` — certificados emitidos (`GET /v1/certificate/me`).

## Criterios de evaluación (reconstruidos a partir del enunciado por pasos recibido)
- La conexión a GitHub y a la API de 4Geeks funciona y está verificada con una llamada real, no solo configurada.
- No se usa ninguna vía de conexión inventada o no documentada (se corrigió explícitamente un comando de CLI que Kai sugirió y que no existe).
- Cada una de las 6 skills depende de un único endpoint, documentado con su URL, cabecera de autenticación y campos usados.
- Cada skill se probó con una llamada real contra la API (no con datos de ejemplo) y el resultado se registró en `SKILL_LOG.md`.
- No se expone el token (`FOURGEEKS_TOKEN`) ni el token OAuth de GitHub en ningún fichero del repositorio.
