---
title: ADR-018 Nuevo sistema de diseño "Seashell & Sapphire"
tags: [waggo, adr, frontend, diseño]
status: Aceptada
date: 2026-10-24
---

# ADR-018 Nuevo sistema de diseño "Seashell & Sapphire"

Volver a [[00 - MOC Waggo]] · Guía: [[Frontend - Expo feature-based]] · Reemplaza: [[ADR-013 Tema oscuro mobile-first con amarillo y Helvetica|ADR-013]] · Amplía/ajusta: [[ADR-015 Selector de tema claro-oscuro|ADR-015]]

> [!info] Estado: **Aceptada** · 2026-10-24 · Mockup: `Vault Waggo/Sistema de diseño - Seashell & Sapphire.html`

## Contexto
ADR-013 fijó un tema **oscuro por defecto**, amarillo como único acento, Helvetica y estilo suizo sin íconos. El PO trajo un nuevo mockup ("Seashell & Sapphire": fondo crema cálido, azul zafiro como acento, tipografía Baloo 2 + Inter, esquinas muy redondeadas e íconos de línea) y pidió reemplazar el sistema de diseño anterior por este. El mockup anterior (`Sistema de diseño-html.zip`/carpeta, el que originó ADR-013/ADR-015) queda **superado**: se retira de la raíz del vault.

## Decisión
- **Paleta** (claro es ahora el tema por defecto; oscuro sigue disponible, ADR-015 no cambia el mecanismo de selector):

  | Token | Claro | Oscuro |
  |---|---|---|
  | `background` (`--body-bg`) | `#FFF0E9` | `#16212E` |
  | `surface` | `#FFFFFF` | `#1C2733` |
  | `surface2` | `#FBE2D3` | `#22303E` |
  | `border` | `#E8D2C3` | `#2A3B4D` |
  | `text` (`--body-color`) | `#16212E` | `#FFF0E9` |
  | `muted` (`--text-secondary`) | `#3A4A5C` | `#8FA5B8` |
  | `primary` | `#3059A4` | `#5B9BD5` |
  | `primaryText` | `#FFFFFF` | `#16212E` |
  | `secondary` | `#006E87` | `#3FA3BD` |
  | `info` | `#2E9EC4` | `#7FD4EC` |
  | `warning` | `#C97A2B` | `#F2A65A` |
  | `success` | `#1F7A52` | `#4CC38A` |
  | `danger` | `#C23B3B` | `#FF7A7A` |

  A diferencia de ADR-013 (mismo amarillo en ambos temas), aquí **el acento cambia de tono entre claro y oscuro** (500 en claro, 300 en oscuro) para mantener contraste AA sobre cada fondo — así lo documenta el propio mockup.
- **Tipografía**: **Baloo 2** (700) para títulos y encabezados (`display`, `title`, `subtitle`/encabezado); **Inter** (400 cuerpo, 700 etiquetas, 500 micro) para el resto. Se retira Helvetica. Los nombres de los tokens de `typography` en `theme.ts` no cambian (`display/title/subtitle/body/caption/eyebrow`, más un nuevo `micro`) para no romper los ~100 usos ya existentes en pantallas — solo cambian sus valores (familia, tamaño, interlineado) a la escala del mockup (Display 34/38, Título 24/30, Encabezado 18/24, Cuerpo 16/24, Apoyo 14/20, Etiqueta 13/16 mayúsculas, Micro 12/16).
- **Radios**: más redondeados (8/14/20/28 y *pill* en botones) en vez de 6/12 (ADR-013).
- **Espaciado**: la escala existente (4/8/16/24/32) ya coincide con la nueva base-4 del mockup; no cambia.
- **Íconos permitidos**: se retira la regla "sin íconos decorativos" de ADR-013. El mockup usa íconos de línea (stroke, sin relleno) junto al texto en botones y controles; se adoptan igual, como parte del lenguaje visual, no como decoración suelta.
- **Tema por defecto: claro.** Difiere de ADR-013 ("arranca en oscuro"); el selector de ADR-015 sigue funcionando igual, solo cambia cuál es el estado inicial.
- **Mobile first se mantiene sin cambios** (columna de máximo 480 px, acción principal abajo) — eso es de ADR-013 y no lo toca este mockup.

## Consecuencias
- ✅ Como el proyecto ya obligaba "solo tokens de `theme.ts`, nada suelto" (ADR-013, checklist de mobile), actualizar los *valores* de los tokens alcanza para que toda la app adopte el nuevo look sin tocar pantalla por pantalla.
- ✅ Mejor contraste: el acento ya no es idéntico en ambos temas, pensado para pasar AA en cada fondo.
- ⚠️ Carga de fuentes nueva (`@expo-google-fonts/baloo-2`, `@expo-google-fonts/inter`) con `useFonts` en `src/app/_layout.tsx`; la app debe esperar a que carguen antes de pintar (antes no hacía falta, Helvetica es del sistema).
- ⚠️ Los componentes que ya dibujaban su propio ícono o layout fijo asumiendo "sin íconos" (ninguno crítico hoy) se revisan cuando se toquen, no se migran todos de una vez.
- ⏳ Pendiente (fuera de esta decisión): íconos de línea reales en `Button`/controles (hoy el mockup los muestra, el código base de componentes no los agrega todavía); revisar cada pantalla contra el mockup para detalles finos de layout.

## Relación con ADR-013 y ADR-015
ADR-013 queda **superada** por esta decisión en todo lo relativo a paleta, tipografía, radios e íconos; su principio de "mobile first, una acción principal, sin colores sueltos" se mantiene y pasa a regir bajo esta paleta nueva. ADR-015 (selector claro/oscuro) no cambia: solo se invierte cuál de los dos es el modo inicial.
