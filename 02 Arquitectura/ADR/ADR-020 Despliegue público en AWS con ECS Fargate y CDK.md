---
title: ADR-020 Despliegue público en AWS con ECS Fargate y CDK
tags: [waggo, adr, devops, aws, infraestructura]
status: Aceptada
date: 2026-10-27
---

# ADR-020 Despliegue público en AWS con ECS Fargate y CDK

Volver a [[00 - MOC Waggo]] · Guía de operación: [[Despliegue en AWS - Guía de operación]] · Contexto: [[Contenedores y despliegue]], [[CI-CD]] · Spec: `waggo-workspace/specs/INF-001-despliegue-publico-en-aws.md`

> [!info] Estado: **Aceptada** · 2026-10-27 · Decisión del PO. Resuelve el "destino del piloto: por decidir" de [[Contenedores y despliegue]].

## Contexto
Waggo corría solo en local (Docker Compose). Para el piloto hace falta que la app esté accesible públicamente en `mywaggo.com`, que cada cambio que llega a `main` se publique solo y que ningún secreto ni clave de larga vida viva en GitHub. La cuenta de AWS ya existía por Amazon SES (correos de confirmación, ver [[Guía AWS SES - prompt de configuración]]).

El destino se había dejado abierto entre Azure Container Apps, Railway, Fly.io o un VPS con Compose.

## Decisión
1. **Un solo ambiente**: producción del piloto, región `us-east-1`. Staging se evalúa cuando haya usuarios reales.
2. **API en ECS Fargate** detrás de un Application Load Balancer con HTTPS (el 80 redirige al 443), usando la imagen Docker actual sin cambios (verificación en `/health/ready`).
3. **Base de datos en RDS PostgreSQL 17** en subred privada, cifrada y con copias automáticas (PostGIS soportado). Solo el grupo de seguridad de la API puede llegar al puerto 5432.
4. **Web (Expo, `expo export --platform web`) en S3 privado + CloudFront** en `app.mywaggo.com`; el sitio público (landing, privacidad, términos) en otro bucket + CloudFront en `mywaggo.com` y `www`.
5. **Infraestructura como código con AWS CDK (TypeScript)** en un repositorio propio, **privado**: `waggo-infra` ([[ADR-001 Repositorios separados]]). El repo es privado porque su configuración incluye datos de la cuenta.
6. **CD por GitHub Actions con OIDC**, sin access keys: un rol IAM por repo (`waggo-api`, `waggo-mobile`) que solo puede asumir ese repo en `main`, con el sujeto OIDC por *ids inmutables* (dueño y repositorio), de modo que un repo renombrado o recreado no puede suplantarlo.
7. **Despliegue automático al merge** (el merge ya exige PR + CI verde). Las migraciones de base de datos son un paso explícito del despliegue de la API.
8. **Imágenes en Amazon ECR** (no en GHCR, como se había pensado en CD-1): la API las usa desde ahí y el rol de despliegue solo necesita permisos de AWS.
9. **DNS en Cloudflare** (registros *solo DNS*, sin proxy) en vez de Route 53; los certificados ACM se validan con CNAME manuales.
10. **Correo**: Amazon SES con el rol de la tarea ECS (sin claves); el buzón de contacto `contacto@mywaggo.com` se reenvía con Cloudflare Email Routing.
11. **Costo controlado**: presupuesto mensual de US$60 con alertas por correo (estimado actual: US$45–50 al mes).

## Alternativas descartadas
| Opción | Por qué no |
|---|---|
| App Runner | Menos control de red hacia RDS privado y hay que verificar su disponibilidad para cuentas nuevas. |
| VPS o EC2 con Compose | Barato, pero sin despliegues gestionados, sin rollback automático y con mantenimiento del servidor. |
| Azure Container Apps, Railway, Fly.io | La cuenta, SES y el dominio ya estaban en AWS; evita repartir el piloto en varias nubes. |
| Route 53 para el DNS | El dominio y su DNS ya estaban en Cloudflare; un cambio de DNS no aportaba al piloto. |
| Claves de acceso de IAM para el CI | Larga duración y riesgo de fuga; OIDC entrega credenciales temporales. |

## Consecuencias
- ✅ `app` y `api` públicos con HTTPS, despliegue automático y rollback del servicio ECS (circuit breaker) si la nueva versión no pasa el health check.
- ✅ La infraestructura se puede recrear desde código; cada cambio pasa por `cdk diff` antes de aplicarse.
- ✅ Ningún secreto en el repo: la clave de cifrado, la clave de firma JWT y la contraseña de la base viven en Secrets Manager.
- ⚠️ Costo fijo mensual (balanceador, RDS, Fargate) aunque haya poco tráfico; el presupuesto avisa antes de pasar el límite.
- ⚠️ Sin NAT por costo: las tareas Fargate van en subredes públicas con grupos de seguridad cerrados; la base queda aislada.
- ⚠️ Mientras SES siga en *sandbox*, solo llegan correos a direcciones verificadas ([[Despliegue en AWS - Guía de operación#Correo (SES)|ver guía]]).
- ⚠️ Los pagos siguen siendo una pasarela **simulada** en producción ([[ADR-014 Pasarela de pagos detrás de un puerto]]).
- ⏳ Pendiente: publicar eventos de rebote y queja de SES en SNS; EAS Build para las apps móviles (CD-3); revisar la política de privacidad y términos con criterio legal antes de abrir al público.
