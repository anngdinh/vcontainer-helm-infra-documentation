# Use nginx proxy in VKS

bài NLB(layer4) -> nginx-ingress(layer7) trong k8s -> pod backend là bài phổ biến note cho 1 docs về config realip xuyên suốt 3 lớp này

## Pre-requirement

- Already install VNGCloud Controller Manager
- Update to dev version of VNGCloud Controller Manager

  ```bash
  kubectl patch deployment -n kube-system vngcloud-controller-manager -p '{"spec": {"template": {"spec": {"containers": [{"name":"vngcloud-controller-manager","image":"vcr.vngcloud.vn/60108-annd2-ingress/vngcloud-controller-manager:v0.2.0"}]}}}}'
  kubectl patch deployment -n kube-system vngcloud-controller-manager -p '{"spec": {"template": {"spec": {"containers": [{"name":"vngcloud-controller-manager","image":"vcr.vngcloud.vn/81-vks-public/vngcloud-controller-manager:v0.2.0"}]}}}}'


  kubectl patch statefulsets -n kube-system vngcloud-ingress-controller -p '{"spec": {"template": {"spec": {"containers": [{"name":"vngcloud-ingress-controller","image":"vcr.vngcloud.vn/60108-annd2-ingress/vngcloud-ingress-controller:v0.2.0"}]}}}}'
  kubectl patch statefulsets -n kube-system vngcloud-ingress-controller -p '{"spec": {"template": {"spec": {"containers": [{"name":"vngcloud-ingress-controller","image":"vcr.vngcloud.vn/81-vks-public/vngcloud-ingress-controller:v0.2.0"}]}}}}'
  ```

- Install sample app to debug via Helm

  ```bash
  helm repo add anngdinh https://anngdinh.github.io/helm-charts
  helm install goapp anngdinh/goapp-debug
  ```

- Install nginx-ingress-controller via Helm

  ```bash
  helm install nginx-ingress-controller oci://ghcr.io/nginxinc/charts/nginx-ingress --namespace kube-system
  ```

## Update configmap for nginx-ingress-controller

```bash
kubectl edit cm -n kube-system nginx-ingress-controller
```

```yaml
data:
  proxy-protocol: "True"
  real-ip-header: proxy_protocol
  real-ip-recursive: "True"
  set-real-ip-from: 0.0.0.0/0
```

## Update service nginx-ingress-controller

```bash
kubectl annotate service -n kube-system nginx-ingress-controller-controller vks.vngcloud.vn/proxy-protocol="http,https"
```

## Apply sample ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: kkk.example.com
    http:
      paths:
      - backend:
          service:
            name: prometheus-node-exporter
            port:
              number: 9100
        path: /metrics
        pathType: Exact
```

## Check in log

```bash
kubectl logs $(kubectl get pods -n kube-system -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' | grep nginx-ingress-controller-controller) -n kube-system -f
```

Curl sample:

```bash
curl -H 'Host: kkk.example.com' http://_____IP_____/metrics
```
