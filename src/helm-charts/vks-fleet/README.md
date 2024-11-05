# VKS Fleet

## Multi-Cluster Services API

[Link](https://github.com/kubernetes/enhancements/tree/master/keps/sig-multicluster/1645-multi-cluster-services-api#release-signoff-checklist)

### Goals

- Consume a service in another cluster (or multiple clusters) as a single service.
- When a service is consumed from another cluster its behavior should be predictable and consistent with how it would be consumed within its own cluster.
- Allow gradual rollout of changes in a multi-cluster environment.
- Create building blocks for multi-cluster tooling.
- Support multiple implementations.
- Leave room for future extension and new use cases.

### Terminology

- **clusterset** - name for a group of clusters
  - namespace sameness
- **mcs-controller** - A controller that syncs services across clusters and makes them available for multi-cluster service discovery and connectivity.

### CRDs

- ServiceExport
  - specify which services should be exposed across all clusters in the clusterset
  - created in each cluster that the underlying Service resides in
- ServiceImport
  - managed by the MCS implementation's mcs-controller.
  - created in each cluster that wants to consume the service
  - contains the information needed to connect to all exported services

### User Stories

- Different ClusterIP Services Each Deployed to Separate Cluster
  - service frontend in cluster A and service backend in cluster B
  - service frontend wants to consume service backend via a single DNS name
  - if migrate service backend to cluster C, service frontend should not need to be updated
- Single Service Deployed to Multiple Clusters
  - have multiple stateless deploy backend services in different clusters in local, regional, global
  - fronend services in any clusters can access backend in priority order based on availability and locality.
  - Routing to service should optimize for cost metric (e.g. prioritize traffic local to zone, region).

### Design Details

```yaml
apiVersion: multicluster.k8s.io/v1alpha1
kind: ServiceExport
metadata:
  name: my-svc
  namespace: my-ns
status:
  conditions:
  - type: Ready
    status: "True"
    lastTransitionTime: "2020-03-30T01:33:51Z"
  - type: InvalidService
    status: "False"
    lastTransitionTime: "2020-03-30T01:33:55Z"
  - type: Conflict
    status: "True"
    lastTransitionTime: "2020-03-30T01:33:55Z"
    message: "Conflicting type. Using \"ClusterSetIP\" from oldest service export in \"cluster-1\". 2/5 clusters disagree."
```

- A `Service` without a corresponding `ServiceExport` in its local cluster will not be exported even if other clusters are exporting a `Service` with the same namespaced name.
- All information about the service, including ports, backends and topology, will continue to be stored in the Service objects, which are each name mapped to a ServiceExport.
- Deleting a ServiceExport will stop exporting the name-mapped Service
- Should use RBAC rules to prevent creation of `ServiceExports` in select namespaces (`default`, `kube-system`, etc.)

```yaml
apiVersion: multicluster.k8s.io/v1alpha1
kind: ServiceImport
metadata:
  name: my-svc
  namespace: my-ns
spec:
  ips:
  - 42.42.42.42
  type: "ClusterSetIP" # or "Headless"
  ports:
  - name: http
    protocol: TCP
    port: 80
  sessionAffinity: None
status:
  clusters:
  - cluster: us-west2-a-my-cluster
```

- All clusters containing the service's namespace will import the service by default.
  - all exporting clusters will also import the multi-cluster service
- If all `ServiceExport` instances are deleted, each `ServiceImport` will also be deleted from all clusters.
- The `ServiceImport.Spec.IP` (VIP) can be used to access this service from within this cluster.

#### ClusterSetIP

A non-headless ServiceImport is expected to have an associated IP address, the clusterset IP, which may be accessed from within an importing cluster. This IP may be a single IP used clusterset-wide or assigned on a per-cluster basis, but is expected to be consistent for the life of a ServiceImport from the perspective of the importing cluster. Requests to this IP from within a cluster will route to backends for the aggregated Service.

#### DNS

Optional, but recommended.

## GKE

### Multi Cluster Ingress (MCI)

- Deploying the app in multiple clusters
- Deploying `MultiClusterService` in config cluster

  ```yaml
  apiVersion: networking.gke.io/v1
  kind: MultiClusterService
  metadata:
    name: whereami-mcs
    namespace: whereami
  spec:
    template:
      spec:
        selector:
          app: whereami
        ports:
        - name: web
          protocol: TCP
          port: 8080
          targetPort: 8080
  ```

- `MultiClusterService` creates a derived **headless** `Service` in every cluster that matches.
- Why headless?
  - This service is not intended to be accessed directly by other services in the cluster.
  - pool member is pod IP
- Deploying `MultiClusterIngress` in config cluster

  ```yaml
  apiVersion: networking.gke.io/v1
  kind: MultiClusterIngress
  metadata:
    name: whereami-ingress
    namespace: whereami
  spec:
    template:
      spec:
        backend:
          serviceName: whereami-mcs
          servicePort: 8080
  ```

### Multi Cluster Services (MCS)

## EKS

[repo](https://github.com/aws/aws-cloud-map-mcs-controller-for-k8s)

## Cilium Cluster Mesh

### Service Global

[docs](https://docs.cilium.io/en/stable/network/clustermesh/clustermesh/)

- All clusters must be configured with the same datapath mode (Encapsulation or Native-Routing)
- PodCIDR ranges in all clusters and all nodes must be non-conflicting
- Nodes in all clusters must have IP connectivity between each other
- The network between clusters must allow the inter-cluster communication.

### Multi-Cluster Services API (Beta)

[link](https://docs.cilium.io/en/latest/network/clustermesh/mcsapi/)

- Installing CoreDNS multicluster
