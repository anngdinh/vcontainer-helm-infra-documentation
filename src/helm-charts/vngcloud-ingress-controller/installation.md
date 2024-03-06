<div style="float: right;"><img src="../../images/01.png" width="160px" /></div><br>

# Installation

## Prepare the Service Account key pair

The **Vngcloud Ingress Controllers** plugin necessitates a **Service Account** for executing operations on load-balancer and vserver resources. Users can establish a **Service Account** by accessing [the IAM dashboard](https://hcm-3.console.vngcloud.vn/iam/service-accounts). These policy is required:

* vserver
  * GetCluster
* vlb
  * ListLoadBalancers
  * ListLoadBalancerPools
  * ListLoadBalancerBySubnet
  * ListLoadBalancerListeners
  * ListLoadBalancerMembers
  * ListLoadBalancerL7Policy
  * ListCertificateAuthority
  * GetLoadBalancer
  * GetLoadBalancerListener
  * GetLoadBalancerL7Policy
  * GetCertificateAuthority
  * GetLoadBalancerPool
  * CreateLoadBalancer
  * CreateLoadBalancerListener
  * UpdateLoadBalancerListener
  * UpdateLoadBalancerL7Policy
  * CreateLoadBalancerL7Policy
  * DeleteLoadBalancerListener
  * DeleteLoadBalancerL7Policy
  * CreateLoadBalancerPool
  * UpdateListLoadBalancerMembers
  * DeleteLoadBalancerPool
  * UpdateLoadBalancerPool
  * DeleteCertificateAuthority
  * ImportCertificateAuthority

## Install the `vngcloud-ingress-controller` chart

* First, add this repo:

  ```bash
  helm repo add vks-helm-charts https://vngcloud.github.io/vks-helm-charts
  helm repo update
  ```

* Get `cluster-id` from dashboard

* Install

  ```bash=
  helm install vngcloud-ingress-controller vks-helm-charts/vngcloud-ingress-controller \
    --set cloudConfig.global.clientID=__________________________ \
    --set cloudConfig.global.clientSecret=__________________________ \
    --set cloudConfig.clusterID=__________________________
  ```

## Verify the installation

After the installation is complete, execute the following command to verify the status of the `vngcloud-ingress-controller` pods:

```bash=
kubectl get pods -n kube-system | grep vngcloud-ingress-controller
```

User can get the log of the plugin by executing the following command:

```bash=
kubectl logs -n kube-system vngcloud-ingress-controller-0 -f
```

# Upgrade the `vngcloud-ingress-controller` chart to latest version

If you followed the instructions in the [Install the vngcloud-ingress-controller chart](#install-the-vngcloud-ingress-controller-chart) section to install the `vngcloud-ingress-controller` chart, execute the following command to upgrade the chart:

```bash=
helm upgrade vngcloud-ingress-controller anngdinh/vngcloud-ingress-controller
```

# Uninstallation

If you followed the instructions in the [Install the vngcloud-ingress-controller chart](#install-the-vngcloud-ingress-controller-chart) section to install the `vngcloud-ingress-controller` chart, execute the following command to uninstall the chart:

```bash=
helm uninstall vngcloud-ingress-controller
```
