---
title: ADR-001 Repositorios separados
tags: [waggo, adr]
status: Aceptada
date: 2026-09-26
---

# ADR-001 Repositorios separados

Volver a [[00 - MOC Waggo]]

> [!info] Estado: **Aceptada** · 2026-09-26

## Contexto
Necesitamos decidir si backend y mobile viven en un monorepo o en repos separados.

## Decisión
Dos repositorios en la cuenta personal de GitHub **Faebb**: `waggo-api` y `waggo-mobile`. Ambos se clonan dentro de `source\repos\Waggo\`.

## Consecuencias
- ✅ Pipelines, versionado y permisos independientes; un dev front no necesita el SDK de .NET.
- ✅ Cada repo se despliega a su ritmo.
- ⚠️ El contrato de la API debe versionarse (`/api/v1`) y publicarse vía OpenAPI.
- ⚠️ Cambios que tocan ambos lados requieren dos PR coordinados (se enlazan entre sí).
