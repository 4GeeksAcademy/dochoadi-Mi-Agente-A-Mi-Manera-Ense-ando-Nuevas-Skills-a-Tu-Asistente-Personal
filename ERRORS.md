# Registro de errores y soluciones

Registro de incidencias encontradas durante la configuración, cómo se diagnosticaron y cómo se resolvieron.

## 2026-10-27 — Bot de Telegram borrado tras la primera configuración

**Síntoma:** se creó un bot inicial (`@Dochoadi_bot`) vía BotFather, se configuró en OpenClaw (`openclaw channels add --channel telegram --token <token>`) y quedó `connected`. Poco después, el bot fue borrado desde Telegram (BotFather), dejando la configuración de OpenClaw apuntando a un token inválido.

**Diagnóstico:**
```
openclaw channels status --probe
# → error:Unauthorized | Telegram ingress worker exited with code 1, stopped, disconnected
```

**Solución:**
1. Eliminar la cuenta de canal inválida (no se puede simplemente sobrescribir sin confirmación interactiva):
   ```
   openclaw channels remove --channel telegram --account default --delete
   ```
2. Crear un bot nuevo en BotFather (`/newbot`) → `@Kai_assistance_dochoadi_bot`.
3. Reconfigurar con el nuevo token:
   ```
   openclaw channels add --channel telegram --token <nuevo_token>
   ```

**Lección:** no borrar el bot desde BotFather mientras OpenClaw lo tiene configurado; si hace falta, borrar primero la cuenta del canal en OpenClaw.

---

## 2026-10-27 — Error "Unauthorized" tras configurar el token nuevo

**Síntoma:** después de agregar el token del bot nuevo (`@Kai_assistance_dochoadi_bot`), `openclaw channels status --probe` seguía mostrando:
```
error:Unauthorized | Telegram ingress worker exited with code 1, stopped, disconnected, transport:11m ago
```

**Diagnóstico:**
1. Se verificó el token directamente contra la API de Telegram para descartar que fuera inválido:
   ```
   curl -s "https://api.telegram.org/bot<token>/getMe"
   # → {"ok":true, "result": {... "username":"Kai_assistance_dochoadi_bot" ...}}
   ```
   El token era válido — el bot existe y responde. El problema no era el token.
2. El dato `transport:11m ago` sugería que el worker de ingestión de Telegram seguía corriendo con el proceso/token anterior en memoria (caché), sin recoger la nueva configuración.

**Solución:** reiniciar el gateway para forzar a que el worker de Telegram recargue la configuración actual:
```
openclaw gateway restart
```
Tras el reinicio, `openclaw channels status --probe` mostró `running, connected, ... works`.

**Lección:** después de cambiar el token de un canal que ya estuvo activo, reiniciar el gateway en vez de asumir que el cambio se recoge en caliente.

---

## 2026-10-27 — Pairing/emparejamiento requerido antes de usar el bot

**Síntoma:** al escribirle al bot por primera vez, Telegram respondió con un código de emparejamiento y la instrucción de aprobarlo desde el servidor:
```
Ask the bot owner to approve with:
openclaw pairing approve telegram VCBGM9ZH
```

**Diagnóstico:** `openclaw doctor` ya había advertido previamente: "No command owner is configured" — sin un operador aprobado, el bot no procesa mensajes de usuarios desconocidos.

**Solución:**
```
openclaw pairing approve telegram VCBGM9ZH
```
Esto registró automáticamente `commands.ownerAllowFrom` con el ID de Telegram del estudiante como dueño/operador del bot.

**Lección:** el primer mensaje a un canal nuevo no siempre llega directo al agente — puede requerir aprobación de "dueño" antes de empezar a conversar normalmente.

---

## 2026-10-27 — OAuth del MCP de Zapier: el callback local no es alcanzable desde el navegador del usuario

**Síntoma:** tanto `mcporter auth zapier` como `openclaw mcp login zapier` generan una URL de autorización de Zapier cuyo `redirect_uri` apunta a `http://127.0.0.1:<puerto>/...` — un servidor que levanta el propio proceso **en el VPS**. El estudiante se conecta desde su Mac/navegador, no desde el VPS, así que ese `127.0.0.1` no apunta a ningún sitio alcanzable para él.

