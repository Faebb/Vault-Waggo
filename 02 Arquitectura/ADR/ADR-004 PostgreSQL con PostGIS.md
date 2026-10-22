---
title: ADR-004 PostgreSQL con PostGIS
tags: [waggo, adr]
status: Aceptada
date: 2026-09-26
---

# ADR-004 PostgreSQL con PostGIS

Volver a [[00 - MOC Waggo]]

> [!info] Estado: **Aceptada** · 2026-09-26

## Contexto
El núcleo del producto es geográfico: cercanía (RF-006), rastreo (RF-008), geocercas (RF-009).

## Decisión
PostgreSQL 17 con extensión **PostGIS**, accedido con EF Core + Npgsql + NetTopologySuite.

## Consecuencias
- ✅ Consultas de cercanía y contención de polígonos en la base de datos, con índices GiST.
- ✅ Open source, disponible como imagen Docker oficial.
- ⚠️ Las pruebas de integración deben usar PostgreSQL real (Testcontainers).
