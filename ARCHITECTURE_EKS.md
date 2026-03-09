# Food Delivery Architecture (EKS + ALB)

Last updated: 2026-03-09

## Topology

```text
Internet
  |
  | DNS records in Route 53
  | - food.trungdevops.vn
  | - admin.food.trungdevops.vn
  v
AWS Application Load Balancer (internet-facing)
  - TLS cert from ACM
  - Listener 80 -> redirect to 443
  - Listener 443 -> host/path routing
  |
  v
Amazon EKS cluster
  Namespace: food-delivery
  IngressClass: alb (AWS Load Balancer Controller)
  Ingress: food-delivery-ingress-alb
    host food.trungdevops.vn
      /api*    -> service/backend:4000
      /images* -> service/backend:4000
      /*       -> service/frontend:80
    host admin.food.trungdevops.vn
      /*       -> service/admin:80
  |
  +-> Service frontend (ClusterIP:80) -> frontend deployment/pods
  +-> Service admin    (ClusterIP:80) -> admin deployment/pods
  +-> Service backend  (ClusterIP:4000) -> backend deployment/pods
                                         + PVC mounted at /app/uploads
  |
  v
External MongoDB (Atlas or self-hosted MongoDB)
```

## Request Flow

1. Client resolves domain in Route 53.
2. Traffic hits ALB and is terminated with ACM certificate.
3. ALB routes by host/path to Kubernetes services through Ingress rules.
4. Service forwards to pods (`frontend`, `admin`, `backend`).
5. Backend reads ConfigMap/Secret, uses PVC for uploads, and connects to MongoDB.

## Notes For This Repo

- `k8s/08-ingress-alb.yaml` is the ALB ingress manifest for EKS.
- Replace `alb.ingress.kubernetes.io/certificate-arn` with your real ACM ARN.
- Ensure AWS Load Balancer Controller is installed before applying ingress.
- Keep only one active ingress option at a time (`nginx`, `haproxy`, or `alb`) to avoid routing confusion.
- Alternative architecture with HAProxy controller is documented in `ARCHITECTURE_EKS_HAPROXY.md`.
