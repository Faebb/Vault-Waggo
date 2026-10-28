---
title: Estado de implementación
tags: [waggo, planeacion, trazabilidad]
updated: 2026-10-26
---

# Estado de implementación

Volver a [[00 - MOC Waggo]] · Plan: [[Roadmap MVP]]

> [!info] Para qué sirve esta nota
> Es el puente entre el vault y el código: qué RF ya existen en los repos, en qué archivos viven y qué pruebas los cubren. Se actualiza en cada PR que cierre o cambie un RF (ver [[Definition of Done]]).

## Repositorios
| Repo | Rama | Último commit | Estado |
|---|---|---|---|
| `Faebb/waggo-api` | `main` | `feat(identity): seed admin, owner and verified walker test accounts (DEV-001)` (#21) | Login sin 2FA (ADR-017), perfil propio nombre/celular y usuarios de prueba DEV-001 (mergeado a main, 2026-10-26) |
| `Faebb/waggo-mobile` | `main` | `feat(nav): bottom tab bar for owner + skeleton loading states` (#21) | Navegación por pestañas y skeletons (ADR-019); perfil propio y cola de verificación de paseadores ya integrados (mergeado a main, 2026-10-25) |
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

### RF-008 GPS en tiempo real · RF-011 Métricas del paseo — ✅ slice TDD (por polling; SignalR 🚧 en progreso)
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

### RF-013 Chat interno — ✅ slice TDD (por polling; SignalR 🚧 en progreso)
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

### Tiempo real con SignalR (RF-008, RF-012, RF-013, RNF-007) — 🚧 en progreso
> [!warning] Aún no está en `main` (2026-10-26)
> El trabajo existe solo como cambios **sin commitear** en los worktrees `.worktrees/api-realtime`, `.worktrees/mobile-realtime` y `.worktrees/workspace-realtime` (rama `feat/realtime-signalr`, sin PR). La spec tampoco está en `main` del workspace. La tabla describe el diseño previsto; en `main` la app sigue usando solo polling.

Spec (en el worktree): `waggo-workspace/specs/RF-008-RF-013-tiempo-real-signalr.md`. El hub solo avisa *qué* cambió; la app recarga ese recurso por la API REST y deja el polling como respaldo de 30 s.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain / Application | `Enums/Walks/WalkUpdateKind`; puerto `IWalkLiveUpdates`, llamado por los 7 handlers que cambian un paseo | `*HandlerTests` |
| Api | `/hubs/walks` (`WalkHub.Follow`, misma regla que `GET /walks/{id}`), `SignalRWalkLiveUpdates`, token por `?access_token=` | `WalkHubTests`, `WalkHubJwtTests` |
| Mobile | `shared/realtime` (`walkUpdates`, `useWalkUpdates`) en `useWalk`, `useRoute`, `useWalkAlerts`, `useChat`; `@microsoft/signalr` | `*.test.ts[x]` junto a cada archivo |

### RF-001 + RF-005 Identidad y login — ✅
Spec: `waggo-workspace/specs/RF-001-RF-005-identidad-y-login.md` · Decisión: [[ADR-016 Identidad propia con bcrypt y JWT emitido por la API|ADR-016]] (identidad propia) + [[ADR-017 Login sin 2FA obligatorio y federación OAuth con Google y Microsoft|ADR-017]] (retira el 2FA por correo del login).
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Identity/{User, RefreshToken}`, `IdentityErrors` | `UserTests`, `RefreshTokenTests` |
| Application | `Identity/Commands/{Register, ConfirmEmail, Login, RefreshSession}`; puertos `IUserRepository`, `IRefreshTokenRepository`, `IPasswordHasher`, `ITokenService`, `IEmailSender` | `*HandlerTests` de cada caso de uso |
| Infrastructure | `BCryptPasswordHasher` (RNF-001), `JwtTokenService` (HS256, RNF-002), `DevelopmentEmailSender` (log, como `SimulatedPaymentGateway`); `identity.{users, refresh_tokens}` (migraciones `AddUsers`, `AddLoginAndRefreshTokens`, `RemoveLoginChallenges`) | `BCryptPasswordHasherTests`, `JwtTokenServiceTests` |
| Api | `/auth/{register, confirm, login, refresh}` (los únicos `AllowAnonymous` de la API); `AddWaggoAuthentication` valida con `IssuerSigningKey` propia, ya no `Authority` externo | `IdentityEndpointTests`, `LoginEndpointTests`, `JwtPipelineTests` (pipeline JWT real de punta a punta) |
| Mobile | `src/features/auth` (registro, confirmación, login directo sin pantalla de 2FA); `shared/auth` (sesión, refresh silencioso, almacenamiento seguro con *fallback* a `AsyncStorage` en web — `expo-secure-store` no tiene implementación real ahí); reemplazó `shared/api/devIdentity` (eliminado) | `*.test.ts[x]` junto a cada archivo |

> [!info] Rutas posteriores
> Además de `/auth/{register, confirm, login, refresh}`: `/auth/{resend-confirmation, forgot-password, reset-password, logout}` y `/users/me/{password, email, email/confirm}` — ver [[#Recuperación y seguridad de la cuenta (RF-001 + RF-005) — ✅ 5 slices TDD]].

> [!info] Roles
> Todo `User` confirmado es `owner`; `walker` se añade si tiene un `WalkerProfile` aprobado (RF-002/RF-003); `admin` es una bandera manual sembrada por migración.

> [!info] ADR-017 — 2FA retirado ✅ y login con Google (web) ✅; falta Microsoft
> Se retiró el 2FA por correo (arriba) y la federación con **Google ya funciona en la web** (sección siguiente). Siguen pendientes **Microsoft** y el login nativo de Google en iOS/Android (necesita el development build de EAS) — spec: `waggo-workspace/specs/RF-001-RF-005-login-social-google-microsoft.md` (estado `En progreso`).

### RF-001 + RF-005 Login con Google (web) — ✅ slice TDD
Spec: `waggo-workspace/specs/RF-001-RF-005-login-social-google-microsoft.md` · Decisión: [[ADR-017 Login sin 2FA obligatorio y federación OAuth con Google y Microsoft|ADR-017]]. PRs: waggo-api #32, waggo-mobile #31, waggo-infra #7. Desplegado en producción.
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `Entities/Identity/ExternalLogin`, `Enums/Identity/ExternalProvider`, `User.RegisterExternal` y `User.ConfirmByExternalProvider`, errores `Identity.{ExternalTokenInvalid, ExternalEmailNotVerified, ExternalProviderUnavailable}` | `ExternalLoginTests`, `UserExternalTests` |
| Application | `Identity/Commands/LoginWithExternalProvider/` (un caso de uso por proveedor); puertos `IExternalIdentityProvider` e `IExternalLoginRepository` | `LoginWithExternalProviderHandlerTests` |
| Infrastructure | `GoogleIdentityProvider` (firma RS256 con las claves rotativas de Google, emisor, audiencia y vigencia), `ExternalLoginRepository`, `GoogleAuthOptions` (`Authentication:Google:ClientIds`), tabla `identity.external_logins` (migración `AddExternalLogins`) | `GoogleIdentityProviderTests` (claves RSA de prueba) |
| Api | `POST /api/v1/auth/login/google` (anónimo) + `GoogleLoginRequestValidator` | `GoogleLoginEndpointTests` (proveedor falso, PostgreSQL real), `GoogleLoginRequestValidatorTests` |
| Mobile (web) | `src/shared/auth/{googleIdentity, GoogleSignInButton}`, `authApi.googleLogin`, `useGoogleLogin`, botón "Continuar con Google" en `LoginScreen` y `RegisterScreen` | `googleIdentity.test.ts`, `GoogleSignInButton.test.tsx`, pruebas de las dos pantallas y de `authApi` |

> [!important] Decisiones de seguridad (cambian la spec original)
> - Se vincula por correo **solo la primera vez y solo si Google lo marca verificado**; después se entra por el id del proveedor (`sub`), nunca por correo.
> - **Pre-secuestro:** si el correo tenía una cuenta que nunca se confirmó, se confirma al entrar con Google y su contraseña anterior se reemplaza por una inutilizable.
> - Las cuentas creadas con Google no tienen contraseña; pueden fijar una con "olvidé mi contraseña".
> - Sin IDs de cliente configurados, la API rechaza todo token.
> - En **Microsoft** no se vinculará solo por correo (su documentación dice que el claim `email` no está garantizado ni es inmutable): se identificará por `oid`/`sub`. Política exacta pendiente del PO.

> [!warning] Mientras la app de Google esté en modo prueba
> Solo entran los usuarios de prueba agregados en la pantalla de consentimiento (hasta 100). Para abrirla a cualquiera hay que **publicarla** en Google Cloud; con permisos básicos no exige revisión. Ver [[Despliegue en AWS - Guía de operación#Login con Google]].

### Perfil de usuario (nombre y celular) — ✅ slice TDD (parcial)
Spec: `waggo-workspace/specs/RF-001-RF-005-perfil-de-usuario.md` (estado `Borrador`: lo que falta es proveedores vinculados, paseador verificado y eliminar cuenta; cambiar contraseña, cambiar correo y cerrar sesión ya están en la spec de seguridad de la cuenta). Este slice (nombre completo y celular) ya se implementó en ambos repos **sin pasar antes por `Lista`** — ver desviación reportada.
| Capa | Archivos | Pruebas |
|---|---|---|
| Application | `Waggo.Application/Identity/Queries/GetMyProfile/`, `Commands/UpdateMyProfile/`; `UserProfileResponse` | `Tests/Waggo.Application.UnitTests/Identity/{Queries/GetMyProfile, Commands/UpdateMyProfile}` |
| Infrastructure | Migración `AddUserFullNameAndPhone` (`identity.users.{full_name, phone}`) | vía integración |
| Api | `Waggo.Api/Endpoints/Identity/UsersEndpoints.cs` — `GET/PUT /api/v1/users/me` (cualquier rol autenticado) | pruebas de integración de `Users` |
| Mobile | `src/features/profile/` — `profileApi` (`getMyProfile`, `updateMyProfile`), `useMyProfile`, `SettingsScreen` (resumen + entrada "Información personal"), `PersonalInfoScreen`, `EditProfileScreen` | `*.test.ts[x]` junto a cada archivo |

### UX-003 Perfil y configuración de cuenta — ✅ slice TDD
No es un RF del SRS: extiende `Identity` (RF-001/ADR-016) en vez de ampliar el contrato de registro. Spec: `waggo-workspace/specs/UX-003-perfil-y-configuracion-de-cuenta.md` · Decisión: [[ADR-021 Perfil de usuario como extensión de Identity|ADR-021]].
| Capa | Archivos | Pruebas |
|---|---|---|
| Domain | `User.{FullName, Phone, UpdateProfile, ChangePassword}`; 4 errores nuevos en `IdentityErrors` | `UserTests` (casos nuevos) |
| Application | `Identity/{PasswordPolicy, UserProfileResponse}`, `Identity/Queries/GetMyProfile/`, `Identity/Commands/{UpdateProfile, ChangePassword}/`; `RegisterCommandValidator` ahora delega en `PasswordPolicy` | `GetMyProfileHandlerTests`, `UpdateProfileHandlerTests`, `ChangePasswordHandlerTests` |
| Infrastructure | `UserConfiguration` (`full_name`, `phone`), migración `AddUserProfileFields` | vía integración |
| Api | `Waggo.Api/Endpoints/Users/` — `GET/PATCH /api/v1/users/me`, `POST /api/v1/users/me/password` (cualquier usuario autenticado, sin rol específico) | `UsersEndpointTests`, `UpdateProfileRequestValidatorTests`, `ChangePasswordRequestValidatorTests` |
| Mobile | `src/features/profile/` — `ProfileScreen` (`/perfil`), `EditProfileScreen` (`/perfil/editar`), `ChangePasswordScreen` (`/perfil/contrasena`); `httpClient.patch` nuevo; enlace "Perfil" en `_layout.tsx` solo con sesión; reutiliza `logout()` de `features/auth` | `*.test.ts[x]` junto a cada archivo |

> [!warning] Discrepancia vault ↔ código
> Esta nota ya listaba un slice "Recuperación de contraseña (RF-001 + RF-005)" con `Identity/PasswordPolicy` compartida. Al implementar UX-003 se verificó que ese código **no existe** en `waggo-api` (ni `ForgotPassword`/`ResetPassword`, ni la spec correspondiente en `specs/`). `Identity/PasswordPolicy` se creó de cero en este slice; si la recuperación de contraseña se retoma, debe reutilizarla. Ver detalle en la spec UX-003.

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

### Correos reales (Amazon SES) y código de 6 dígitos (RF-001 + RF-005) — ✅
> [!success] En `main` (2026-10-27)
> Mergeados `waggo-api#22` (envío con SES), `waggo-api#23` (código de 6 dígitos y reenvío), `waggo-mobile#22` y `waggo-workspace#23` (spec). Probado con SES real en *sandbox*: llegó el correo con el código y `POST /auth/confirm` lo aceptó.

Spec: `waggo-workspace/specs/RF-001-RF-005-envio-de-correos-reales.md` · Configuración de AWS: [[Guía AWS SES - prompt de configuración]]. El correo sale por Amazon SES (`us-east-1`) detrás de `IEmailSender`; la confirmación pasa a un código de 6 dígitos que vence a los 30 min, se invalida tras 5 intentos fallidos y se puede reenviar (espera de 60 s). Cambia el contrato: `POST /auth/confirm {email, code}` reemplaza al `GET` y aparece `POST /auth/resend-confirmation {email}`.
| Capa | Archivos | Pruebas |
|---|---|---|
| Infrastructure | `Options/Identity/{EmailOptions, EmailProvider, SesOptions, EmailOptionsValidator}`, `Services/Identity/SesEmailSender` (`AWSSDK.SimpleEmailV2`), selección por `Email:Provider` en `DependencyInjection` | `SesEmailSenderTests`, `EmailOptionsValidatorTests` |
| Api | `EmailStartupGuard` (Production exige `Ses`), `IdentityEndpoints` (`POST /confirm`, `POST /resend-confirmation`), `ConfirmRequest`, `ResendConfirmationRequest` + validadores | `EmailStartupGuardTests`, `ConfirmRequestValidatorTests`, `ResendConfirmationRequestValidatorTests`, `IdentityEndpointTests` |
| Application / Domain | `User` (`ConfirmationAttempts`, `Confirm`, `TryIssueConfirmationCode`), `ConfirmEmail*`, `ResendConfirmation/*`, `ConfirmationEmail`, `ITokenGenerator.GenerateNumericCode`, `EmailErrors.SendFailed`, `IdentityErrors.ConfirmationAttemptsExceeded`; `RegisterHandler` envía antes de guardar; migración `AddConfirmationAttempts` | `UserTests`, `ConfirmEmailHandlerTests`, `ResendConfirmationHandlerTests`, `RegisterHandlerTests`, `SecureTokenGeneratorTests` |
| Mobile | `src/features/auth` — `authApi` (`confirmEmail(email, code)`, `resendConfirmation`), `useAuth`, `ConfirmScreen` (código de 6 dígitos y "Reenviar código" con cuenta regresiva de 60 s) | `authApi.test.ts`, `ConfirmScreen.test.tsx` |

> [!info] Pendiente de este slice (operativo, no de código)
> Registrar el dominio y salir del *sandbox* de SES (hoy solo se puede escribir a correos verificados), y un configuration set con SNS para rebotes y quejas. El correo de recuperación de contraseña ya existe, ver la sección siguiente.

### Recuperación y seguridad de la cuenta (RF-001 + RF-005) — ✅ 5 slices TDD
Spec: `waggo-workspace/specs/RF-001-RF-005-recuperacion-y-seguridad-de-cuenta.md` (`Implementada`, 2026-10-27). Reutiliza `IEmailSender` y el código de 6 dígitos del slice de correos reales: vence a los 15 min, 5 intentos fallidos, espera de 60 s entre envíos y solo se guarda su hash. Las respuestas no revelan qué correos tienen cuenta, y cada cambio de contraseña o de correo manda un aviso de seguridad.
| Slice | Qué entrega | API | Mobile |
|---|---|---|---|
| A. Olvidé mi contraseña | `POST /auth/forgot-password`, `POST /auth/reset-password`: el código restablece la contraseña, cierra todas las sesiones y levanta el bloqueo por intentos de login | `User` (`TryIssuePasswordResetCode`, `ResetPassword`), `Identity/Commands/{ForgotPassword, ResetPassword}`, `PasswordStrength`, migración `AddPasswordResetCode` (api#24) | `ForgotPasswordScreen` (`/olvide-contrasena`), `ResetPasswordScreen` (`/restablecer-contrasena`), enlace en `LoginScreen`, `useResendCountdown` (mobile#23) |
| B. Cerrar sesión | `POST /auth/logout {refreshToken}` revoca ese refresh token (idempotente, anónimo) | `Identity/Commands/Logout` (api#25) | `authApi.logout`: borra la sesión local primero y revoca en la API sin bloquear (mobile#24) |
| C. Cambiar contraseña | `PUT /users/me/password`: pide la actual, deja abierta la sesión que cambia y cierra las demás | `User.ChangePassword`, `Identity/Commands/ChangePassword`, `RevokeAllForUserExceptAsync` (api#26) | `ChangePasswordScreen` (`/configuracion/contrasena`), entrada en Configuración (mobile#25) |
| D. Cambiar correo | `POST /users/me/email` manda el código al correo **nuevo**; `POST /users/me/email/confirm` lo aplica, cierra las demás sesiones y avisa al correo anterior | `User` (`RequestEmailChange`, `ConfirmEmailChange`), `Identity/Commands/{RequestEmailChange, ConfirmEmailChange}`, `SessionCloser`, `SecurityNotice`, migración `AddEmailChange` (api#27) | `ChangeEmailScreen` en dos pasos (`/configuracion/correo`) (mobile#26) |
| E. Cuenta sin confirmar | Login con `Identity.AccountNotConfirmed`: la app pide un código nuevo y abre la pantalla del código | — | `LoginScreen` (`onAccountNotConfirmed`) (mobile#27) |

> [!info] Decisiones tomadas al implementar
> Una contraseña incorrecta en cambiar contraseña o cambiar correo cuenta para el mismo bloqueo del login (5 intentos, 15 min): un access token robado no debe servir para adivinar la contraseña. Los endpoints de confirmación de cambio de correo y de cambio de contraseña reciben el `refreshToken` de la sesión actual (opcional) para dejarla abierta. Ver la spec.

> [!warning] Probado a mano solo en parte
> Los endpoints de los slices A y D se recorrieron contra la API local con JWT real y el proveedor `Development`. La app no se probó a mano en el navegador ni con SES real para estos flujos. En Development con `Authentication:UseDevelopmentUser` activo, los endpoints `/users/me/*` responden `Server.Unexpected` (el usuario falso no es un Guid): para probarlos a mano hay que arrancar con `Authentication__UseDevelopmentUser=false`.

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
| Usuarios de prueba en Development/Testing (DEV-001, `Seed:Users`): admin `admin@waggo.local` / `Admin1234`, dueño `dueno@waggo.local` / `Dueno1234`, paseador aprobado `paseador@waggo.local` / `Paseador1234` — ver [[API - Pipeline, seguridad y errores#Usuarios de prueba (solo Development y Testing)]]. (PR api#21, workspace#22, mergeados 2026-10-26) | `Waggo.Domain/Entities/Identity/User.Seed`, `Waggo.Infrastructure/{Options,Services}/Identity/{SeedUsersOptions,SeedUserOptions,DevelopmentUsersSeeder}`; spec `waggo-workspace/specs/DEV-001-usuarios-de-prueba.md` | `DevelopmentUsersSeederTests`, `SeededUsersTests`, `ProductionSettingsTests` |
| Sistema de diseño "Seashell & Sapphire" — paleta clara por defecto (oscuro vía el selector), Baloo 2 + Inter, radios más redondeados, íconos permitidos — reemplaza el tema oscuro+amarillo+Helvetica | `waggo-mobile/src/shared/ui/theme.ts` (+ `Button`, `_layout.tsx`) · [[ADR-018 Nuevo sistema de diseño Seashell y Sapphire|ADR-018]] (reemplaza [[ADR-013 Tema oscuro mobile-first con amarillo y Helvetica|ADR-013]]) | — |
| Navegación por pestañas inferior (dueño `(tabs)`: Inicio · Paseos · Mascotas · Configuración; paseador `(walker-tabs)`: Inicio · Configuración) y componente `Skeleton` compartido reemplazando `ActivityIndicator` en Inicio/Configuración/Mascotas/Paseos, con mensaje de error si falla la carga; "Perfil" se renombra a "Configuración" (`ProfileMenuScreen` → `SettingsScreen`) | `waggo-mobile/src/app/(tabs)/`, `src/app/(walker-tabs)/`, `src/shared/ui/Skeleton.tsx`, `src/features/profile/screens/SettingsScreen.tsx` · [[ADR-019 Navegación por pestañas y estados de carga con skeletons|ADR-019]] | — |

### INF-001 Despliegue público en AWS — ✅ (producción del piloto)
No es un RF del SRS: es infraestructura. Spec: `waggo-workspace/specs/INF-001-despliegue-publico-en-aws.md` · Decisión: [[ADR-020 Despliegue público en AWS con ECS Fargate y CDK|ADR-020]] · Operación: [[Despliegue en AWS - Guía de operación]].
| Capa | Qué | Dónde |
|---|---|---|
| Infraestructura | Stacks CDK de red, datos, registro, certificados, pipeline OIDC, API, web, sitio público y presupuesto | repo privado `waggo-infra` (pruebas Jest + CI con `cdk synth`) |
| API | `deploy.yml`: imagen a ECR, migraciones como paso (`--migrate-only`), servicio ECS y smoke test | `waggo-api/.github/workflows/deploy.yml` |
| Web | `deploy.yml` (`deploy-web`): `expo export`, S3, invalidación de CloudFront y smoke test | `waggo-mobile/.github/workflows/deploy.yml` |
| Sitio público | Landing, privacidad y términos en `mywaggo.com` | `waggo-infra/site/` |

> [!warning] Pendiente
> Acceso a producción de SES (solicitud en revisión: hasta entonces el correo solo llega a direcciones verificadas), eventos de rebote y queja a SNS, EAS Build (CD-3) y revisión legal de privacidad y términos.

## Pendiente por RF
| RF | Módulo | Sprint |
|---|---|---|
| RF-001 + RF-005 login social: **Microsoft** y Google en iOS/Android (Google en la web ya está) — spec `En progreso` (ADR-017) | `Identity` | 1 |
| RF-003 (proveedor de verificación, foto y documento escaneado) | `Identity` | 2 |
| RF-014 push (Expo Push o FCM/APNs; la bandeja ya existe) | `Messaging` | 6 |
| RF-015 – RF-018 (proveedor real, tarjetas y cuentas de paseadores) | `Payments` | 7 |
| RF-021 | `Analytics` | 9 |

## Deuda técnica conocida
- [x] `EFCore.NamingConventions` (snake_case) y schema por módulo — aplicado con `Pets` ([[Base de datos - PostgreSQL]]).
- [x] NetTopologySuite para PostGIS — aplicado con `Walks` (`GeoPointConverter`).
- [ ] 🚧 Test de arquitectura que impida referencias cruzadas entre módulos — `ModuleTests` con lista explícita ([[ADR-002 Clean Architecture en el backend|ADR-002]]). En progreso: sin commitear en `.worktrees/api-module-boundaries` (rama `test/module-boundaries`, sin PR).
- [ ] 🚧 Umbral de cobertura bloqueante en el backend — Domain + Application ≥ 80 % (`scripts/coverage-gate.sh`). En progreso: mismo worktree que `ModuleTests`.
- [ ] Publicación de imágenes en GHCR (CD-1, ver [[CI-CD]]).
- [ ] Límite de intentos por IP en `/auth/*` (hoy el límite es por cuenta: 5 intentos por código y bloqueo de login).
- [ ] El registro revela si un correo ya tiene cuenta (`Identity.EmailAlreadyRegistered`); la recuperación y el reenvío sí responden igual para todos.
- [ ] Probar a mano en la app, con SES real, los flujos de recuperación y cambio de correo.
- [ ] Revocar también el access token al cerrar sesión (hoy sigue valiendo hasta que vence, 15 min).
- [ ] Una prueba de `AddPetScreen` (mobile) falló de forma intermitente en corridas completas locales; no se reprodujo ni se encontró la causa. El timeout por defecto de Jest ya subió a 15 s.
