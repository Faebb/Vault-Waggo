---
title: ADR-002 Clean Architecture en el backend
tags: [waggo, adr]
status: Aceptada
date: 2026-09-26
---

# ADR-002 Clean Architecture en el backend

Volver a [[00 - MOC Waggo]]

> [!info] Estado: **Aceptada** · 2026-09-26

## Contexto
El SRS pide un monolito modular migrable a microservicios (RNF-009). El equipo es pequeño y junior/mixto.

## Decisión
Clean Architecture clásica con 4 proyectos (`Domain`, `Application`, `Infrastructure`, `Api`) y **carpetas por módulo** dentro de cada capa. Reglas de dependencia verificadas con NetArchTest.

## Consecuencias
- ✅ Estructura conocida y documentada; curva de aprendizaje baja.
- ✅ El dominio queda aislado y 100 % testeable sin infraestructura (ideal para TDD).
- ⚠️ Menos aislamiento entre módulos que un monolito modular: se mitiga con carpetas/namespaces por módulo y, cuando se mergee (🚧 en progreso, sin mergear), con `Tests/Waggo.ArchitectureTests/ModuleTests.cs`, que solo permitirá las dependencias entre módulos de una lista explícita. Con él, una dependencia nueva romperá el build hasta agregarla a la lista y justificarla aquí.

### Dependencias permitidas entre módulos (2026-10-20)
| Módulo | Puede usar | Por qué |
|---|---|---|
| Identity | Walkers | El rol `walker` de la sesión sale del perfil de paseador aprobado |
| Walks | Pets, Pricing, Payments, Walkers, Notifications | Un paseo es de las mascotas del dueño, se cotiza, se retiene en escrow, lo toma un paseador verificado y avisa cada cambio |
| Payments | Walks | El pago pertenece a un paseo y valida quién participa |
| Tracking | Walks, Notifications | La ruta y las alertas son de un paseo; las alertas avisan |
| Messaging | Walks, Notifications | El chat es de un paseo; `NotificationConfiguration` vive en el schema `messaging` |
| Notifications | Walks | Cada aviso apunta a un paseo |
| Pets, Pricing, Walkers | — | No dependen de otros módulos |

> [!warning] Ciclos conocidos
> Walks ↔ Payments y Walks ↔ Notifications. Si se separan en servicios, habrá que romperlos con eventos (p. ej., "paseo terminado" → cobro y aviso).
- 🔁 Revisar esta decisión cuando existan 3+ módulos con tráfico real.
