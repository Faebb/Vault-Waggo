---
title: Despliegue en AWS - Guía de operación
tags: [waggo, aws, devops, operación]
created: 2026-10-28
updated: 2026-10-28
---

# Despliegue en AWS — Guía de operación

Volver a [[00 - MOC Waggo]] · Decisión: [[ADR-020 Despliegue público en AWS con ECS Fargate y CDK|ADR-020]] · Spec: `waggo-workspace/specs/INF-001-despliegue-publico-en-aws.md` · Antecedentes: [[Contenedores y despliegue]], [[CI-CD]], [[Guía AWS SES - prompt de configuración]]

> [!warning] Esta nota es pública
> El vault y los repos `waggo-api`, `waggo-mobile` y `waggo-workspace` son públicos. **Nunca** pongas aquí ni en esos repos: el id de la cuenta de AWS, ARNs, nombres de buckets o de recursos, ids de distribuciones, endpoints de la base, claves, ni datos de operación. Esos datos viven en `waggo-infra` (repo **privado**, `lib/config.ts`) y en la consola de AWS.

## Qué existe hoy en AWS
Región `us-east-1`. Un solo ambiente (producción del piloto). Todo se creó con CDK desde `waggo-infra`.

| Stack CDK | Qué contiene | Notas |
|---|---|---|
| `WaggoNetwork` | VPC sin NAT (subredes públicas y aisladas), grupo de seguridad de la API | Sin NAT por costo |
| `WaggoData` | RDS PostgreSQL 17 privada, cifrada, copias 7 días, protección contra borrado | Contraseña en Secrets Manager; solo la API llega al puerto 5432 |
| `WaggoRegistry` | Repositorio ECR `waggo-api` (conserva las últimas 10 imágenes) | Las imágenes las sube **solo el CI** |
| `WaggoCertificate` | Certificado ACM para `api.` y `app.mywaggo.com` | Validación DNS manual en Cloudflare |
| `WaggoPipeline` | Proveedor OIDC de GitHub y rol de despliegue de `waggo-api` | Sin access keys |
| `WaggoApi` | ECS Fargate + ALB HTTPS (el 80 redirige al 443), SES por el rol de la tarea | Solo existe con `-c imageTag=<sha>` |
| `WaggoWeb` | S3 privado + CloudFront en `app.mywaggo.com` y el rol de despliegue de `waggo-mobile` | SPA: 403/404 devuelven `index.html` |
| `WaggoSiteCertificate` | Certificado ACM para `mywaggo.com` y `www` | Validación DNS manual en Cloudflare |
| `WaggoSite` | S3 privado + CloudFront en `mywaggo.com` y `www` con la landing, privacidad y términos | Los archivos salen de `waggo-infra/site/` |
| `WaggoBudget` | Presupuesto mensual de US$60 con alertas (80 % gastado, 100 % pronosticado, 100 % gastado) | El correo se pasa con `-c budgetEmail=…` |
| `CDKToolkit` | Bootstrap de CDK | — |

Otras piezas fuera de CDK: DNS en **Cloudflare** (registros *solo DNS*, sin proxy), Cloudflare Email Routing (`contacto@mywaggo.com` reenvía al correo del proyecto), identidad de dominio y SES en la consola de AWS, secretos creados a mano en Secrets Manager.

| URL | Qué es |
|---|---|
| `https://api.mywaggo.com` | API (`/health/ready` para comprobar) |
| `https://app.mywaggo.com` | App web (Expo) |
| `https://mywaggo.com` y `www` | Sitio público (landing, `/privacidad.html`, `/terminos.html`) |

**Costo estimado:** unos US$45–50 al mes (balanceador, RDS y Fargate son lo principal). Presupuesto de alerta: US$60.

## Cómo se publica cada cosa

### Código de la API o de la base de datos
Los cambios de BD (migraciones EF) viajan en el mismo PR de `waggo-api`. Al hacer *squash and merge* a `main`, `deploy.yml`: construye la imagen y la sube a ECR (etiqueta = SHA), registra una nueva definición de tarea, **aplica las migraciones** con una tarea de un solo uso (`--migrate-only`), actualiza el servicio ECS y prueba `https://api.mywaggo.com/health/ready`. Si la migración falla, el servicio **no** se actualiza. Si la versión nueva no pasa el health check, ECS conserva la anterior.

### La web
Al hacer merge a `main` en `waggo-mobile`, `deploy.yml` (job `deploy-web`) hace `expo export --platform web` con `EXPO_PUBLIC_API_URL=https://api.mywaggo.com`, sube a S3 (primero los assets con hash, con caché larga; luego `index.html` y el resto sin caché), invalida CloudFront y prueba `https://app.mywaggo.com`. Variables del repo (no son secretos): `AWS_ROLE_ARN`, `WEB_BUCKET`, `WEB_DISTRIBUTION_ID`.

### Cambios que tocan BD, API y web a la vez
Un PR por repo y **en este orden**: (1) API + migración; (2) esperar que el despliegue termine en verde; (3) web. Entre los dos despliegues conviven versiones: agregar campos o rutas es seguro; **renombrar o borrar solo en dos releases**. Las columnas no se borran en el mismo release que deja de usarlas (patrón expandir/contraer).

