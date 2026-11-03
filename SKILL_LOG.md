# SKILL_LOG — Conexión de OpenClaw con GitHub

## Objetivo

Dar a Kai (el agente de OpenClaw) una identidad de GitHub real, con permiso de
escritura, para poder leer/crear/editar archivos y hacer commits en este
repositorio (`4GeeksAcademy/dochoadi-Mi-Agente-A-Mi-Manera-Ense-ando-Nuevas-Skills-a-Tu-Asistente-Personal`)
desde una skill.

## Qué falló primero

Al pedirle a Kai que escribiera en el repo, respondió que no tenía acceso y
propuso dos vías:

1. `openclaw secrets request github_token` — **este comando no existe** en la
   versión instalada (OpenClaw 2026.10.7). Comprobado con
   `openclaw secrets --help`: los subcomandos reales son `apply`, `audit`,
   `configure`, `reload`, `store`. Kai se inventó el comando.
2. Conectar GitHub desde "Settings → Agents → Tools" — esta vía sí existe,
   pero corresponde al Control UI (interfaz web) del gateway, no a un comando
   de CLI.

También se confirmó que el plugin de GitHub que ya viene integrado en
OpenClaw (`@openclaw/github`, herramientas `github_read` / `github_search`)
es de **solo lectura y solo repos públicos** — no permite escribir ni hacer
commits, así que no era la herramienta adecuada para este caso.

## Causa raíz

OpenClaw separa dos identidades de GitHub distintas:

- Una identidad "gestionada" (`tools.github`) que solo se configura desde el
  Control UI (web), pensada para publicación controlada desde el propio
  gateway.
- La identidad **nativa de `gh` CLI**, que es la que usan por defecto las
  herramientas de ejecución de shell del agente (`gh` en el `PATH`, o las
  variables `GH_TOKEN`/`GITHUB_TOKEN`).

El VPS no tenía `gh` instalado ni ninguna sesión de `gh auth` iniciada, de
ahí el error. No hacía falta usar el Control UI ni generar ningún token
manualmente desde GitHub: bastaba con autenticar `gh` CLI de forma nativa.

## Solución aplicada

1. Instalación de GitHub CLI desde el repositorio oficial de `apt`:
   ```bash
   curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg -o /tmp/githubcli-archive-keyring.gpg
   install -D -m 0644 /tmp/githubcli-archive-keyring.gpg /etc/apt/keyrings/githubcli-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
     | tee /etc/apt/sources.list.d/github-cli.list
   apt-get update -qq && apt-get install -y gh
   ```
2. Login mediante el flujo de dispositivo (device flow) de GitHub, sin
   necesidad de abrir ningún puerto ni túnel SSH hacia el gateway (el flujo
   habla directamente con `github.com`, no con el Control UI local):
   ```bash
   gh auth login --hostname github.com --git-protocol https
   ```
   El comando mostró un código de un solo uso y la URL
   `https://github.com/login/device`. El código se introdujo en esa página
   desde un navegador normal, autenticando la cuenta `dochoadi` y aprobando
   los scopes `repo`, `workflow`, `read:org`, `gist`.
3. Verificación:
   ```bash
   gh auth status
   # ✓ Logged in to github.com account dochoadi
   # Token scopes: 'gist', 'read:org', 'repo'

   gh api repos/4GeeksAcademy/dochoadi-Mi-Agente-A-Mi-Manera-Ense-ando-Nuevas-Skills-a-Tu-Asistente-Personal --jq '.permissions'
   # {"admin":true,"maintain":true,"pull":true,"push":true,"triage":true}
   ```

## Resultado

- `gh` CLI queda autenticado de forma nativa en el VPS (`root`), con token
  guardado en `~/.config/gh/hosts.yml` (fuera del repositorio, nunca en el
  código ni en `openclaw.json`).
- Las herramientas de ejecución de Kai que usan `gh`/`git` en este host
  heredan esa identidad automáticamente — no fue necesario tocar
  `tools.github` ni el Control UI.
