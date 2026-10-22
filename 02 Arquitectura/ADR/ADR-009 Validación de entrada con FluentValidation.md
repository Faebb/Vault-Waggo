---
title: ADR-009 Validación de entrada con FluentValidation
tags: [waggo, adr]
status: Aceptada
date: 2026-09-28
---

# ADR-009 Validación de entrada con FluentValidation

Volver a [[00 - MOC Waggo]] · Relacionado: [[Convención de respuestas y logging]] · [[Convenciones de código]]

> [!info] Estado: **Aceptada** · 2026-09-28

## Contexto
Los casos de uso reciben queries y commands desde la API y hay que validar su forma (campos obligatorios, rangos, formatos) antes de ejecutar la lógica. Se necesita una forma única y declarativa de hacerlo.

## Decisión
> [!note] Actualizado 2026-09-28: **dos capas**
> 1. **DTO (depuración)** en `Waggo.Api`: `<X>RequestValidator` sobre el request, ejecutado por el endpoint filter `.WithRequestValidation<TRequest>()`; códigos `Request.Required` / `Request.InvalidValue`.
> 2. **Negocio** en `Waggo.Application`: `<Query|Command>Validator`, primer paso del handler.
> Detalle en [[API - Pipeline, seguridad y errores#Validación en dos capas]].

- Las validaciones de entrada se hacen con **FluentValidation** (v12, licencia Apache-2.0).
- Un validador por query/command: `<Query|Command>Validator : AbstractValidator<T>`, `internal sealed`, en la misma carpeta del caso de uso. Se registran solos con `AddValidatorsFromAssembly`.
- Cada regla usa un error del catálogo del módulo: `.WithError(PricingErrors.InvalidDuration)` (código + mensaje en español). Si una regla no lo tiene, el código queda `Validation.<Propiedad>`.
- **Primer paso de todo handler**:
  ```csharp
  WaggoResponse<QuoteFareQuery> validation = await validator.ValidateToResponseAsync(query, ct);
  response.ConcatStacks(validation);
  if (!response.IsValid)
  {
      return response;
  }
  ```
- El **dominio conserva sus invariantes** (ej. `WalkDuration.Create`) como última defensa; FluentValidation valida la entrada, el dominio protege el modelo.

## Consecuencias
- ✅ Reglas declarativas, fáciles de leer y de probar (`ValidateAsync` sobre el validador).
- ✅ Todos los errores de validación llegan al cliente en el mismo `WaggoApiResponse` (HTTP 400).
- ⚠️ Algunas reglas se repiten entre el validador y el dominio (ej. rango de duración); se comparten constantes para no desincronizarlas.
- La prueba de arquitectura exige que los validadores sean `sealed` y no públicos.
