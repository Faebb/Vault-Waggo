---
title: Convenciones de código
tags: [waggo, convencion, backend, frontend]
aliases: [Code conventions, Estilo de código]
created: 2026-09-28
---

# Convenciones de código

Volver a [[00 - MOC Waggo]] · Decisión: [[ADR-008 Convenciones de código obligatorias]] · Relacionado: [[Convención de respuestas y logging]] · [[Convenciones de Git y ramas]]

> [!tip] Principio número uno
> **La simplicidad es primero.** Un código comprensible a la vista es mejor que uno ingenioso: menos clases, menos magia (nada implícito), nombres que dicen lo que hacen. Estructura de carpetas: [[Estructura de carpetas del backend]].

> [!important] Se hacen cumplir, no se sugieren
> - **Backend:** las reglas del `.editorconfig` están en nivel `warning`, y con `TreatWarningsAsErrors` + `EnforceCodeStyleInBuild` **rompen el build**. El CI además corre `dotnet format whitespace --verify-no-changes`.
> - **Mobile:** ESLint (reglas propias + Expo) y **Prettier**. El CI corre `npm run format:check` y `npm run lint`.
> - Si una regla no aplica en un caso puntual, se discute en el PR; no se desactiva en silencio.

## Idioma
| Qué | Idioma |
|---|---|
| Código (tipos, métodos, variables), comentarios, XML docs, commits, nombres de pruebas | **Inglés** |
| Mensajes **públicos** al usuario (`WaggoMessage` con `Visibility.Public`, textos de UI) | **Español** |
| Mensajes solo de log (`Internal`), mensajes de excepciones técnicas | Inglés |
| Códigos de mensaje (`Pricing.InvalidDuration`) | Inglés — son contrato |
| Documentación del vault | Español; puente en el [[Glosario]] |

## Backend (C# / .NET 10)

### Nombres
| Elemento | Convención | Ejemplo |
|---|---|---|
| Tipos, métodos, propiedades, eventos, namespaces | `PascalCase` | `FareCalculator`, `Calculate` |
| Interfaces | `I` + `PascalCase` | `IPricingTableProvider` |
| Parámetros de tipo | `T` + `PascalCase` | `TResponse` |
| Métodos `async` | Sufijo `Async` | `GetCurrentAsync` |
| Parámetros y variables locales | `camelCase` | `durationMinutes` |
| Campos privados de instancia | `_camelCase` | `_errors` |
| Campos `private static readonly` | `s_camelCase` | `s_someError` |
| Constantes (`const`, cualquier visibilidad) | `PascalCase` | `MaxPageSize` |
| Campos públicos `static readonly` (catálogos) | `PascalCase` | `PricingErrors.InvalidDuration` |
| Siglas | Solo la primera en mayúscula | `Id`, `Api`, `Http` (no `ID`, `API`) |

### `var`: nunca
Siempre tipo explícito. Para no repetir el tipo, usa `new()` con tipo destino:
```csharp
WaggoResponse<FareQuoteResponse> response = new();
QuoteFareQuery query = new(walkType, durationMinutes);
foreach (WaggoMessage message in response.Errors) { ... }
(int status, string code, string message) = Resolve(exception);
```

### Archivos y namespaces
- **Un tipo por archivo**; el archivo se llama igual que el tipo.
- Namespace **file-scoped** (`namespace Waggo.Domain.Pricing;`) y **igual a la carpeta**.
- `using` **fuera** del namespace, `System.*` primero, sin `using` sin uso (el build falla).
- Líneas de máximo **120** caracteres.

### Tipos y estructura
- Clases **`sealed` por defecto**; solo se abren cuando hay herencia real.
- `record` para DTOs, value objects, queries y commands.
- **Constructores primarios** para inyectar dependencias en servicios y handlers.
- Modificador de acceso **siempre explícito** (`private`, `internal`...).
- Campos que no cambian: `readonly`.
- Sin `this.`; usar `string`, `int`... en vez de `String`, `Int32`.
- Enums en singular, con valores explícitos desde 1 (`Individual = 1`).

### Estilo
- **Llaves siempre**, incluso en `if` de una línea; estilo Allman (llave en su propia línea).
- Cuerpo de expresión (`=>`) solo en miembros de **una línea**.
- Sin `#region`. Sin números mágicos: constantes con nombre.
- Collection expressions (`[]`, `[.. source]`), pattern matching y `switch` expressions cuando aclaran.

