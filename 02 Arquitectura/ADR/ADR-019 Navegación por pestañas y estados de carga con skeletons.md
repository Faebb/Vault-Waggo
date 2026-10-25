---
title: ADR-019 Navegación por pestañas y estados de carga con skeletons
tags: [waggo, adr, frontend, navegación, diseño]
status: Aceptada
date: 2026-10-25
---

# ADR-019 Navegación por pestañas y estados de carga con skeletons

Volver a [[00 - MOC Waggo]] · Guía: [[Frontend - Expo feature-based]] · Amplía: [[ADR-018 Nuevo sistema de diseño Seashell y Sapphire|ADR-018]]

> [!info] Estado: **Aceptada** · 2026-10-25 · Referencia visual: mockup de Figma "Food Delivery App (Community)" (community file, node-id 38-1923)

## Contexto
La app solo tenía un `Stack` raíz (`src/app/_layout.tsx`): cada pantalla se alcanza empujando otra sobre la anterior desde accesos dentro de Inicio (`Shortcut`s). El dueño llega a Mascotas, Paseos o Perfil solo por ese camino — no hay una barra de navegación persistente — y no existe forma de volver a Inicio salvo deshacer la pila. El usuario lo señaló directamente: "hay muchas páginas a las que es difícil acceder, no hay un menú", pidiendo además un diseño "más profesional y con skeletons", usando como referencia el mockup de Figma de arriba (header oscuro con decoración radial, tarjetas con sombra suave y esquinas muy redondeadas, badges de rating, chips de categoría).

Las pantallas que cargan datos (`useMyProfile`, `useMyPets`, `useMyWalks`, etc.) hoy muestran un `ActivityIndicator` centrado mientras cargan — funcional pero no es el estándar "profesional" que pide el mockup (que usa placeholders con la forma del contenido final).

## Decisión
1. **Barra de pestañas inferior para el rol dueño** (Expo Router `Tabs`, grupo de ruta `(tabs)`): **Inicio · Paseos · Mascotas · Perfil**. Vive en `src/app/(tabs)/_layout.tsx`; cada pestaña es la pantalla "raíz" de esa sección (`inicio`, `paseos/index`, `mascotas/index`, `perfil/index`). Las pantallas de detalle (`paseos/[id]`, `paseos/nuevo`, `mascotas/nueva`, `perfil/info`, `perfil/editar`, `notificaciones`, `cotizar`, `admin/*`) siguen viviendo en el `Stack` raíz, empujadas **sobre** las pestañas (ocultan la barra al entrar, como cualquier app de este estilo) — no hace falta un stack anidado por pestaña.
   - Rol paseador (`paseador/*`) **no** cambia en este ADR: hoy tiene una sola pantalla principal; se revisita cuando tenga más de 2-3 secciones propias.
   - Iconos: se reutiliza/extiende el set de línea ya creado para auth (`PawIcon`, `MapRouteIcon`, etc., ADR-018) en vez de añadir una librería de iconos.
2. **Componente `Skeleton` compartido** (`src/shared/ui/Skeleton.tsx`): bloque con `radius` y tamaño configurables, animación de brillo (shimmer) con `Animated`/`react-native-reanimated` ya presente en Expo. Reemplaza los `ActivityIndicator` de pantallas con datos as-you-scroll (Inicio, Perfil, Mascotas, Paseos) por una silueta de la pantalla cargada (p. ej. tarjetas grises del mismo alto/ancho que las reales). Patrón: cada pantalla define su propio `<XScreenSkeleton />` compuesto con `Skeleton`, igual que ya hace cada pantalla con sus propios estilos via `useThemedStyles`.
3. **Pulido visual de tarjetas de lista** (Inicio, Paseos, Mascotas) inspirado en el mockup: sombra suave + borde sutil en vez de solo borde, miniatura/ícono más prominente a la izquierda, metadatos secundarios en `colors.muted`. Se mantiene la paleta Seashell & Sapphire (ADR-018): nada de los colores naranja/negro del mockup, solo su *estructura* (jerarquía, densidad, uso de sombra y espacio).

## Consecuencias
- ✅ Resuelve el acceso directo a Mascotas/Paseos/Perfil desde cualquier punto de la app (pestañas siempre visibles), sin rehacer la arquitectura de rutas de detalle.
- ✅ El patrón `Skeleton` es incremental: no migra las ~15 pantallas de una vez; se aplica primero donde ya hay `ActivityIndicator` (Inicio, Perfil) y se extiende pantalla por pantalla en PRs siguientes.
- ⚠️ El rol paseador queda sin pestañas por ahora (asimetría temporal entre roles); se documenta como pendiente, no como decisión final.
- ⚠️ Mover `inicio.tsx`, `paseos/index.tsx`, `mascotas/index.tsx`, `perfil/index.tsx` a `(tabs)/` cambia sus rutas internas (sin cambiar lo que ve el usuario) — hay que revisar cualquier `router.push('/inicio')`, etc. literal en el código y tests.
- ⏳ Pendiente (fuera de esta decisión): aplicar el mismo pulido visual a Notificaciones, Admin, Cotizar y a las pantallas de paseo en vivo (tracking); pestañas para el rol paseador cuando tenga más secciones.
