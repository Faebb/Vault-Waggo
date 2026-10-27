---
title: Guía AWS SES - prompt de configuración
tags: [waggo, desarrollo, aws, correo]
created: 2026-10-26
updated: 2026-10-26
---

# Guía AWS SES — prompt de configuración

Volver a [[00 - MOC Waggo]] · Spec: `waggo-workspace/specs/RF-001-RF-005-envio-de-correos-reales.md` · Decisión: [[ADR-016 Identidad propia con bcrypt y JWT emitido por la API|ADR-016]] (identidad propia) + [[ADR-017 Login sin 2FA obligatorio y federación OAuth con Google y Microsoft|ADR-017]]

> [!abstract] Para qué sirve
> Waggo manda sus correos (código de confirmación de cuenta y, luego, recuperación de contraseña) con **Amazon SES** detrás del puerto `IEmailSender`. Esta nota guarda el prompt para que un asistente de IA te guíe, paso a paso, a crear en AWS lo necesario. Se pega entero en un chat nuevo y se contesta una fase a la vez.

> [!warning] Reglas
> - **Nunca** pegues en un chat la *secret access key* de AWS ni ninguna contraseña. El prompt le prohíbe pedirlas.
> - Las llaves no van en el repo, ni en `appsettings*.json`, ni en el vault.
> - Los límites, precios y nombres de pantalla de AWS cambian: el prompt le pide al asistente verificarlos con la documentación vigente.

## Decisiones tomadas (2026-10-26)
| Tema | Decisión |
|---|---|
| Región de SES | `us-east-1` (Virginia). Las identidades y la salida del sandbox son por región: no cambiarla sin repetir todo |
| Dominio | Aún no existe: se registra en Route 53 |
| Remitente | `no-reply@<dominio>` con nombre visible "Waggo" |
| Contenido del correo | Código numérico de 6 dígitos |
| Perfil de AWS CLI | `waggo-ses` |

## Fases
| Fase | Qué logra | Necesita dominio |
|---|---|---|
| 0 | Cuenta de AWS segura y presupuesto | No |
| 1 | SES en *sandbox*: enviar a direcciones verificadas (tus pruebas) | No |
| 2 | Usuario IAM con permiso mínimo y credenciales locales | No |
| 3 | Registrar el dominio en Route 53 | — |
| 4 | Identidad de dominio en SES, DKIM/SPF/DMARC y salida del sandbox (correos a cualquier persona) | Sí |
| 5 | Rebotes y quejas, y rol IAM para el despliegue | Sí |

Las fases 1–2 y la 3 se pueden hacer a la vez: mientras el dominio se registra, ya se prueba el envío con un correo personal.

