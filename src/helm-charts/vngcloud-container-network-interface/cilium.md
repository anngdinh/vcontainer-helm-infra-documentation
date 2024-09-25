# How to check cilium cluster health

## Prerequisites

- Install [Cilium CLI](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli)
- Allow all traffic in security group if you want to do test End-To-End Connectivity Testing.

## Check status

You can check the status of Cilium cluster by running the following command:

```bash
cilium status --wait
```

```bash
    /¯¯\
 /¯¯\__/¯¯\    Cilium:             OK
 \__/¯¯\__/    Operator:           OK
 /¯¯\__/¯¯\    Envoy DaemonSet:    OK
 \__/¯¯\__/    Hubble Relay:       OK
    \__/       ClusterMesh:        disabled

Deployment             hubble-relay       Desired: 1, Ready: 1/1, Available: 1/1
Deployment             hubble-ui          Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet              cilium-envoy       Desired: 2, Ready: 2/2, Available: 2/2
Deployment             cilium-operator    Desired: 2, Ready: 2/2, Available: 2/2
DaemonSet              cilium             Desired: 2, Ready: 2/2, Available: 2/2
Containers:            cilium-operator    Running: 2
                       cilium             Running: 2
                       hubble-relay       Running: 1
                       hubble-ui          Running: 1
                       cilium-envoy       Running: 2
Cluster Pods:          10/10 managed by Cilium
Helm chart version:    
Image versions         cilium-operator    vcr.vngcloud.vn/60108-annd2-ingress/cilium/operator-generic:latest: 2
                       cilium             vcr.vngcloud.vn/81-vks-public/cilium/cilium:v1.16.1: 2
                       hubble-relay       vcr.vngcloud.vn/81-vks-public/cilium/hubble-relay:v1.16.1: 1
                       hubble-ui          vcr.vngcloud.vn/81-vks-public/cilium/hubble-ui-backend:v0.13.1: 1
                       hubble-ui          vcr.vngcloud.vn/81-vks-public/cilium/hubble-ui:v0.13.1: 1
                       cilium-envoy       vcr.vngcloud.vn/81-vks-public/cilium/cilium-envoy:v1.29.7-39a2a56bbd5b3a591f69dbca51d3e30ef97e0e51: 2
```

## Check connectivity

First, check connectivity between nodes. You should see total number of nodes and the result of ICMP and HTTP connectivity between nodes.

```bash
kubectl -n kube-system exec ds/cilium -- cilium-health status --probe

Probe time:   2024-09-25T08:21:22Z
Nodes:
  vks-annd2-rua-thau-1c308 (localhost):
    Host connectivity to 10.111.0.3:
      ICMP to stack:   OK, RTT=219.002µs
      HTTP to agent:   OK, RTT=243.964µs
    Endpoint connectivity to 10.111.128.137:
      ICMP to stack:   OK, RTT=232.362µs
      HTTP to agent:   OK, RTT=324.762µs
  vks-annd2-rua-thau-60c0f:
    Host connectivity to 10.111.0.8:
      ICMP to stack:   OK, RTT=1.173948ms
      HTTP to agent:   OK, RTT=781.549µs
    Endpoint connectivity to 10.111.129.125:
      ICMP to stack:   OK, RTT=1.277224ms
      HTTP to agent:   OK, RTT=584.425µs
```

## End-To-End Connectivity Testing

Moreover, you can run [Cilium End-To-End Connectivity Testing](https://docs.cilium.io/en/stable/contributing/testing/e2e/).

```bash
cilium connectivity test
...
✅ All 32 tests (263 actions) successful, 2 tests skipped, 1 scenarios skipped.
```

## Network performance test

Cilium provides a [network performance test](https://docs.cilium.io/en/stable/contributing/testing/e2e/#network-performance-test) tool to measure the network performance of the cluster. You can run the following command to test the network performance:

```bash
cilium connectivity perf
...
[=] Test [network-perf] [1/1]
...
```

## Troubleshooting

### Cilium agent is not running

If cilium agent is not running or `0/1     Running`, you can check status of CiliumNode. Its show node name, podCIDR and IPAM operator status.

```bash
kubectl get ciliumnodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.ipam.podCIDRs[0]}{"\t\t"}{.status.ipam.operator-status}{"\n"}{end}'

vks-annd2-rua-thau-1c308        10.111.128.0/24         {}
vks-annd2-rua-thau-60c0f        10.111.129.0/24         {}
```

There is some common issues:

- `{"error":"waiting for provider to allocate CIDR"}`: The IPAM provider is not ready to allocate CIDR. You should check the Event history in portal.

## Limitations
