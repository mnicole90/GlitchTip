# Déploiement GlitchTip — srv-apps

## Pré-requis

- Serveur : `root@149.71.44.117` (port 2200)
- Réseaux Docker `shared-db` et `apps` déjà présents
- PostgreSQL `apps-postgres-1` accessible sur le réseau `shared-db`
- Cloudflare Tunnel déjà configuré sur srv-apps

## 1. Créer la base de données

```bash
ssh -p 2200 root@149.71.44.117
docker exec -it apps-postgres-1 psql -U postgres <<SQL
CREATE USER glitchtip WITH PASSWORD 'mot_de_passe_ici';
CREATE DATABASE glitchtip OWNER glitchtip;
SQL
```

## 2. Déposer les fichiers sur le serveur

```bash
mkdir -p /opt/apps/glitchtip
scp -P 2200 compose.srv-apps.yml root@149.71.44.117:/opt/apps/glitchtip/compose.yml
```

## 3. Créer le fichier .env

```bash
# Sur le serveur :
cat > /opt/apps/glitchtip/.env <<EOF
SECRET_KEY=$(openssl rand -hex 32)
DB_PASSWORD=mot_de_passe_ici
RESEND_API_KEY=re_xxxxxxxxxx
S3_SECRET_KEY=la_cle_secrete_jotelulu
GLITCHTIP_DOMAIN=https://glitchtip.kodesaas.com
EOF
```

## 4. Démarrer

```bash
cd /opt/apps/glitchtip
docker compose -f compose.yml --env-file .env up -d
docker compose -f compose.yml --env-file .env exec web python manage.py createsuperuser
```

## 5. Route Cloudflare Tunnel

Dashboard CF Zero Trust → Tunnels → srv-kode01-002 → Public Hostnames → Add :
- Subdomain : `glitchtip`
- Domain : `kodesaas.com`
- Service : `http://glitchtip-web-1:8000`

DNS Cloudflare : CNAME `glitchtip.kodesaas.com` → `<tunnel-id>.cfargotunnel.com` (proxied)

## 6. Smoke test

```bash
curl -I https://glitchtip.kodesaas.com
```
