---
title: ADR-007 Envolventes WaggoResponse y WaggoApiResponse
tags: [waggo, adr]
status: Aceptada
date: 2026-09-28
---

# ADR-007 Envolventes WaggoResponse y WaggoApiResponse

Volver a [[00 - MOC Waggo]] · Guía de uso: [[Convención de respuestas y logging]]

> [!info] Estado: **Aceptada** · 2026-09-28 · Reemplaza el `Result<T>` + `ProblemDetails` del Sprint 0

## Contexto
- Se necesita un formato único para todas las respuestas de la API (éxito, error y listados paginados) que el mobile pueda consumir siempre igual.
- Los métodos no deben escribir directamente en el log: el desarrollador decide dónde se escribe, y los mensajes de operaciones internas no se deben perder.
- El proyecto usa **Serilog**.

## Decisión
1. **`WaggoResponse<T>`** (Domain, sin dependencias) es el retorno obligatorio de toda operación: `Data`, tres pilas (errores, warnings, infos), `IsValid` y **`ConcatStacks`** para pasar las pilas de las operaciones llamadas. *(Simplificada el 2026-09-28: una sola clase, sin conversiones implícitas ni helpers extra.)*
2. **`WriteLogs(ILogger, operation)`** (Application) escribe todas las pilas con su nivel; se llama una sola vez donde el desarrollador decida.
3. **`WaggoApiResponse<T>`** (Api) es el DTO de **todas** las respuestas HTTP: `success`, `data`, `pagination` (`null` si no aplica), `errors`, `warnings`, `infos`, `traceId`. Solo lleva mensajes `Public`.
4. El código HTTP se mantiene semántico (400, 404, 409...) según el `ErrorType` del primer error.
5. Excepciones, errores de binding y rutas inexistentes también se envuelven.
6. Serilog es el proveedor de logging; el código usa `ILogger<T>`.

## Consecuencias
- ✅ Contrato único y predecible para el mobile.
- ✅ Warnings e infos de capas internas llegan hasta el endpoint sin perderse.
- ✅ Control explícito de dónde y cuándo se escribe al log.
- ✅ Mensajes internos separados de los públicos (seguridad, RNF-005).
- ⚠️ Un poco más de código en cada método (`ConcatStacks` + `if (!response.IsValid) return`).
- ⚠️ Cambio de contrato respecto al Sprint 0: el mobile se actualizó al mismo tiempo.
