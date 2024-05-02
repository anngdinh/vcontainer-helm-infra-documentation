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