### Nombres por tipo de artefacto
| Artefacto | Nombre | Ubicación |
|---|---|---|
| Query / Command | `<Verbo><Sustantivo>Query` / `Command` | `Waggo.Application/<Módulo>/Queries|Commands/<CasoDeUso>/` |
| Handler | `<Verbo><Sustantivo>Handler` (`internal sealed`) | mismo folder |
| Validador (FluentValidation) | `<Query o Command>Validator` (`internal sealed`), reglas con `.WithError(<Módulo>Errors.X)` | mismo folder |
| DTO de respuesta | `<Sustantivo>Response` | mismo folder |
| Puerto (interfaz) | `I<X>Provider`, `I<X>Repository`, `I<X>Gateway` | `Waggo.Application/Common/Interfaces/<Módulo>/` |
| Adaptador | `<Tecnología><X>Provider` (ej. `ConfigurationPricingTableProvider`) | `Waggo.Infrastructure/Services/<Módulo>/` |
| Catálogo de errores | `<Módulo>Errors` | `Waggo.Domain/Errors/<Módulo>/` |
| Códigos de warnings/infos | `<Módulo>Messages` | `Waggo.Application/Common/Constants/<Módulo>/` |
| Endpoints | `<Módulo>Endpoints` + `Map<Módulo>Endpoints()` | `Waggo.Api/Endpoints/<Módulo>/` (DTO en `Requests/`, validador en `Validators/`) |
| Configuración EF | `<Entidad>Configuration` | `Waggo.Infrastructure/Persistence/Configurations/<Módulo>/` |
| Options | `<X>Options` con `SectionName` | `Waggo.Infrastructure/Options/<Módulo>/` |

### Asincronía
- `CancellationToken` **siempre** como último parámetro de los métodos async y siempre se propaga.
- Nunca `.Result`, `.Wait()` ni `async void`.
- No se usa `ConfigureAwait(false)` (ASP.NET Core no tiene `SynchronizationContext`).

### Errores y logging
- Toda operación devuelve `WaggoResponse`; las reglas de negocio no lanzan excepciones → [[Convención de respuestas y logging]].
- Excepciones solo para errores de programación: `ArgumentNullException.ThrowIfNull`, `ArgumentOutOfRangeException`, `InvalidOperationException`.
- Logging vía `ILogger<T>` (Serilog detrás) y `WriteLogs`; plantillas estructuradas (`[LoggerMessage]`), nunca interpolación en el mensaje del log.

### Comentarios
- XML doc (`/// <summary>`) en tipos y miembros públicos que no se explican solos.
- Los comentarios explican el **por qué**, no el qué. Nada de código comentado.
- Referencia al RF cuando aplique: `// RF-019: ...`.

### Pruebas
- Nombre: `Method_Scenario_ExpectedResult` (única excepción al PascalCase sin guiones).
- Arrange / Act / Assert separados por una línea en blanco; **un comportamiento por prueba**.
- Datos de prueba con **Object Mother** (`PricingTableMother`) o builders; nada de valores mágicos repetidos.
- Aserciones con **Shouldly**; dobles con **NSubstitute**; afirmar sobre `code`, no sobre textos.

## Mobile (TypeScript / React Native)

### Nombres y archivos
| Elemento | Convención | Ejemplo |
|---|---|---|
| Componentes (archivo y símbolo) | `PascalCase.tsx` | `FareQuoteCard.tsx` |
| Hooks | `useX.ts` | `useFareQuote.ts` |
| Resto de módulos | `camelCase.ts` | `pricingApi.ts`, `formatMoney.ts` |
| Pruebas | junto al archivo: `*.test.ts(x)` | `formatMoney.test.ts` |
| Tipos | `PascalCase` | `FareQuote` |
| Constantes de módulo | `UPPER_SNAKE_CASE` | `WALK_TYPES` |
| Variables y funciones | `camelCase` | `getFareQuote` |

### Reglas (ESLint las hace cumplir)
- **Solo named exports**; `export default` únicamente en rutas de Expo Router (`src/app/**`).
- `type` en vez de `interface`; **prohibido `any`**.
- `import type` / `type` inline para tipos.
- Imports ordenados: externos → `@/...` → relativos, separados por línea en blanco.
- Una feature se importa **solo por su `index.ts`** (`@/features/pricing`), nunca por sus archivos internos.
- `===` siempre; sin `console.log` (solo `warn` / `error`).
- Respuestas de la API validadas con **Zod** a través de `httpClient`.

### Formato (Prettier)
Ancho **120**, comillas simples, punto y coma, comas finales, fin de línea LF. `npm run format` corrige; `npm run format:check` valida (CI).

## Comandos
```bash
# Backend
dotnet build                                   # falla con cualquier violación de estilo
dotnet format                                  # corrige formato y estilo automáticamente
dotnet format whitespace --verify-no-changes   # lo que corre el CI

# Mobile
npm run format        # Prettier --write
npm run lint          # ESLint
npm run typecheck     # tsc --noEmit
```
