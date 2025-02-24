# Lab 2: Deployment vs. StatefulSet

## Objective
This lab demonstrates the difference between **Deployments** and **StatefulSets**, followed by creating a **MySQL StatefulSet** and a **Service** to access it.

## Prerequisites
- A running Kubernetes cluster.
- `kubectl` installed and configured.
- Basic knowledge of Kubernetes resources (Pods, Deployments, Services, and StatefulSets).

---

## 1. Deployment vs. StatefulSet: Key Differences

| Feature          | Deployment | StatefulSet |
|-----------------|------------|-------------|
| Pod Identity   | Identical pods (random names) | Unique, stable pod identities (ordered names) |
| Scaling        | Stateless, all pods are the same | Ordered scaling and termination |
| Storage       | Ephemeral by default | Persistent storage using PersistentVolumeClaims (PVCs) |
| Use Case      | Stateless apps like web servers | Stateful apps like databases |

**Conclusion:** Use **Deployments** for **stateless** applications and **StatefulSets** for **stateful** applications like MySQL, which require persistent storage and stable network identities.

---

## 2. Creating a MySQL StatefulSet

### 2.1. **Storage Class (Optional: If dynamic provisioning is required)**
Create a **StorageClass** to provision persistent volumes dynamically:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mysql-storage-class
provisioner: kubernetes.io/aws-ebs  # Change based on your cloud provider
```
Apply it:
```bash
kubectl apply -f mysql-storage-class.yaml
```

### 2.2. **Persistent Volume Claim (PVC) for MySQL**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
Apply it:
```bash
kubectl apply -f mysql-pvc.yaml
```

### 2.3. **StatefulSet for MySQL**
Create `mysql-statefulset.yaml`:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```
Apply it:
```bash
kubectl apply -f mysql-statefulset.yaml
```

Verify the StatefulSet:
```bash
kubectl get statefulsets
kubectl get pods -l app=mysql
```

---

## 3. Exposing MySQL with a Service

### 3.1. **Headless Service for MySQL**
Create `mysql-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysql
  clusterIP: None  # Headless service for direct pod access
```
Apply it:
```bash
kubectl apply -f mysql-service.yaml
```

Verify the service:
```bash
kubectl get services mysql
```

### 3.2. **Connect to MySQL from Another Pod**
```bash
kubectl run mysql-client --image=mysql:latest -it --rm -- bash
mysql -h mysql-0.mysql -uroot -p
```
---

**Congratulations!** You have successfully deployed a **MySQL StatefulSet** and exposed it with a **headless service** in Kubernetes. 🎉

