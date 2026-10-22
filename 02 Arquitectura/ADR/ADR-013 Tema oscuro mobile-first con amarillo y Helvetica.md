---
title: ADR-013 Tema oscuro mobile-first con amarillo y Helvetica
tags: [waggo, adr, frontend, diseño]
status: Aceptada
date: 2026-09-30
---

# ADR-013 Tema oscuro mobile-first con amarillo y Helvetica

Volver a [[00 - MOC Waggo]] · Guía: [[Frontend - Expo feature-based]]

> [!info] Estado: **Aceptada** · 2026-09-30 · Decidida por el equipo al revisar la primera versión de UX-001

## Contexto
La primera versión de la bienvenida (UX-001) usaba fondo claro, verde, emojis como íconos y tarjetas centradas. Se veía genérica, "hecha con IA", y no transmitía una identidad propia. La app se usa sobre todo en el celular, en la calle y con una mano.

## Decisión
- **Mobile first**: se diseña para el ancho de un teléfono. En tablet y web el contenido mantiene una columna de máximo 480 px, y la acción principal va abajo, al alcance del pulgar.
- **Tema oscuro** único (`userInterfaceStyle: dark`): fondo `#0F0F0D`, superficies `#1A1A17`, bordes `#2C2B26`, texto `#F4F1E8`, texto secundario `#9A968A`.
- **Amarillo `#FFC527` como único acento**: acción principal, selección y cifras clave. El texto sobre amarillo es casi negro (`#14120A`).
- **Rojo `#FF6B5E` (`colors.danger`) solo para errores y emergencias** (RF-012): el botón "Emergencia" y el aviso de alerta. Nunca como decoración, para que siga significando "atención".
- **Helvetica** como tipografía: `Helvetica Neue` en iOS y `Helvetica` en web. Android no incluye Helvetica y usa su sans del sistema; no se empaqueta la fuente porque su licencia es comercial ([[ADR-006 Sin librerías comerciales en el núcleo|ADR-006]]).
- **Estilo tipográfico suizo**: titulares grandes alineados a la izquierda, listas numeradas (`01`, `02`, `03`) con separadores finos, sin emojis ni íconos decorativos.
- Todos los valores viven en `src/shared/ui/theme.ts` (`colors`, `fontFamily`, `typography`, `spacing`, `radius`). Las pantallas no usan colores ni tamaños sueltos.

## Consecuencias
- ✅ Identidad visual reconocible y coherente en todas las pantallas, incluido el cotizador (RF-019).
- ✅ Buen contraste del amarillo sobre fondo oscuro para la acción principal.
- ⚠️ En Android la tipografía no es Helvetica. Si se quiere idéntica, habría que licenciarla o elegir una alternativa libre (por ejemplo Inter) con un ADR nuevo.
- ⚠️ No hay tema claro. Si se pide, se agregan los tokens claros en `theme.ts` y se cambia `userInterfaceStyle` a `automatic`.