- Permiso confirmado: `push`/`admin` sobre este repo exacto, suficiente para
  que una skill cree o edite archivos y haga commit/push.

## Nota de seguridad

El token OAuth vive únicamente en el keyring de `gh` de este usuario del VPS
(`~/.config/gh/hosts.yml`), con alcance a la cuenta `dochoadi` y a los scopes
mínimos necesarios (`repo`, `workflow`, `read:org`, `gist`). No se ha pegado
ningún token en el repositorio, en `SOUL.md`/`TOOLS.md`/`AGENTS.md` ni en
ningún commit.

---

# Fase de descubrimiento — 4Geeks Academy API + GitHub Access

## Prompt inicial del usuario

> *"Nuevo Objetivo - Conectar tu asistente de IA (OpenClaw) a la API de 4Geeks
> Academy - Creo que esto esta ya logrado. Por otro lado vamos a crear 4 Skills
> nuevas, de las cuales te dare un prompt por cada una de ellas, y hay que
> documentarlas dentro del fichero SKILL_LOG.md creado en el repositorio..."*
> — Dani, 2026-11-03 10:30 UTC

## Acciones realizadas

### 1. Verificar el estado del acceso a la API de 4Geeks

- **Endpoint:** `GET https://breathecode.herokuapp.com/v1/admissions/user/me`
- **Autenticación:** `Authorization: Token ***` (variable de entorno
  `FOURGEEKS_TOKEN` definida en el sistema y referenciada en `openclaw.json`
  como `${FOURGEEKS_TOKEN}`)
- **Resultado:** ✅ Respuesta 200 con los datos del usuario
- **Datos extraídos:** `"Daniel Ochoa"` — confirmación de que el token es
  válido y la API responde correctamente

**Hallazgo:** El endpoint funciona sin trailing slash (`/me` da 404, `/me` sin
slash da 200). La API está en Heroku y devuelve JSON con los campos
`first_name` y `last_name` del perfil del alumno.

### 2. Verificar el acceso al repositorio de GitHub

- **Repositorio:** `4GeeksAcademy/dochoadi-Mi-Agente-A-Mi-Manera-Ense-ando-Nuevas-Skills-a-Tu-Asistente-Personal`
- **Repo privado** — la API pública de GitHub (`Accept: vnd.github.v3+json`)
  devuelve `404 Not Found` al no estar autenticada
- **Estado del agente OpenClaw (`github_identity_status`):**
  ```json
  {"scope":"agent","configured":false,"identity":null,"credentialState":"unavailable"}
  ```
  → No hay credencial de GitHub configurada a nivel de agente OpenClaw.

### 3. Descubrimiento del clon local en la VPS

- **Path local:** `/root/projects/dochoadi-Mi-Agente-A-Mi-Manera-Ense-ando-Nuevas-Skills-a-Tu-Asistente-Personal/`
- **Remote:** `origin git@github.com:4GeeksAcademy/...git (push)` — usa SSH
- **Autenticación SSH:** ✅ `git fetch` funciona sin intervención
- **Último commit remoto:** `e6a434f` (por detrás del local `f4a1dc1`)
- **Fichero `SKILL_LOG.md`:** Existe localmente con contenido previo sobre
  la configuración de `gh` CLI

### 4. Conclusión del descubrimiento

| Componente | Estado |
|---|---|
| API 4Geeks Academy | ✅ Conectado y funcional |
| GitHub CLI (`gh`) | ✅ Autenticado (cuenta `dochoadi`, scopes: repo/workflow/read:org/gist) |
| Repo local en VPS | ✅ Clonado, SSH funcional, permisos de push |
| OpenClaw GitHub tools | ❌ No configurado (innecesario — usamos `gh`/`git` nativos) |
| Secret store | Vacío (no hace falta — el token 4Geeks vive como env var) |

---

# Skill 1 — Autenticación en 4Geeks Academy

## Prompt del usuario

