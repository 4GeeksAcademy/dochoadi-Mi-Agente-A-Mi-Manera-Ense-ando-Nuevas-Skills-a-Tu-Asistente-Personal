---
name: borrador-email-voz
description: Redacta un borrador de email a partir de una idea breve, usando mi firma y mi tono directo definidos en TOOLS.md y USER.md. Actívala cuando pida "escribe un email a...", "redacta un correo para..." o expresiones equivalentes.
---

# Skill: Borrador de email con mi voz

## Descripción
A partir de una idea breve mía (destinatario + qué quiero decir), redacta un borrador de email completo con mi firma y mi tono directo, listo para que lo revise antes de enviarlo.

## Cuándo usar
Cuando pida "escribe un email a...", "redacta un correo para...", "mándale un email a X diciendo..." o active esta skill explícitamente.

## Prerrequisitos
- Gmail disponible vía Zapier (cuenta `dodochoadiaz@gmail.com`, ver `TOOLS.md`).
- Destinatario y una idea del contenido — si falta cualquiera de los dos, pregunta antes de redactar.

## Procedimiento
1. Confirma destinatario y la idea/puntos clave del mensaje.
2. Redacta el email: directo, sin frases de relleno ("espero que estés bien", "quería comentarte que..."), en línea con el tono definido en `USER.md`.
3. Firma con "Daniel Ochoa" (sin cargo ni empresa, salvo que se pida lo contrario para ese email concreto).
4. Guarda el resultado como borrador en Gmail (o, si no es posible, muestra el texto completo en el chat).
5. Pide confirmación explícita del contenido final antes de que se envíe — nunca lo envía por su cuenta.

## Salida esperada
Un borrador en Gmail (o el texto completo en el chat) con el email redactado, firmado, y pendiente de mi aprobación. Se sabe que funcionó si necesito corregir poco o nada antes de enviarlo.

## Casos especiales
- Si no tengo claro el destinatario o el objetivo del correo, pregunta antes de escribir nada — no redacta un email genérico "de relleno".
- Nunca envía el correo sin mi confirmación explícita del contenido final, aunque yo haya dado la orden inicial de "mándaselo" (regla de `AGENTS.md`/`TOOLS.md`).
- Si pido un tono distinto para ese email concreto (más formal, por ejemplo), lo respeta solo para ese caso, sin cambiar la convención por defecto.
