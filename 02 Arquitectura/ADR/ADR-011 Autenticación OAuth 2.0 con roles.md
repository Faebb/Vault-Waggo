---
title: ADR-011 Autenticación OAuth 2.0 con roles
tags: [waggo, adr, seguridad]
status: Aceptada
date: 2026-09-28
---

# ADR-011 Autenticación OAuth 2.0 con roles

Volver a [[00 - MOC Waggo]] · Guía: [[API - Pipeline, seguridad y errores]]

> [!info] Estado: **Aceptada** · 2026-09-28 · RF-005, RNF-002
> [!warning] Parcialmente superada
> "Pendiente: elegir proveedor" quedó resuelto en [[ADR-016 Identidad propia con bcrypt y JWT emitido por la API|ADR-016]]: el proveedor somos nosotros (JWT propio, sin OAuth externo). Roles, políticas y la autenticación falsa en Development de este ADR siguen vigentes.

## Contexto
La API necesita autenticación estándar y autorización por tipo de usuario, sin fricción en desarrollo.

## Decisión
- **OAuth 2.0**: la API actúa como *resource server* y valida access tokens **JWT Bearer**. Configuración genérica (`Authority`, `Audience`, `RoleClaimType`) para poder usar cualquier proveedor; el proveedor concreto se decide después.
- **Autorización basada en roles**: `owner`, `walker`, `admin`; políticas `OwnerPolicy`, `WalkerPolicy` (admin pasa ambas) y `AdminPolicy`. Política por defecto: usuario autenticado.
- **Development sin autenticación**: `DevelopmentAuthenticationHandler` autentica un usuario falso con roles configurables (header `X-Dev-Roles` para probar otros roles). Bloqueado en Production.

## Consecuencias
- ✅ Independiente del proveedor (Keycloak, Entra ID, Auth0...).
- ✅ Las reglas por rol se prueban en desarrollo e integración sin servidor OAuth.
- ⚠️ Cada proveedor entrega los roles en un claim distinto: se ajusta `RoleClaimType` (y, si viene anidado como en Keycloak, se agregará una transformación de claims).
- ⏳ Pendiente: elegir proveedor; 2FA (RF-005) lo resuelve el proveedor.