**Intentos:**
1. Túnel SSH (`ssh -L <puerto>:127.0.0.1:<puerto> root@VPS`) — en teoría válido, pero el proceso de `mcporter auth` tiene un timeout de 300s y expiró dos veces mientras se coordinaba el túnel (incluyendo un intento donde el túnel no llegó a confirmarse y dio `ERR_CONNECTION_REFUSED` / `channel N: open failed: connect failed: Connection refused`).
2. Al reintentar sin esperar demasiado entre pasos, `mcporter auth zapier` sí completó la autorización (quedó cacheada) — confirmado porque una ejecución posterior terminó instantáneamente sin pedir nueva autorización, y en Zapier (pestaña Clients) el cliente `OpenClaw` pasó de `Pending` a `Connected`.
3. Para la autenticación propia de **OpenClaw** (`openclaw mcp login zapier`, independiente de la de `mcporter`), se usó un método más simple que no requiere túnel: el propio comando imprime en el log `Open this URL to authorize... If the callback cannot reach this terminal, run openclaw mcp login zapier --code <code>`. El estudiante autorizó en su navegador, el navegador intentó redirigir a `127.0.0.1:8989/oauth/callback?code=...` (dando error de conexión, visible pero sin problema), y se copió el valor de `code=` de la barra de direcciones para pasarlo manualmente:
   ```
   openclaw mcp login zapier --code <código_copiado_de_la_url>
   ```

**Solución final:** usar el flag `--code <código>` en vez de depender de que el callback local sea alcanzable — evita por completo la necesidad de túnel SSH.

**Lección:** cuando un flujo OAuth de un MCP corre en un servidor remoto (VPS) y el navegador está en otra máquina, buscar primero si la herramienta ofrece un modo de "pegar el código manualmente" antes de montar un túnel SSH — es más simple y más rápido.

---

## 2026-10-27 — MCP de Zapier conectado pero falla con "SSE error: Non-200 status code (405)"

**Síntoma:** tras autenticar correctamente (`openclaw mcp login zapier` exitoso), `openclaw mcp probe zapier` fallaba con:
```
SSE error: Non-200 status code (405); server unavailable
```

**Diagnóstico:** el servidor se había registrado sin especificar el transporte HTTP, y OpenClaw intentaba conectarse por defecto usando SSE (Server-Sent Events), que el endpoint de Zapier no acepta (responde 405 Method Not Allowed a ese tipo de conexión).

**Solución:** forzar el transporte correcto (`streamable-http`) al guardar la configuración del servidor:
```
openclaw mcp set zapier '{"url":"https://mcp.zapier.com/api/v1/connect","auth":"oauth","transport":"streamable-http"}'
```
Tras esto, `openclaw mcp probe zapier` mostró `zapier: 17 tools, Codex approval auto`.

**Lección:** si un MCP HTTP falla con error 405 en el handshake, revisar/forzar el tipo de transporte (`streamable-http` vs `sse`) antes de asumir que es un problema de autenticación o de red.

---

## 2026-10-27 — Captura de pantalla expuso el email de un tercero real en un repositorio público

**Síntoma:** durante una prueba inicial del flujo, se le pidió a Kai enviar un email real a una dirección de un tercero (no el estudiante) con el asunto "soy el bot de Dani es una prueba". La captura de esa conversación se subió al repositorio **sin redactar el email del destinatario**, en el commit `fe7263a`. El repositorio (`dochoadi-Configurando-tu-Agente-de-IA-Personal-con-OpenClaw`) es **público**.

**Diagnóstico:** en la revisión final de brechas se identificó que esto viola el criterio de evaluación "no se exponen credenciales ni datos personales en los materiales entregados" — el email de un tercero real, sin su consentimiento, es un dato personal, y además se le envió un correo real sin motivo legítimo más allá de la prueba.

**Solución aplicada:**
1. Se reemplazó la captura por una versión con el email del destinatario tachado/redactado visualmente.
2. **Limitación conocida y aceptada:** la versión original (sin redactar) permanece en el historial de git de ese commit anterior, ya pusheado a GitHub antes de detectarse el problema. Reescribir el historial (`git filter-repo` + `push --force`) habría sido la solución completa, pero el estudiante decidió conscientemente no hacerlo y dejar el historial como está, asumiendo que el dato estuvo expuesto brevemente en un repositorio público académico.

**Lección:** no usar direcciones de correo de terceros reales en pruebas de integraciones de email, ni siquiera como "prueba" — usar siempre una dirección propia, una de pruebas controlada (ej. un alias propio), o un servicio de email descartable. Revisar las capturas de pantalla por datos de terceros **antes** de subirlas, no después.

---

## Mi Agente a Mi Manera: Enseñando Nuevas Skills a Tu Asistente Personal

Registro de incidencias de este proyecto (configuración de los cinco archivos de identidad e implementación de skills). Se completa a medida que avanza el trabajo.

