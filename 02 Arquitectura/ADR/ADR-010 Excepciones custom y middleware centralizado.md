---
title: ADR-010 Excepciones custom y middleware centralizado
tags: [waggo, adr]
status: Aceptada
date: 2026-09-28
---

# ADR-010 Excepciones custom y middleware centralizado

Volver a [[00 - MOC Waggo]] · Guía: [[API - Pipeline, seguridad y errores]]

> [!info] Estado: **Aceptada** · 2026-09-28 · Complementa [[ADR-007 Envolventes WaggoResponse y WaggoApiResponse|ADR-007]]

## Contexto
Se quiere manejo centralizado de errores con el código HTTP correcto, logs con el detalle técnico y mensajes al usuario que nunca expongan excepciones ni código.

## Decisión — modelo híbrido
- **Errores esperados** (validaciones de DTO y de negocio, reglas que pueden acumularse) → errores en `WaggoResponse`.
- **Situaciones que impiden continuar** → excepciones custom en `Waggo.Domain/Exceptions`, todas `sealed` y derivadas de `WaggoException`: `NotFoundException` (404), `ConflictException` (409), `UnauthorizedException` (401), `ForbiddenException` (403), `BusinessRuleException` (422), `ExternalServiceException` (503). Cada una lleva un `Error` público (código + mensaje en español) y un detalle técnico opcional solo para el log.
- **`ExceptionHandlingMiddleware`** es el primer middleware del pipeline: convierte la excepción en `WaggoApiResponse` con el status correcto; excepciones desconocidas → 500 con mensaje genérico que pide contactar al administrador con el `traceId`.
- La excepción completa (tipo, mensaje, stack) **solo** va al log (Serilog): `Warning` para las conocidas, `Error` para las desconocidas.

## Consecuencias
- ✅ Un solo lugar decide status y formato de los errores.
- ✅ Imposible filtrar detalles internos al cliente (probado en `ExceptionHandlingMiddlewareTests`).
- ⚠️ Hay que elegir bien: si el error se puede reportar junto a otros → `AddError`; si corta el flujo → excepción.
