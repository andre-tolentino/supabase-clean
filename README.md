# Supabase Self-Hosted + MinIO

Complete Supabase Self-Hosted configuration with S3 storage via MinIO, aligned with the official [supabase/supabase](https://github.com/supabase/supabase) repository (master branch - June 2026).

## Included Services

| Service | Version | Description |
|---------|---------|-------------|
| **Studio** | `2026.06.22` | Administration dashboard |
| **Kong** | `3.9.3` | API Gateway |
| **Auth** | `v2.191.0` | Authentication (GoTrue) |
| **PostgREST** | `v14.13` | Automatic REST API |
| **Realtime** | `v2.112.0` | Real-time (WebSocket) |
| **Storage** | `v1.61.5` | File storage |
| **imgproxy** | `v3.31.4` | Image processing |
| **pg-meta** | `v0.96.6` | Postgres management |
| **Edge Functions** | `v1.74.2` | Serverless functions (Deno) |
| **Logflare** | `1.43.1` | Logs and analytics |
| **Vector** | `0.53.0` | Log collection |
| **Postgres** | `17.6.1.139` | Database |
| **Supavisor** | `2.9.7` | Connection pooler |
| **MinIO** | latest | S3-compatible storage |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) >= 20.10
- [Docker Compose](https://docs.docker.com/compose/install/) >= 2.0
- Minimum: 4GB RAM, 2 vCPUs, 40GB SSD
- Recommended: 8GB+ RAM, 4+ vCPUs, 80GB+ SSD

## Quick Start

```bash
# 1. Clone or copy the folder
cd supabase-clean

# 2. Edit .env with your configuration
nano .env

# 3. Start all services
docker compose up -d

# 4. Verify all services are running
docker compose ps
```

## Configuration

### 1. Environment Variables

Edit the `.env` file and configure:

```bash
# Postgres password (CHANGE for production!)
POSTGRES_PASSWORD=your-super-secret-and-long-password

# JWT Secret (CHANGE for production!)
JWT_SECRET=your-jwt-token-with-at-least-32-characters

# Dashboard password
DASHBOARD_USERNAME=admin
DASHBOARD_PASSWORD=your-dashboard-password

# Supabase public URL
SUPABASE_PUBLIC_URL=https://your-domain.com
API_EXTERNAL_URL=https://your-domain.com

# Your application URL (for auth redirects)
SITE_URL=https://your-app.com
```

### 2. Generate Keys (Optional - Recommended)

To use asymmetric API keys (ES256), run the generation scripts from the official repository:

```bash
# Clone the official repository just for the scripts
git clone --depth 1 --filter=blob:none --sparse https://github.com/supabase/supabase
cd supabase
git sparse-checkout set docker/utils
cd docker/utils

# Generate the keys
sh generate-keys.sh
sh add-new-auth-keys.sh

# Copy the generated keys to your .env
# and add them to your supabase-clean/.env
```

### 3. OAuth (Social Login)

To enable Google, GitHub, etc., add to `.env`:

```bash
# Google
GOOGLE_ENABLED=true
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_SECRET=your-secret

# GitHub
GITHUB_ENABLED=true
GITHUB_CLIENT_ID=your-client-id
GITHUB_SECRET=your-secret
```

And uncomment the corresponding lines in `docker-compose.yml` in the `auth` section.

### 4. HTTPS (Production)

For OAuth to work in production, HTTPS is required. Use a reverse proxy (Caddy or Nginx):

```bash
# Example with Caddy
# Add to docker-compose.yml:
services:
  caddy:
    image: caddy:2
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
```

## Access

| Service | URL | Credentials |
|---------|-----|-------------|
| **Studio (Dashboard)** | `http://localhost:8000` | `DASHBOARD_USERNAME` / `DASHBOARD_PASSWORD` |
| **REST API** | `http://localhost:8000/rest/v1/` | Via `ANON_KEY` or `SERVICE_ROLE_KEY` |
| **Auth** | `http://localhost:8000/auth/v1/` | - |
| **Storage** | `http://localhost:8000/storage/v1/` | Via `ANON_KEY` or `SERVICE_ROLE_KEY` |
| **Realtime** | `ws://localhost:8000/realtime/v1/` | Via `ANON_KEY` |
| **Edge Functions** | `http://localhost:8000/functions/v1/` | Via `ANON_KEY` or `SERVICE_ROLE_KEY` |
| **Postgres** | `localhost:5432` | `postgres` / `POSTGRES_PASSWORD` |
| **MinIO Console** | `http://localhost:9001` | `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` |

## Useful Commands

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# View logs for a specific service
docker compose logs -f auth
docker compose logs -f storage
docker compose logs -f db

# View logs for all services
docker compose logs -f

# Service status
docker compose ps

# Restart a specific service
docker compose restart auth

# Update images
docker compose pull
docker compose up -d

# Stop and remove data (WARNING: deletes everything!)
docker compose down -v
```

## File Structure

```
supabase-clean/
├── docker-compose.yml          # Main configuration
├── .env                        # Environment variables (DO NOT commit!)
├── .env.example                # Variables template
├── README.md                   # This file
└── volumes/
    ├── api/
    │   ├── kong.yml            # Kong configuration
    │   └── kong-entrypoint.sh  # Startup script
    ├── db/
    │   ├── realtime.sql        # Realtime schema
    │   ├── webhooks.sql        # Webhooks and pg_net
    │   ├── roles.sql           # User passwords
    │   ├── jwt.sql             # JWT configuration
    │   ├── _supabase.sql       # _supabase database
    │   ├── logs.sql            # _analytics schema
    │   └── pooler.sql          # _supavisor schema
    ├── functions/
    │   └── main/
    │       └── index.ts        # Edge Functions runtime
    ├── logs/
    │   └── vector.yml          # Vector configuration
    ├── pooler/
    │   └── pooler.exs          # Supavisor configuration
    ├── snippets/               # Studio snippets
    └── storage/                # Storage files (local)
```

## Backup

### Database Backup

```bash
# Full backup
docker compose exec db pg_dump -U postgres postgres > backup.sql

# Backup a specific schema
docker compose exec db pg_dump -U postgres -n public postgres > backup_public.sql

# Restore backup
cat backup.sql | docker compose exec -T db psql -U postgres
```

### Storage Backup

```bash
# Files are stored in volumes/storage/
# Copy the entire directory
cp -r volumes/storage/ /path/to/backup/storage/
```

## Updating

```bash
# 1. Stop services
docker compose down

# 2. Update docker-compose.yml and volumes/ files
# (copy from new versions)

# 3. Pull new images
docker compose pull

# 4. Start again
docker compose up -d

# 5. Verify everything is working
docker compose ps
docker compose logs -f
```

## Troubleshooting

### Container won't start

```bash
# Check logs of the failing container
docker compose logs <container-name>

# Check if all services are healthy
docker compose ps
```

### Database connection error

```bash
# Check if Postgres is running
docker compose exec db pg_isready -U postgres

# Restart Postgres
docker compose restart db
```

### Storage not working

```bash
# Check if MinIO is running
docker compose logs minio

# Check if bucket was created
docker compose exec minio mc alias set supa-minio http://minio:9000 supa-storage secret1234
docker compose exec minio mc ls supa-minio/
```

### Logs not showing in Studio

```bash
# Check if Vector and Logflare are running
docker compose logs vector
docker compose logs analytics

# Restart log services
docker compose restart vector analytics
```

## Security

- **CHANGE** all default passwords before using in production
- **DO NOT** expose Postgres directly to the internet
- Use **HTTPS** in production
- Configure **firewall** to restrict access
- Keep images **updated**

## License

This configuration is based on [Supabase](https://github.com/supabase/supabase) (Apache 2.0 License).