### 2026-11-01 — `openclaw doctor` avisa de que `TOOLS.md` se fusionará con `AGENTS.md`

**Síntoma:** tras crear `TOOLS.md`, `openclaw doctor` mostró el aviso: *"TOOLS.md migration preview — ~/.openclaw/workspace/TOOLS.md will be archived and merged into AGENTS.md when customized."*

**Diagnóstico:** se inspeccionó el código fuente del paquete instalado (`/usr/lib/node_modules/openclaw/dist/doctor-tools-md-migration-*.mjs`). Esta versión de OpenClaw trata `TOOLS.md` como una convención heredada ("legacy"): el aviso es solo una vista previa informativa. La fusión real únicamente ocurre si se ejecuta `openclaw doctor --fix`, no con un `openclaw doctor` normal.

**Decisión:** no ejecutar `openclaw doctor --fix` para esta parte, de forma consciente, precisamente para mantener `TOOLS.md` como archivo independiente — que es lo que exige el checklist del proyecto (5 archivos separados). Es una diferencia de comportamiento de esta versión de OpenClaw frente al enunciado genérico del curso, igual que ocurrió con el campo "Greeting" de `IDENTITY.md` en el primer ejercicio.

**Lección:** antes de ejecutar comandos de "auto-fix" de OpenClaw, revisar qué cambian exactamente — pueden alterar la estructura de archivos que un ejercicio requiere mantener separada.

---

### 2026-11-01 — `USER.md` cerca de su límite de caracteres (bootstrap)

**Síntoma:** `openclaw doctor` reportó `USER.md: 3.932 chars (98% of max/file 4.000)` — riesgo de truncado del archivo al cargarse en cada sesión.

**Diagnóstico:** al añadir el perfil real y las reglas de modo "advisor"/"builder" sobre el contenido ya existente (saludo + personalidad Donna Paulsen), el archivo se acercó al límite fijo de 4.000 caracteres que OpenClaw reserva para `USER.md`.

**Solución:** se le pidió a Kai, en conversación, que condensara la redacción de sus propias directivas sin perder ninguna (frases más cortas, fusión de bloques redundantes), dejando margen. Resultado: 3.220 caracteres (80% del límite).

**Lección:** vigilar el tamaño de `USER.md` a medida que se añade contexto real; dejar margen para futuras actualizaciones en vez de llenarlo justo al límite.

---

### 2026-11-01 — GitHub, Google Drive y Google Tasks no están conectados vía Zapier

**Síntoma:** al probar la skill `resumen-github-diario` desde Telegram con un input real, Kai respondió que no tiene acceso a GitHub ("ni gh CLI instalado ni token OAuth configurado") y propuso configurar un token clásico de GitHub o reconectarlo desde Settings → Agents → Tools.

**Diagnóstico:** se verificó con Kai, usando sus propias herramientas de Zapier (`zapier__discover_zapier_actions`, `zapier__inspect_zapier_actions`), sin configurar nada nuevo, qué apps están realmente habilitadas. Resultado: solo **Gmail, Google Calendar, Google Docs y Google Sheets** están conectadas y operativas. **GitHub, Google Drive y Google Tasks no están conectados** — Drive aparece en el catálogo de Zapier pero no está habilitado, Tasks ni siquiera aparece, y GitHub no tiene ninguna conexión activa (ni por Zapier ni como herramienta nativa del agente).

Esto contradice el enunciado genérico del proyecto, que lista GitHub y Google Tasks/Drive como servicios ya accesibles vía Zapier — en la práctica, la conexión de Zapier de este estudiante (hecha en el ejercicio anterior) solo habilitó 4 de las 7 apps mencionadas en la guía.

**Por qué no se soluciona conectando esas apps:** el enunciado de este proyecto prohíbe explícitamente configurar una API externa o un flujo OAuth nuevo ("Todas las skills deben funcionar con las herramientas actualmente conectadas... Cualquier skill que requiera una nueva conexión no será aceptada"). Conectar GitHub, Drive o Tasks ahora violaría esa regla, aunque técnicamente sería posible.

**Decisión inicial:** se rediseñó `resumen-github-diario` para no depender de GitHub, y se consideró rediseñar `triaje-bandeja-entrada` para usar Google Sheets en vez de Tasks.

**Actualización — el estudiante conecta Drive y Tasks manualmente:** durante la revisión, el estudiante conectó GitHub, Google Drive y Google Tasks a Zapier por su cuenta, fuera de esta conversación. Esto generó un conflicto directo con el criterio de evaluación "No se ha configurado ninguna API externa ni flujo OAuth nuevo como parte del proyecto" y con el enunciado ("cualquier skill que requiera una nueva conexión no será aceptada").

