---
title: Requerimientos funcionales
tags: [waggo, requerimientos, srs]
source: Requerimientos_App_Paseo_Perros.xlsx
created: 2026-09-26
---

# Requerimientos funcionales

Volver a [[00 - MOC Waggo]] · Fuente: `Requerimientos_App_Paseo_Perros.xlsx` (hoja *Requerimientos Funcionales*)

> [!info] Resumen
> 20 RF, todos en alcance **MVP** — 16 prioridad Alta, 4 Media. La columna *Módulo backend* indica dónde vive cada RF en [[Backend - Clean Architecture]].

> [!warning] Hallazgo
> La numeración salta de **RF-019 a RF-021** (no existe RF-020). Confirmar con el PO si falta un requerimiento o es un error de numeración.

## Gestión de Usuarios y Perfiles → módulo `Identity` / `Pets`
| ID | Requerimiento | Actor | Prio | Criterio de aceptación | Integración |
|---|---|---|---|---|---|
| RF-001 | Registro de dueños | Dueño | Alta | Registro con correo/teléfono, confirmación y cuenta activa. | — |
| RF-002 | Registro de paseadores | Paseador | Alta | Carga datos, documento y foto; cuenta queda *pendiente de verificación*. | API identidad |
| RF-003 | Verificación de identidad y antecedentes | Paseador / Plataforma | Alta | Solo paseadores con biometría y antecedentes aprobados reciben paseos. | API biométrica + antecedentes |
| RF-004 | Perfil de mascota | Dueño | Media | Una o más mascotas con historial médico cifrado. | — |
| RF-005 | Autenticación 2FA | Dueño / Paseador | Alta | Credenciales + segundo factor; intentos fallidos registrados. | — |

## Búsqueda y Matching → módulo `Walks`
| ID | Requerimiento | Actor | Prio | Criterio de aceptación | Integración |
|---|---|---|---|---|---|
| RF-006 | Matching por cercanía | Dueño | Alta | Paseadores verificados ordenados por cercanía. | Maps |
| RF-007 | Solicitud y agendamiento | Dueño | Alta | Selecciona tipo/horario; paseador acepta o rechaza. | Push |

## Rastreo y Seguridad en Tiempo Real → módulo `Tracking`
| ID | Requerimiento | Actor | Prio | Criterio de aceptación | Integración |
|---|---|---|---|---|---|
| RF-008 | GPS en tiempo real | Dueño / Paseador | Alta | Ruta y posición actualizada durante el paseo. | Maps |
| RF-009 | Geocercas | Plataforma | Alta | Salida del perímetro → alerta a dueño y plataforma. | Maps |
| RF-010 | Detección de anomalías | Plataforma | Alta | Detención > umbral → notificación automática. | — |
| RF-011 | Métricas del paseo | Dueño | Media | Resumen con distancia y duración al finalizar. | Maps |
| RF-012 | Alertas de emergencia | Dueño / Paseador | Alta | Botón de emergencia → push prioritaria. | Push |

## Comunicación → módulo `Messaging`
| ID | Requerimiento | Actor | Prio | Criterio de aceptación | Integración |
|---|---|---|---|---|---|
| RF-013 | Chat interno | Dueño / Paseador | Media | Mensajes dentro de la app durante el servicio. | — |
| RF-014 | Notificaciones de recorrido | Dueño / Paseador | Alta | Push al iniciar, finalizar y ante avisos. | Push |

## Pagos → módulo `Payments`
| ID | Requerimiento | Actor | Prio | Criterio de aceptación | Integración |
|---|---|---|---|---|---|
| RF-015 | Pasarela integrada | Dueño | Alta | Pago dentro de la app antes de iniciar; sin efectivo. | Stripe / MercadoPago |
| RF-016 | Escrow | Plataforma | Alta | Monto retenido desde la confirmación hasta finalizar. | Stripe / MercadoPago |
| RF-017 | Split payments | Plataforma / Paseador | Alta | Transferencia automática al paseador al completar. | Split payments |
| RF-018 | Retención de comisión | Plataforma | Alta | Comisión configurada se retiene antes de dispersar. | Stripe / MercadoPago |
| RF-019 | Cálculo de tarifa | Dueño | Alta | Precio visible antes de confirmar, según duración/tipo. | — |

> [!example] Implementado como ejemplo TDD
> **RF-019** y **RF-018** son el primer *slice* construido con TDD en ambos repos (cotización de tarifa + desglose de comisión). Ver [[Flujo TDD#Ejemplo real RF-019]].

## Operación, Reportes y Métricas → módulo `Analytics`
| ID | Requerimiento | Actor | Prio | Criterio de aceptación | Integración |
|---|---|---|---|---|---|
| RF-021 | Tablero de KPIs | Plataforma | Media | Retención, conversión, DAU/MAU, LTV/CAC actualizados. | — |

Relacionado: [[Requerimientos no funcionales]] · [[Roadmap MVP]]
