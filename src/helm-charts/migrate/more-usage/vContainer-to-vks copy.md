# How to migrate vContainer to VKS (Advance)

## In both cluster (source and target)

Create a vStorage Project, Container, IAM Key,...
Create a file `credentials-velero` with content:

```yaml
[default]
aws_access_key_id=________CLIENT_ID____________
aws_secret_access_key=________CLIENT_SECRET____________
```

Define bucket:

```bash
export BUCKET=___________
```

Install velero cli:

```bash
curl -OL https://github.com/vmware-tanzu/velero/releases/download/v1.13.2/velero-v1.13.2-linux-amd64.tar.gz
tar -xvf velero-v1.13.2-linux-amd64.tar.gz
cp velero-v1.13.2-linux-amd64/velero /usr/local/bin
```

Install velero to cluster:

```bash
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.9.0 \
    --use-node-agent \
    --use-volume-snapshots=false \
    --secret-file ./credentials-velero \
    --bucket $BUCKET \
    --backup-location-config region=hcm03,s3ForcePathStyle="true",s3Url=https://hcm03.vstorage.vngcloud.vn

# flag `--default-volumes-to-fs-backup` will backup all persistent volume as file system volume
```

## In source cluster

### Aware of resource

* Ingress controller
  * If source cluster already use `vngcloud-ingress-controller`: add annotation `vks.vngcloud.vn/ignore: true` for every Ingress resource.
  * Others ingress controller will cause no impact, except the `nginx-ingress-controller` in vContainer.
* Controller manager: Resource control service type LoadBalancer. Only 1 CCM can work in a cluster.
  * If you plan to use `vngcloud-controller-manager` in target cluster: add annotation `vks.vngcloud.vn/ignore: true` for every Service type LoadBalancer.
  * If not, others CCM will not work in VKS default cluster. You must uninstall `vngcloud-controller-manager` in VKS target cluster first.
* Admission webhooks

### Mark volume to backup and resource unnnecessary

Link to helper file: [helper.sh](https://raw.githubusercontent.com/anngdinh/vcontainer-helm-infra-documentation/main/src/helm-charts/migrate/helper.sh). Create a file named `helper.sh` and grant execute permission.

#### 1. Convert hostPath volume to Persistent Volume to include in backup

Because velero not support backup hostPath volume, you must convert in to Persistent Volume.

```bash
./helper.sh check_hostPath
```

#### 2. Mark Persistent Volume to include in backup

All Persistent Volumes' data will be stored in vStorage. You must add this annotation for every pod using PVC with volume name: `backup.velero.io/backup-volumes=volume1,volume2`

Or using helper. It'll detect all Pod using PVC and print command to run.

```bash
./helper.sh mark_volume
```

#### 3. Mark resource in exclude in backup

Because VKS is fully managed cluster, you don't need to include system resource such as: `calico`, `kube-dns`, `kube-scheduler`, `kube-apiserver`,... In addition, some resource in vContainer will be excluded such as: `magnum-auto-healer`, `cluster-autoscaler`, `csi-cinder`,...

```bash
./helper.sh mark_exclude
```

#### 4. Check label and taint of node

Maybe resource in source cluster using labels and taints to schedule. Make sure these important labels and taints is exist in taget cluster.

```bash
./helper.sh check_node_label
./helper.sh check_node_taint
```

#### 5. Check total

Run this command to check again before create backup

```bash
./helper.sh check
```

#### 6. Mapping Storage Class

We need convert Storage Class in source cluster to target cluster. Assume you have 2 Storage Class below:

```bash
root@vks-dev:/# kubectl get sc
NAME                            PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
sc-iops-200-retain (default)    csi.vngcloud.vn   Retain          Immediate           true                   81s
sc-ssd-10000-delete (default)   csi.vngcloud.vn   Delete          Immediate           true                   14d
```

And you want to convert to 2 Storage Class `ssd-200`, `ssd-10000` in target cluster arcordingly, you have to apply this yaml in target cluster:

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
  sc-iops-200-retain: ssd-200
  sc-ssd-10000-delete: ssd-10000
```

#### 6. Create backup

```bash
velero backup create minato --exclude-namespaces velero \
    --include-cluster-resources=true \
    --wait

velero backup describe minato --details
```

## In target cluster

Create new Storage Class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name:  ssd-3000                           # [1] The StorageClass name, CAN be changed
provisioner: bs.csi.vngcloud.vn                     # The CSI driver name, MUST set this value
parameters:
  type: vtype-61c3fc5b-f4e9-45b4-8957-8aa7b6029018  # Change it to your volume type UUID from portal
  ispoc: "true"
allowVolumeExpansion: true
```

Create restore in target cluster:

```bash
velero restore create --from-backup minato
```

## After migrate

### Source cluster

```bash
./helper.sh unmark
```

### Target cluster

```bash
./helper.sh unmark
```
