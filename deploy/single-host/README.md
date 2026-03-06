# Single Host Deployment (No Kubernetes)

This folder is for running the project on one VM/server using Docker Compose + Nginx reverse proxy.

## Architecture

- `food.trungdevops.vn`:
  - `/` -> frontend
  - `/api/*` -> backend
  - `/images/*` -> backend
- `admin.food.trungdevops.vn`:
  - `/` -> admin

## Prerequisites

- Docker + Docker Compose plugin installed on the host.
- DNS records:
  - `food.trungdevops.vn` -> server public IP
  - `admin.food.trungdevops.vn` -> server public IP
- Backend env file exists at `backend/.env` (copy from `backend/.env.example` and fill secrets).

Required in `backend/.env`:
- `MONGO_URL`
- `JWT_SECRET`
- `STRIPE_SECRET_KEY`
- `FRONTEND_URL=http://food.trungdevops.vn`
- `CORS_ORIGINS=http://food.trungdevops.vn,http://admin.food.trungdevops.vn`

## Run

```bash
cd deploy/single-host
docker compose up -d --build
```

No `.env` file is required for this folder unless you want to override defaults.

## Verify

- Frontend: `http://food.trungdevops.vn`
- Admin: `http://admin.food.trungdevops.vn`
- API check: `http://food.trungdevops.vn/api/food/list`

## HTTPS

Recommended for production: put TLS in front of this Nginx (certbot or cloud load balancer).