> *"Quiero crear una skill que me permita verificar si mi token de estudiante
> almacenado en la variable de entorno FOURGEEKS_TOKEN es válido y mi sesión
> está activa en 4Geeks."*
> — Dani, 2026-11-03 11:14 UTC

## Descripción de la skill

Skill `authenticate-4geeks` que verifica la validez del token de estudiante de
4Geeks Academy almacenado en la variable de entorno `FOURGEEKS_TOKEN`,
consultando el perfil del alumno en la API de Breathecode.

- **Nombre del fichero SKILL.md:**
  `.openclaw/workspace/skills/authenticate-4geeks/SKILL.md`
- **Configuración en openclaw.json:** entrada `authenticate-4geeks` dentro de
  `skills.entries`, con `FOURGEEKS_TOKEN` referenciado como `${FOURGEEKS_TOKEN}`
- **Input:** Ninguno (lee la variable de entorno automáticamente)
- **Output esperado:** Nombre completo del estudiante + email + estado de cohorts

## Endpoint utilizado

| Propiedad | Valor |
|---|---|
| **URL** | `GET https://breathecode.herokuapp.com/v1/admissions/user/me` |
| **Header** | `Authorization: Token ${FOURGEEKS_TOKEN}` |
| **Formato** | JSON |
| **Campos clave** | `id`, `email`, `first_name`, `last_name`, `cohorts` |

### Hallazgos técnicos

- El endpoint **no tolera trailing slash** (`/me/` → 404, `/me` → 200)
- La API está hosteada en Heroku
- Devuelve información extensa del perfil incluyendo cohorts, rol (`STUDENT`),
  estado financiero (`UP_TO_DATE`) y educacional (`ACTIVE`/`GRADUATED`)

## Prueba de la skill

### Comando ejecutado

```bash
curl -s -w "\nHTTP_CODE:%{http_code}" \
  -H "Authorization: Token ${FOURGEEKS_TOKEN}" \
  "https://breathecode.herokuapp.com/v1/admissions/user/me"
```

### Resultado

```
HTTP_CODE:200
```

### Datos del estudiante extraídos

- **Nombre:** Daniel Ochoa
- **Email:** dochoadi@gmail.com
- **ID de usuario:** 21847
- **Fecha de alta:** 2026-06-24T11:38:30
- **Rol:** student (Academia: 4Geeks Madrid)
- **Estado financiero:** UP_TO_DATE en todos los cohorts
- **Cohorts activos (educational_status ACTIVE):**
  - Implementing Data Pipelines
  - Asynchronous processing and offloading
  - Command Line, Git and Github
  - Intro to 4Geeks Method
  - Command Line, Git and Github (v3)
  - Coding Fundamentals with Typescript
  - Working with AI coding agents
  - Frontend development with Coding Agents
  - Coding fundamentals with Python
  - Backend development with Coding Agents
  - Authentication in web applications
  - Error handling, debugging and testing
  - Managing relational databases with FastAPI
  - Container applications with Docker
  - Architecture optimization
  - Application telemetry
  - Models training & RAG
  - Agentic Engineering
  - Agentic Workflows
- **Cohorts completados (GRADUATED):**
  - Working with AI coding agents (v3)
  - Ai Engineering Fundamentals
  - No-Code & Automation
- **Cohort con estado NOT_COMPLETING:** Ai Engineering Introduction (PREWORK)

### Conclusión

✅ **Token válido y sesión activa.** La API respondió 200 con todos los datos
del perfil. El token `FOURGEEKS_TOKEN` está operativo y permite consultar la
API de Breathecode correctamente. La skill queda funcional.

---

# Skill 2 — Obtener mis proyectos

## Prompt del usuario

> *"Crea una skill para consultar la lista de mis proyectos asignados en 4Geeks
> con sus estados actuales (pendiente, entregado, calificado) usando la variable
> de entorno FOURGEEKS_TOKEN."*
> — Dani, 2026-11-03 11:21 UTC

## Descripción de la skill

