---
title: ADR-015 Selector de tema claro/oscuro
tags: [waggo, adr, frontend, diseño]
status: Aceptada
date: 2026-10-01
---

# ADR-015 Selector de tema claro/oscuro

Volver a [[00 - MOC Waggo]] · Guía: [[Frontend - Expo feature-based]] · Amplía [[ADR-013 Tema oscuro mobile-first con amarillo y Helvetica|ADR-013]]

> [!warning] Ajustada por [[ADR-018 Nuevo sistema de diseño Seashell y Sapphire|ADR-018]] · 2026-10-24
> El mecanismo descrito aquí (mini-store, `useThemedStyles`, persistencia local) sigue vigente tal cual. Lo que cambia: los valores de `lightColors`/`darkColors` (nueva paleta, el acento ya no es idéntico en ambos temas) y el modo por defecto, que pasa de oscuro a **claro**. El mockup `Vault Waggo/Sistema de diseño-html` que originó esta ADR fue reemplazado por `Sistema de diseño - Seashell & Sapphire.html`.

> [!info] Estado: **Aceptada** · 2026-10-01 · Pedida por el PO a partir del mockup "Sistema de diseño v1"

## Contexto
ADR-013 dejó el tema oscuro como el único disponible y anotó explícitamente: *"No hay tema claro. Si se pide, se agregan los tokens claros en `theme.ts` y se cambia `userInterfaceStyle` a `automatic`"*. El PO trajo un mockup (`Vault Waggo/Sistema de diseño-html`) con los tokens de ambos temas y un selector "Claro/Oscuro" en la cabecera, y pidió aplicar ese estilo a la app.

## Decisión
- **Se agregan tokens claros** (`lightColors`) junto a los oscuros ya aceptados (`darkColors`) en `src/shared/ui/theme.ts`. El acento amarillo (`#FFC527`) y su texto casi negro (`#14120A`) se mantienen **idénticos** en ambos temas (ADR-013: único acento); no se adoptan los hex ligeramente distintos del mockup (`#FFC61A`, etc.) para no alterar la identidad ya aceptada sin que lo pida el PO.
- **La app sigue arrancando en oscuro** (no sigue `userInterfaceStyle: automatic` ni el esquema del sistema operativo): cambiar de tema es una acción explícita del usuario, para no alterar la experiencia ya aceptada en ADR-013 sin que el usuario lo pida. `app.json` no cambia.
- **Selector `ThemeToggle`**: control de dos opciones, "Claro"/"Oscuro", solo texto (sin íconos, estilo suizo de ADR-013), en la cabecera del `Stack` raíz (`src/app/_layout.tsx`), visible en todas las pantallas.
- **Persistencia local**: la elección se guarda en el dispositivo (`@react-native-async-storage/async-storage`, clave `waggo:theme-mode`), no en el backend; es una preferencia de UI, no un dato de negocio.
- **Mecanismo**: `theme.ts` deja de exportar `colors` como objeto plano. Expone un mini-store fuera de React (`getThemeMode`/`setThemeMode`) y los hooks `useThemeMode`, `useColors` y `useThemedStyles`. Las pantallas que antes hacían `const styles = StyleSheet.create({...})` a nivel de módulo (cerrando sobre `colors`) ahora llaman `const styles = useThemedStyles((colors) => ({...}))` como primera línea del componente, para que el `StyleSheet` se recalcule cuando cambia el tema. No hace falta envolver la app en un `ThemeProvider`: los hooks funcionan en cualquier componente sin un ancestro especial, así que las ~30 pantallas y componentes existentes no necesitaron un wrapper nuevo en sus pruebas.
- **Tipografía, espaciado y radios no cambian**: el mockup los documenta iguales en ambos temas (no son parte del "tema").

## Consecuencias
- ✅ El selector queda disponible sin backend ni `ThemeProvider` envolviendo la app; cada pantalla sigue usando solo tokens de `theme.ts`.
- ✅ Toda la app (cabeceras, pantallas, componentes compartidos) respeta el tema activo de inmediato al cambiar el selector.
- ⚠️ Se reescribió el `StyleSheet.create` de cada pantalla/componente existente para moverlo dentro del componente vía `useThemedStyles`; es un cambio mecánico pero de alcance amplio (~30 archivos).
- ⚠️ Pregunta abierta para el PO: ¿el tema por defecto debería seguir el del sistema operativo en lugar de arrancar siempre en oscuro? Se asumió que no (ver Decisión); si el PO prefiere seguir el sistema, es cambiar el valor inicial de `mode` en `theme.ts` y `userInterfaceStyle` a `automatic` en `app.json`.
