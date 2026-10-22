---
title: ADR-003 Expo y TypeScript
tags: [waggo, adr]
status: Aceptada
date: 2026-09-26
---

# ADR-003 Expo y TypeScript

Volver a [[00 - MOC Waggo]]

> [!info] Estado: **Aceptada** · 2026-09-26

## Contexto
RNF-010 pide iOS y Android con React Native o Flutter. Se eligió React Native.

## Decisión
Usar **Expo** (managed workflow, Expo Router, EAS Build) con **TypeScript strict** y arquitectura *feature-based*.

## Consecuencias
- ✅ Sin Xcode/Android Studio para empezar; builds en la nube con EAS.
- ✅ Exporta también a Web → contenedor Docker.
- ✅ Soporte oficial para push, mapas, ubicación en segundo plano (config plugins).
- ⚠️ Librerías nativas no incluidas en Expo Go requieren *development build*.
