---
title: Requerimientos no funcionales
tags: [waggo, requerimientos, srs]
source: Requerimientos_App_Paseo_Perros.xlsx
created: 2026-09-26
---

# Requerimientos no funcionales

Volver a [[00 - MOC Waggo]]

| ID | Categoría | Requerimiento | Prio | Cómo lo atendemos |
|---|---|---|---|---|
| RNF-001 | Seguridad | Contraseñas con bcrypt | Alta | `IPasswordHasher` en Application, implementación BCrypt en Infrastructure. Test: el hash nunca es igual al texto plano y verifica correctamente. |
| RNF-002 | Seguridad | Sesión con JWT | Alta | JWT Bearer en la API (access corto + refresh token rotativo). Ver [[Backend - Clean Architecture#Seguridad]]. |
| RNF-003 | Seguridad | Cifrado de datos sensibles | Alta | Cifrado a nivel de campo (value converters de EF Core) + llaves fuera del código (secrets / Key Vault). |
| RNF-004 | Seguridad / Cumplimiento | No almacenar tarjetas | Alta | Solo tokens de la pasarela; **test de arquitectura** prohíbe entidades con campos tipo `CardNumber`. |
| RNF-005 | Cumplimiento | GDPR / Ley 1581 de 2012 (Colombia) | Alta | Consentimiento explícito, borrado/anonimización, minimización de datos. |
| RNF-006 | Rendimiento | 500 usuarios / 50 paseadores (piloto) | Alta | Pruebas de carga con k6 antes del lanzamiento. |
| RNF-007 | Rendimiento | Picos concurrentes y GPS sin caídas | Alta | SignalR para tracking (✅ `/hubs/walks`), ingesta de posiciones por lotes, PostGIS con índices GiST. Al escalar a varias instancias hará falta un *backplane* (Redis). |
| RNF-008 | Disponibilidad | Alta disponibilidad GPS, pagos, push | Alta | Contenedores sin estado, health checks, reintentos (Polly / `Microsoft.Extensions.Http.Resilience`), outbox para eventos. |
| RNF-009 | Arquitectura | Monolito modular migrable a microservicios | Media | Clean Architecture con **carpetas por módulo** dentro de cada capa. Ver [[ADR-002 Clean Architecture en el backend]]. |
| RNF-010 | Portabilidad | iOS y Android | Media | Expo + React Native (también exporta Web). Ver [[ADR-003 Expo y TypeScript]]. |
| RNF-011 | Mantenibilidad | Scrum/Kanban, sprints de 2 semanas, Jira o Linear | Media | Ver [[Convenciones de Git y ramas]] y [[Definition of Done]]. |

> [!tip] RNF verificables con pruebas
> Cada RNF debe tener al menos una prueba automática que lo "vigile": tests de arquitectura (RNF-004, RNF-009), tests de integración de seguridad (RNF-001, RNF-002) y pruebas de carga (RNF-006, RNF-007). Ver [[Estrategia de pruebas]].
