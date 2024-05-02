# Solution Overview

Before the migration, analyze all resources in the source clusters and then determine the migration solution. Resources that can be migrated include resources inside and outside the clusters.

These are resources that can be migrated:

* **Resources inside a cluster:**
  * **All objects in a cluster, including pods, jobs, Services, Deployments, and ConfigMaps.** You are not advised to migrate the resources in the velero and kube-system namespaces.
  * **PersistentVolumes (PVs) mounted to containers:** Due to restrictions of the Restic tool, migration is not supported for the hostPath storage volume. For details about how to solve the problem, see Storage Volumes of the HostPath Type Cannot Be Backed Up.
* Resources outside a cluster
  * **On-premises image repository:** You can migrate to vCR.
  * **Non-containerized database:** You can migrate to vDB.
  * **Non-local storage, such as object storage:** You can migrate to vStorage

## Migration Process

The cluster migration process is as follows:

1. **Plan resources for the target cluster.**

2. **Migrate resources outside a cluster.**
3. **Install the migration tool.**
4. **Migrate resources in the cluster.**
    * Backing Up Applications in the Source Cluster
    * Restoring Applications in the Target Cluster
5. **Update resources accordingly.** After the migration, cluster resources may fail to be deployed. Update the faulty resources. The possible adaptation problems are as follows:
    * Updating Images
    * Updating Services
    * Updating the Storage Class
    * Updating Databases

6. **Perform additional tasks.** After cluster resources are properly deployed, verify application functions after the migration and switch service traffic to the target cluster. After confirming that all services are running properly, bring the source cluster offline.
