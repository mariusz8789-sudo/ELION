# ELION v5.9 - Autonomous Software Development Agent

Full-stack autonomous agent system with self-healing capabilities, multi-language support, and production-ready deployment.

## Features

- **Frontend**: React 18 + TypeScript + Tailwind CSS + i18n (EN/PL/AR)
- **Backend**: Node.js + tRPC + Express + JWT Auth
- **Worker**: BullMQ + Redis + Self-Fix Loop + Adaptive Memory
- **Database**: PostgreSQL + ChromaDB
- **Deployment**: Docker Compose + Kubernetes manifests
- **Payments**: Stripe integration with webhooks

## Quick Start

### Prerequisites

- Docker 20.10+
- Docker Compose 2.0+
- Node.js 18+ (for local development)
- pnpm 8+ (for local development)

### Fast-Track Deployment

```bash
# Clone and enter directory
cd elion-v5.9

# One-command deployment
bash scripts/ac_fast_start.sh
```

This will:
1. Check Docker availability
2. Generate `.env` with secure JWT_SECRET
3. Start all services (postgres, redis, chroma, backend, worker, frontend)
4. Run health checks
5. Display service URLs

### Manual Deployment

```bash
# 1. Create .env from template
cp .env.example .env

# 2. Generate JWT_SECRET
openssl rand -hex 32

# 3. Edit .env and add JWT_SECRET

# 4. Start services
docker compose -f docker-compose.prod.yml up -d --build

# 5. Wait for services to initialize
sleep 15

# 6. Check health
curl http://localhost:3001/api/health
curl http://localhost:3000
```

## Service Ports

- **Frontend**: http://localhost:3000
- **Backend**: http://localhost:3001
- **PostgreSQL**: localhost:5432
- **Redis**: localhost:6379
- **ChromaDB**: http://localhost:8000

## Health Checks

```bash
# Backend API
curl http://localhost:3001/api/health

# Frontend
curl http://localhost:3000

# Redis
docker exec elion-redis redis-cli ping

# PostgreSQL
docker exec elion-postgres pg_isready -U elion_user
```

## Smoke Tests

### 1. Frontend i18n

1. Open http://localhost:3000
2. Click language buttons: EN, PL, AR
3. Verify navigation labels change

### 2. Queue Processing

```bash
# Enqueue a test job (via backend API)
curl -X POST http://localhost:3001/api/trpc/createJob \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Test job"}'

# Check worker logs
docker compose -f docker-compose.prod.yml logs worker
```

### 3. Stripe Webhooks (Optional)

```bash
# Terminal 1: Start Stripe CLI listener
stripe listen --forward-to http://localhost:3001/api/webhooks/stripe

# Copy whsec_... value and add to .env
echo "STRIPE_WEBHOOK_SECRET=whsec_..." >> .env

# Restart backend
docker compose -f docker-compose.prod.yml restart backend

# Terminal 2: Trigger test event
stripe trigger payment_intent.succeeded

# Check backend logs
docker compose -f docker-compose.prod.yml logs backend | grep webhook
```

## Local Development

```bash
# Install dependencies
pnpm install

# Start all services in dev mode
pnpm dev

# Build all packages
pnpm build

# Run tests
pnpm test
```

## Project Structure

```
elion-v5.9/
├── packages/
│   ├── frontend/       # React + Vite + i18n
│   ├── backend/        # Node + tRPC + Express
│   ├── worker/         # BullMQ + Self-Fix Loop
│   └── sdk/            # tRPC client SDK
├── infra/
│   ├── nginx/          # Nginx reverse proxy
│   └── k8s/            # Kubernetes manifests
├── scripts/            # Deployment scripts
├── playbooks/          # DevOps playbooks
├── assets/brand/       # ELION logo
├── docker-compose.prod.yml
├── .env.example
└── README.md
```

## Environment Variables

See `.env.example` for all available configuration options.

Required variables:
- `JWT_SECRET` - Generate with `openssl rand -hex 32`
- `DATABASE_URL` - PostgreSQL connection string

Optional variables:
- `STRIPE_SECRET_KEY` - For payment processing
- `STRIPE_WEBHOOK_SECRET` - For webhook verification
- `MAPBOX_TOKEN` - For map features
- `WORKER_CONCURRENCY` - Number of concurrent jobs (default: 3)

## Troubleshooting

### Backend 500 / Health FAIL

```bash
# Check logs
docker compose -f docker-compose.prod.yml logs backend --tail=50

# Verify DATABASE_URL
grep DATABASE_URL .env

# Restart backend
docker compose -f docker-compose.prod.yml restart backend
```

### Frontend Not Loading

```bash
# Check logs
docker compose -f docker-compose.prod.yml logs frontend --tail=50

# Rebuild frontend
docker compose -f docker-compose.prod.yml up -d --build frontend
```

### Queue Not Processing

```bash
# Check Redis
docker exec elion-redis redis-cli ping

# Check worker logs
docker compose -f docker-compose.prod.yml logs worker --tail=50

# Restart worker
docker compose -f docker-compose.prod.yml restart worker
```

## Production Recommendations

1. Use managed PostgreSQL (AWS RDS, Google Cloud SQL, etc.)
2. Use Redis cluster for high availability
3. Set up load balancer for backend instances
4. Enable auto-scaling based on CPU/memory
5. Configure CDN for frontend assets
6. Set up monitoring (Prometheus + Grafana)
7. Enable log aggregation (ELK or Loki)
8. Implement backup automation

## License

Proprietary - All rights reserved

## Version

5.9.0 - November 2025

