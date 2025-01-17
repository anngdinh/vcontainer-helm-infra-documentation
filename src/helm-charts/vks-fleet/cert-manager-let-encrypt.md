# Nginx Ingress Controller + Cert-Manager + Let's Encrypt

- Prerequisites
  - Kubernetes cluster
  - A "real" domain
  - An email address
- Reference
  - [Cert-Manager](https://cert-manager.io/docs/tutorials/acme/nginx-ingress/)
- Notes
  - DO NOT create any ingress nginx

## Install Nginx Ingress Controller

After installing Nginx Ingress Controller, point your domain to the external IP of the Nginx Ingress Controller.

## Install Cert-Manager

```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.16.2 \
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
    email: ______________________                                           # Change to your email
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
      - http01:
          ingress:
            ingressClassName: nginx
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
  name: go-httpbin
  annotations:
    cert-manager.io/issuer: "letsencrypt-staging"
    acme.cert-manager.io/http01-edit-in-place: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - ______________________                   # Change to your domain
    secretName: quickstart-example-tls
  rules:
  - host: ______________________               # Change to your domain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: clusterip
            port:
              number: 80
```

Check certificate created:

```bash
kubectl get certificate

NAME                     READY   SECRET                   AGE
quickstart-example-tls   True    quickstart-example-tls   16m     # Ready should be True
```

Check certificate details:

```bash
kubectl describe certificate quickstart-example-tls

Name:         quickstart-example-tls
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
  Self Link:               /apis/cert-manager.io/v1/namespaces/default/certificates/quickstart-example-tls
  UID:                     68d43400-ea92-11e8-82f8-42010a8a00b5
Spec:
  Dns Names:
    www.example.com
  Issuer Ref:
    Kind:       Issuer
    Name:       letsencrypt-staging
  Secret Name:  quickstart-example-tls
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
kubectl delete secret quickstart-example-tls
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
    email: ______________________                                 # Change to your email
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            ingressClassName: nginx
```

Update Ingress annotation to use `letsencrypt-prod`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-httpbin
  annotations:
    cert-manager.io/issuer: "letsencrypt-prod"  # Change to letsencrypt-prod
    acme.cert-manager.io/http01-edit-in-place: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - ______________________                   # Change to your domain
    secretName: quickstart-example-tls
  rules:
  - host: ______________________               # Change to your domain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: clusterip
            port:
              number: 80
```