**Decisión final del estudiante:**
- **GitHub:** se descarta por completo. Pese a la conexión añadida, el acceso seguía siendo inestable en las pruebas (Kai reportó accesos contradictorios: una vez "conectado, 20 acciones", otra vez "sin acceso"), y el riesgo de incumplimiento del criterio no compensaba. La skill 1 (`resumen-github-diario`) se sustituye por **Diario de aprendizaje**, que solo usa Google Docs (ya conectado desde el ejercicio anterior).
- **Google Drive y Google Tasks:** el estudiante decide mantenerlas, ya que ambas aparecen mencionadas explícitamente en el enunciado del proyecto como servicios disponibles vía Zapier. La skill `triaje-bandeja-entrada` se mantiene con su diseño original (crea Google Tasks), sin rediseñar a Sheets.

**Riesgo asumido y documentado:** Google Drive y Google Tasks se conectaron a Zapier **durante este proyecto**, no antes. Esto es, en sentido estricto, una configuración nueva de conexión, lo cual el enunciado pide evitar. El estudiante asume este riesgo de interpretación conscientemente (mismo patrón que el riesgo documentado en el ejercicio anterior sobre el criterio del evento de Calendar) — ver la tabla de verificación final en `TRACKING.md`.

**Lección:** antes de diseñar skills a partir de un enunciado genérico, verificar con el propio agente (sin configurar nada nuevo) qué servicios están realmente conectados — la lista de la guía puede no coincidir con la conexión real del estudiante. Y: una vez identificado un posible incumplimiento de un criterio explícito, documentarlo con la misma transparencia tanto si se revierte como si se asume el riesgo.

---

### 2026-11-01 — "MCP error -32000: Connection closed" al probar `triaje-bandeja-entrada`

**Síntoma:** al probar la skill desde Telegram, Kai no pudo completar la búsqueda de correos no leídos en Gmail y empezó a intentar workarounds por su cuenta ("voy a probar un enfoque diferente..."). El estudiante interpretó, por un mensaje de estado ambiguo, que OpenClaw había eliminado por completo el servidor MCP de Zapier.

**Diagnóstico:**
1. Se pidió a Kai el error literal (sin workarounds): `MCP error -32000: Connection closed`, en la llamada `zapier__execute_zapier_read_action` → `gmail_find_email` (`query: "is:unread in:inbox"`).
2. Se revisó `openclaw mcp show zapier` y `openclaw mcp list`: la configuración seguía intacta en `openclaw.json`, el servidor **no** se había eliminado.
3. Se revisó el log del gateway (`/tmp/openclaw/openclaw-<fecha>.log`) y aparecían varios mensajes `[state/lease] Waiting for state lease core:mcp-oauth/zapier-... held by <otro-proceso>`, justo alrededor del momento del fallo — indicando contención: la prueba real en Telegram y los comandos de diagnóstico lanzados desde el CLI en paralelo compitieron por la misma credencial OAuth de Zapier, y una de las llamadas se cortó a media conexión.
4. Se volvió a probar `openclaw mcp probe zapier` en frío (sin otra llamada concurrente): respondió correctamente con 17 herramientas, sin errores.

**Solución:** ninguna reconfiguración necesaria — fue un fallo transitorio de contención, no una pérdida real de la conexión. Simplemente evitar lanzar pruebas reales por Telegram al mismo tiempo que comandos de diagnóstico desde el CLI contra el mismo MCP.

**Lección:** un mensaje de error de conexión de un MCP no implica que la configuración se haya perdido — verificar primero `mcp show`/`mcp list`/`mcp probe` antes de asumir que hay que reinstalar o reconectar nada. Evitar lanzar llamadas concurrentes al mismo servidor MCP (prueba real + diagnóstico) para no generar falsos positivos de desconexión.

**Actualización — el fallo persiste y es real, no solo contención:** se repitió la prueba en una llamada única, aislada, sin ningún otro proceso usando el MCP de Zapier en paralelo. Resultado: mismo error exacto, `MCP error -32000: Connection closed`, en el mismo punto (`zapier__execute_zapier_read_action` → `gmail_find_email`). Se descarta la hipótesis de contención como causa única.

