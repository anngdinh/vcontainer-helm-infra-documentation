# How to migrate other cloud to VKS

## EKS

### Backup EKS

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install wordpress bitnami/wordpress \
    --set mariadb.primary.persistence.enabled=true \
    --set mariadb.primary.persistence.storageClass=gp2 \
    --set mariadb.primary.persistence.size=20Gi \
    --set persistence.enabled=false \
    --namespace tantm3 \
    --create-namespace

kubectl -n tantm3 annotate pod/wordpress-mariadb-0 backup.velero.io/backup-volumes=data

velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.9.0 \
    --use-node-agent \
    --use-volume-snapshots=false \
    --secret-file ./credentials-velero \
    --bucket __________________________________ \
    --backup-location-config region=hcm03,s3ForcePathStyle="true",s3Url=https://hcm03.vstorage.vngcloud.vn

./helper.sh mark_exclude -c
./helper.sh mark_volume -c

velero backup create ns --exclude-namespaces velero \
    --wait

velero backup create cluster-rs --include-namespaces "" \
  --include-cluster-resources=true
```

### Restore EKS

```bash
# velero restore create --item-operation-timeout 1m --from-backup cluster-rs \
#     --exclude-resources="MutatingWebhookConfiguration,ValidatingWebhookConfiguration"

# velero restore create --item-operation-timeout 1m --from-backup ns

# velero restore create --item-operation-timeout 1m --from-backup cluster-rs
```

## GKE

### Backup GKE

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install wordpress bitnami/wordpress \
    --set mariadb.primary.persistence.enabled=true \
    --set mariadb.primary.persistence.storageClass=standard-rwo \
    --set mariadb.primary.persistence.size=20Gi \
    --set persistence.enabled=false \
    --namespace tantm3 \
    --create-namespace

kubectl -n tantm3 annotate pod/wordpress-mariadb-0 backup.velero.io/backup-volumes=data

velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.9.0 \
    --use-node-agent \
    --use-volume-snapshots=false \
    --secret-file ./credentials-velero \
    --bucket __________________________________ \
    --backup-location-config region=hcm03,s3ForcePathStyle="true",s3Url=https://hcm03.vstorage.vngcloud.vn

./helper.sh mark_exclude -c
./helper.sh mark_volume -c

velero backup create ns --exclude-namespaces velero \
    --wait

velero backup create cluster-rs --include-namespaces "" \
  --include-cluster-resources=true
```

### Restore GKE

```bash
velero restore create --item-operation-timeout 1m --from-backup cluster-rs \
    --exclude-resources="MutatingWebhookConfiguration,ValidatingWebhookConfiguration"

velero restore create --item-operation-timeout 1m --from-backup ns

velero restore create --item-operation-timeout 1m --from-backup cluster-rs
```

### Note GKE

* GKE do not allow deploy daemonsets in all node but velero only need deploy it in node have mount pv, should adjust taint and toleration to do it
* default resource request is cpu:500m and mem:512M, we can change at install step or adjust in deployment yaml
