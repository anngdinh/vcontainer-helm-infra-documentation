# How to migrate VKS to VKS

Deploy a sample service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: annd2
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  selector:
    app: nginx
  type: NodePort
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
  namespace: annd2
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: disk-ssd
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: disk-ssd
      namespace: annd2
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: sc-iops-200-retain
      resources:
        requests:
          storage: 40Gi
```

Modify data

```bash
kubectl exec -n annd2 -it web-0 bash
cd /usr/share/nginx/html
echo -e "<html>\n<head>\n  <title>Annd2</title>\n</head>\n<body>\n  <h1>Hello, Tantm3</h1>\n</body>\n</html>" > index.html
```

Access to Nodeport URL, we will see "Hello, Annd2".

We have to install Snapshot Controller by VNGCLOUD:

```bash
helm repo add vks-helm-charts https://vngcloud.github.io/vks-helm-charts
helm repo update
helm install vngcloud-snapshot-controller vks-helm-charts/vngcloud-snapshot-controller \
  --replace --namespace kube-system
```

Apply this file to create default Volume Snapshot for every PVC

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

Create a vStorage Project, Container, IAM Key,...
Create a file `credentials-velero` with content:

```yaml
[default]
aws_access_key_id=<AWS_ACCESS_KEY_ID>
aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>
```

Install velero cli:

```bash
curl -OL https://github.com/vmware-tanzu/velero/releases/download/v1.13.2/velero-v1.13.2-linux-amd64.tar.gz
tar -xvf velero-v1.13.2-linux-amd64.tar.gz
cp velero-v1.13.2-linux-amd64/velero /usr/local/bin
```

Install velero to cluster:

```bash
velero install --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.0,velero/velero-plugin-for-csi:v0.7.0 \
  --secret-file ./credentials-velero \
  --bucket annd2-01 \
  --backup-location-config region=hcm03,s3ForcePathStyle="true",s3Url=https://hcm03.vstorage.vngcloud.vn \
  --use-node-agent \
  --features=EnableCSI

# Enable CSI client:
velero client config set features=EnableCSI
```

Create backup

```bash
velero backup create tantm3 --include-cluster-scoped-resources="" \
    --include-namespace-scoped-resources="*" \
    --include-namespaces annd2 \
    --wait

# --snapshot-move-data is Specify whether snapshot data should be moved

velero backup describe tantm3 --details
```

Create restore to another namespace:

```bash
velero restore create --from-backup tantm3
```
