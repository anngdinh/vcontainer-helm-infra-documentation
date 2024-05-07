# Troubleshooting

## Storage Volumes of the HostPath Type Cannot Be Backed Up

Both HostPath and Local volumes are local storage volumes. However, the Restic tool integrated in Velero cannot back up the PVs of the HostPath type and supports only the Local type. Therefore, you need to replace the storage volumes of the HostPath type with the Local type in the source cluster.

1. Create a storage class for the Local volume. Example YAML:

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: local
    provisioner: kubernetes.io/no-provisioner
    volumeBindingMode: WaitForFirstConsumer
    ```

2. Change the hostPath field to the local field, specify the original local disk path of the host machine, and add the nodeAffinity field. Example YAML:

    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: mysql-pv
      labels: 
        app: mysql
    spec:
      accessModes:
      - ReadWriteOnce
      capacity:
        storage: 5Gi
      storageClassName: local     # Storage class created in the previous step
      persistentVolumeReclaimPolicy: Delete
      local:
        path: "/mnt/data"     # Path of the attached local disk
      nodeAffinity:
        required:
          nodeSelectorTerms:
          - matchExpressions:
            - key: kubernetes.io/hostname
              operator: Exists
    ```

3. Run the following commands to verify the creation result:

    ```bash
    $ kubectl get pv -A
    NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE
    mysql-pv   5Gi        RWO            Delete           Available                       local                   3s
    ```

## More usage

* restore từ namespace này qua namespace khác
* thay đổi restore order với `--restore-resource-priorities`
* thay đổi Storage Classes bằng config map
* thay đổi Image Repositories bằng config map
* thay đổi PVC selected-node (in PVC, can select node for PVC by this annotation `volume.kubernetes.io/selected-node: <node-name>`).
* when restore service nodeport, nodeport number is delete and auto assign again in new cluster (can use flag `--preserve-nodeports=true` to keep)
* velero không sửa lại resource nếu đã tồn tại, trừ service account: merge các quyền của nó và thêm các label và annotation (dùng flag `--existing-resource-policy=update` để đổi)
* incremental backup dựa trên PVCs
*

## Các lỗi có thể xảy ra

* velero server nằm trên node bị lệch time so với s3 server, ko tạo được backup.
* daemonset node-agent bị taint và tolleration chặn trên node, không copy được filesystem được mount vô node, ko backup được volume
* persistent volume đã bị sử dụng, ko claim được nữa
* name volume backup in annotation is `pod.spec.volumes`, not pvc name
* `hostPath` volumes are not supported. Local persistent volumes are supported.

## Restic and Kopia perfromance and impact

Tham khảo performance do Velero cung cấp: [link](https://velero.io/docs/main/performance-guidance/)

```bash=
helm repo add vmonitor-platform https://vngcloud.github.io/helm-charts-vmonitor

helm install vmonitor-metric-agent vmonitor-platform/vmonitor-metric-agent \
    --set vmonitor.iamClientID=____________________ \
    --set vmonitor.iamClientSecret=____________________ \
    --set vmonitor.clusterName=____________________
```

Cài node-exporter cho cụm VKS, ta có những thông số (CPU, RAM,...) khi trong cụm đang sử dụng như sau:

| Usecase                            | CPU  | RAM  |
|:---------------------------------- |:----:|:----:|
| không có velero                    | Text | Text |
| có velero nhưng ko chạy backup     | Text | Text |
| có velero, chạy backup bằng kopia  | Text | Text |
| có velero, chạy backup bằng restic | Text | Text |

Impact của kopia lớn hơn so với restic. Nhưng theo docs của Velero benchmark:

* Với lượng data ít, số file nhiều, kopia tốn CPU hơn, tuy nhiên lưu trữ và thời gian tốt hơn, restic sẽ tốn lượng storage cực lớn.
* Với lượng data tăng dần, kopia tốn CPU và max RAM usage hơn, tuy nhiên lưu trữ và thời gian tốt hơn. Kopia sẽ dễ OOM kill hơn.
* Khi lượng data trong mỗi file lớn (>=1GB), Restic lại ngốn CPU và thời gian lâu hơn, dễ bị timeout hơn, tuy nhiên max memory thấp hơn.

**Kết luận:**

* Kopia tốn ít thời gian hơn với cùng thông số kĩ thuật
* Trường hợp sao lưu lượng dữ liệu lớn hoặc các tệp nhỏ có dung lượng lớn thì Kopia vẫn tốt hơn
* Nên tính toán resource phân bổ trước để tránh trường hợp timeout hoặc OOM.

## Incremental backup

```bash
kubectl exec -n annd2 -it nginx bash
cd /usr/share/nginx/html
git clone https://github.com/torvalds/linux.git
```

Chắc chắn các tool về backup thì sẽ luôn có incremental backup. Nhưng khi tích hợp cùng với velero thì cần phải kiểm tra lại hiệu suất [tham khảo](https://github.com/gilbertchen/benchmarking). Cần đo các thông số (khoảng thời gian, sizes of the backup storage) sau các bước với restic và kopia như sau:

* Đầu tiên, clone source linux, tạo backup
* Checkout to commit 01/01/2022 `git checkout -f 60c332029c8da6f4ef791807fcbfbd98e71a5fbd`, tạo backup
* Checkout to commit 01/01/2024 `git checkout -f 77e01b49e35f24ebd1659096d5fc5c3b75975545`, tạo backup
* Checkout to commit 01/01/2023 `git checkout -f 5b24ac2dfd3eb3e36f794af3aa7f2828b19035bd`, tạo backup
* Checkout to commit 01/01/2013 `git checkout -f 1b8d52e63c53fd271846a56b7b1e3f622fd6a0a8`, tạo backup

| Data | kopia (time) | kopia (size) | restic (time) | restic (size) |
|:----:|:------------:|:------------:|:-------------:|:-------------:|
|  0   |     720      |              |      540      |     4.46      |
|  1   |      24      |     7.11     |      33       |     4.65      |
|  2   |      19      |     7.73     |      24       |     4.75      |
|  3   |      15      |     8.13     |      26       |     4.89      |
|  3   |      15      |     8.57     |      25       |     5.02      |
