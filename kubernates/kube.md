# Kubernetes Deployment for Camunda and MySQL

This guide outlines the process of deploying **Camunda** and **MySQL** using Kubernetes. Camunda will use MySQL for data persistence, and both services will be deployed as Docker containers in a Kubernetes environment.

## Table of Contents
1. [Understanding the Setup](#understanding-the-setup)
2. [MySQL Deployment Details](#mysql-deployment-details)
    1. [Persistent Volume Claim](#persistent-volume-claim)
    2. [MySQL Deployment](#mysql-deployment)
    3. [MySQL Service](#mysql-service)
3. [Camunda Deployment Details](#camunda-deployment-details)
    1. [Camunda Deployment](#camunda-deployment)
    2. [Camunda Service](#camunda-service)
4. [Applying Kubernetes Configuration](#applying-kubernetes-configuration)
    1. [Step 1: Apply MySQL Resources](#step-1-apply-mysql-resources)
    2. [Step 2: Apply Camunda Resources](#step-2-apply-camunda-resources)
5. [Validating the Setup](#validating-the-setup)
6. [Expose Camunda Publicly](#expose-camunda-publicly)
7. [Debugging Tips](#debugging-tips)
8. [Reference Document](#reference-document)

## Understanding the Setup

We are deploying two services:
1. **MySQL**: A relational database to store Camunda process-related data.
2. **Camunda**: A process automation engine connecting to MySQL for persistence.

**Technologies Used:**
- **Kubernetes**: Orchestrates containerized services.
- **Docker**: Provides container images for MySQL and Camunda.
- **Persistent Volumes**: Ensures MySQL data persists even if the pod is restarted.
- **Networking**: Allows communication between Camunda and MySQL via Kubernetes services.

## MySQL Deployment Details

### Persistent Volume Claim

MySQL requires persistent storage to ensure data is not lost if the pod restarts. A **PersistentVolumeClaim** (PVC) is created to request storage from the Kubernetes cluster.

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



### MySQL Deployment

The **MySQL Deployment** specifies how to run the MySQL container, configure its environment variables, and attach the persistent volume.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
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
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: rootpass
            - name: MYSQL_DATABASE
              value: camunda
            - name: MYSQL_USER
              value: camunda_user
            - name: MYSQL_PASSWORD
              value: camunda_pass
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql-pvc
```

### MySQL Service

A **Service** is required to expose the MySQL deployment within the Kubernetes cluster so Camunda can connect to it.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysql
  type: ClusterIP
```

## Camunda Deployment Details

### Camunda Deployment

The **Camunda Deployment** specifies how to run the Camunda container and configure it to connect to MySQL.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: camunda
  labels:
    app: camunda
spec:
  replicas: 1
  selector:
    matchLabels:
      app: camunda
  template:
    metadata:
      labels:
        app: camunda
    spec:
      containers:
        - name: camunda
          image: sinugaud/camunda-mysql-app:latest
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_DATASOURCE_DRIVERCLASSNAME
              value: com.mysql.cj.jdbc.Driver
            - name: SPRING_DATASOURCE_URL
              value: jdbc:mysql://mysql:3306/camunda?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
            - name: SPRING_DATASOURCE_USERNAME
              value: camunda_user
            - name: SPRING_DATASOURCE_PASSWORD
              value: camunda_pass
          readinessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
```

### Camunda Service

A **Service** exposes Camunda within the cluster or externally to users.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: camunda
  labels:
    app: camunda
spec:
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    app: camunda
  type: ClusterIP
```

## Applying Kubernetes Configuration

### Step 1: Apply MySQL Resources
1. Save the YAML files (`mysql-pvc.yaml`, `mysql-deployment.yaml`, and `mysql-service.yaml`).
2. Apply them using:
   ```bash
   kubectl apply -f mysql-pvc.yaml
   kubectl apply -f mysql-deployment.yaml
   kubectl apply -f mysql-service.yaml
   ```

### Step 2: Apply Camunda Resources
1. Save the YAML files (`camunda-deployment.yaml` and `camunda-service.yaml`).
2. Apply them using:
   ```bash
   kubectl apply -f camunda-deployment.yaml
   kubectl apply -f camunda-service.yaml
   ```

## Validating the Setup

### Check Pods
```bash
kubectl get pods
```
Ensure both MySQL and Camunda pods are running.

### Check Services
```bash
kubectl get services
```
Verify that `mysql` and `camunda` services are created.

### Access Camunda
If using `kubectl port-forward`, you can expose Camunda locally:
```bash
kubectl port-forward service/camunda 8080:8080
```
Access the Camunda Web UI at [http://localhost:8080](http://localhost:8080).

## Expose Camunda Publicly
If you want Camunda to be accessible outside the cluster, modify its service to `LoadBalancer`:
```yaml
spec:
  type: LoadBalancer
```
This provides an external IP (if supported by your Kubernetes provider).

## Debugging Tips
- **Check Pod Logs**:
  ```bash
  kubectl logs <pod-name>
  ```
- **Describe Resources**:
  ```bash
  kubectl describe pod <pod-name>
  ```
- **Test MySQL Connectivity**:
  ```bash
  kubectl exec -it <mysql-pod-name> -- mysql -u camunda_user -p
  ```

## Reference Document
For additional information and reference, check the [Camunda Deployment Documentation](https://docs.google.com/document/d/1i4_jEBXxVAc6A57c-6XypNfHzABPh9QkRg4qmzXEBg4/edit?usp=sharing).
```

---
