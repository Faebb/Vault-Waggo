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
1. **Barra de pestañas inferior para los dos roles** (Expo Router `Tabs`, un grupo de ruta por rol):
   - Dueño, grupo `(tabs)`: **Inicio · Paseos · Mascotas · Configuración**. Vive en `src/app/(tabs)/_layout.tsx`; cada pestaña es la pantalla "raíz" de esa sección (`inicio`, `paseos/index`, `mascotas/index`, `configuracion/index`).
   - Paseador, grupo `(walker-tabs)`: **Inicio · Configuración** (`paseador/index`, `paseador/configuracion`) — hoy el paseador tiene una sola pantalla principal (solicitudes + paseos asignados en una vista), así que dos pestañas alcanzan; se amplía si gana más secciones propias.
   - En ambos casos, las pantallas de detalle (`paseos/[id]`, `paseos/nuevo`, `mascotas/nueva`, `configuracion/info`, `configuracion/editar`, `notificaciones`, `cotizar`, `admin/*`, `paseador/[id]`, `paseador/notificaciones`) siguen viviendo en el `Stack` raíz, empujadas **sobre** las pestañas (ocultan la barra al entrar) — no hace falta un stack anidado por pestaña.
   - Iconos: se reutiliza/extiende el set de línea ya creado para auth (`PawIcon`, `MapRouteIcon`, etc., ADR-018) en vez de añadir una librería de iconos.
2. **"Perfil" se renombra a "Configuración"**: la pestaña ya no es solo los datos de la cuenta, es el menú de ajustes — el acceso al perfil ("Información personal") pasa a ser una entrada *dentro* de Configuración, junto a una fila nueva "Tema" (el selector claro/oscuro, que antes solo vivía en el header que las pantallas de pestañas ya no muestran) y "Cerrar sesión". `ProfileMenuScreen` se renombra a `SettingsScreen`; sus accesos a Mascotas/Paseos ahora son opcionales (`onOpenPets?`, `onOpenWalks?`) para que el paseador reuse el mismo componente sin esas entradas, que no aplican a su rol.
3. **Componente `Skeleton` compartido** (`src/shared/ui/Skeleton.tsx`): bloque con `radius` y tamaño configurables, animación de opacidad (pulso) con `Animated` de React Native (sin añadir `react-native-reanimated`). Reemplaza los `ActivityIndicator` de pantallas con datos (Inicio, Configuración, Mascotas, Paseos) por una silueta de la pantalla cargada (p. ej. tarjetas grises del mismo alto/ancho que las reales), y las pantallas que antes se quedaban en blanco si la carga fallaba ahora muestran un mensaje de error. Patrón: cada pantalla define su propio `<XScreenSkeleton />` compuesto con `Skeleton`, igual que ya hace cada pantalla con sus propios estilos via `useThemedStyles`.
4. **Pulido visual de tarjetas de lista** (Inicio, Paseos, Mascotas) inspirado en el mockup: sombra suave + borde sutil en vez de solo borde, miniatura/ícono más prominente a la izquierda, metadatos secundarios en `colors.muted`. Se mantiene la paleta Seashell & Sapphire (ADR-018): nada de los colores naranja/negro del mockup, solo su *estructura* (jerarquía, densidad, uso de sombra y espacio).

## Consecuencias
- ✅ Resuelve el acceso directo a Mascotas/Paseos/Configuración desde cualquier punto de la app, para los dos roles, sin rehacer la arquitectura de rutas de detalle.
- ✅ El patrón `Skeleton` es incremental: no migra las ~15 pantallas de una vez; se aplica primero donde ya hay `ActivityIndicator` (Inicio, Configuración, Mascotas, Paseos) y se extiende pantalla por pantalla en PRs siguientes.
- ⚠️ Mover `inicio.tsx`, `paseos/index.tsx`, `mascotas/index.tsx`, `perfil/*` a `(tabs)/configuracion/` y `paseador/index.tsx` a `(walker-tabs)/` cambia rutas internas y el nombre del componente de perfil — se revisó cada `router.push('/perfil')`/`'/paseador'` literal y sus tests.
- ⏳ Pendiente (fuera de esta decisión): aplicar el mismo pulido visual a Notificaciones, Admin, Cotizar y a las pantallas de paseo en vivo (tracking).
