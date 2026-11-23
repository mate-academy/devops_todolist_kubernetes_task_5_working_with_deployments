# TodoApp Kubernetes Deployment Guide

## 1. Overview

This document describes how to deploy the TodoApp application to a Kubernetes cluster using:

- Kubernetes Deployment
- RollingUpdate deployment strategy
- Resource requests and limits
- Horizontal Pod Autoscaler (HPA)
- Kubernetes Service

All application resources must be deployed in the `mateapp` namespace.

### Kubernetes manifests

The deployment consists of three manifests:

| File | Purpose |
|---|---|
| `deployment.yml` | Defines the application Deployment, replicas, container configuration, and resource allocation |
| `hpa.yml` | Defines automatic horizontal scaling based on CPU and memory utilization |
| `service.yml` | Exposes the application through a Kubernetes Service |

The Deployment and HPA must use the `mateapp` namespace.

---

## 2. Prerequisites

Before deploying the application, ensure that:

1. A Kubernetes cluster is running.
2. `kubectl` is installed and configured to communicate with the cluster.
3. The cluster has Metrics Server installed and operational.

Metrics Server is required because the HPA uses CPU and memory utilization metrics.

Verify cluster connectivity:

```bash
kubectl cluster-info
kubectl get nodes
```

Verify Metrics Server:

```bash
kubectl top nodes
```

If the command returns resource usage information, Metrics Server is working.

---

## 3. Create the Namespace

Create the namespace where all application resources will reside:

```bash
kubectl create namespace mateapp
```

Verify:

```bash
kubectl get namespaces
```

---

## 4. Deployment Configuration

The Deployment manages the application Pods and ensures that the desired number of replicas are running.

### Initial replica count

The Deployment specifies:

```yaml
replicas: 2
```

Two replicas are maintained during normal idle operation.

Running two Pods provides basic redundancy. If one Pod becomes unavailable, the Deployment controller attempts to maintain the desired number of available replicas.

The HPA also has a minimum replica count of two, ensuring that automatic scaling never reduces the application below this number.

### Resource requests and limits

The container uses the following resource configuration:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

**Resource requests**

Requests represent the resources Kubernetes reserves when scheduling a Pod.

- CPU: `250m` (0.25 CPU core).
- Memory: `64Mi` (64 mebibytes).

These values provide a reasonable initial allocation for a lightweight application that is expected to have relatively low resource consumption during idle operation.

The scheduler uses these values to determine whether a node has sufficient available resources to accommodate the Pod.

**Resource limits**

Limits define the maximum resources the container can consume.

- CPU: `500m` (0.5 CPU core).
- Memory: `128Mi` (128 mebibytes).

CPU consumption is throttled when the container reaches its CPU limit.

If the container exceeds its memory limit, it can be terminated with an OOMKilled status.

The limits provide resource isolation and prevent an individual application container from consuming excessive node resources.

These values are initial estimates and should be adjusted after observing actual resource consumption under normal and peak workloads.

---

## 5. RollingUpdate Strategy

The Deployment uses the RollingUpdate strategy:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

RollingUpdate allows Kubernetes to replace existing Pods gradually when a new application version is deployed.

This avoids terminating all existing Pods simultaneously.

**Configuration explanation**

- `maxUnavailable: 1` — Kubernetes can have at most one fewer available Pod than the desired replica count during the update.
- `maxSurge: 1` — Kubernetes can temporarily create one additional Pod above the desired replica count.

With two desired replicas, Kubernetes can temporarily run up to three Pods during an update.

This configuration provides a balance between availability and deployment speed.

The application has two replicas, so allowing one unavailable Pod ensures that Kubernetes can progressively replace the old instances while retaining an available instance, assuming the remaining Pod is healthy.

Readiness probes are important here because Kubernetes uses readiness status to determine whether a new Pod can receive traffic.

---

## 6. Horizontal Pod Autoscaler Configuration

The Horizontal Pod Autoscaler automatically adjusts the number of application replicas according to resource utilization.

The configuration uses:

```yaml
minReplicas: 2
maxReplicas: 5
```

### Minimum replicas: 2

The application always maintains at least two replicas.

This provides basic redundancy and avoids scaling down to a single application instance.

### Maximum replicas: 5

The application can scale up to five replicas when resource consumption increases.

This provides additional capacity during periods of higher demand while keeping the maximum resource consumption controlled.

### CPU and memory metrics

