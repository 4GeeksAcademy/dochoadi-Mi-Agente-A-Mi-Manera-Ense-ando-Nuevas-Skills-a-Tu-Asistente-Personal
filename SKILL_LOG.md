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
