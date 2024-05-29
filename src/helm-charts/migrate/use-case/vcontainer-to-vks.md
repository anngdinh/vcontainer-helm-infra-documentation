# How to migrate vContainer to VKS

* You must have vStotage project, bucket (container), IAM key
* Have file `credentials-velero` include iam key
* Install velero CLI
* Download helper bash script and grand execute permission [velero_helper.sh](https://raw.githubusercontent.com/vngcloud/velero/main/velero_helper.sh)

## Backup vContainer

Annotate persistent volume and label resource exclude from backup

```bash
./velero_helper.sh mark_volume -c
./velero_helper.sh mark_exclude -c
```

Create backup. You have to create 2 version backup for cluster resource and namespace resource

```bash
velero backup create vcontainer-full-backup --exclude-namespaces velero \
    --include-cluster-resources=true \
    --wait
```

### Restore vContainer

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

Restore backup

```bash
velero restore create --item-operation-timeout 1m --from-backup vcontainer-full-backup
```
