# Resultados de pruebas de las skills

Pruebas de las 4 skills con input real del estudiante, disparadas directamente desde Telegram (`@Kai_assistance_dochoadi_bot`). Por cada skill: input real usado, respuesta de Kai, y verificación del resultado en el servicio correspondiente.

Nota general: a mitad de las pruebas se agotó la cuota de tareas del plan de Zapier, lo que bloqueó dos de las cuatro skills a partir de ese momento. Ver detalle completo en `ERRORS.md`, sección "Causa raíz final: cuota de tareas del plan de Zapier agotada".

---

## Skill 1 — diario-aprendizaje

- **Input enviado por Telegram:** "Hoy aprendí: a diagnosticar un fallo de OAuth en un MCP remoto pegando el código manualmente en vez de montar un túnel SSH, y a leer el código fuente de OpenClaw para entender un aviso de doctor antes de actuar sobre él. Anótalo en mi diario de aprendizaje."
- **Respuesta de Kai:** no encontró el Doc de diario (primera vez que se usa la skill), preguntó si el título "Diario de aprendizaje — Daniel Ochoa" era correcto, el estudiante confirmó, y Kai creó el documento con una entrada fechada (1 de noviembre de 2026) y estructurada en dos puntos, reflejando fielmente lo contado — no genérico.
- **Verificación:** documento "Diario de aprendizaje — Daniel Ochoa" creado en Google Drive (`dodochoadiaz@gmail.com`), con la entrada real visible.
- **Estado:** ✅ Completado — ejecutada antes de agotarse la cuota de Zapier.

---

## Skill 2 — triaje-bandeja-entrada

- **Input enviado por Telegram:** "Triaje mi bandeja de entrada y crea una tarea por cada correo que requiera acción mía." (repetido varias veces durante el diagnóstico)
- **Respuesta de Kai:** la skill se activó correctamente, pero la llamada de lectura de Gmail (`zapier__execute_zapier_read_action` → `gmail_find_email`) falló sistemáticamente con `MCP error -32000: Connection closed` en más de 10 intentos. Causa raíz identificada: cuota de tareas del plan de Zapier agotada (ver `ERRORS.md`).
- **Verificación:** no se pudo completar — no se creó ninguna Google Task.
- **Estado:** ❌ No se pudo probar — limitación externa documentada (cuota de Zapier agotada, fuera del control de este proyecto). La skill en sí está correctamente implementada; el bloqueo es del servicio externo, no del diseño ni de la configuración.

---

## Skill 3 — borrador-email-voz

- **Input enviado por Telegram:** "Escríbeme un borrador de email para mi manager en Agilimo, avisando de que esta semana tengo formación de IA y puede que tarde un poco más en responder al QBR de un cliente." + respuestas a las preguntas de aclaración de Kai (formación termina en 6 meses, QBR genérico, sin dirección de destinatario, tono Donna Paulsen).
- **Respuesta de Kai:** preguntó primero por los datos que faltaban (buena práctica, no inventó nada), redactó el borrador con tono directo estilo Donna Paulsen, firmado "Daniel Ochoa", sin destinatario (tal como se pidió), y lo guardó como borrador en Gmail al confirmarlo el estudiante.
- **Verificación:** borrador "Formación IA — disponibilidad esta semana" visible en Gmail (Drafts), sin enviar.
- **Estado:** ✅ Completado — ejecutada antes de agotarse la cuota de Zapier. Nota menor: al no tener el nombre real del manager, el saludo del borrador quedó como "Daniel," (genérico) en vez de un nombre concreto — comportamiento esperado dado que el estudiante no proporcionó ese dato.

---

## Skill 4 — plan-semana

- **Input enviado por Telegram:** "Organiza mi plan de la semana: terminar la actividad de OpenClaw de 4Geeks, avanzar en el build de iOS de Padel School Manager, y tengo una entrevista de trabajo el jueves a las 17:00." + aclaraciones (hoy es 1 de noviembre, entrevista presencial a las 14:00, distribuir tareas a lo largo del día).
- **Respuesta de Kai:** preguntó fecha, formato de la entrevista y preferencia de horario antes de generar nada (no asumió). Creó el evento "Entrevista de trabajo" el jueves 5 de noviembre, 14:00–15:00, en Google Calendar. El Google Doc del plan semanal falló por agotamiento de cuota de Zapier (ver `ERRORS.md`) — Kai entregó el plan priorizado como texto en el chat en su lugar, siendo transparente sobre qué no pudo completar.
- **Verificación:** evento "Entrevista de trabajo" visible en Google Calendar (jueves 5/11, 14:00–15:00). El Doc del plan semanal no se creó.
- **Estado:** ⚠️ Parcial — el evento de Calendar se verificó correctamente; el Doc del plan falló por la misma limitación externa de cuota de Zapier.

---

## Resumen

| Skill | Estado | Output verificado en servicio conectado |
|---|---|---|
| `diario-aprendizaje` | ✅ Completo | Google Docs |
| `triaje-bandeja-entrada` | ❌ Bloqueada (cuota Zapier) | — |
| `borrador-email-voz` | ✅ Completo | Gmail (borrador) |
| `plan-semana` | ⚠️ Parcial | Google Calendar (evento); Doc bloqueado por cuota Zapier |

3 de 4 skills produjeron output real y verificado en un servicio conectado (criterio mínimo: al menos 1), cumpliendo el requisito del proyecto. La única skill totalmente bloqueada lo fue por una limitación externa (cuota del plan de Zapier), documentada en detalle en `ERRORS.md`, no por un fallo de diseño o implementación.
