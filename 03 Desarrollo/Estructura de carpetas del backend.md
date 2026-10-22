---
title: Estructura de carpetas del backend
tags: [waggo, convencion, backend, estructura]
aliases: [Estructura de carpetas, Folder structure]
created: 2026-09-28
---

# Estructura de carpetas del backend

Volver a [[00 - MOC Waggo]] · Decisión: [[ADR-012 Estructura de carpetas por tipo]] · Relacionado: [[Convenciones de código]] · [[Backend - Clean Architecture]]

> [!important] Regla
> **La simplicidad es primero.** Cada archivo está donde su nombre de carpeta dice qué es: enums en `Enums/`, interfaces en `Interfaces/`, helpers en `Extensions/` o `Helpers/`, autenticación en `Authentication/`. Un tipo por archivo. Los namespaces son iguales a la carpeta (el build lo exige).

`*` = carpetas o archivos futuros, para saber dónde irán.

```text
waggo-api/
├─ Waggo.Domain/                              ← reglas de negocio puras, sin dependencias
│  ├─ Common/          Error, WaggoMessage, WaggoResponse, BaseEntity*
│  ├─ Entities/*       <Módulo>/  Owner, Walker, Pet, Walk…  (objetos con identidad que se guardan en BD)
│  ├─ ValueObjects/    Pricing/  Money, WalkDuration, CommissionRate, WalkRate, PricingTable, FareBreakdown
│  ├─ Enums/           Common/  ErrorType, MessageVisibility · Pricing/  WalkType
│  ├─ Errors/          Pricing/  PricingErrors         (catálogo de errores de negocio)
│  ├─ Exceptions/      WaggoException, NotFoundException, ConflictException, ForbiddenException,
│  │                   UnauthorizedException, BusinessRuleException, ExternalServiceException
│  └─ Services/        Pricing/  FareCalculator        (lógica de negocio que no es de una sola entidad)
│
├─ Waggo.Application/                         ← casos de uso
│  ├─ Common/
│  │  ├─ Interfaces/   ICommand, ICommandHandler, IQuery, IQueryHandler
│  │  │                Pricing/  IPricingTableProvider · Persistence/  IWaggoDbContext*
│  │  ├─ Extensions/   WaggoResponseLoggingExtensions, ValidationExtensions, PagedListExtensions  (helpers)
│  │  ├─ Models/       PageRequest, PagedList
│  │  └─ Constants/    WaggoRoles · Pricing/  PricingMessages
│  ├─ Pricing/
│  │  └─ Queries/QuoteFare/   QuoteFareQuery, QuoteFareHandler, QuoteFareQueryValidator, FareQuoteResponse
│  ├─ Walks/*          Commands/<CasoDeUso>/ · Queries/<CasoDeUso>/
│  └─ DependencyInjection.cs
│
├─ Waggo.Infrastructure/                      ← detalles técnicos
│  ├─ Persistence/
│  │  ├─ Context/        WaggoDbContext
│  │  ├─ Configurations/  <Módulo>/  <Entidad>Configuration   (mapeo a tablas; Pets/PetConfiguration)
│  │  ├─ Converters/      EncryptedStringConverter            (columnas cifradas, RNF-003)
│  │  ├─ Migrations/      generadas con dotnet ef (código generado, sin reglas de estilo)
│  │  ├─ Interceptors/*
│  │  └─ Seeds/*
│  ├─ Services/        Pricing/  ConfigurationPricingTableProvider · Pets/  PetRepository · Security/  AesGcmFieldEncryptor
│  ├─ Options/         Pricing/  PricingOptions, WalkRateOptions · Security/  EncryptionOptions   (leen appsettings)
│  └─ DependencyInjection.cs
│
├─ Waggo.Api/                                 ← HTTP
│  ├─ Endpoints/
│  │  └─ Pricing/      PricingEndpoints · Requests/  QuoteFareRequest · Validators/  QuoteFareRequestValidator
│  ├─ Infrastructure/                         ← lo transversal del Api
│  │  ├─ Middleware/      ExceptionHandlingMiddleware
│  │  ├─ Filters/         RequestValidationFilter
│  │  ├─ Authentication/  DevelopmentAuthenticationHandler, DevelopmentAuthenticationOptions
│  │  ├─ Authorization/   WaggoPolicies
│  │  ├─ Responses/       WaggoApiResponse, WaggoApiPagination, WaggoApiMessage,
│  │  │                   WaggoApiResponseFactory, WaggoApiResultExtensions
│  │  ├─ Errors/          ApiErrors, RequestErrors, StatusCodeResponses
│  │  ├─ Settings/        AuthenticationSettings, JwtSettings, DevelopmentUserSettings, RateLimitingSettings
│  │  ├─ Extensions/      ServiceCollectionExtensions, PipelineExtensions, RequestValidationExtensions
│  │  ├─ Helpers/         TraceIdHelper
│  │  └─ Realtime/        SignalRWalkLiveUpdates (implementa IWalkLiveUpdates con el hub)
│  └─ Program.cs
│
└─ Tests/                                     ← mismas carpetas que el proyecto que prueban
   ├─ Waggo.Domain.UnitTests/       Common/, Exceptions/, ValueObjects/Pricing/, Services/Pricing/, TestData/
   ├─ Waggo.Application.UnitTests/  Common/Extensions/, Common/Models/, Pricing/Queries/QuoteFare/, TestDoubles/
   ├─ Waggo.Api.UnitTests/          Endpoints/Pricing/Validators/, Infrastructure/Middleware/, Infrastructure/Responses/
   ├─ Waggo.Api.IntegrationTests/   Endpoints/Pricing/, Health/, Responses/, Infrastructure/
   └─ Waggo.ArchitectureTests/
```

