# How to migrate VKS to VKS

* Prepare velero in both cluster in More usage
* Download helper bash script and grand execute permission [velero_helper.sh](https://raw.githubusercontent.com/vngcloud/velero/main/velero_helper.sh)
* (Optional) Deploy some sample app in More usage

## In both cluster (source and target)

Install velero to cluster:

```bash
velero install --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.0,velero/velero-plugin-for-csi:v0.7.0 \
  --secret-file ./credentials-velero \
  --bucket __________________________ \
  --backup-location-config region=hcm03,s3ForcePathStyle="true",s3Url=https://hcm03.vstorage.vngcloud.vn \
  --use-node-agent \
  --features=EnableCSI

# Enable CSI client:
velero client config set features=EnableCSI
```

We have to install Snapshot Controller by VNGCLOUD:

```bash
helm repo add vks-helm-charts https://vngcloud.github.io/vks-helm-charts
helm repo update
helm install vngcloud-snapshot-controller vks-helm-charts/vngcloud-snapshot-controller \
  --replace --namespace kube-system
```

## In source cluster

Annotate persistent volume and label resource exclude from backup

```bash
./velero_helper.sh mark_volume -c
./velero_helper.sh mark_exclude -c
```

Create `VolumeSnapshotClass`

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: vngcloud-vsclass
  labels:
    velero.io/csi-volumesnapshot-class: "true"
driver: bs.csi.vngcloud.vn
deletionPolicy: Delete

# user can choose the VolumeSnapshotClass by setting annotation velero.io/csi-volumesnapshot-class_disk.csi.cloud.com: "test-snapclass" on backup resource.
# user can choose the VolumeSnapshotClass by setting annotation velero.io/csi-volumesnapshot-class: "test-snapclass" on PersistentVolumeClaim resource.
```

Create backup

```bash
velero backup create vks-full-backup \
  --exclude-namespaces velero \
  --include-cluster-resources=true \
  --wait

# --snapshot-move-data is Specify whether snapshot data should be moved

velero backup describe vks-full-backup --details
```

## In target cluster

Restore backup

```bash
velero restore create --from-backup vks-full-backup
```
