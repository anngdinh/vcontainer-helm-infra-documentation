# Migrate Cluster from another platform to VKS

* Prepare velero in both cluster in More usage
* Download helper bash script and grand execute permission [velero_helper.sh](https://raw.githubusercontent.com/vngcloud/velero/main/velero_helper.sh)
* (Optional) Deploy some sample app in More usage

## EKS

### Backup EKS

Annotate persistent volume and label resource exclude from backup

```bash
./velero_helper.sh mark_volume -c
./velero_helper.sh mark_exclude -c
```

Create backup. You have to create 2 version backup for cluster resource and namespace resource

```bash
velero backup create eks-cluster --include-namespaces "" \
  --include-cluster-resources=true \
  --wait

velero backup create eks-namespace --exclude-namespaces velero \
    --wait
```

### Restore EKS

Apply this yaml to convert `StorageClass` between 2 cluster

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: change-storage-class-config
  namespace: velero
  labels:
    velero.io/plugin-config: ""
    velero.io/change-storage-class: RestoreItemAction
data:
  _______old_storage_class_______: _______new_storage_class_______  # <= Adjust here
  _______old_storage_class_______: _______new_storage_class_______  # <= Adjust here
```

Restore backup in order

```bash
velero restore create --item-operation-timeout 1m --from-backup eks-cluster \
    --exclude-resources="MutatingWebhookConfiguration,ValidatingWebhookConfiguration"

velero restore create --item-operation-timeout 1m --from-backup eks-namespace

velero restore create --item-operation-timeout 1m --from-backup eks-cluster
```

## GKE

### Backup GKE

Annotate persistent volume and label resource exclude from backup

```bash
./velero_helper.sh mark_volume -c
./velero_helper.sh mark_exclude -c
```

Create backup. You have to create 2 version backup for cluster resource and namespace resource

```bash
velero backup create gke-cluster --include-namespaces "" \
  --include-cluster-resources=true \
  --wait

velero backup create gke-namespace --exclude-namespaces velero \
    --wait
```

### Restore GKE

Apply this yaml to convert `StorageClass` between 2 cluster

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: change-storage-class-config
  namespace: velero
  labels:
    velero.io/plugin-config: ""
    velero.io/change-storage-class: RestoreItemAction
data:
  _______old_storage_class_______: _______new_storage_class_______  # <= Adjust here
  _______old_storage_class_______: _______new_storage_class_______  # <= Adjust here
```

Restore backup in order

```bash
velero restore create --item-operation-timeout 1m --from-backup gke-cluster \
    --exclude-resources="MutatingWebhookConfiguration,ValidatingWebhookConfiguration"

velero restore create --item-operation-timeout 1m --from-backup gke-namespace

velero restore create --item-operation-timeout 1m --from-backup gke-cluster
```

### Note GKE

* GKE do not allow deploy daemonsets in all node but velero only need deploy it in node have mount pv, should adjust taint and toleration to do it
* default resource request is cpu:500m and mem:512M, we can change at install step or adjust in deployment yaml
