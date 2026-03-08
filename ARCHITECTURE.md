# Food Delivery Architecture (Current Kubernetes Runtime)

Last verified: 2026-03-08 (live cluster)

## Topology

```text
Internet
  |
  | DNS A records
  | - food.trungdevops.vn
  | - admin.food.trungdevops.vn
  v
GCP Load Balancer (created by Service type LoadBalancer)
  External IP: 34.124.236.203
  |
  v
Namespace: ingress-nginx
  Service: ingress-nginx-controller (LoadBalancer)
    - 80  -> NodePort 30931
    - 443 -> NodePort 31063
  Pods: ingress-nginx-controller x3 (Running, spread across 3 nodes)
  IngressClass: nginx (controller: k8s.io/ingress-nginx)
  |
  v
Namespace: food-delivery
  Ingress: food-delivery-ingress (class nginx, ADDRESS 34.124.236.203)
    host food.trungdevops.vn
      /api*    -> service/backend:4000
      /images* -> service/backend:4000
      /*       -> service/frontend:80
    host admin.food.trungdevops.vn
      /*       -> service/admin:80

  Service frontend (ClusterIP:80) -> frontend deployment (3/3 available)
  Service admin    (ClusterIP:80) -> admin deployment (3/3 available)
  Service backend  (ClusterIP:4000) -> backend deployment (1/1 available)
  PVC backend-uploads-pvc (2Gi, Bound) mounted at /app/uploads
  |
  v
External MongoDB on Rancher host
  34.101.158.188:27017
  authSource=admin
  app DB: food_delivery
```

## Request Flow

1. Client resolves domain to `34.124.236.203`.
2. GCP LB forwards traffic to `ingress-nginx-controller` service.
3. NGINX Ingress Controller reads `food-delivery-ingress` (`ingressClassName: nginx`).
4. Controller routes by host/path to `frontend`, `backend`, or `admin` service.
5. Backend reads config/secret via `envFrom`, writes uploads to PVC, and connects to MongoDB host `34.101.158.188`.

## Active vs Inactive Ingress

- Active: `food-delivery-ingress` (class `nginx`)
- Inactive: `food-delivery-ingress-haproxy` (class `haproxy`, no HAProxy controller installed)

## Runtime Notes

- Backend logs show `DB Connected`.
- Current signin issue is data-level (`User Doesn't exist`) when no user is present in `food_delivery.users`, not DB connectivity.
- Backend PVC uses `ReadWriteOnce` (`standard-rwo`), so backend should stay `replicas=1` unless storage is migrated to RWX.