The HPA monitors two resource metrics:

**CPU utilization: 70%**

The HPA targets average CPU utilization of 70% relative to the CPU requests.

For example:

- CPU request: 250m.
- Target utilization: 70%.
- Target average consumption: 175m per Pod.

When average utilization exceeds the target, Kubernetes can increase the number of replicas.

**Memory utilization: 70%**

The HPA also targets average memory utilization of 70% relative to the memory requests.

For example:

- Memory request: 64Mi.
- Target utilization: 70%.
- Target average consumption: approximately 44.8Mi per Pod.

Monitoring both metrics allows the application to scale in response to either CPU-intensive or memory-intensive workloads.

The HPA evaluates the metrics and calculates the desired replica count. When multiple metrics are configured, Kubernetes selects the highest replica recommendation.

Therefore, if CPU utilization suggests three replicas but memory utilization suggests four, the HPA can scale to four.

Note that scaling is not instantaneous. Kubernetes uses stabilization windows and scaling policies to avoid unnecessary fluctuations.

---

## 7. Deploy the Application

From the directory containing the Kubernetes manifests, execute:

### Step 1: Create the namespace

```bash
kubectl create namespace mateapp
```

If the namespace already exists, this command will report that it already exists.

### Step 2: Deploy the application

```bash
kubectl apply -f deployment.yml
```

### Step 3: Create the Horizontal Pod Autoscaler

```bash
kubectl apply -f hpa.yml
```

### Step 4: Deploy the Service

```bash
kubectl apply -f service.yml
```

### Step 5: Verify deployment

Check the Deployment:

```bash
kubectl get deployments -n mateapp
```

Check the Pods:

```bash
kubectl get pods -n mateapp
```

Check the HPA:

```bash
kubectl get hpa -n mateapp
```

Check the Service:

```bash
kubectl get services -n mateapp
```

---

## 8. Accessing the Application

The application listens on container port `8000`.

A Kubernetes Service must expose the application to make it accessible.

For local development and testing, port forwarding can be used:

```bash
kubectl port-forward service/todoapp 80:8000 -n mateapp
```

After starting port forwarding, access the application through:

```text
http://localhost:8000
```

Port forwarding keeps the connection active only while the command is running.

For external access, an appropriate Service configuration such as NodePort or LoadBalancer, or an Ingress, must be configured depending on the cluster environment.

---

## 9. Monitoring Autoscaling

To observe CPU and memory consumption:

```bash
kubectl top pods -n mateapp
```

To monitor HPA decisions:

```bash
kubectl get hpa -n mateapp --watch
```

To inspect detailed autoscaling information:

```bash
kubectl describe hpa todoapp -n mateapp
```

To observe Deployment events:

```bash
kubectl describe deployment todoapp -n mateapp
```

During increased workload, the HPA can increase the number of replicas from two up to five.

When resource utilization decreases, the HPA can reduce the number of replicas, but never below two.

---

## 10. Updating the Application

When a new application image is available, update the image in `deployment.yml`.

For example:

```yaml
image: kulyk404/todoapp:3.1.0
```

Apply the updated manifest:

```bash
kubectl apply -f deployment.yml
```

Kubernetes will perform a RollingUpdate according to the configured strategy.

Monitor the update:

```bash
kubectl rollout status deployment/todoapp -n mateapp
```

If the new version causes problems, roll back:

```bash
kubectl rollout undo deployment/todoapp -n mateapp
```

---

## 11. Cleanup

To remove the application resources:

```bash
kubectl delete -f deployment.yml
kubectl delete -f hpa.yml
kubectl delete -f service.yml
```

To remove the namespace and all remaining resources inside it:

```bash
kubectl delete namespace mateapp
```

---

## 12. Expected Configuration

After deployment, the expected configuration is:

| Parameter | Configuration |
|---|---|
| Namespace | mateapp |
| Deployment | todoapp |
| Initial replicas | 2 |
| Minimum replicas | 2 |
| Maximum replicas | 5 |
| Deployment strategy | RollingUpdate |
| Max unavailable | 1 |
| Max surge | 1 |
| CPU request | 250m |
| CPU limit | 500m |
| Memory request | 64Mi |
| Memory limit | 128Mi |
| CPU scaling target | 70% |
| Memory scaling target | 70% |
| Container port | 8000 |

The application should normally run two replicas and automatically scale up to five replicas when CPU or memory utilization increases.