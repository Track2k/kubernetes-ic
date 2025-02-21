# Kubernetes Assignment Solutions

This document contains the fixed YAML files for the Kubernetes assignments. These files address the issues present in the original faulty configurations [here](https://github.com/infracloudio/citadel-internal/blob/master/modules/kubernetes/intermediate/assignments/README.md).

## 1. Fixed YAML: `app-with-pvc-faulty-fix.yaml`

### Issues:
- Invalid storage class in the PersistentVolumeClaim (PVC), changed it to standard.
- PVC not properly bound due to storage class mismatch.

### Fixed YAML:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
  namespace: advanced-workload
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: "standard" # fixed storage class

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: advanced-workload
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
        - name: app-container
          image: nginx:latest
          volumeMounts:
            - name: app-storage
              mountPath: /usr/share/nginx/html
      volumes:
        - name: app-storage
          persistentVolumeClaim:
            claimName: app-pvc  # Fixed
```

## 2. Fixed YAML: `network-policy-faulty-fix.yaml`

### Issues:
- Database selector should match the correct labels for the database(mysql).
- NetworkPolicy incorrectly configured, preventing necessary communication between the web application and its database.

### Fixed YAML:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mysql-network-policy
  namespace: advanced-workload
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            name: advanced-workload
        podSelector:
          matchLabels:
            app: app
      ports:
        - protocol: TCP
          port: 3306
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: advanced-workload
      ports:
        - protocol: TCP
          port: 80
```

## 3. Fixed YAML: `mysql-statefulset-faulty-fix.yaml`

### Issues:
- Invalid storage class changed it to "standard" .
- Missing volume for MySQL data persistence.
- Incorrect container image version.

### Fixed YAML:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
  namespace: advanced-workload
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
        - name: mysql-container
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: rootpass
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-storage
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
        storageClassName: "standard"  # fixed storage class
```