**Diagnóstico adicional:**
- `zapier__inspect_zapier_actions` (solo inspección, no llama a la API real de Gmail) funciona siempre, sin fallos.
- `zapier__execute_zapier_read_action` (reenvía la llamada a la API real de Gmail a través de Zapier) falla siempre con el mismo error, en más de 10 intentos.
- Conectividad de red desde el VPS hacia `mcp.zapier.com` verificada: responde en ~0.3s (HTTP 401 sin credenciales, como se espera sin auth).
- Kai eliminó y volvió a añadir el servidor MCP de Zapier (`openclaw config.unset` + `config.set`) como intento de solución — no cambió el resultado.

**Estado: limitación técnica no resuelta, documentada y aceptada.** El patrón (falla específicamente en la acción que reenvía la llamada a la API real de Gmail, no en la capa de transporte/red del VPS ni en la configuración de OpenClaw) apunta a un problema del lado de Zapier al conectar con la API de Gmail para esa cuenta/conexión concreta — posiblemente la conexión de Gmail dentro del panel de Zapier necesita reautorizarse, algo que solo se puede verificar/arreglar desde el dashboard web de Zapier, fuera del alcance de este CLI. Se decide no seguir reintentando dentro de esta sesión y, en su lugar, demostrar que el resto de la integración Zapier sí funciona con otra skill que no dependa de leer Gmail (`plan-semana`, que usa Docs + Calendar).

**Lección:** cuando una acción específica de un MCP falla de forma consistente mientras otras acciones del mismo servidor funcionan, el problema suele estar en esa integración concreta (en este caso, la conexión Gmail dentro de Zapier), no en la configuración general del MCP — reiniciar/reconfigurar el servidor completo no soluciona un problema de una conexión de app específica dentro de Zapier.

**Causa raíz real, encontrada después (ver entrada siguiente): la cuota de tareas del plan de Zapier estaba agotada.** Las hipótesis anteriores (contención de lease, problema de la conexión Gmail) quedan descartadas como causa principal — eran síntomas correlacionados, no la causa.

---

### 2026-11-01 — Causa raíz final: cuota de tareas del plan de Zapier agotada

**Síntoma:** patrón consistente en las 4 skills probadas: cualquier acción que **ejecuta** algo contra un servicio real (`zapier__execute_zapier_read_action` / `zapier__execute_zapier_write_action` — buscar correos, crear un Doc) falla con `MCP error -32000: Connection closed`; cualquier acción que solo **inspecciona** metadatos (`zapier__inspect_zapier_actions`) funciona siempre.

**Diagnóstico final:** Kai identificó, en una de las pruebas, que el patrón exacto (inspect = 0 coste, execute = consume cuota) coincide con el comportamiento de un plan de Zapier que ha agotado su límite de tareas del período de facturación. Cada llamada de `execute_*` intentaba consumir una tarea de la cuenta de Zapier, era rechazada por falta de cuota, y el servidor MCP cerraba la conexión al recibir ese rechazo — de ahí el "Connection closed" genérico, que no reflejaba un problema de red, OAuth ni configuración de OpenClaw (todo eso se descartó correctamente en las entradas anteriores).

**Impacto en las pruebas de las 4 skills:**
- `diario-aprendizaje` ✅ — se ejecutó **antes** de agotarse la cuota; funcionó de punta a punta.
- `borrador-email-voz` ✅ — también se ejecutó antes de agotarse la cuota; borrador guardado correctamente en Gmail.
- `plan-semana` ⚠️ parcial — el evento de Calendar se creó correctamente; el Google Doc del plan semanal falló por falta de cuota (Kai entregó el plan como texto en el chat en su lugar).
- `triaje-bandeja-entrada` ❌ — nunca pudo completarse; cada intento de búsqueda de correos consumía (o intentaba consumir) cuota ya agotada.

**Solución:** no depende de OpenClaw ni de este proyecto — hay que ampliar el plan de Zapier (`zapier.com/app/settings/billing`) o esperar al reinicio del período de facturación. Se documenta como limitación externa aceptada, no como un fallo de la configuración del agente.

**Nota aparte:** durante la investigación, Kai modificó varias veces por su cuenta un skill propio de OpenClaw (`execute-zapier-actions`, un skill del sistema, no uno de los 4 de este proyecto) usando su mecanismo de aprendizaje automático ("💾 Learned: updated execute-zapier-actions"). Es un comportamiento nativo de OpenClaw (el propio mensaje permite revertirlo con "undo") y no afectó a ninguno de los archivos de este repositorio.

**Lección:** cuando un MCP de terceros falla de forma selectiva (las llamadas "baratas" funcionan, las que cuestan no), sospechar primero de límites de cuota/plan antes que de la configuración técnica — habría ahorrado varias rondas de diagnóstico.
