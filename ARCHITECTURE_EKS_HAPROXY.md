# Food Delivery Architecture (EKS + HAProxy Ingress Controller)

Last updated: 2026-03-09

## Full Topology

```text
User Browser
  |
  | 1) DNS resolve in Route 53
  |    - food.trungdevops.vn
  |    - admin.food.trungdevops.vn
  v
AWS Load Balancer (public NLB/ALB)
  |
  | 2) Forward traffic to HAProxy Ingress Service
  v
Namespace: haproxy-controller
  Service: haproxy-kubernetes-ingress (LoadBalancer/NodePort)
  Deployment: haproxy-ingress-controller (pods)
  IngressClass: haproxy
  |
  | 3) Controller watches Ingress resources and applies host/path rules
  v
Namespace: food-delivery
  Ingress: food-delivery-ingress-haproxy
    host food.trungdevops.vn
      /api*    -> service/backend:4000
      /images* -> service/backend:4000
      /*       -> service/frontend:80
    host admin.food.trungdevops.vn
      /*       -> service/admin:80
  |
  v
ClusterIP Services
  frontend:80 -> frontend deployment/pods
  admin:80    -> admin deployment/pods
  backend:4000 -> backend deployment/pods
                  + PVC mounted at /app/uploads
                  + env from ConfigMap/Secret
  |
  v
External MongoDB (Atlas or self-hosted MongoDB)
```

## Request Flow

1. Client resolves domain in Route 53.
2. Traffic enters AWS load balancer and is forwarded to HAProxy Ingress Controller service.
3. HAProxy controller reads `food-delivery-ingress-haproxy` (`ingressClassName: haproxy`).
4. Controller routes by host/path to `frontend`, `admin`, or `backend` service.
5. Backend reads ConfigMap/Secret, uses PVC for uploads, and connects to MongoDB.

## If HAProxy Controller Is Missing

`k8s/07-ingress-haproxy.yaml` is only a routing declaration. Without HAProxy Ingress Controller pods:

- No component watches `IngressClass=haproxy`.
- No runtime routing config is generated.
- Traffic cannot be routed by host/path to services.
- Result is typically 404, 503, or timeout depending on LB/service wiring.

## Notes For This Repo

- Use ingress manifest: `k8s/07-ingress-haproxy.yaml`.
- Ensure HAProxy Ingress Controller is installed and exposes a `LoadBalancer` service.
- Keep only one ingress path active at a time in production:
  - ALB direct: `k8s/08-ingress-alb.yaml`
  - HAProxy controller: `k8s/07-ingress-haproxy.yaml`
