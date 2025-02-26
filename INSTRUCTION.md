# INSTRUCTION.md

## Description

This instruction contains the steps for deploying the **mateapp** application on Kubernetes using the `deployment.yml` and `hpa.yml` manifests. It also explains the strategy, resource settings, and HPA configuration, as well as how to access the application after deployment.

## Steps to Deploy the Application on Kubernetes

### 1. Creating Manifest Files

#### 1.1 Create `deployment.yml` File

This file configures the deployment of the **mateapp** application on Kubernetes, including the **RollingUpdate** strategy, resource requests, and limits.

**`deployment.yml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mateapp-deployment
  namespace: mateapp
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: mateapp
    spec:
      containers:
        - name: mateapp
          image: mateapp:latest
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
