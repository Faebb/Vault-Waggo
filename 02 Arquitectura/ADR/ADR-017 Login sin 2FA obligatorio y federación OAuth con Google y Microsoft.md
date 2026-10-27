---
title: ADR-017 Login sin 2FA obligatorio y federación OAuth con Google y Microsoft
tags: [waggo, adr, seguridad]
status: Aceptada
date: 2026-10-24
---

# ADR-017 Login sin 2FA obligatorio y federación OAuth con Google y Microsoft

Volver a [[00 - MOC Waggo]] · Guía: [[API - Pipeline, seguridad y errores]] · Supersede parcialmente: [[ADR-016 Identidad propia con bcrypt y JWT emitido por la API|ADR-016]]

> [!info] Estado: **Aceptada** · 2026-10-24 · RF-001, RF-005

## Contexto
ADR-016 fijó identidad propia con un segundo factor obligatorio (código de 6 dígitos al correo) en cada login, y dejó explícitamente fuera de alcance el login social ("login social (Google/Apple) no está en alcance de RF-001/RF-005"). En la práctica:
- El 2FO por correo exige un proveedor de correo real para probarlo de punta a punta; en Development/Testing el código solo queda en el log (`DevelopmentEmailSender`), lo que vuelve cada prueba de login un paso manual (ir al log a copiar el código) sin aportar nada sobre el flujo que ya se prueba con el correo+contraseña.
- El plan de producto ahora pide poder crear cuenta e iniciar sesión con **Google, Microsoft y correo** (Apple queda fuera por ahora: requiere Apple Developer Program, de pago, y no se justifica todavía).
- Si una cuenta se crea con Google o Microsoft, ese proveedor ya ofrece su propio segundo factor (p. ej. Google Authenticator) — pedir además un código de Waggo es redundante para esas cuentas.

## Decisión
- **Se retira el segundo factor por correo (RF-005)**: `POST /api/v1/auth/login` valida contraseña y devuelve la sesión (`accessToken` + `refreshToken`) directamente, sin paso intermedio. Se eliminan `LoginChallenge`, `VerifyLoginCode`, la tabla `identity.login_challenges` y el endpoint `/auth/login/verify`. Lo demás de RF-005 no cambia: intentos fallidos limitados y bloqueo de 15 min (ahora es la única defensa de fuerza bruta), access token corto + refresh rotativo (RNF-002).
- **Federación OAuth 2.0 con Google y Microsoft** (Apple: fuera de alcance, se retoma cuando haya cuenta de desarrollador): el móvil obtiene el `id_token` del proveedor (Google vía `expo-auth-session`, Microsoft vía Entra ID / MSAL), lo manda a la API, y la API:
  1. Verifica la firma contra las claves públicas (JWKS) del proveedor — Google: `https://www.googleapis.com/oauth2/v3/certs`, emisor `https://accounts.google.com`; Microsoft: `https://login.microsoftonline.com/common/discovery/v2.0/keys`, emisor por tenant.
  2. Valida `aud` contra el *Client ID* registrado en `Authentication:Google:ClientId` / `Authentication:Microsoft:ClientId`.
  3. Busca una cuenta por `(Provider, ProviderUserId)` en una tabla nueva `identity.external_logins`; si no existe, busca por correo normalizado y **la vincula** (el proveedor ya verificó ese correo, así que cuenta como `EmailConfirmed = true`); si tampoco existe por correo, crea un `User` nuevo ya confirmado.
  4. Emite el mismo JWT propio de siempre (ADR-016) — el resto de la API no sabe ni le importa cómo entró el usuario.
- **El correo+contraseña sigue existiendo** (RF-001 sin cambios: registro, confirmación por enlace, login) como alternativa para quien no quiera usar Google/Microsoft.
- **Vinculación por correo es automática y sin confirmación extra**: si ya existe una cuenta por correo+contraseña y el usuario entra con Google/Microsoft usando el mismo correo, se vincula a la cuenta existente en vez de crear una duplicada.
- **Nada cambia en roles ni políticas**: `owner`/`walker`/`admin` se siguen calculando igual al emitir el token (ADR-016); el origen del login no afecta el rol.

## Consecuencias
- ✅ Login de un solo paso: menos fricción para desarrollo (prueba local sin ir a buscar un código) y para el usuario final.
- ✅ Google y Microsoft ya hacen su propio 2FA cuando el usuario lo activa en su cuenta; Waggo no necesita reinventarlo para esas cuentas.
- ⚠️ Las cuentas de solo correo+contraseña se quedan **sin segundo factor** por ahora — aceptado como riesgo temporal del piloto (RNF-006), no para producción a mayor escala sin revisarlo.
- ⚠️ Se borra código y una tabla ya implementados y probados (`LoginChallenge` y relacionados) — no hay usuarios reales todavía, así que no hay migración de datos que perder.
- ⏳ Pendiente: Apple Sign In (cuenta de desarrollador de pago), proveedor real de correo para producción (sigue pendiente desde ADR-016), rotación/revisión de claves de Google/Microsoft antes de ir a producción.

## Relación con ADR-016
ADR-016 sigue vigente en todo lo que no contradice esta decisión: identidad propia como emisor de JWT, `BCryptPasswordHasher`, roles y políticas, refresh token rotativo. Solo cambia la sección "2FA (RF-005)" (se retira) y la nota de "pendiente: login social" (ahora en alcance, Google y Microsoft).
