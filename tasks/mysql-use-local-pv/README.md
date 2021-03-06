# MySQL db server uses persistent volume for database

# Create a local directory on a worker node
```sudo mkdir /home/pv1```
```sudo chmod 777 /home/pv1```
# Create StorageClass, PersistentVolume and PersistentVolumeClaim
```
cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage-mysql-demo
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-k8s-worker-1-pv1
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage-mysql-demo
  local:
    path: /home/pv1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-worker-1
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc1-for-mysql
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage-mysql-demo
  resources:
    requests:
      storage: 1Gi
EOF
```
# Create MySQL deployment and service
```
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-dep
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      hostname: myssqlhostname
      containers:
      - name: mysql
        image: mysql
        ports:
        - containerPort: 3306
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: storage
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: my-secret-pw
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: pvc1-for-mysql
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql
  name: mysql-service
spec:
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    app: mysql
status:
  loadBalancer: {}
EOF
```
