# Overview

## What is CNI?

![CNI](https://miro.medium.com/v2/resize:fit:1050/1*gYqKRBOFfhdZW6Li63kLaw.png)

CNI (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of supported plugins. CNI concerns itself only with network connectivity of containers and removing allocated resources when the container is deleted. Because of this focus, CNI has a wide range of support and the specification is simple to implement.

## The Kubernetes network model

- Every Pod in a cluster gets its own unique cluster-wide IP address.
- Kubernetes imposes the following fundamental requirements on any networking implementation
  - pods can communicate with all other pods on any other node without NAT
  - agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node

## Requirements for our CNI

- Kubernetes network model
- Security groups for pods

## GKE

<https://medium.com/cloudzone/gke-networking-options-explained-demonstrated-5c0253415eba>

GKE currently supports two network modes: routes-based and VPC-native. The network mode defines how traffic is routed:

- between Kubernetes pods within the same node,
- between nodes of the same cluster
- other network-enabled resources on the same VPC, such as virtual machines (VMs).

### Routes-Based Network Mode

![alt text](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*aMm9g2tDiAlBXg_K8CAmHQ.png)

- node has a single IP address from the subnet it is launched on (10.128.0.0/20)
- pod will also get an IP, however, is not registered in the VPC itself
- each node reserves a /24 IP address range that is unique within the VPC and not in use by any subnet (10.0.1.0/24, 10.0.2.0/24, ... ~ 254 IP/node)
- GKE then automatically creates a new route, using the node assigned /24 as the destination IP range and the node instance as the next hop.
- traffic between pods on different nodes is routed through the node’s IP address

Networking inside a single GKE Node
![alt text](https://miro.medium.com/v2/resize:fit:720/format:webp/1*gihqRa2G_PmIOUKld4g7-w.png)

- each pod has a fully isolated network from the node it runs on, using Linux network namespaces
- node connects to each of these eth0 interfaces on the pods using a virtual Ethernet (veth) device, which is always created as a pair between two network namespaces
- all veth interfaces on the node are connected together using a Layer 2 software-defined bridge **cbr0**

### VPC-Native Network Mode

- This network mode uses a nifty feature called alias IP ranges
- it is possible to have an IP range assigned to the same network interface
- These addresses can then be assigned to applications or containers running inside the VM without requiring additional network interfaces.
- it is possible to add one or more secondary CIDR ranges to a subnet and use subsets of them as alias IP ranges.
- uses separate secondary address ranges for pods and Kubernetes services
- **Alias IP ranges**
  - Google Cloud alias IP ranges let you assign ranges of internal IP addresses as aliases to a virtual machine's (VM) network interfaces
  - useful if you have multiple services running on a VM and you want to assign each service a different IP address. Alias IP ranges also work with GKE Pods.
  - You can also allocate alias IP ranges from that primary range, or you can add a secondary range to the subnet and allocate alias IP ranges from secondary range

![alt text](https://miro.medium.com/v2/resize:fit:720/format:webp/1*JUu9Zy_Wiz5EphjGJcIYvA.png)

- Each Node still reserves necessary IP range for ~ 110 pods per node, a /24 is required. But smaller is ok
- inside of a node — it is precisely the same as for clusters using the routes-based network mode
- no need to use the custom static routes created for each GKE Node

While this seems like a small difference between the two modes, it is **highly advantageous**.

- Pod IP addresses are natively routable within the cluster's VPC network and other VPC networks connected to it by VPC Network Peering.
- Pod IP addresses are reserved in the VPC network before the Pods are created in your cluster. This prevents conflict with other resources in the VPC network and allows you to better plan IP address allocations.
- Pod IP address ranges do not depend on custom static routes. They do not consume the system-generated and custom static routes quota. Instead, automatically-generated subnet routes handle routing for VPC-native clusters.
- You can create firewall rules that apply to just Pod IP address ranges instead of any IP address on the cluster's nodes.
- Pod IP address ranges, and subnet secondary IP address ranges in general, are accessible from on-premises networks connected with Cloud VPN or Cloud Interconnect using Cloud Routers.
- Some features, such as network endpoint groups (NEGs), only work with VPC-native clusters.

- no drawbacks to choosing a VPC-native cluster
- firewall rules can be applied to individual pods (for routes-based clusters, the finest granularity level would have been an entire node)
- Service (ClusterIP) addresses are only available from within the cluster. If you need to access within the cluster's VPC, create an internal Network Load Balancer.

## AWS EKS

[Amazon VPC Container Network Interface(VPC CNI)](https://github.com/aws/amazon-vpc-cni-k8s)

![CNI](https://aws.github.io/aws-eks-best-practices/networking/vpc-cni/image-3.png)

- allows Kubernetes Pods to have the same IP address as they do on the VPC network
- all containers inside the Pod share a network namespace, and they can communicate with each-other using local ports.
- has two components
  - CNI Binary, which will setup Pod network to enable Pod-to-Pod communication. The CNI binary runs on a node root file system and is invoked by the kubelet when a new Pod gets added to, or an existing Pod removed from the node.
  - ipamd, a long-running node-local IP Address Management (IPAM) daemon and is responsible
    - managing ENIs on a node
    - maintaining a warm-pool of available IP addresses or prefix

- When an instance is created, EC2 creates and attaches a primary ENI associated with a primary subnet. The Pods that run in hostNetwork mode use the primary IP address assigned to the node primary ENI and share the same network namespace as the host.

![eks-network](https://aws.github.io/aws-eks-best-practices/networking/index/image.png)
