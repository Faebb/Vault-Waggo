---
title: Estado de implementación
tags: [waggo, planeacion, trazabilidad]
updated: 2026-10-20
---

# Estado de implementación

Volver a [[00 - MOC Waggo]] · Plan: [[Roadmap MVP]]

> [!info] Para qué sirve esta nota
> Es el puente entre el vault y el código: qué RF ya existen en los repos, en qué archivos viven y qué pruebas los cubren. Se actualiza en cada PR que cierre o cambie un RF (ver [[Definition of Done]]).

## Repositorios
| Repo | Rama | Último commit | Estado |
|---|---|---|---|
| `Faebb/waggo-api` | `main` | `feat(identity): login, 2FA by email and rotating JWT session (RF-005)` | Identidad y login completo (mergeado a main, 2026-10-20) |
| `Faebb/waggo-mobile` | `main` | `feat(auth): register, login, 2FA and session screens (RF-001, RF-005)` | Identidad y login completo (mergeado a main, 2026-10-20) |
| `Faebb/waggo-workspace` | `main` | `docs(specs): mark the mobile slice of RF-001/RF-005 as implemented` | Todas las specs al día (mergeado a main, 2026-10-20) |

## Trazabilidad RF → código
### RF-019 Cálculo de tarifa · RF-018 Retención de comisión (cálculo) — ✅ slice TDD
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Waggo.Domain/{ValueObjects,Enums,Errors,Services}/Pricing/` — `FareCalculator`, `Money`, `WalkDuration`, `CommissionRate`, `PricingTable`, `WalkRate`, `WalkType`, `FareBreakdown`, `PricingErrors` | `Tests/Waggo.Domain.UnitTests/Pricing/*` |
| Application | `Waggo.Application/Pricing/Queries/QuoteFare/` — `QuoteFareQuery`, `QuoteFareQueryValidator`, `QuoteFareHandler`, `FareQuoteResponse`; puerto `IPricingTableProvider` | `Tests/Waggo.Application.UnitTests/Pricing/QuoteFareHandlerTests.cs` |
| Infrastructure | `Waggo.Infrastructure/Services/Pricing/ + Options/Pricing/` — `ConfigurationPricingTableProvider`, `PricingOptions` (sección `Pricing` de `appsettings.json`) | vía integración |
| Api | `Waggo.Api/Endpoints/Pricing/` — `PricingEndpoints` (`GET /api/v1/pricing/quote`, política Owner), `QuoteFareRequest` + validador DTO | `Tests/Waggo.Api.IntegrationTests/Pricing/FareQuoteEndpointTests.cs` |
| Mobile | `src/features/pricing/` — `pricingApi` (Zod), `useFareQuote`, `FareQuoteCard`, `FareQuoteScreen`, `formatMoney` | `*.test.ts[x]` junto a cada archivo |

> [!warning] Solo cálculo
> RF-018 está implementado como **desglose** (comisión / pago al paseador). La retención real en la pasarela sigue pendiente (Sprint 7).

### UX-001 Pantalla de bienvenida — ✅ slice TDD (solo mobile)
No es un RF del SRS: es la entrada de la app, que llevará al registro (RF-001, RF-002) y al login (RF-005). Spec: `waggo-workspace/specs/UX-001-pantalla-de-bienvenida.md`.
| Capa | Archivos | Pruebas |
|---|---|---|
| Rutas | `src/app/index.tsx` (`/`, bienvenida) · `src/app/cotizar.tsx` (`/cotizar`, RF-019) | — (rutas delgadas) |
| Mobile | `src/features/onboarding/` — `WelcomeScreen`, `ValueCard`, `valuePillars` | `WelcomeScreen.test.tsx`, `ValueCard.test.tsx` |
| UI compartida | `src/shared/ui/Button.tsx`; tema oscuro con amarillo y Helvetica en `theme.ts` ([[ADR-013 Tema oscuro mobile-first con amarillo y Helvetica|ADR-013]]), aplicado a toda la app | `Button.test.tsx`, `theme.test.tsx` |

### UX-002 Selector de tema claro/oscuro — ✅ slice TDD (solo mobile)
No es un RF del SRS: amplía el sistema de diseño de UX-001. Spec: `waggo-workspace/specs/UX-002-selector-de-tema.md`.
| Capa | Archivos | Pruebas |
|---|---|---|
| Mobile | `src/shared/ui/{theme.ts, ThemeToggle.tsx, useHydrateThemeMode.ts}` ([[ADR-015 Selector de tema claro-oscuro|ADR-015]]), `src/app/_layout.tsx` (cabecera con el selector), tokens claros aplicados en toda pantalla/componente que usaba `colors` | `theme.test.tsx`, `ThemeToggle.test.tsx`, `useHydrateThemeMode.test.tsx` |

### RF-004 Perfil de mascota · RNF-003 cifrado — ✅ slice TDD
Spec: `waggo-workspace/specs/RF-004-perfil-de-mascota.md`. Primer módulo con base de datos.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Waggo.Domain/{Entities,Enums,Errors}/Pets/` — `Pet` (invariantes, máximo 10 por dueño), `PetSize`, `PetErrors` | `Tests/Waggo.Domain.UnitTests/Entities/Pets/PetTests` |
| Application | `Waggo.Application/Pets/` — `RegisterPet`, `ListMyPets`, `GetPet`, `PetResponse`; puertos `ICurrentUser`, `IPetRepository` | `Tests/Waggo.Application.UnitTests/Pets/**` |
| Infrastructure | `PetConfiguration` (schema `pets`, snake_case), migración `AddPets`, `PetRepository`, `AesGcmFieldEncryptor` + `EncryptedStringConverter` (`Encryption:Key`) | `Tests/Waggo.Infrastructure.UnitTests/Services/Security/AesGcmFieldEncryptorTests` |
| Api | `Waggo.Api/Endpoints/Pets/` — `POST /api/v1/pets`, `GET /api/v1/pets`, `GET /api/v1/pets/{id}` (política Owner); `HttpCurrentUser`; header `X-Dev-User-Id` | `Tests/Waggo.Api.IntegrationTests/Endpoints/Pets/PetsEndpointTests`, `RegisterPetRequestValidatorTests` |
| Mobile | `src/features/pets/` — `MyPetsScreen` (`/mascotas`), `AddPetScreen` (`/mascotas/nueva`), `petsApi`, `usePets`; `httpClient.post`, `TextField`, `ActionFooter` | `*.test.ts[x]` junto a cada archivo |

> [!info] RNF-003 aplicado
> Las notas médicas se guardan cifradas con AES-256-GCM (prueba de integración: la columna `pets.medical_notes` no contiene el texto plano). El documento del paseador (RF-002) usa el mismo conversor (`identity.walker_profiles.document_number`).

### RF-007 Solicitud y agendamiento de paseo — ✅ (dueño + paseador)
Specs: `waggo-workspace/specs/RF-007-solicitud-de-paseo-dueno.md` y `RF-007-solicitud-de-paseo-paseador.md`. RF-007 completo: el dueño pide y el paseador acepta.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Waggo.Domain/{Entities,Enums,Errors}/Walks/` — `Walk` (`Request`, `Cancel`, tarifa congelada), `WalkStatus`, `WalkErrors`; `ValueObjects/Walks/GeoPoint` | `WalkTests`, `GeoPointTests` |
| Application | `Waggo.Application/Walks/` — `RequestWalk` (reusa `FareCalculator`), `CancelWalk`, `ListMyWalks`, `GetWalk`; puerto `IWalkRepository` | `Tests/Waggo.Application.UnitTests/Walks/**` |
| Infrastructure | `WalkConfiguration` (schema `walks`, `pet_ids uuid[]`, `pickup_location geography(Point, 4326)` + GiST), `GeoPointConverter` (NetTopologySuite), migración `AddWalks` (extensión `postgis`), `WalkRepository` | vía integración |
| Api | `Waggo.Api/Endpoints/Walks/` — `POST/GET /api/v1/walks`, `GET /walks/{id}`, `POST /walks/{id}/cancel` (política Owner) | `WalksEndpointTests`, `RequestWalkRequestValidatorTests` |
| Mobile | `src/features/home` (`/inicio`), `src/features/walks` (`/paseos/nuevo`, `/paseos/[id]`, `/paseos`), `shared/location/getCurrentLocation` (`expo-location`) | `*.test.ts[x]` junto a cada archivo |
| Paseador (API) | `Walk.Accept` (primero gana, `xmin`), `GET /walks/available` (orden por distancia haversine), `POST /walks/{id}/accept`, `GET /walks/assigned`; `GET /walks/{id}` también para el paseador asignado | `WalkerEndpointTests`, `WalkConcurrencyTests`, `AcceptWalkHandlerTests`, `ListAvailableWalksHandlerTests` |
| Paseador (mobile) | `src/features/walker` (`/paseador`, `/paseador/[id]`); requiere sesión autenticada con rol `walker` (`shared/auth`, ver RF-001+RF-005) | `WalkerHomeScreen.test.tsx`, `AssignedWalkScreen.test.tsx` |

### RF-008 GPS en tiempo real · RF-011 Métricas del paseo — ✅ slice TDD (en vivo con SignalR)
Spec: `waggo-workspace/specs/RF-008-paseo-en-vivo.md`.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Walk.Start/Finish`, `Entities/Tracking/TrackPoint`, `Services/Tracking/RouteMetrics` (distancia y duración), `TrackingErrors` | `WalkTests`, `RouteMetricsTests` |
| Application | `Walks/Commands/{StartWalk,FinishWalk}`, `Tracking/Commands/RecordTrack` (lotes 1–100), `Tracking/Queries/GetRoute`; puerto `ITrackPointRepository` | `StartAndFinishWalkHandlerTests`, `TrackingHandlersTests` |
| Infrastructure | `tracking.track_points` (GiST en la posición, BRIN en el tiempo), migración `AddTracking` | vía integración |
| Api | `POST /walks/{id}/start`, `/finish`, `/track`; `GET /walks/{id}/track` | `TrackingEndpointTests` |
| Mobile | `src/features/tracking` (`RouteView` SVG, `useRoute`, `useTrackingBeacon`); paseador inicia/termina en `/paseador/[id]`; el dueño ve la ruta en `/paseos/[id]` | `*.test.ts[x]` junto a cada archivo |

### RF-012 Alertas de emergencia — ✅ slice TDD (sin push)
Spec: `waggo-workspace/specs/RF-012-alerta-de-emergencia.md`. La notificación push prioritaria queda para RF-014.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Tracking/WalkAlert` (`RaiseEmergency`), `AlertKind`, `AlertParty`, `AlertErrors` | `WalkAlertTests` |
| Application | `Tracking/Commands/RaiseEmergency`, `Tracking/Queries/ListWalkAlerts`, `IWalkAlertRepository` | `AlertHandlersTests` |
| Infrastructure / Api | `tracking.walk_alerts` (migración `AddWalkAlerts`); `POST /walks/{id}/emergency`, `GET /walks/{id}/alerts` | `AlertEndpointTests` |
| Mobile | `src/features/tracking` — `WalkSafety`, `EmergencyButton`, `AlertBanner` en las pantallas del dueño y del paseador | `*.test.ts[x]` junto a cada archivo |

### RF-013 Chat interno — ✅ slice TDD (en vivo con SignalR)
Spec: `waggo-workspace/specs/RF-013-chat-del-paseo.md`.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Messaging/WalkMessage`, `MessageErrors`, `Enums/Walks/WalkParty`, `Walk.PartyOf` | `WalkMessageTests`, `WalkTests` |
| Application | `Messaging/Commands/SendMessage`, `Messaging/Queries/ListMessages` (`?after=`), `IWalkMessageRepository` | `MessageHandlersTests` |
| Infrastructure / Api | `messaging.walk_messages` (índice por paseo y fecha, migración `AddWalkMessages`); `POST/GET /walks/{id}/messages` | `MessageEndpointTests` |
| Mobile | `src/features/messaging` — `ChatScreen` en `/paseos/[id]/chat` y `/paseador/[id]/chat` | `*.test.ts[x]` junto a cada archivo |

### RF-009 Geocercas · RF-010 Anomalías — ✅ slice TDD
Spec: `waggo-workspace/specs/RF-009-RF-010-alertas-automaticas.md`. Radio 1,5 km y detención de 10 min a menos de 30 m (provisionales).
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Services/Tracking/WalkMonitor`, `AlertKind.{Geofence,Anomaly}`, `WalkAlert.RaiseAutomatic` | `WalkMonitorTests` |
| Application | `RecordTrackHandler` evalúa cada lote y guarda las alertas | `TrackingHandlersTests` |
| Infrastructure / Api | `raised_by` admite nulos (migración `AllowSystemAlerts`) | `AutomaticAlertEndpointTests` |
| Mobile | `AlertBanner` con textos por tipo | `AlertBanner.test.tsx` |

### RF-006 Matching por cercanía — ✅
Spec: `waggo-workspace/specs/RF-006-matching-por-cercania.md`. `GET /walks/available` con ubicación: `ST_DWithin` (5 km) + `ORDER BY ST_Distance` en PostGIS (`WalkRepository.ListRequestedNearAsync`). "Solo paseadores verificados" llegó con RF-002/RF-003. Pruebas: `WalkerEndpointTests`, `ListAvailableWalksHandlerTests`.

### Tiempo real con SignalR (RF-008, RF-012, RF-013, RNF-007) — ✅ slice TDD
Spec: `waggo-workspace/specs/RF-008-RF-013-tiempo-real-signalr.md`. El hub solo avisa *qué* cambió; la app recarga ese recurso por la API REST y deja el polling como respaldo de 30 s.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain / Application | `Enums/Walks/WalkUpdateKind`; puerto `IWalkLiveUpdates`, llamado por los 7 handlers que cambian un paseo | `*HandlerTests` |
| Api | `/hubs/walks` (`WalkHub.Follow`, misma regla que `GET /walks/{id}`), `SignalRWalkLiveUpdates`, token por `?access_token=` | `WalkHubTests`, `WalkHubJwtTests` |
| Mobile | `shared/realtime` (`walkUpdates`, `useWalkUpdates`) en `useWalk`, `useRoute`, `useWalkAlerts`, `useChat`; `@microsoft/signalr` | `*.test.ts[x]` junto a cada archivo |

### RF-001 + RF-005 Identidad y login — ✅
Spec: `waggo-workspace/specs/RF-001-RF-005-identidad-y-login.md` · Decisión: ADR-016 (identidad propia, sin proveedor OAuth externo — resuelve el "pendiente" de ADR-011).
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Identity/{User, LoginChallenge, RefreshToken}`, `IdentityErrors` | `UserTests`, `LoginChallengeTests`, `RefreshTokenTests` |
| Application | `Identity/Commands/{Register, ConfirmEmail, Login, VerifyLoginCode, RefreshSession}`; puertos `IUserRepository`, `ILoginChallengeRepository`, `IRefreshTokenRepository`, `IPasswordHasher`, `ITokenGenerator`, `ITokenService`, `IEmailSender` | `*HandlerTests` de cada caso de uso |
| Infrastructure | `BCryptPasswordHasher` (RNF-001), `JwtTokenService` (HS256, RNF-002), `SecureTokenGenerator`, `DevelopmentEmailSender` (log, como `SimulatedPaymentGateway`); `identity.{users, login_challenges, refresh_tokens}` (migraciones `AddUsers`, `AddLoginAndRefreshTokens`) | `BCryptPasswordHasherTests`, `JwtTokenServiceTests`, `SecureTokenGeneratorTests` |
| Api | `/auth/{register, confirm, login, login/verify, refresh}` (los únicos `AllowAnonymous` de la API); `AddWaggoAuthentication` valida con `IssuerSigningKey` propia, ya no `Authority` externo | `IdentityEndpointTests`, `LoginEndpointTests`, `JwtPipelineTests` (pipeline JWT real de punta a punta) |
| Mobile | `src/features/auth` (registro, confirmación, login, 2FA); `shared/auth` (sesión, refresh silencioso, almacenamiento seguro con *fallback* a `AsyncStorage` en web — `expo-secure-store` no tiene implementación real ahí); reemplazó `shared/api/devIdentity` (eliminado) | `*.test.ts[x]` junto a cada archivo |

> [!info] Roles
> Todo `User` confirmado es `owner`; `walker` se añade si tiene un `WalkerProfile` aprobado (RF-002/RF-003); `admin` es una bandera manual sembrada por migración.

### Recuperación de contraseña (RF-001 + RF-005) — ✅ slice TDD
Spec: `waggo-workspace/specs/RF-001-RF-005-recuperacion-de-contrasena.md`. Token de un solo uso al correo (1 h, solo el hash en la base); respuesta idéntica exista o no la cuenta; el reset revoca todas las sesiones y limpia el bloqueo.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `User.RequestPasswordReset`, `User.ResetPassword`; `IdentityErrors.{ResetTokenInvalid, ResetTokenExpired}` | `UserTests` |
| Application | `Identity/Commands/{ForgotPassword, ResetPassword}`, `Identity/PasswordPolicy` (compartida con el registro) | `ForgotPasswordHandlerTests`, `ResetPasswordHandlerTests` |
| Infrastructure / Api | columnas `password_reset_token_{hash,expires_at}` en `identity.users` (migración `AddPasswordReset`); `POST /auth/password/forgot`, `POST /auth/password/reset` (`AllowAnonymous`) | `PasswordResetEndpointTests`, validadores DTO |
| Mobile | `src/features/auth` — `ForgotPasswordScreen` (`/olvide-contrasena`), `ResetPasswordScreen` (`/restablecer`), enlace en `LoginScreen` | `*.test.ts[x]` junto a cada archivo |

### RF-002 + RF-003 Paseadores verificados — ✅ slice TDD (verificación manual)
Spec: `waggo-workspace/specs/RF-002-RF-003-paseadores-verificados.md`. Mientras se elige el proveedor de identidad y antecedentes, un administrador aprueba o rechaza. Foto y documento escaneado quedan para un slice con almacenamiento de archivos.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Walkers/WalkerProfile` (`Register`, `Approve`, `Reject`), `VerificationStatus`, `DocumentType`, `WalkerErrors` | `WalkerProfileTests` |
| Application | `Walkers/Commands/{RegisterWalker, ApproveWalker, RejectWalker}`, `Walkers/Queries/{GetMyWalkerProfile, ListWalkersForReview}`, `WalkerVerification` en `AcceptWalk` y `ListAvailableWalks` | `AcceptWalkHandlerTests`, `ListAvailableWalksHandlerTests` (con `WalkerMother`) |
| Infrastructure / Api | `identity.walker_profiles` con `document_number` cifrado (migración `AddWalkerProfiles`); `/walkers/me` y `/admin/walkers` | `WalkerProfileEndpointTests` |
| Mobile | `src/features/walker` — `WalkerGate` y `BecomeWalkerScreen` en `/paseador` | `*.test.ts[x]` junto a cada archivo |

### RF-015 – RF-018 Pagos con escrow — ✅ slice TDD (pasarela simulada)
Spec: `waggo-workspace/specs/RF-015-RF-018-pagos-con-escrow.md`. Decisión: [[ADR-014 Pasarela de pagos detrás de un puerto|ADR-014]]. Se retiene al pedir, se libera al cancelar y se cobra con split al terminar.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Payments/WalkPayment` (`Hold`, `Capture`, `Release`), `PaymentStatus`, `PaymentErrors` | `WalkPaymentTests` |
| Application | `IPaymentGateway`, `IWalkPaymentRepository`; `RequestWalk`/`CancelWalk`/`FinishWalk`; `Payments/Queries/{GetWalkPayment, GetMyEarnings}` | `RequestWalkHandlerTests`, `CancelWalkHandlerTests`, `StartAndFinishWalkHandlerTests` |
| Infrastructure / Api | `payments.walk_payments` (migración `AddWalkPayments`), `SimulatedPaymentGateway`, `PaymentOptions`; `/walks/{id}/payment`, `/payments/earnings` | `PaymentEndpointTests` |
| Mobile | `src/features/payments` — `PaymentSummary` y `EarningsCard` | `*.test.ts[x]` junto a cada archivo |

### RF-014 Notificaciones del recorrido — ✅ slice TDD (en la app, sin push)
Spec: `waggo-workspace/specs/RF-014-notificaciones-del-recorrido.md`. La tabla `messaging.notifications` servirá de *outbox* para el push (RNF-008).
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Notifications/Notification` (`ForWalk`, `MarkRead`), `NotificationKind`, `NotificationPriority` | `NotificationTests` |
| Application | `INotificationRepository`; avisos en `AcceptWalk`, `StartWalk`, `FinishWalk`, `CancelWalk`, `RaiseEmergency`, `RecordTrack`; `ListMyNotifications`, `MarkNotificationsRead` | handlers de paseos y alertas |
| Infrastructure / Api | `messaging.notifications` (migración `AddNotifications`); `GET /notifications`, `POST /notifications/read` | `NotificationEndpointTests` |
| Mobile | `src/features/notifications` — `NotificationBell` y `NotificationsScreen` | `*.test.ts[x]` junto a cada archivo |

### Transversal — ✅
| Qué | Dónde | RNF |
|---|---|---|
| Reglas de capas + handlers `sealed internal` | `Tests/Waggo.ArchitectureTests/LayerTests.cs` | RNF-009 |
| Prohibición de datos de tarjeta en el dominio | `LayerTests.Domain_NeverStoresCardData_Rnf004` | RNF-004 |
| `WaggoResponse<T>` (`Data`, pilas, `IsValid`, `ConcatStacks`), `Error`, `WaggoMessage` | `Waggo.Domain/Common/` · [[ADR-007 Envolventes WaggoResponse y WaggoApiResponse|ADR-007]] | — |
| `WriteLogs` (Serilog vía `ILogger<T>`), `PageRequest`, `PagedList<T>`, `ValidateToResponseAsync` / `WithError` (FluentValidation) | `Waggo.Application/Common/` · [[ADR-009 Validación de entrada con FluentValidation|ADR-009]] | — |
| `WaggoApiResponse<T>` + manejo global (excepciones, binding, 404) | `Waggo.Api/Infrastructure/Responses/` · `Tests/Waggo.Api.UnitTests`, `EnvelopeTests` | RNF-005 |
| Handlers CQRS propios (sin MediatR) | `Waggo.Application/Common/Interfaces/` | [[ADR-006 Sin librerías comerciales en el núcleo|ADR-006]] |
| Pipeline ordenado: middleware de excepciones, Serilog, status pages, CORS, OAuth 2.0 JWT + roles (usuario dev), rate limiting, `/health/live` y `/health/ready`, OpenAPI (dev) | `Waggo.Api/Infrastructure/Extensions/`, `Infrastructure/Authentication/`, `Infrastructure/Errors/` · [[API - Pipeline, seguridad y errores]] | RNF-002, RNF-007, RNF-008 |
| Excepciones custom (`WaggoException` y derivadas) | `Waggo.Domain/Exceptions/` · [[ADR-010 Excepciones custom y middleware centralizado|ADR-010]] | — |
| Validación de DTO (capa 1) | `Waggo.Api/Infrastructure/Filters/`, `Endpoints/Pricing/Validators/QuoteFareRequestValidator` | — |
| `WaggoDbContext` (vacío, listo para módulos) | `Waggo.Infrastructure/Persistence/Context/` | — |
| `httpClient` (desenvuelve `WaggoApiResponse`) + `ApiError`, `env`, UI base | `waggo-mobile/src/shared/` | — |
| CI (formato, build, tests, cobertura, docker) | `.github/workflows/ci.yml` en ambos repos | — |
| Convenciones de código obligatorias (`.editorconfig` estricto, Prettier + ESLint) | `.editorconfig`, `eslint.config.js`, `.prettierrc.json` · [[ADR-008 Convenciones de código obligatorias|ADR-008]] | RNF-011 |

## Pendiente por RF
| RF | Módulo | Sprint |
|---|---|---|
| RF-003 (proveedor de verificación, foto y documento escaneado) | `Identity` | 2 |
| RF-014 push (Expo Push o FCM/APNs; la bandeja ya existe) | `Messaging` | 6 |
| RF-015 – RF-018 (proveedor real, tarjetas y cuentas de paseadores) | `Payments` | 7 |
| RF-021 | `Analytics` | 9 |

## Deuda técnica conocida
- [x] `EFCore.NamingConventions` (snake_case) y schema por módulo — aplicado con `Pets` ([[Base de datos - PostgreSQL]]).
- [x] NetTopologySuite para PostGIS — aplicado con `Walks` (`GeoPointConverter`).
- [x] Test de arquitectura que impida referencias cruzadas entre módulos — `ModuleTests` con lista explícita ([[ADR-002 Clean Architecture en el backend|ADR-002]]).
- [x] Umbral de cobertura bloqueante en el backend — Domain + Application ≥ 80 % (`scripts/coverage-gate.sh`).
- [ ] Publicación de imágenes en GHCR (CD-1, ver [[CI-CD]]).
