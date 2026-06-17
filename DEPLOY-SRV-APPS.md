# Déploiement GlitchTip — srv-apps

## Pré-requis

- Serveur : `root@149.71.44.117` (port 2200)
- Réseau Docker `apps` déjà présent (Cloudflare Tunnel)
- Cloudflare Tunnel déjà configuré sur srv-apps
- PostgreSQL dédié inclus dans le compose (container `glitchtip-postgres-1`)

## 1. Déposer les fichiers sur le serveur

```bash
mkdir -p /opt/apps/glitchtip
scp -P 2200 compose.srv-apps.yml root@149.71.44.117:/opt/apps/glitchtip/compose.yml
```

## 2. Créer le fichier .env

```bash
# Sur le serveur :
cat > /opt/apps/glitchtip/.env <<EOF
SECRET_KEY=$(openssl rand -hex 32)
DB_PASSWORD=$(openssl rand -hex 16)
RESEND_API_KEY=re_xxxxxxxxxx
S3_SECRET_KEY=la_cle_secrete_jotelulu
EOF
```

## 3. Démarrer

```bash
cd /opt/apps/glitchtip
docker compose -f compose.yml --env-file .env up -d
docker compose -f compose.yml --env-file .env exec web python manage.py createsuperuser
```

## 4. Route Cloudflare Tunnel

Dashboard CF Zero Trust → Tunnels → srv-kode01-002 → Public Hostnames → Add :
- Subdomain : `monitor`
- Domain : `kodesaas.com`
- Service : `http://glitchtip-web-1:8000`

DNS Cloudflare : CNAME `monitor.kodesaas.com` → `<tunnel-id>.cfargotunnel.com` (proxied)

## 5. Smoke test

```bash
curl -I https://monitor.kodesaas.com
```

## Mise à jour

```bash
cd /opt/apps/glitchtip
docker compose -f compose.yml --env-file .env pull
docker compose -f compose.yml --env-file .env up -d
```

## Backup PostgreSQL

À ajouter dans `/opt/scripts/backup-databases.sh` :
```bash
docker exec glitchtip-postgres-1 pg_dump -U glitchtip glitchtip | gzip > /backups/glitchtip-$(date +%Y%m%d).sql.gz
```
