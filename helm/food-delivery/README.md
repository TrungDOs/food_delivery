# food-delivery Helm Chart

Helm chart nay dong goi cac thanh phan:

- `backend`
- `frontend`
- `admin`
- `ingress`
- `backend` configmap, secret va PVC uploads

## Cai dat

```bash
helm upgrade --install food-delivery ./helm/food-delivery -n food-delivery
```

Neu muon de Helm tao namespace:

```bash
helm upgrade --install food-delivery ./helm/food-delivery -n food-delivery --create-namespace --set namespace.create=true
```

## Tuy chinh nhanh

```bash
helm upgrade --install food-delivery ./helm/food-delivery \
  -n food-delivery \
  --set backend.image.tag=v1 \
  --set frontend.image.tag=v1 \
  --set admin.image.tag=v1
```

Ingress mac dinh dang dung `nginx`. Neu can HAProxy, doi:

```bash
helm upgrade --install food-delivery ./helm/food-delivery \
  -n food-delivery \
  --set ingress.className=haproxy \
  --set ingress.annotations.haproxy\\.org/timeout-client=60s \
  --set ingress.annotations.haproxy\\.org/timeout-server=60s \
  --set ingress.annotations.nginx\\.ingress\\.kubernetes\\.io/proxy-body-size=null
```
