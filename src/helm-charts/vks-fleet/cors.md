# CORS

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-header-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: custom-header-app
  template:
    metadata:
      labels:
        app: custom-header-app
    spec:
      containers:
      - name: flask-app
        image: python:3.9-slim
        ports:
        - containerPort: 8000
        command: ["/bin/sh", "-c"]
        args:
          - |
            pip install flask &&
            echo '
            from flask import Flask, make_response
            app = Flask(__name__)
            @app.route("/")
            def hello():
                resp = make_response("Custom Headers Working!")
                resp.headers["X-Custom-Header"] = "MyValue"
                resp.headers["X-Server-Name"] = "custom-header-app"
                resp.headers["Access-Control-Allow-Origin"] = "*"  # For CORS
                return resp
            app.run(host="0.0.0.0", port=8000)
            ' > app.py &&
            python app.py
---
apiVersion: v1
kind: Service
metadata:
  name: custom-header-service
spec:
  selector:
    app: custom-header-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: NodePort
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    vks.vngcloud.vn/enable-autoscale: "true"
  name: custom-header-ingress
spec:
  ingressClassName: vngcloud
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: custom-header-service
            port:
              number: 80
```