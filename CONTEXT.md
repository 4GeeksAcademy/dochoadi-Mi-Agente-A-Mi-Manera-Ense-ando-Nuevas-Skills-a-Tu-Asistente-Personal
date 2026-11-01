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
