---
title: ADR-008 Convenciones de código obligatorias
tags: [waggo, adr]
status: Aceptada
date: 2026-09-28
---

# ADR-008 Convenciones de código obligatorias

Volver a [[00 - MOC Waggo]] · Guía: [[Convenciones de código]]

> [!info] Estado: **Aceptada** · 2026-09-28

## Contexto
- El `.editorconfig` inicial era el de la plantilla de .NET: todas sus reglas en `silent`/`suggestion`, y el código ya las contradecía (`var` en todas partes, campos estáticos sin `s_`).
- El mobile no tenía formateador.
- Se busca que cualquier persona (o agente) que escriba código en Waggo produzca el mismo estilo sin depender de la revisión manual.

## Decisión
1. **Nunca `var`** en C#: tipo explícito; `new()` con tipo destino para no repetir.
2. Nombres según la convención del runtime de .NET: `_camelCase` para campos de instancia, **`s_camelCase`** para `private static readonly`, `PascalCase` para constantes y campos públicos.
3. Mensajes **públicos en español**; código, logs internos y códigos de mensaje en inglés.
4. Cumplimiento **estricto**: reglas del `.editorconfig` en `warning` (rompen el build por `TreatWarningsAsErrors` + `EnforceCodeStyleInBuild`); `dotnet format` en CI; Prettier + ESLint en mobile con `format:check` en CI.

## Consecuencias
- ✅ Estilo homogéneo verificable automáticamente; menos discusiones de estilo en los PR.
- ✅ Los agentes de IA reciben retroalimentación inmediata del build.
- ⚠️ `GenerateDocumentationFile=true` es necesario para detectar `using` sin uso (IDE0005); el aviso CS1591 (XML doc faltante) está desactivado.
- ⚠️ Los campos `private static` **no** `readonly` caen en la regla `_camelCase` (limitación de las reglas de nombres del `.editorconfig`); en la práctica casi no existen.
- ⚠️ El ancho de 120 caracteres no lo valida el build en C#; se revisa en el PR.
