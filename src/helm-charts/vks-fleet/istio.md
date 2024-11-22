# Preserve source IP address Istio Ingress gateway

## Reference

- [Getting started with Istio](https://istio.io/latest/docs/setup/getting-started)
- [Maintaining Traffic Transparency: Preserving Client Source IP in Istio](https://jimmysongio.medium.com/maintaining-traffic-transparency-preserving-client-source-ip-in-istio-69359fecd5f0)

## Setup test environment

### Install Istio

```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

helm install istio-base istio/base -n istio-system --create-namespace --wait

kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
{ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.1.0" | kubectl apply -f -; }


helm install istiod istio/istiod --namespace istio-system --set profile=ambient
helm install istio-ingressgateway istio/gateway -n istio-system
```

### Create test resources

Firstly, make sure namespace is labeled with `istio-injection=enabled`.

```bash
kubectl label namespace default istio-injection=enabled
```

Then, create debug pod and service. (NOTE: Should not use `nginx` pod because it has some [issue](https://medium.com/@syedhassaniiui/istio-common-issues-59340ae20241)).

```bash
kubectl create deployment echo-server --image=mccutchen/go-httpbin
kubectl expose deployment echo-server --name=clusterip --port=80 --target-port=8080 --type=ClusterIP
```

Create a service entry for the service.

```bash
cat > config.yaml <<EOF
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: clusterip-gateway
spec:
  selector:
    istio: ingressgateway # Choose the appropriate selector for your environment
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
      - "clusterip.jimmysong.io" # Replace with the desired hostname
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: clusterip-virtualservice
spec:
  hosts:
  - "clusterip.jimmysong.io" # Replace with the same hostname as in the Gateway
  gateways:
  - clusterip-gateway # Use the name of the Gateway here
  http:
  - route:
      - destination:
          host: clusterip.default.svc.cluster.local # Replace with the actual hostname of your Service
          port:
            number: 80 # Port of the Service

EOF
kubectl apply -f config.yaml
```

Open 3 terminal to watch log from `istio-ingressgateway` and `echo-server` pod.

```bash
kubectl -n istio-system logs -l app=istio-ingressgateway -f
```

```bash
kubectl logs -l app=echo-server -f -c istio-proxy
```

```bash
kubectl logs -l app=echo-server -f
```

Open one more terminal to send request to the service.

```bash
watch -n 1 "curl -H \"Host: clusterip.jimmysong.io\" ________________" # Replace with the IP of the Istio Ingress Gateway
```

Expected output:

- All IP addresses to 2 pods are the same (load balancer IP).

## Preserve source IP address

Firstly, enable `proxy-protocol` in LB to preserve source IP address.

```bash
kubectl annotate service -n istio-system istio-ingressgateway vks.vngcloud.vn/enable-proxy-protocol="*"
```

Then, update gateway to use `PROXY` protocol. Be careful, this command can **OVERWRITE** the current configuration.

```bash
kubectl patch deployment istio-ingressgateway -n istio-system -p '{"spec":{"template":{"metadata":{"annotations":{"proxy.istio.io/config":"{\"gatewayTopology\":{\"proxyProtocol\":{}}}"}}}}}'
```

Expected output:

- All IP addresses to 2 pods are same (source IP address of request).

## Clean up

```bash
kubectl delete deployment echo-server
kubectl delete service clusterip
kubectl delete -f config.yaml

helm -n istio-system uninstall istio-base istiod istio-ingressgateway
```
