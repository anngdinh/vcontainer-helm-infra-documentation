# VNGCLOUD Ingress Controller + Cert-Manager + Let's Encrypt

- Prerequisites
  - Kubernetes cluster
  - A "real" domain. In my case, it's `stag.annd2.pro` and `pro.annd2.pro`
  - An email address
- Reference
  - [Cert-Manager](https://cert-manager.io/docs/tutorials/acme/nginx-ingress/)
- Notes
  - **DO NOT** create any ingress "vngcloud"

## Install Cert-Manager

```bash
helm repo add jetstack https://charts.jetstack.io --force-update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.17.0 \
  --set crds.enabled=true
```

## Let's Rock with Let's Encrypt

Deploy foo app

```bash
kubectl create deployment echo-server --image=mccutchen/go-httpbin
kubectl expose deployment echo-server --name=clusterip --port=80 --target-port=8080 --type=ClusterIP
```

### Test with Staging

Create Issuer, change `email` in this yaml file.

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: example@gmail.com # Change to your email
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
      - http01:
          ingress:
            ingressClassName: vngcloud
```

Check Issuer created:

```bash
kubectl describe issuer letsencrypt-staging
```

It's conditions should look like this:

```yaml
Status:
  Acme:
    Uri:  https://acme-staging-v02.api.letsencrypt.org/acme/acct/7374163
  Conditions:
    Last Transition Time:  ...
    Message:               The ACME account was registered with the ACME server
    Reason:                ACMEAccountRegistered
    Status:                True
    Type:                  Ready
```

Deploy ingress, change domain in this yaml file.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: stag.annd2.pro
  annotations:
    cert-manager.io/issuer: "letsencrypt-staging"
    acme.cert-manager.io/http01-edit-in-place: "true"
    vks.vngcloud.vn/implementation-specific-params: '[{"host":"","path":"/","rules":[{"type":"PATH","compare":"STARTS_WITH","value":"/"}],"action":{"action":"REDIRECT_TO_URL","redirectUrl":"https://stag.annd2.pro","redirectHttpCode":301,"keepQueryString":true}}]'
spec:
  ingressClassName: vngcloud
  tls:
  - hosts:
    - stag.annd2.pro                   # Change to your domain
    secretName: secret-stag.annd2.pro
  rules:
  - host: stag.annd2.pro               # Change to your domain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: clusterip
            port:
              number: 80
  - http: # this rule redirect to https, please remove after challenge is completed
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: clusterip
            port:
              number: 80
```

After ingress created, update DNS record to point to external IP. Wait for a while to challenge to be completed. When challenge is taking place, a new path like `/.well-known/acme-challenge/______` will be created in your ingress. After challenge is completed, the path will be removed and the ingress will be updated with a new secret. Please remove the redirect to https part and annotation `vks.vngcloud.vn/implementation-specific-params` in the ingress after challenge is completed.

Check certificate created:

```bash
kubectl get certificate

NAME                     READY   SECRET                   AGE
secret-stag.annd2.pro   True    secret-stag.annd2.pro   16m     # Ready should be True
```

Check certificate details:

```bash
kubectl describe certificate secret-stag.annd2.pro

Name:         secret-stag.annd2.pro
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  cert-manager.io/v1
Kind:         Certificate
Metadata:
  Cluster Name:
  Creation Timestamp:  2018-11-17T17:58:37Z
  Generation:          0
  Owner References:
    API Version:           networking.k8s.io/v1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  Ingress
    Name:                  kuard
    UID:                   a3e9f935-ea87-11e8-82f8-42010a8a00b5
  Resource Version:        9295
  Self Link:               /apis/cert-manager.io/v1/namespaces/default/certificates/secret-stag.annd2.pro
  UID:                     68d43400-ea92-11e8-82f8-42010a8a00b5
Spec:
  Dns Names:
    www.example.com
  Issuer Ref:
    Kind:       Issuer
    Name:       letsencrypt-staging
  Secret Name:  secret-stag.annd2.pro
Status:
  Acme:
    Order:
      URL:  https://acme-staging-v02.api.letsencrypt.org/acme/order/7374163/13665676
  Conditions:
    Last Transition Time:  2018-11-17T18:05:57Z
    Message:               Certificate issued successfully
    Reason:                CertIssued
    Status:                True
    Type:                  Ready
Events:
  Type     Reason          Age                From          Message
  ----     ------          ----               ----          -------
  Normal   CreateOrder     9m                 cert-manager  Created new ACME order, attempting validation...
  Normal   DomainVerified  8m                 cert-manager  Domain "www.example.com" verified with "http-01" validation
  Normal   IssueCert       8m                 cert-manager  Issuing certificate...
  Normal   CertObtained    7m                 cert-manager  Obtained certificate from ACME server
  Normal   CertIssued      7m                 cert-manager  Certificate issued Successfully
```

Try curl to your domain, it'll return cert are self-signed. It's OKAY because it's staging.

```bash
curl -kivL -H 'Host: ______DOMAIN______' 'http://_____IP_____'
```

Delete resources:
  
```bash
kubectl delete ingress go-httpbin
kubectl delete issuer letsencrypt-staging
kubectl delete secret secret-stag.annd2.pro
kubectl delete secret letsencrypt-staging
```

### Test with Production

Create Production Issuer, change `email` in this yaml file.

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: example@gmail.com                                 # Change to your email
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            ingressClassName: vngcloud
```

Update Ingress annotation to use `letsencrypt-prod`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pro.annd2.pro
  annotations:
    cert-manager.io/issuer: "letsencrypt-prod"  # Change to letsencrypt-prod
    acme.cert-manager.io/http01-edit-in-place: "true"
    vks.vngcloud.vn/implementation-specific-params: '[{"host":"","path":"/","rules":[{"type":"PATH","compare":"STARTS_WITH","value":"/"}],"action":{"action":"REDIRECT_TO_URL","redirectUrl":"https://pro.annd2.pro","redirectHttpCode":301,"keepQueryString":true}}]'
spec:
  ingressClassName: vngcloud
  tls:
  - hosts:
    - pro.annd2.pro                   # Change to your domain
    secretName: secret-pro.annd2.pro
  rules:
  - host: pro.annd2.pro               # Change to your domain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: clusterip
            port:
              number: 80
  - http:
      paths:
      - path: /
        pathType: ImplementationSpecific
        backend:
          service:
            name: clusterip
            port:
              number: 80
```
