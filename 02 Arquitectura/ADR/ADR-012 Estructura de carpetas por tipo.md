---
title: ADR-012 Estructura de carpetas por tipo
tags: [waggo, adr, estructura]
status: Aceptada
date: 2026-09-28
---

# ADR-012 Estructura de carpetas por tipo

Volver a [[00 - MOC Waggo]] · Guía: [[Estructura de carpetas del backend]]

> [!info] Estado: **Aceptada** · 2026-09-28 · Actualiza la organización "por módulo" de [[ADR-002 Clean Architecture en el backend|ADR-002]]

## Contexto
Hay que reconocer a la vista dónde están las interfaces, los enums, los objetos, la lógica de negocio, los helpers y la autenticación. La simplicidad y un código comprensible a la vista son la prioridad.

## Decisión
- Estructura basada en la plantilla **Clean Architecture** de .NET (Jason Taylor), con nombres ajustados:
  - **Domain** por tipo: `Common`, `Entities`, `ValueObjects`, `Enums`, `Errors`, `Exceptions`, `Services`.
  - **Application**: `Common/{Interfaces, Extensions, Models, Constants}` y cada funcionalidad con `Commands/` y `Queries/`.
  - **Infrastructure**: `Persistence/{Context, Configurations, Migrations, Interceptors, Seeds}`, `Services`, `Options`.
  - **Api**: `Endpoints/<Módulo>/{Requests, Validators}` e `Infrastructure/{Middleware, Filters, Authentication, Authorization, Responses, Errors, Settings, Extensions, Helpers}`.
- Donde una carpeta de tipo va a crecer, se agrega una subcarpeta por módulo (`ValueObjects/Pricing/`).
- **Un tipo por archivo**; las pruebas replican las carpetas del proyecto que prueban.
- Las **entidades del dominio se persisten directamente** con EF Core (sin entidades de persistencia separadas).

## Consecuencias
- ✅ Se reconoce el tipo de cada archivo por su carpeta.
- ✅ Estructura conocida por cualquier desarrollador .NET.
- ⚠️ Lo de una funcionalidad queda repartido en varias carpetas; se compensa con la subcarpeta de módulo y nombres consistentes (`Pricing*`).