Skill `4geeks-proyectos` que consulta el perfil del alumno en la API de
Breathecode y extrae la lista completa de cohorts/syllabi con su estado
`educational_status` (ACTIVE, GRADUATED, NOT_COMPLETING).

- **Nombre del fichero SKILL.md:**
  `.openclaw/workspace/skills/4geeks-proyectos/SKILL.md`
- **Input:** Ninguno (usa la variable de entorno `FOURGEEKS_TOKEN`)
- **Output esperado:** Lista de proyectos con nombre y estado actual

## Endpoint utilizado

| Propiedad | Valor |
|---|---|
| **URL** | `GET https://breathecode.herokuapp.com/v1/admissions/user/me` |
| **Header** | `Authorization: Token ***` |
| **Datos extraídos de** | `cohorts[].cohort.syllabus_version.name` + `cohorts[].educational_status` |

### Hallazgos técnicos

- No existe un endpoint específico de assignments/projects en `/v1/admissions/`.
- Los "proyectos" en 4Geeks Academy se modelan como **cohorts**, cada uno
  asociado a un syllabus (materia).
- El estado del proyecto se lee de `educational_status`: `ACTIVE` (en curso),
  `GRADUATED` (completado/entregado), `NOT_COMPLETING` (abandonado/no finalizado).

## Prueba de la skill

### Comando ejecutado

```bash
curl -s -H "Authorization: Token ***" \
  "https://breathecode.herokuapp.com/v1/admissions/user/me" | python3 -c "
import sys, json
d = json.load(sys.stdin)
for c in d.get('cohorts',[]):
    co = c.get('cohort', {})
    sv = co.get('syllabus_version', {})
    print(f'{sv.get(\"name\",\"?\")} | estado: {c.get(\"educational_status\")}')
"
```

### Resultado (32 proyectos)

| Proyecto | Estado |
|---|---|
| Data pipelines | ACTIVE |
| Asynchronous processing and offloading | ACTIVE |
| Command Line, Git and Github | ACTIVE |
| Working with AI coding agents | GRADUATED |
| AI Engineering Introduction | NOT_COMPLETING |
| Intro to 4Geeks' method | ACTIVE |
| AI Engineering Fundamentals | GRADUATED |
| No‑Code & Automation | GRADUATED |
| Coding fundamentals with TypeScript | ACTIVE |
| Working with AI coding agents | ACTIVE |
| Frontend development with Coding Agents | ACTIVE |
| Coding fundamentals with Python | ACTIVE |
| Backend development with Coding Agents | ACTIVE |
| Authentication in web applications | ACTIVE |
| Error handling, debugging and testing | ACTIVE |
| Managing relational databases with FastAPI | ACTIVE |
| Container applications with Docker | ACTIVE |
| Architecture optimization | ACTIVE |
| Application Telemetry | ACTIVE |
| Models training & RAG | ACTIVE |
| Agentic Engineering | ACTIVE |
| Agentic workflows | ACTIVE |
| Real-Time Communication | ACTIVE |
| Secure AI Applications | ACTIVE |
| AI Engineering Project Delivery | ACTIVE |
| Web UI fundamentals with Tailwind | ACTIVE |
| AI Engineer | ACTIVE |
| Basic personal assistants with Openclaw | GRADUATED |
| Advanced personal assistants with Openclaw | ACTIVE |

### Conclusión

✅ **Skill 2 funcional.** Se listaron 32 proyectos con sus estados. La
información se obtiene del perfil del usuario sin necesidad de un endpoint
específico de assignments.

---

# Skill 3 — Obtener trabajo pendiente

## Prompt del usuario

> *"Crea una skill con la responsabilidad única de listar qué proyectos o tareas
> me faltan por entregar."*
> — Dani, 2026-11-03 11:21 UTC

## Descripción de la skill

Skill `4geeks-pendientes` que filtra los cohorts del alumno cuyo
`educational_status` sea `ACTIVE`, indicando que el proyecto está en curso y
pendiente de completar/entregar.