### El sitio público
Edita `waggo-infra/site/` y despliega: `cdk deploy WaggoSite --exclusively`. Sube los archivos e invalida CloudFront.

### Infraestructura o variables de entorno
En `waggo-infra`, siempre en este orden: `cdk diff` → revisar → `cdk deploy <Stack> --exclusively`. Todo cambio de CDK entra por PR (el CI corre pruebas y `cdk synth` sin credenciales de AWS).

## Rollback
- **API:** en GitHub, Actions → workflow *Deploy* de `waggo-api` → *Run workflow* con `image_tag` = SHA anterior. Las migraciones **no** se revierten: se corrigen hacia adelante.
- **Web:** volver a ejecutar el despliegue del commit anterior, o un PR de revert.
- **Base de datos:** copias automáticas de RDS como último recurso.

## Trabajar desde un computador nuevo
1. Instalar AWS CLI v2, Node 22, Git y Docker. Si se usan los MCP de AWS: `uv` y un Python del sistema (el Python que descarga `uv` puede fallar en Windows; usar `--no-managed-python --python <ruta>`).
2. Entrar con `aws login --profile waggo-admin` (usuario administrador con MFA). Las credenciales son temporales; si caducan, repetir el login. Los perfiles de claves de larga duración quedan solo para SES.
3. Clonar `waggo-infra` (privado) junto a los demás repos y correr `npm ci`.
4. CDK siempre con `AWS_PROFILE=waggo-admin`.

## Trampas conocidas
- **`docker login` a ECR puede fallar en Windows** ("stub received bad data"). No hace falta: las imágenes las sube solo el CI.
- **CDK y el CI comparten la imagen de la API.** Para desplegar `WaggoApi` con CDK: `cdk deploy WaggoApi --exclusively -c imageTag=<sha que corre hoy> -c desiredCount=1`. El SHA actual se lee de la definición de tarea en ECS. Sin `--exclusively`, CDK arrastra stacks dependientes (una vez se rompió un export).
- **Sujeto OIDC con ids inmutables:** GitHub emite `repo:<dueño>@<id>/<repo>@<id>:ref:refs/heads/main`. Los ids de dueño y repos están en `waggo-infra/lib/config.ts`; para un repo nuevo se obtiene el id con la API de GitHub (`GET /repos/<dueño>/<repo>`).
- **Cloudflare:** todos los registros que apuntan a AWS van **solo DNS** (nube gris). Con proxy, ACM y CloudFront dejan de validar o entregan mal. En el alta de un registro, apagar el interruptor de proxy antes de guardar.
- **Secretos:** se crean a mano en Secrets Manager (clave de cifrado, clave de firma JWT). **La clave de cifrado debe respaldarse fuera de AWS**: sin ella no se pueden leer las notas médicas ni los documentos cifrados.
- **Usuarios de prueba (DEV-001):** el seed solo corre en `Development` y `Testing`; en producción `ASPNETCORE_ENVIRONMENT=Production` y `Authentication__UseDevelopmentUser=false`. No se crean usuarios de prueba.
- **Pagos:** producción usa `Payments__Provider=Simulated`; no se cobra dinero real.
- **Presupuesto:** el correo de alertas se pasa al desplegar (`-c budgetEmail=…`); no se guarda en el repo.

## Correo (SES)
- Dominio `mywaggo.com` verificado en SES (Easy DKIM, MAIL FROM `mail.mywaggo.com` con SPF, DMARC `p=none` por ahora). Remitente: `noreply@mywaggo.com`.
- La API manda con el rol de la tarea ECS; no usa claves.
- **Acceso a producción:** la solicitud estuvo en estado *DENIED* porque `mywaggo.com` no abría ningún sitio y el caso pedía más datos. Se publicó el sitio y se respondió el caso con los datos pedidos (sitio, tipo de correo, volumen, origen de los destinatarios, rebotes y quejas, y un ejemplo). Una solicitud denegada **no se puede reenviar por API** (`ConflictException`); se responde en el caso de AWS Support.
- Mientras SES siga en *sandbox*, solo se entrega a direcciones verificadas; el registro de otras personas no recibe el código. Revisar con `GetAccount` (`ProductionAccessEnabled`).
- La lista de supresión de cuenta está activa para rebotes y quejas. **Pendiente:** publicar esos eventos en SNS con un *Configuration Set*.
- `contacto@mywaggo.com`: Cloudflare Email Routing reenvía al correo del proyecto. Responder desde Gmail sale con la cuenta personal, no con `contacto@`.

## Pendientes
- [ ] Respuesta de AWS al caso de acceso a producción de SES.
- [ ] Rebotes y quejas de SES a SNS (Configuration Set).
- [ ] EAS Build para las apps iOS y Android (CD-3).
- [ ] Revisión legal de la política de privacidad y de los términos (hoy son un borrador de piloto).
- [ ] Endurecer la conexión a RDS (validar el certificado en vez de confiar en él).
- [ ] Evaluar staging cuando haya usuarios reales.
