---
title: Backlog - Perfil de usuario y panel de administración
tags: [waggo, planeacion, backlog]
created: 2026-10-25
updated: 2026-10-25
---

# Backlog: Perfil de usuario y panel de administración

Volver a [[00 - MOC Waggo]] · Plan: [[Roadmap MVP]] · Surge de: [[ADR-017 Login sin 2FA obligatorio y federación OAuth con Google y Microsoft|ADR-017]]

> [!info] Para qué sirve esta nota
> Dos ideas de producto identificadas al cerrar el login sin 2FA (2026-10-25), que se pueden desarrollar **en paralelo** al login social (Google/Microsoft) sin pisarse con ese trabajo. Todavía no son specs (`Borrador` lo son en `waggo-workspace/specs/`); esta nota es el porqué y el qué se quiere, antes de pasar por `/new-spec`.

## 1. Perfil de usuario

### Por qué ahora
Antes de ADR-016/RF-001+RF-005 no existía una cuenta real que "configurar" — no tenía sentido una pantalla de perfil. Ahora que el login es real (correo+contraseña, y pronto Google/Microsoft), el usuario necesita un lugar para ver y controlar su propia cuenta.

### Qué se quiere
- [x] Ver y editar nombre completo y celular — implementado (`GET/PUT /api/v1/users/me`, pantalla "Información personal" dentro de Configuración); ver [[Estado de implementación]].
- [x] Cambiar el correo de la cuenta — implementado (código al correo nuevo y aviso al anterior); ver [[Estado de implementación]].
- [x] Cambiar contraseña — implementado, junto con "olvidé mi contraseña".
- Ver qué proveedores tiene vinculados a la cuenta (correo siempre; Google/Microsoft una vez exista esa spec) y poder desvincular uno si hay más de un método de entrada.
- Ver si es un paseador verificado (hoy no hay ningún indicador de esto en la app; `WalkerGate` lo sabe pero no se muestra en ningún otro lado).
- [x] Cerrar sesión — implementado (revoca el refresh token en la API).
- Eliminar la cuenta (RNF de privacidad / buena práctica — pendiente definir qué pasa con paseos históricos, pagos y mascotas al eliminarla).

### Preguntas abiertas
- [ ] ¿Eliminar cuenta es borrado real o desactivación (soft delete)? Afecta paseos/pagos históricos.
- [x] ¿Se puede cambiar el correo libremente o requiere reconfirmación? — Requiere un código enviado al correo nuevo (decidido 2026-10-27).
- [ ] ¿Desvincular un proveedor social requiere que quede al menos un método de entrada activo?

## 2. Panel de administración

### Por qué ahora
RF-002/RF-003 (verificación de paseadores) y la revisión de emergencias (RF-012) ya existen en la API, pero la única forma de operarlos hoy es `curl` con los headers `X-Dev-User-Id`/`X-Dev-Roles` a mano (ver `waggo-api/README.md`). Eso no es sostenible ni siquiera para el piloto (RNF-006: 500 dueños / 50 paseadores) — alguien del equipo necesita aprobar paseadores y atender emergencias sin abrir una terminal.

### Qué se quiere
- Aprobar / rechazar paseadores pendientes (ya existe `GET /admin/walkers?status=Pending`, `POST /admin/walkers/{id}/approve`, `POST /admin/walkers/{id}/reject` — falta la UI).
- Ver alertas de emergencia y anomalías activas (`GET /walks/{id}/alerts`) para poder reaccionar, no solo que quede guardado.
- Ver paseos en curso (operación en vivo, no solo lo que ve cada dueño/paseador por separado).
- Protegido por `WaggoPolicies.Admin` igual que hoy; no es una app nueva, es una sección dentro de la misma app (o un cliente web mínimo) solo visible para `IsAdmin = true`.

### Preguntas abiertas
- [ ] ¿Vive dentro de `waggo-mobile` (rutas `/admin/*` visibles solo para admins) o es un cliente web aparte? Mobile reutiliza todo lo que ya existe (auth, `httpClient`, tema); un cliente web separado sería más trabajo pero no compite por espacio de pantalla con las pantallas de dueño/paseador.
- [ ] ¿Quién más además del admin sembrado necesita este acceso en el piloto?

## Siguiente paso
Cuando el PO priorice una de las dos, `/new-spec` la convierte en una spec `Borrador` en `waggo-workspace/specs/` con tabla de ejemplos y contrato, siguiendo el flujo normal (spec → TDD → docs). Ya hay dos specs `Borrador` de arranque: `RF-001-RF-005-perfil-de-usuario.md` y `RF-002-RF-003-panel-de-administracion.md`.
