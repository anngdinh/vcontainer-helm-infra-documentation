# How to migrate VKS to VKS

Deploy a sample service

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: annd2
---
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
          storage: 20Gi
```

Modify data

```bash
kubectl exec -n annd2 -it web-0 bash
cd /usr/share/nginx/html
echo -e "<html>\n<head>\n  <title>Annd2</title>\n</head>\n<body>\n  <h1>Hello, Tantm3</h1>\n</body>\n</html>" > index.html
```

Access to Nodeport URL, we will see "Hello, Tantm3".

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
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.9.0 \
    --use-node-agent \
    --use-volume-snapshots=false \
    --secret-file ./credentials-velero \
    --bucket annd2-01 \
    --backup-location-config region=hcm03,s3ForcePathStyle="true",s3Url=https://hcm03.vstorage.vngcloud.vn
```

Mark volume to backup, otherwise skip:

```bash
kubectl -n annd2 annotate pod/web-0 backup.velero.io/backup-volumes=disk-ssd
```

Create map storage class on target cluster:

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
  standard-rwo: sc-iops-200-retain
```

Create backup

```bash
velero backup create tantm3 --include-cluster-scoped-resources="" \
    --include-namespace-scoped-resources="*" \
    --include-namespaces annd2 \
    --parallel-files-upload 20 --wait

velero backup describe tantm3 --details
```

Create restore in target cluster:

```bash
velero restore create --from-backup tantm3
```