## ¿Dónde pongo...?
| Qué | Dónde |
|---|---|
| Un enum | `Waggo.Domain/Enums/<Módulo>/` |
| Una entidad (se guarda en BD, tiene Id) | `Waggo.Domain/Entities/<Módulo>/` |
| Un objeto de valor (sin Id) | `Waggo.Domain/ValueObjects/<Módulo>/` |
| Errores de negocio | `Waggo.Domain/Errors/<Módulo>/<Módulo>Errors.cs` |
| Lógica de negocio de varias entidades | `Waggo.Domain/Services/<Módulo>/` |
| Una interfaz (puerto) | `Waggo.Application/Common/Interfaces/<Módulo>/` |
| Un caso de uso que cambia datos | `Waggo.Application/<Módulo>/Commands/<CasoDeUso>/` |
| Un caso de uso de lectura | `Waggo.Application/<Módulo>/Queries/<CasoDeUso>/` |
| Un helper (método de extensión) | `…/Common/Extensions/` o `Waggo.Api/Infrastructure/Extensions/` |
| Constantes y códigos de mensajes | `Waggo.Application/Common/Constants/<Módulo>/` |
| Implementación de una interfaz | `Waggo.Infrastructure/Services/<Módulo>/` |
| Mapeo de una entidad a su tabla | `Waggo.Infrastructure/Persistence/Configurations/<Módulo>/` |
| Clase que lee appsettings | `Waggo.Infrastructure/Options/<Módulo>/` o `Waggo.Api/Infrastructure/Settings/` |
| Un endpoint | `Waggo.Api/Endpoints/<Módulo>/<Módulo>Endpoints.cs` |
| DTO de entrada y su validador | `Waggo.Api/Endpoints/<Módulo>/Requests/` y `Validators/` |
| Middleware, filtros, auth | `Waggo.Api/Infrastructure/{Middleware, Filters, Authentication, Authorization}/` |
| Hub de SignalR · implementación de un puerto que lo usa | `Waggo.Api/Endpoints/<Módulo>/<X>Hub.cs` · `Waggo.Api/Infrastructure/Realtime/` |

## Persistencia
Por simplicidad, **las entidades del dominio (`Domain/Entities`) se guardan directamente con EF Core**; no hay una segunda clase "de base de datos" por cada entidad. En `Persistence/Configurations/` solo se describe cómo se mapea cada entidad a su tabla.