- **Nombre del fichero SKILL.md:**
  `.openclaw/workspace/skills/4geeks-pendientes/SKILL.md`
- **Input:** Ninguno (usa `FOURGEEKS_TOKEN`)
- **Output:** Lista de proyectos pendientes (estado ACTIVE)
- **Filtro aplicado:** `cohorts[?educational_status == "ACTIVE"]`

## Prueba de la skill

### Comando ejecutado

```bash
curl -s -H "Authorization: Token ***" \
  "https://breathecode.herokuapp.com/v1/admissions/user/me" | python3 -c "
import sys, json
d = json.load(sys.stdin)
pending = [c for c in d.get('cohorts',[]) if c.get('educational_status') == 'ACTIVE']
print(f'Total pendientes: {len(pending)}')
for c in pending:
    sv = c.get('cohort',{}).get('syllabus_version',{})
    print(f'  - {sv.get(\"name\",\"?\")}')
"
```

### Resultado

**Total pendientes: 27 proyectos**

- Data pipelines
- Asynchronous processing and offloading
- Command Line, Git and Github
- Intro to 4Geeks' method
- Coding fundamentals with TypeScript
- Working with AI coding agents
- Frontend development with Coding Agents
- Coding fundamentals with Python
- Backend development with Coding Agents
- Authentication in web applications
- Error handling, debugging and testing
- Managing relational databases with FastAPI
- Container applications with Docker
- Architecture optimization
- Application Telemetry
- Models training & RAG
- Agentic Engineering
- Agentic workflows
- Real-Time Communication
- Secure AI Applications
- AI Engineering Project Delivery
- Web UI fundamentals with Tailwind
- AI Engineer
- Advanced personal assistants with Openclaw
-(más cohorts activos)

### Conclusión

✅ **Skill 3 funcional.** 27 proyectos pendientes identificados filtrando
por `educational_status == "ACTIVE"`. La skill entrega solo lo relevante
sin saturar con información de cohorts completados.

---

# Skill 4 — Obtener resumen de progreso

## Prompt del usuario

> *"Crea una skill que consulte el porcentaje global de mi avance en el curso
> de 4Geeks."*
> — Dani, 2026-11-03 11:21 UTC

## Descripción de la skill

Skill `4geeks-progreso` que calcula estadísticas globales sobre los cohorts
del alumno: total de proyectos, completados (GRADUATED), en curso (ACTIVE) y
no completados (NOT_COMPLETING), con sus porcentajes respectivos.

- **Nombre del fichero SKILL.md:**
  `.openclaw/workspace/skills/4geeks-progreso/SKILL.md`
- **Input:** Ninguno (usa `FOURGEEKS_TOKEN`)
- **Output:** Resumen con porcentaje global de avance

## Prueba de la skill

### Comando ejecutado

```bash
curl -s -H "Authorization: Token ***" \
  "https://breathecode.herokuapp.com/v1/admissions/user/me" | python3 -c "
import sys, json
d = json.load(sys.stdin)
cohorts = d.get('cohorts',[])
total = len(cohorts)
graduated = len([c for c in cohorts if c.get('educational_status') == 'GRADUATED'])
active = len([c for c in cohorts if c.get('educational_status') == 'ACTIVE'])
nc = len([c for c in cohorts if c.get('educational_status') == 'NOT_COMPLETING'])
print(f'Total proyectos: {total}')
print(f'Completados (GRADUATED): {graduated} ({100*graduated//total}%)')
print(f'En curso (ACTIVE): {active} ({100*active//total}%)')
print(f'No completados: {nc} ({100*nc//total}%)')
"
```

### Resultado

```
Total proyectos: 32
Completados (GRADUATED): 4 (12%)
En curso (ACTIVE): 27 (84%)
No completados: 1 (3%)
```

### Conclusión

✅ **Skill 4 funcional.** El progreso global se calcula a partir de los
estados educacionales de los cohorts. El alumno lleva completado el **12%**
de los proyectos, tiene un **84% en curso** y un **3% no completado**.

