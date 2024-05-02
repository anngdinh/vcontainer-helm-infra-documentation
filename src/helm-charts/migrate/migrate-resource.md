# Migrating Resources in a Cluster (Velero)

## Application Scenarios

## Prerequisites

## Backing Up Applications in the Source Cluster

1. (Optional) To back up the data of a specified storage volume in the pod, add an annotation to the pod. The annotation template is as follows:

    ```bash
    kubectl -n <namespace> annotate pod/<pod_name> backup.velero.io/backup-volumes=<volume_name_1>,<volume_name_2>,...
    # Eg: kubectl annotate pod/wordpress-758fbf6fc7-s7fsr backup.velero.io/backup-volumes=wp-storage
    # Eg: kubectl annotate pod/mysql-5ffdfbc498-c45lh backup.velero.io/backup-volumes=mysql-storage

    ```

2. Back up the application.

    You can create default backup with all resource or specific more by adding these flags. See more [Resource filtering](https://velero.io/docs/v1.13/resource-filtering/)

    * `--include-namespaces`: backs up resources in a specified namespace.

        ```bash
        velero backup create <backup-name> --include-namespaces <namespace>
        ```

    * `--include-resources`: backs up the specified resources.

        ```bash
        velero backup create <backup-name> --include-resources deployments
        ```

    * `--selector`: backs up resources that match the selector.

        ```bash
        velero backup create <backup-name> --selector <key>=<value>
        ```

    If the following information is displayed, the backup task is successfully created:

    ```bash
    Backup request "wordpress-backup" submitted successfully. Run `velero backup describe wordpress-backup` or `velero backup logs wordpress-backup` for more details.
    ```

3. Check the backup status.

    ```bash
    $ velero backup get
    NAME               STATUS      ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
    wordpress-backup   Completed   0        0          2021-10-14 15:32:07 +0800 CST   29d       default            <none>
    ```

    In addition, you can go to the object bucket to view the backup files. The backups path is the application resource backup path, and the restic path is the PV data backup path.

## Restoring Applications in the Target Cluster

1. (Optional) Changing PV/PVC Storage Classes
    Velero can change the storage class of persistent volumes and persistent volume claims during restores. To configure a storage class mapping, create a config map in the Velero namespace like the following:

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
      <old-storage-class>: <new-storage-class>
      <old-storage-class>: <new-storage-class>
    ```

2. (Optional) Changing Pod/Deployment/StatefulSet/DaemonSet/ReplicaSet/ReplicationController/Job/CronJob Image Repositories [docs](https://velero.io/docs/v1.13/restore-reference/)

3. Use the Velero tool to create a restore and specify a backup named wordpress-backup to restore

    ```bash
    velero restore create --from-backup wordpress-backup
    ```

    You can run the `velero restore get` statement to view the application restoration status.

4. After the restoration is complete, check whether the application is running properly.
