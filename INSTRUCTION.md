# INSTRUCTION.md

## Application Deployment Guide for Kubernetes

This document explains how to deploy the **todoapp** application into a Kubernetes cluster. It also provides justification for Deployment configuration choices, resource requests/limits, update strategy, Horizontal Pod Autoscaler configuration, and instructions for accessing the application.

---

## 1. Prerequisites

Ensure you have the following:

* `kubectl` installed
* Access to a Kubernetes cluster
* Namespace `mateapp` created:

```bash
kubectl create namespace mateapp
```

---

## 2. Deployment Configuration and Justification

### 2.1 Number of Replicas

```yaml
replicas: 2
```

Two replicas are chosen to ensure:

* **High availability** — if one pod fails, the second continues handling traffic.
* **Zero-downtime deployments** during RollingUpdate strategy.

### 2.2 Resource Requests and Limits

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

**Justification:**

* **CPU request (100m):** The application has a low idle CPU footprint, allowing efficient scheduling.
* **Memory request (128Mi):** Provides guaranteed memory for stable operation.
* **CPU limit (500m):** Prevents the pod from consuming too much CPU during spikes.
* **Memory limit (256Mi):** Ensures memory usage remains under control, preventing node pressure.

These settings ensure efficient cluster resource usage while maintaining performance and stability.

### 2.3 RollingUpdate Strategy

```yaml
strategy:
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

**Explanation:**

* **maxUnavailable: 1** — Only one pod can be offline during updates, ensuring continuous availability.
* **maxSurge: 1** — Allows adding one extra pod during rollout to speed up deployment without overloading the cluster.

This strategy guarantees a safe, controlled update process with minimal disruption.

---

## 3. Horizontal Pod Autoscaler (HPA) Configuration and Reasoning

### HPA Settings

```yaml
minReplicas: 2
maxReplicas: 5
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

### Justification

* **minReplicas: 2** — Matches the Deployment baseline for redundancy.
* **maxReplicas: 5** — Provides enough headroom to handle peak loads.
* **CPU utilization @ 70%** — A standard threshold where scaling becomes beneficial.
* **Memory utilization @ 70%** — Ensures scaling when memory pressure increases, preventing slowdowns.

Using both CPU and memory ensures more accurate, balanced autoscaling.

---

## 4. How to Access the Application After Deployment

Once the Deployment, Service, and (optionally) Ingress are applied, you can access the application in one of two common ways:

### 4.1 Using a NodePort Service

Run:

```bash
kubectl get svc -n mateapp
```

Look for a `NodePort` value and access the application via:

```
http://<node-ip>:<node-port>
```

### 4.2 Using an Ingress

If you created an Ingress resource:

```
http://<your-domain>
```

Or retrieve the ingress endpoint:

```bash
kubectl get ingress -n mateapp
```

---

## 5. Applying the Manifests

Deploy the application using:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

---

## Deployment Complete

Your application is now deployed and accessible through the configured Service or Ingress.