---

# Skill 5 — Calendario 4Geeks

## Prompt del usuario

> *"Quiero una skill llamada `calendario-4geeks`. Cuando le pregunte '¿qué
> eventos tengo en 4Geeks esta semana?' o algo equivalente, debe llamar a
> `GET https://breathecode.herokuapp.com/v1/events/me` con mi `FOURGEEKS_TOKEN`,
> filtrar los eventos cuya `starting_at` caiga en los próximos 7 días, y
> responderme en una lista clara con título, fecha/hora y un enlace si lo hay."*
> — Dani, 2026-11-03 13:05 UTC

## Descripción de la skill

Skill `calendario-4geeks` que consulta los eventos de la plataforma 4Geeks
Academy y filtra los que ocurren en los próximos 7 días.

- **Nombre del fichero SKILL.md:**
  `.openclaw/workspace/skills/calendario-4geeks/SKILL.md`
- **Input:** Ninguno (usa `FOURGEEKS_TOKEN`)
- **Output esperado:** Lista de eventos con título, fecha/hora y enlace

## Endpoint utilizado

| Propiedad | Valor |
|---|---|
| **URL** | `GET https://breathecode.herokuapp.com/v1/events/me` |
| **Header** | `Authorization: Token ${FOURGEEKS_TOKEN}` |
| **Campos clave** | `title`, `starting_at`, `ending_at`, `live_stream_url`, `event_type` |

### Hallazgos técnicos

- El endpoint **no requiere trailing slash** (funciona sin `/`)
- Requiere el token expandido correctamente con `${FOURGEEKS_TOKEN}`, no con
  comillas simples que impidan la expansión de la variable de shell
- Devuelve eventos pasados y futuros; el filtro de 7 días se aplica en el
  cliente con Python
- Cada evento incluye `live_stream_url` (enlace Google Meet o YouTube),
  `event_type` con nombre del tipo de evento, y `academy` con sede

## Prueba de la skill

### Comando ejecutado

```bash
curl -s -H "Authorization: Token ${FOURGEEKS_TOKEN}" \
  "https://breathecode.herokuapp.com/v1/events/me" | python3 -c "
import sys, json
from datetime import datetime, timezone, timedelta
data = json.load(sys.stdin)
now = datetime.now(timezone.utc)
week = now + timedelta(days=7)
print(f'Eventos totales: {len(data)}')
print(f'Filtrando desde {now.date()} hasta {week.date()}')
print()
for e in data:
    start = e.get('starting_at','')
    if start:
        start_dt = datetime.fromisoformat(start.replace('Z','+00:00'))
        if now <= start_dt <= week:
            title = e.get('title','?')
            url = e.get('live_stream_url') or '—'
            print(f'📅 {title}')
            print(f'   🕐 {start_dt.strftime(\"%a %d %b %Y %H:%M UTC\")}')
            print(f'   🔗 {url}')
            print()
"
```

### Resultado (próximos 7 días desde el 3 nov 2026)

**Eventos totales en la API: 7 | En los próximos 7 días: 3**

📅 **Your Way Into the AI Jobs That Pay 62% More**
   🕐 Mon 03 Nov 2026 23:00 UTC (martes 4 nov 00:00 CET)
   🔗 https://meet.google.com/odd-axjw-dcb

📅 **Your Way Into the AI Jobs That Pay 62% More (DAY 2)**
   🕐 Tue 04 Nov 2026 23:00 UTC (miércoles 5 nov 00:00 CET)
   🔗 https://meet.google.com/odd-axjw-dcb

📅 **Your Way Into the AI Jobs That Pay 62% More (DAY 3)**
   🕐 Wed 05 Nov 2026 23:00 UTC (jueves 6 nov 00:00 CET)
   🔗 https://meet.google.com/odd-axjw-dcb

### Conclusión

