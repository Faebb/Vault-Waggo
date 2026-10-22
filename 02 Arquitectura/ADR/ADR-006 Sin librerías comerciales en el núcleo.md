---
title: ADR-006 Sin librerías comerciales en el núcleo
tags: [waggo, adr]
status: Aceptada
date: 2026-09-26
---

# ADR-006 Sin librerías comerciales en el núcleo

Volver a [[00 - MOC Waggo]]

> [!info] Estado: **Aceptada** · 2026-09-26

## Contexto
MediatR (v13+), AutoMapper (v15+) y FluentAssertions (v8+) pasaron a licencias comerciales.

## Decisión
No usarlas. Handlers propios (`ICommandHandler`/`IQueryHandler`), mapeo manual y **Shouldly** para aserciones.

## Consecuencias
- ✅ Sin costos ni riesgo legal.
- ✅ Menos magia: el flujo es explícito y fácil de depurar.
- ⚠️ Un poco más de código de plumbing (mínimo).
