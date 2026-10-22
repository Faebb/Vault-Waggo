---
title: ADR-016 Identidad propia con bcrypt y JWT emitido por la API
tags: [waggo, adr, seguridad]
status: Aceptada
date: 2026-10-20
---

# ADR-016 Identidad propia con bcrypt y JWT emitido por la API

Volver a [[00 - MOC Waggo]] · Guía: [[API - Pipeline, seguridad y errores]] · Supersede parcialmente: [[ADR-011 Autenticación OAuth 2.0 con roles]]

> [!info] Estado: **Aceptada** · 2026-10-20 · RF-001, RF-005, RNF-001, RNF-002

## Contexto
ADR-011 dejó pendiente "elegir proveedor" de OAuth 2.0 (Keycloak, Auth0, Entra ID...) y nunca se escogió. Mientras tanto, el Roadmap (sprint 1: "Registro y autenticación de dueños con JWT + bcrypt") y las RNF ya escritas asumían lo contrario:
- RNF-001 pide un puerto `IPasswordHasher` con implementación bcrypt en Infrastructure — solo tiene sentido si **nosotros** guardamos y verificamos la contraseña.
- RNF-002 pide "access corto + refresh token rotativo" — solo tiene sentido si **nosotros** emitimos y rotamos el token, no un proveedor externo.

Dos decisiones contradictorias conviviendo sin que nadie las resolviera. Para el piloto (RNF-006: 500 dueños / 50 paseadores) levantar o pagar un IdP externo (Keycloak, Auth0, Entra External ID) es una complejidad y un costo que no se justifican todavía.

## Decisión
- **Identidad propia**: módulo `Identity` con entidad `User` (schema `identity.users`, mismo patrón que `identity.walker_profiles`). La API **es** el emisor y el validador de sus propios tokens; no delegamos a un proveedor externo.
- **Contraseñas (RNF-001)**: puerto `IPasswordHasher` en Application; implementación `BCryptPasswordHasher` (paquete `BCrypt.Net-Next`) en Infrastructure.
- **Sesión (RNF-002)**: JWT access token corto (15 min) firmado con una clave simétrica propia (`Authentication:Jwt:SigningKey`, igual patrón que `Encryption:Key` — 32+ bytes aleatorios fuera del código) y un refresh token opaco de un solo uso guardado **hasheado** en `identity.refresh_tokens`; cada uso rota el token (invalida el anterior y emite uno nuevo) y es revocable.
- **Roles**: se mantienen `owner` / `walker` / `admin` y las políticas de ADR-011 sin cambios. Se calculan al emitir el token: todo `User` confirmado es `owner`; `walker` se añade si tiene un `WalkerProfile` con `Status = Approved` (RF-002/RF-003); `admin` es una bandera manual en `User`, sembrada por migración (sin autoservicio).
- **Confirmación de cuenta (RF-001)**: enlace de un solo uso al correo, expira en 24 h; la cuenta queda inactiva (no puede iniciar sesión) hasta confirmarla.
- **2FA (RF-005)**: código de 6 dígitos al correo tras validar la contraseña, expira en 10 min, máximo 5 intentos; solo entonces se emite el JWT.
- **Envío de correos**: puerto `IEmailSender` en Application; en Development/Testing una implementación que solo registra el correo en el log (mismo patrón que `SimulatedPaymentGateway`, ADR-014); el proveedor real (SMTP/SendGrid/SES) se decide cuando haya que enviar correos de verdad.
- **Desarrollo sin autenticación**: `DevelopmentAuthenticationHandler` (ADR-011) se mantiene para no frenar el TDD del resto de módulos; convive con el nuevo JWT real (se activa el que corresponda según `Authentication:UseDevelopmentUser`).

## Consecuencias
- ✅ Sin infraestructura ni costo nuevo; coherente con RNF-001/RNF-002 tal como están escritas; mismo estilo del resto del backend (puertos detrás de interfaces, Clean Architecture, sin librerías comerciales en Domain/Application).
- ✅ Simple de probar: igual que `SimulatedPaymentGateway`, el `IEmailSender` de desarrollo no depende de un servicio externo.
- ⚠️ `AddWaggoAuthentication` (`ServiceCollectionExtensions`) cambia: ya no valida contra un `Authority` (metadatos OIDC de un tercero) sino contra una `IssuerSigningKey` propia. Rompe la lectura de `Authentication:Jwt:Authority` de ADR-011; se reemplaza por `SigningKey`, `Issuer` y `Audience`.
- ⚠️ Clave simétrica (HS256): más simple que un par de llaves RSA, correcto para un solo servicio emisor/validador. Si en el futuro otro servicio necesita validar tokens sin conocer el secreto, se migra a RS256 (par de llaves, como ya se documentó la migración de proveedor).
- ⏳ Pendiente: proveedor real de correo (para producción); SMS si el piloto lo requiere más adelante; login social (Google/Apple) no está en alcance de RF-001/RF-005.

## Relación con ADR-011
ADR-011 sigue vigente en todo lo que no contradice esta decisión: OAuth 2.0 Bearer, roles y políticas (`OwnerPolicy`, `WalkerPolicy`, `AdminPolicy`), autenticación falsa en Development. Solo cambia la respuesta a su "pendiente: elegir proveedor" — el proveedor somos nosotros mismos — y por tanto la configuración de validación del token (`Authority` → `SigningKey`).
