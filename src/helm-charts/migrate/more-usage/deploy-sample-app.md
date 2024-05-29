# Deploy some sample app

## NGINX without PV

## NGINX with PV

## WordPress

Deploy sample app WordPress

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install wordpress bitnami/wordpress \
    --set mariadb.primary.persistence.enabled=true \
    --set mariadb.primary.persistence.storageClass=_____________________ \
    --set mariadb.primary.persistence.size=20Gi \
    --set persistence.enabled=false \
    --namespace tantm3 \
    --create-namespace
```