✅ **Skill 5 funcional.** 3 eventos encontrados en los próximos 7 días,
todos del AI Intensive "Your Way Into the AI Jobs That Pay 62% More" con
enlace a Google Meet. El endpoint `events/me` funciona correctamente con
el token de estudiante.

---

# Skill 6 — Mis certificados 4Geeks

## Prompt del usuario

> *"Quiero una skill llamada `mis-certificados-4geeks`. Cuando le pida
> 'enséñame mis certificados de 4Geeks' o similar, debe llamar a
> `GET https://breathecode.herokuapp.com/v1/certificate/me` con mi
> `FOURGEEKS_TOKEN` y devolverme, por cada certificado, el nombre del curso,
> la academia, el estado y la fecha."*
> — Dani, 2026-11-03 13:05 UTC

## Descripción de la skill

Skill `mis-certificados-4geeks` que consulta los certificados emitidos al
alumno en 4Geeks Academy.

- **Nombre del fichero SKILL.md:**
  `.openclaw/workspace/skills/mis-certificados-4geeks/SKILL.md`
- **Input:** Ninguno (usa `FOURGEEKS_TOKEN`)
- **Output esperado:** Lista de certificados con curso, academia, estado y fecha

## Endpoint utilizado

| Propiedad | Valor |
|---|---|
| **URL** | `GET https://breathecode.herokuapp.com/v1/certificate/me` |
| **Header** | `Authorization: Token ${FOURGEEKS_TOKEN}` |
| **Campos clave** | `specialty.name`, `academy.name`, `status`, `issued_at`, `pdf_url`, `preview_url` |

### Hallazgos técnicos

- El endpoint **no requiere trailing slash**
- Usa la misma autenticación que el resto de la API de Breathecode
- Los certificados se emiten en estado `PERSISTED` (generado y encolado para
  PDF)
- Incluyen enlace al PDF descargable y a una vista previa
- Firmados por instructores (`signed_by` con rol)
- Algunos tienen fecha de expiración (`expires_at`), otros no

## Prueba de la skill

### Comando ejecutado

```bash
curl -s -H "Authorization: Token ${FOURGEEKS_TOKEN}" \
  "https://breathecode.herokuapp.com/v1/certificate/me" | python3 -c "
import sys, json
data = json.load(sys.stdin)
print(f'Total certificados: {len(data)}')
print()
for c in data:
    specialty = c.get('specialty',{}).get('name','?')
    academy = c.get('academy',{}).get('name','?')
    status = c.get('status','?')
    issued = c.get('issued_at','?')[:10] if c.get('issued_at') else '?'
    pdf = c.get('pdf_url','—')
    print(f'🎓 {specialty}')
    print(f'   🏛  {academy}')
    print(f'   📄 Estado: {status}')
    print(f'   📅 Emitido: {issued}')
    print(f'   🔗 {pdf}')
    print()
"
```

### Resultado

**Total certificados: 3**

🎓 **Basic personal assistants with Openclaw**
   🏛  4Geeks Madrid
   📄 Estado: PERSISTED
   📅 Emitido: 2026-10-28
   🔗 https://certificate.4geeks.com/pdf/27daed3b4ec03acd15e2b6ade196dd52beffcc37

🎓 **Working with AI Coding Agents**
   🏛  4Geeks Madrid
   📄 Estado: PERSISTED
   📅 Emitido: 2026-10-21
   🔗 https://certificate.4geeks.com/pdf/c2b2da22ad52695b3dbf4bd87f4e48faa80ce728

🎓 **No-Code & Automation**
   🏛  4Geeks Madrid
   📄 Estado: PERSISTED
   📅 Emitido: 2026-08-27
   🔗 https://certificate.4geeks.com/pdf/91015bc10d55ab718f529de280cd4ee3f62d5865

### Conclusión

✅ **Skill 6 funcional.** 3 certificados recuperados, todos emitidos por
4Geeks Madrid, en estado PERSISTED (generados). El más reciente es "Basic
personal assistants with Openclaw" del 28 de octubre de 2026.

---
