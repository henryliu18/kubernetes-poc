# Deploy Tomcat webserver with Nginx-ingress
## Deployment of Tomcat server 2 replicas, Service on port 80 -> containerPort 8080 and Ingress rules of host to backend service
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: tomcat
  name: tomcat-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      run: tomcat
  template:
    metadata:
      labels:
        run: tomcat
    spec:
      containers:
      - image: tomcat:alpine
        name: tomcat
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    run: tomcat
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tomcat-ingress
spec:
  rules:
  - host: tomcat.example.com
    http:
      paths:
      - backend:
          serviceName: tomcat-service
          servicePort: 80
EOF
```
## Status
```kubectl get pod,svc,ing -o wide```

## Clean up
```bash
kubectl delete ing tomcat-ingress
kubectl delete svc tomcat-service
kubectl delete deployment tomcat-demo
```
