# DEPLOY — Fork de FreshRSS para infra personal (24h.cloud)

Fork de [`FreshRSS/FreshRSS`](https://github.com/FreshRSS/FreshRSS) desplegado en el VPS
Hetzner `ubuntu-gar` vía **Coolify**, en **https://rss.24h.cloud**.

> Este fichero existe **solo en el fork** (no en el upstream), para no generar conflictos.

## Modelo de ramas

| Rama | Rol |
|---|---|
| `edge` | Espejo del upstream **desarrollo** (default del upstream). No se despliega. |
| `latest` | Espejo del upstream **estable** (la actualiza el upstream en cada release). |
| `production` | **Lo que despliega Coolify** (auto-deploy on push). Basada en `latest` + cambios propios. |

Versión base actual: **1.29.1**.

## Despliegue (Coolify)

- Proyecto Coolify `freshrss` · app tipo **Dockerfile** (`Docker/Dockerfile`).
- Build pack: Dockerfile. Rama desplegada: `production`. Auto-deploy on push (GitHub App `coolify-24h`).
- Puerto interno **80** (Apache), enrutado por Traefik vía `Host(rss.24h.cloud)`. Sin puerto al host.
- Volumen persistente: `/var/www/FreshRSS/data` (SQLite + config de usuario + favicons).

### Variables de entorno
- `TZ=Europe/Madrid`
- `CRON_MIN=4,34` — refresco de feeds 2×/h (cron interno del contenedor).
- `TRUSTED_PROXY=<subred docker coolify>` — confía en los `X-Forwarded-*` de Traefik.
- `FRESHRSS_INSTALL` — auto-instalación: `--default-user admin --base-url https://rss.24h.cloud
  --environment production --db-type sqlite --auth-type form --api-enabled --disable-update
  --language es --title FreshRSS`
- `FRESHRSS_USER` — alta del admin: `--user admin --password <secreto> --api-password <secreto> --language es`

> `--disable-update` desactiva el auto-actualizador web: las actualizaciones se hacen por Git/Docker.

## Actualizar a una nueva versión de FreshRSS

```bash
git fetch upstream            # remote 'upstream' = FreshRSS/FreshRSS (añadir si no está)
git checkout latest && git merge --ff-only upstream/latest && git push origin latest
git checkout production && git merge latest && git push origin production   # dispara redeploy
```

Si hay cambios propios en `production`, resolver el merge conservándolos.
