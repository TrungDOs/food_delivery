# Food Delivery Architecture (Kubernetes + Ingress)

```text
                          Internet
                              |
                    DNS A records (same LB IP)
              food.trungdevops.vn / admin.food.trungdevops.vn
                              |
                    +------------------------+
                    |  NGINX Ingress Ctrl    |
                    |  (Kubernetes proxy)    |
                    +------------------------+
                       |                  |
      host=food.trungdevops.vn      host=admin.food.trungdevops.vn
      /            -> frontend-svc   / -> admin-svc
      /api,/images -> backend-svc
                       |
     +-----------------+------------------+
     |                                    |
+------------+                      +-------------+
| frontend   |                      |   admin     |
| Deployment |                      | Deployment  |
| (nginx)    |                      | (nginx)     |
+------------+                      +-------------+
          \                            /
           \   HTTPS calls /api       /
            +------------------------+
            |      backend-svc       |
            +------------------------+
                      |
               +--------------+
               | backend pod  |
               | Node/Express |
               +--------------+
                 |        |
          /app/uploads    | MONGO_URL
            PVC           v
                    +-------------+
                    | MongoDB     |
                    | (Atlas/self)|
                    +-------------+
```

## Request Flow

1. User opens `http://food.trungdevops.vn` -> Ingress -> `frontend` service.
2. Frontend calls `http://food.trungdevops.vn/api/...` -> Ingress routes `/api` to `backend` service.
3. Admin opens `http://admin.food.trungdevops.vn` -> Ingress -> `admin` service, and admin APIs also go to backend.
4. Backend connects to MongoDB via `MONGO_URL`.
5. Uploaded images are served from `/images` and stored on PVC mounted at `/app/uploads`.

## Routing Rules (Current)

- Host `food.trungdevops.vn`:
  - `/` -> frontend
  - `/api` -> backend
  - `/images` -> backend
- Host `admin.food.trungdevops.vn`:
  - `/` -> admin

## Required Environment

- Frontend/Admin build:
  - `VITE_API_URL=http://food.trungdevops.vn`
- Backend runtime:
  - `MONGO_URL=...`
  - `JWT_SECRET=...`
  - `STRIPE_SECRET_KEY=...`
  - `FRONTEND_URL=http://food.trungdevops.vn`
  - `CORS_ORIGINS=http://food.trungdevops.vn,http://admin.food.trungdevops.vn`
