# 🚀 INSTRUCTION.md – ToDo App Kubernetes Deployment Guide

## 📦 1. Deploying the App

### Step-by-step:

```bash
# 1. Create Namespace
kubectl apply -f .infrastructure/namespace.yml

# 2. Deploy Busybox Pod (for testing service DNS access)
kubectl apply -f .infrastructure/busybox.yml

# 3. Deploy Application
kubectl apply -f .infrastructure/deployment.yml

# 4. Create ClusterIP Service
kubectl apply -f .infrastructure/clusterIp.yml

# 5. Apply Horizontal Pod Autoscaler
kubectl apply -f .infrastructure/hpa.yml
```

### ⚙️ 2. Resource Requests and Limits — Explanation

requests: sets the minimum resources that the container is guaranteed to receive (idle/normal operation).

limits: sets the maximum resources that the container can consume.

Moderate values were chosen because the application is lightweight (Django + API), but traffic growth is possible.

### 3. HPA Configuration — Explanation

minReplicas = 2: to avoid downtime and ensure high availability.

maxReplicas = 5: taking into account the expected load of the application.

70% utilization: a balanced level at which scaling will occur on time, without being overly sensitive.

### 4. Strategy Configuration — Explanation
RollingUpdate: allows you to update pods without downtime.

maxUnavailable = 1: a maximum of 1 pod can be unavailable during the update — this ensures continuity.

maxSurge = 1: one more pod can be created temporarily — a balance between update speed and stability.

### 5. How to Access the Application

kubectl port-forward svc/todolist-service 8080:80 -n matepp
http://localhost:8080