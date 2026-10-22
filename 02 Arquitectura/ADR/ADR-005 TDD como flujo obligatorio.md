---
title: ADR-005 TDD como flujo obligatorio
tags: [waggo, adr]
status: Aceptada
date: 2026-09-26
---

# ADR-005 TDD como flujo obligatorio

Volver a [[00 - MOC Waggo]]

> [!info] Estado: **Aceptada** · 2026-09-26

## Contexto
Se busca calidad desde el día uno y un equipo pequeño que pueda refactorizar sin miedo.

## Decisión
Todo cambio de comportamiento sigue **Red → Green → Refactor**. El PR debe mostrar la prueba que cubre el cambio. Cobertura mínima: 80 % en Domain y Application.

## Consecuencias
- ✅ Diseño guiado por uso; menos regresiones.
- ✅ Las pruebas documentan las reglas de negocio (ej. tarifa RF-019).
- ⚠️ Al inicio se siente más lento; se compensa en mantenimiento.
- Ver [[Flujo TDD]].
