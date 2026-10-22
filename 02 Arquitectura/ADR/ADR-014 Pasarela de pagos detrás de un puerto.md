---
title: ADR-014 Pasarela de pagos detrás de un puerto
tags: [waggo, adr, backend, pagos]
status: Aceptada
date: 2026-09-30
---

# ADR-014 Pasarela de pagos detrás de un puerto

Volver a [[00 - MOC Waggo]] · Guía: [[Backend - Clean Architecture]]

> [!info] Estado: **Aceptada** · 2026-09-30 · Tomada al implementar RF-015 – RF-018 sin proveedor elegido

## Contexto
Los pagos (RF-015 – RF-018) son el corazón del modelo tipo Uber: sin efectivo, dinero retenido hasta que el paseo ocurre, comisión y dispersión automáticas. El proveedor (Stripe o MercadoPago) sigue siendo una pregunta abierta del PO, y cada uno llama distinto a las mismas operaciones (autorizar/capturar/cancelar, *payment intents*, *split payments*, *marketplace*).

## Decisión
- La aplicación habla con un **puerto** `IPaymentGateway` con tres operaciones: `HoldAsync` (retener), `CaptureAsync` (cobrar y repartir: comisión para Waggo, `walkerPayout` para el paseador) y `ReleaseAsync` (liberar).
- El estado del dinero vive en Waggo: `WalkPayment` (`Held → Captured | Released`) en `payments.walk_payments`, con la referencia de la pasarela y **nunca datos de tarjeta** (RNF-004).
- Momentos: se retiene al **pedir** el paseo (si se rechaza, no se crea), se libera al **cancelar** y se cobra al **terminar**.
- Mientras no hay proveedor existe solo `SimulatedPaymentGateway`, elegido con `Payments:Provider = Simulated`. Solo Development y Testing lo configuran; en otro ambiente la validación de opciones impide arrancar la API. La simulación rechaza a los usuarios cuyo id contiene `card-declined` (como las tarjetas de prueba de Stripe).
- Si la pasarela no responde, el adaptador lanza `ExternalServiceException` (503) y la operación no se guarda: no quedan paseos terminados sin cobrar.

## Consecuencias
- ✅ El flujo completo (retener, liberar, cobrar, ganancias del paseador) se prueba de punta a punta sin cuentas ni llaves de un proveedor.
- ✅ Cambiar de proveedor es escribir un adaptador nuevo; dominio, casos de uso y app no cambian.
- ⚠️ Registro de tarjetas y cuentas de los paseadores quedan para cuando se elija el proveedor (formulario tokenizado y *onboarding* del proveedor).
- ⚠️ Pasarela y base de datos no comparten transacción: si la pasarela cobra y luego falla el guardado, hay que reconciliar por referencia. Con el proveedor real se usarán llaves de idempotencia.