## El prompt
```text
Actúa como mi guía práctica para configurar Amazon SES (Simple Email Service) y un dominio en AWS. Soy desarrollador, no soy experto en AWS. Respóndeme en español, con pasos cortos y numerados, indicándome la ruta exacta en la consola de AWS (menú > pantalla > botón) y qué debo ver cuando el paso salió bien.

## Contexto del proyecto
- Waggo es una app para pasear perros. Su API es .NET 10 (ASP.NET Core) y está en desarrollo; mi máquina es Windows 11.
- La API ya registra usuarios con correo y contraseña. Al registrarse manda un correo con un código numérico de 6 dígitos para confirmar la cuenta; hoy solo lo escribe en el log. Quiero que lo mande de verdad con SES.
- La API hablará con SES usando el SDK de AWS para .NET (paquete AWSSDK.SimpleEmailV2, operación SendEmail). Las credenciales las tomará de la cadena estándar del SDK (perfil de AWS CLI, variables de entorno o rol IAM), nunca de archivos del repositorio.
- Primero necesito que funcione el envío en desarrollo; después, correos reales a usuarios reales.

## Decisiones ya tomadas (no las cuestiones)
- Región: us-east-1 (N. Virginia). Usa SIEMPRE esa región, explícame que las identidades y la salida del sandbox son por región y recuérdame comprobar la región en la esquina superior derecha de la consola antes de cada paso.
- Todavía NO tengo dominio: quiero registrarlo en Route 53 (Fase 3).
- Remitente final: no-reply@<mi-dominio>, con nombre visible "Waggo".
- Nombre del perfil de AWS CLI: waggo-ses.

## Reglas para ti
1. Trabaja UNA fase a la vez. Al terminar cada fase, pídeme que confirme que todo salió bien antes de pasar a la siguiente.
2. NUNCA me pidas ni me aceptes la Secret Access Key, contraseñas ni códigos de MFA. Si los pego por error, dime que los rote o los elimine de inmediato.
3. Verifica con la documentación oficial vigente de AWS los nombres de las pantallas, los límites del sandbox, los precios y los tiempos de aprobación. Si no estás seguro de algo, dilo en lugar de inventarlo.
4. Avísame de cualquier paso que pueda generar costos y qué pasa si lo olvido encendido.
5. Si algo falla, pídeme el mensaje de error exacto (sin secretos) y ayúdame a diagnosticarlo antes de seguir.

## Fase 0 — Cuenta y presupuesto
- Si no tengo cuenta de AWS, guíame a crearla. Activa MFA en el usuario raíz y no me dejes usar el usuario raíz para el trabajo diario.
- Ayúdame a crear un presupuesto (AWS Budgets) con alerta de costos bajos.

## Fase 1 — SES en sandbox, sin dominio (en us-east-1)
- Explícame qué es el sandbox de SES y sus límites actuales (a quién puedo enviar, cuántos correos por día y por segundo).
- Guíame a crear una identidad de tipo "Email address" para el remitente de pruebas (una dirección mía real) y a abrir el enlace de verificación que llega a mi bandeja.
- Guíame a verificar también 1 o 2 direcciones receptoras mías (en sandbox solo se puede enviar a direcciones verificadas).
- Guíame a mandar un correo de prueba desde la consola de SES y a comprobar que llegó (y a revisar la carpeta de spam).

## Fase 2 — Credenciales para mi API local
- Guíame a crear una política IAM de permiso mínimo para enviar correos: acción ses:SendEmail, limitada a mi identidad remitente, usando la condición ses:FromAddress. Muéstrame el JSON de la política y explícame cada línea. Si esa política no funciona en sandbox (por ejemplo porque también exige permisos sobre la identidad receptora), dime cuál es el ajuste mínimo.
- Guíame a crear un usuario IAM solo para esto (nombre sugerido: waggo-api-ses-dev) con solo esa política, sin acceso a la consola, y a crear su clave de acceso.
- Guíame a guardar la clave en mi máquina SIN pegarla en el chat: instala AWS CLI en Windows y usa `aws configure --profile waggo-ses` (región us-east-1), y luego `AWS_PROFILE=waggo-ses` como variable de entorno. Explícame dónde queda guardada y que nunca debe ir al repositorio.
- Guíame a probar el envío desde la terminal con `aws sesv2 send-email --profile waggo-ses ...` y a leer la respuesta (MessageId = éxito).

## Fase 3 — Registrar el dominio en Route 53
- Guíame a buscar disponibilidad de un dominio en Route 53 (ideas: waggo.app, waggo.co, getwaggo.com u otros parecidos), a comparar precios de las terminaciones y a registrarlo. Dime el costo anual y el de la zona alojada (hosted zone) mensual.
- Explícame el correo de verificación del contacto del registrante (ICANN), cuánto tarda el registro y cómo comprobar que quedó activo y con su hosted zone creada.
- Dime si conviene activar la renovación automática y la protección contra transferencias.

## Fase 4 — Identidad de dominio en SES y salida del sandbox
- Guíame a crear en SES una identidad de tipo "Domain" con Easy DKIM para mi dominio y a publicar sus registros en Route 53 (si hay un botón para publicarlos automáticamente en la hosted zone, úsalo; si no, explícame los 3 CNAME). Muéstrame cómo comprobar que quedó "Verified" y DKIM en "Successful".
- Guíame a configurar un dominio MAIL FROM personalizado (registros MX y TXT/SPF) para que SPF alinee.
- Guíame a publicar un registro DMARC (empezando con p=none y una dirección para los reportes) y explícame cuándo endurecerlo.
- Ayúdame a ajustar la política IAM de la Fase 2 para que ahora permita enviar desde no-reply@<mi-dominio>.
- Guíame a solicitar el acceso a producción (salir del sandbox) en us-east-1: qué pantalla, y redáctame el texto de la solicitud (correo transaccional: código de confirmación de cuenta y recuperación de contraseña; sin marketing; solo a personas que se registran en la app; cómo manejo rebotes y quejas). Dime cuánto suele tardar.

## Fase 5 — Rebotes, quejas y despliegue
- Guíame a crear un Configuration Set con un destino de eventos (SNS) para rebotes y quejas, y a activar la lista de supresión de la cuenta.
- Explícame cómo debe obtener credenciales la API cuando se despliegue en AWS: con un ROL de IAM asignado al servicio donde corra (no con claves de acceso), y qué política adjuntarle.

## Al terminar cada fase, resúmeme en esta tabla (para llevársela a mi asistente de código). Sin claves ni secretos.
| Dato | Valor |
|---|---|
| Región de SES | us-east-1 |
| Dirección remitente verificada (sandbox) | |
| Dominio registrado | |
| Remitente final | no-reply@<dominio> |
| Nombre visible | Waggo |
| Nombre del perfil de AWS CLI | waggo-ses |
| ¿Dominio verificado en SES (DKIM verificado)? | sí / no |
| ¿Salí del sandbox? | sí / no |

Empieza ahora con la Fase 0.
```

## Qué sigue en el código
Con la tabla completa, la API usa `Email:Provider = Ses`, `Email:FromAddress`, `Email:FromName` y `Email:Ses:Region`, y las credenciales salen de `AWS_PROFILE` (o del rol en el servidor). Ver la spec.
