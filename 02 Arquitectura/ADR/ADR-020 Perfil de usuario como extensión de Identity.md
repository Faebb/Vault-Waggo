---
title: ADR-020 Perfil de usuario como extensión de Identity
tags: [waggo, adr, identity]
status: Aceptada
date: 2026-10-24
---

# ADR-020 Perfil de usuario como extensión de Identity

Volver a [[00 - MOC Waggo]] · Guía: [[Backend - Clean Architecture]] · Spec: `waggo-workspace/specs/UX-003-perfil-y-configuracion-de-cuenta.md`

> [!info] Estado: **Aceptada** · 2026-10-24 · UX-003 (no corresponde a un RF del SRS)

## Contexto
El SRS no tiene un requerimiento de "perfil de usuario" ni de "configuración de cuenta". `User` (RF-001, ADR-016) hoy solo guarda `Email`, `PasswordHash` y los campos de confirmación/bloqueo — ningún nombre ni teléfono. Pero toda app necesita, tarde o temprano, una pantalla de "Mi perfil" donde el dueño vea su nombre, su correo, cambie su contraseña y pueda cerrar sesión. Dos caminos posibles:

1. **Ampliar el contrato de registro de RF-001** (`POST /auth/register`) para pedir nombre y teléfono desde el principio.
2. **Completar el perfil después de registrarse**, con un endpoint nuevo y separado, dentro del mismo módulo `Identity`.

RF-001 ya está implementado, probado y mergeado (`IdentityEndpointTests`, `RegisterHandlerTests`, pantallas de `features/auth` en mobile). Cambiar su contrato significa tocar el flujo de registro, su validación de capa 1 y 2, y las pantallas de registro ya construidas — alto riesgo de romper algo que funciona, por una necesidad (el nombre/teléfono) que ni siquiera es parte del SRS.

## Decisión
- **Camino 2**: el perfil se completa **después** de registrarse. No se toca `RegisterCommand`, `RegisterRequest` ni sus validadores.
- `User` gana dos columnas en la misma tabla `identity.users` (no una tabla nueva): `FullName` (string, empieza vacío) y `Phone` (string?, empieza en `null`).
- Se extiende el módulo `Identity` existente (mismo patrón de `Waggo.Application/Identity/`, mismo namespace, mismo `IUserRepository`) en vez de crear un módulo `Users` nuevo — el perfil no es un concepto aparte de la cuenta, es la cuenta misma.
- Endpoints nuevos bajo un grupo `/users` separado de `/auth`: `GET/PATCH /users/me`, `POST /users/me/password`. A diferencia de `/auth` (siempre `AllowAnonymous`), `/users` exige sesión (política por defecto: cualquier usuario autenticado, sin rol específico — el perfil es de cualquiera, dueño o paseador).
- El cambio de contraseña reutiliza la misma política de fuerza que el registro (`Identity/PasswordPolicy`, extraída de `RegisterCommandValidator` en este mismo cambio) en vez de duplicar la regla.
- En mobile, el perfil es una feature nueva (`features/profile/`) que reutiliza `logout()` de `features/auth` para cerrar sesión, en vez de crear una función de sign-out paralela.

## Consecuencias
- ✅ RF-001 queda intacto: su contrato, sus pruebas y sus pantallas no cambian. Cero riesgo de regresión en el flujo de registro.
- ✅ Una sola tabla (`identity.users`), una sola migración (`AddUserProfileFields`), consistente con "una entidad persistida, una tabla" (ADR-012).
- ✅ `Identity/PasswordPolicy` queda como punto único de la regla de fuerza de contraseña; si en el futuro se implementa recuperación de contraseña (ver nota de discrepancia en la spec UX-003 — el vault la documentaba pero el código no existía), debe reutilizar esta misma clase.
- ⚠️ El perfil puede quedar incompleto indefinidamente (`fullName: ""`, `phone: null`) — no hay ningún punto que obligue a completarlo. Aceptado: no hay un RF que lo exija.
- ⚠️ `PATCH /users/me` reemplaza nombre y teléfono completos en cada llamada (no hay "parche" campo por campo); mobile siempre envía ambos campos. Más simple de razonar, a costa de que un cliente que solo quiera cambiar uno debe conocer el otro (mobile lo prefetchea con `GET /users/me` antes de editar).
- ⏳ Pendiente (preguntas abiertas en la spec): si cambiar la contraseña debería revocar las demás sesiones activas, y si "cerrar sesión" debería avisarle al backend para revocar el refresh token.

## Relación con otras decisiones
- No modifica ADR-016 (identidad propia, roles calculados desde `WalkerProfile`/`IsAdmin`): el perfil solo agrega datos, no cambia cómo se determinan los roles.
- Sigue ADR-007 (`WaggoResponse`/`WaggoApiResponse`), ADR-009 (validación en dos capas) y ADR-012 (estructura de carpetas) sin excepciones.
