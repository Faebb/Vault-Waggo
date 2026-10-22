---
title: Glosario
tags: [waggo, proyecto, ddd]
created: 2026-09-26
---

# Glosario (lenguaje ubicuo)

Volver a [[00 - MOC Waggo]]

> [!note] Regla
> El **código se escribe en inglés** y la documentación en español. Esta tabla es el puente entre ambos y evita que un mismo concepto tenga dos nombres.

| Español | Código (inglés) | Definición |
|---|---|---|
| Dueño | `Owner` | Usuario que registra mascotas y solicita paseos. |
| Paseador | `Walker` | Usuario verificado que realiza paseos. |
| Perfil de paseador | `WalkerProfile` | Datos del paseador (nombre, documento cifrado, celular, experiencia) y su verificación. |
| Estado de verificación | `VerificationStatus` | `Pending` (en revisión), `Approved` (recibe paseos), `Rejected` (con motivo). |
| Tipo de documento | `DocumentType` | `CC` cédula, `CE` cédula de extranjería, `PP` pasaporte. |
| Mascota | `Pet` | Perro registrado por un dueño. |
| Paseo | `Walk` | Servicio contratado, con ciclo de vida (solicitado → aceptado → en curso → finalizado / cancelado). |
| Estado del paseo | `WalkStatus` | `Requested` (buscando paseador), `Accepted`, `InProgress`, `Completed`, `Cancelled`. |
| Punto de recogida | `PickupLocation` / `GeoPoint` | Dirección y coordenadas (WGS 84) donde el paseador recoge a los perros. |
| Tipo de paseo | `WalkType` | `Individual`, `Group` (provisionales, pendientes de validar con el PO) — afecta la tarifa. |
| Tarifa | `Fare` | Precio calculado por tipo y duración (RF-019). |
| Comisión | `Commission` | Porcentaje de la plataforma (RF-018). |
| Pago en garantía | `Escrow` | Dinero retenido hasta finalizar el paseo (RF-016). |
| Dispersión | `Payout` | Transferencia al paseador (RF-017). |
| Aviso / notificación | `Notification` | Aviso a un usuario sobre un momento del paseo; se ve en la bandeja "Avisos" (RF-014). |
| Prioridad del aviso | `NotificationPriority` | `High` para emergencias y alertas automáticas, `Normal` para el resto. |
| Pago del paseo | `WalkPayment` | El dinero de un paseo: retenido al pedir, cobrado al terminar o liberado al cancelar. |
| Estado del pago | `PaymentStatus` | `Held` (retenido), `Captured` (cobrado y repartido), `Released` (liberado, sin cobro). |
| Pasarela de pagos | `IPaymentGateway` | Proveedor de pagos detrás de un puerto; hoy `SimulatedPaymentGateway` (ADR-014). |
| Geocerca | `Geofence` | Perímetro autorizado del paseo (RF-009). |
| Anomalía | `Anomaly` | Comportamiento fuera de lo esperado (RF-010). |
| Punto de rastreo | `TrackPoint` | Posición GPS con marca de tiempo (RF-008). |
