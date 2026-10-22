---
title: Definition of Done
tags: [waggo, scrum, proceso]
created: 2026-09-26
---

# Definition of Done

Volver a [[00 - MOC Waggo]]

Una historia está **terminada** cuando:

- [ ] Los criterios de aceptación del RF están cubiertos por pruebas automáticas.
- [ ] El código se escribió con [[Flujo TDD|TDD]] (la prueba existía antes que el código).
- [ ] `dotnet test` / `npm test` pasan en local y en CI.
- [ ] Cobertura de Domain/Application ≥ 80 % (sin bajar respecto a `main`).
- [ ] Cumple las [[Convenciones de código]]: build sin warnings, `dotnet format` / `npm run format:check`, lint (`npm run lint`) y typecheck (`npm run typecheck`) en verde.
- [ ] Los tests de arquitectura pasan.
- [ ] Pasa la checklist del repo en `waggo-workspace/.claude/rules/` (`backend-checklist.md` o `mobile-checklist.md`): archivos en su carpeta ([[Estructura de carpetas del backend]]), validación en dos capas, errores y política de rol.
- [ ] La imagen Docker construye y el `/health` responde.
- [ ] Contrato de API actualizado (OpenAPI) y, si aplica, PR enlazado en el otro repo.
- [ ] Nota en el vault actualizada si cambió una decisión (ADR) o una regla de negocio.
- [ ] PR revisado y aprobado por al menos una persona.

## Definition of Ready (para entrar al sprint)
- [ ] Historia con RF asociado y criterios de aceptación.
- [ ] Tabla de ejemplos (entradas/salidas) acordada con el PO.
- [ ] Dependencias externas identificadas (APIs, llaves, cuentas sandbox).
