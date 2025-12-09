# Deployment

Docker Compose infrastructure for 1337.legal services.

## Services

| Service    | Port | Description                      |
|------------|------|----------------------------------|
| `caddy`    | 443  | Reverse proxy with automatic TLS |
| `database` | 5432 | PostgreSQL database              |
| `backend`  | 3000 | API server (internal)            |
| `relay`    | 25   | SMTP server for email forwarding |
| `tor`      | -    | Tor hidden service proxy         |

## Quick Start

```bash
# Copy environment template
cp .env.example .env

# Edit with your values
nano .env

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f
```

## Configuration

### Environment Variables

Copy `.env.example` to `.env` and configure:

```env
# Database
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password
POSTGRES_HOST=database
POSTGRES_PORT=5432
POSTGRES_DB=legal1337
DATABASE_URL=postgresql://postgres:your_secure_password@database:5432/legal1337

# Authentication
JWT_SECRET=your_jwt_secret

# DKIM (Email Signing)
DKIM_DOMAIN=1337.legal
DKIM_SELECTOR=default
DKIM_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----"

# Relay TLS
RELAY_CERTIFICATES="-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----"
RELAY_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
```

### Caddy

The `caddy/Caddyfile` configures:

- `api.1337.legal` - HTTPS reverse proxy to backend
- Tor hidden service endpoint with API and frontend routing

Place TLS certificates in `caddy/certificates/`:

- `1337.crt` - Certificate chain
- `1337.key` - Private key

### Tor Hidden Service

The `.onion` address is generated on first run and stored in the `tor_data` volume.

To retrieve your onion address:

```bash
docker-compose exec tor cat /var/lib/tor/hidden_service/hostname
```

## Directory Structure

```
Deployment/
├── .env.example        # Environment template
├── docker-compose.yml  # Service definitions
├── caddy/
│   ├── Caddyfile       # Reverse proxy config
│   └── certificates/   # TLS certs (not in git)
└── Tor/
    └── torrc           # Tor daemon config
```

## Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Rebuild images
docker-compose build --no-cache

# View service logs
docker-compose logs -f backend
docker-compose logs -f relay

# Access database
docker-compose exec database psql -U postgres -d legal1337

# Restart a single service
docker-compose restart backend
```

