# Deployment Instructions for Django ToDo App

This document provides step-by-step instructions on how to deploy the Django ToDo application to a Kubernetes cluster and explains the architectural choices made for resources, autoscaling, and update strategies.

---

## How to Deploy the Application

### 1. Prerequisites
Ensure you have access to a Kubernetes cluster (e.g., `kind`, `minikube`) and `kubectl` CLI is configured.

### 2. Apply Manifests
Apply the Deployment and Horizontal Pod Autoscaler manifests using the following commands:

```
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```
### 3. Verify the Deployment
Check the status of the pods and the autoscaler:

```
kubectl get pods -n todoapp
kubectl get hpa -n todoapp
```
## Architectural Choices and Justifications
### 1. Resource Requests and Limits
Requests (CPU: 100m, Memory: 128Mi): Django is a synchronous WSGI/ASGI framework. For a basic ToDo application in an idle state, 128Mi of RAM is more than enough to load the Python runtime, Django core, and cache schemas. 100m CPU (0.1 core) ensures the pod guarantees smooth handling of background health checks and low-intensity traffic.

Limits (CPU: 500m, Memory: 256Mi): Memory limits are set to 256Mi to prevent potential memory leaks from crashing the node (OOMKilled will trigger only for the specific misbehaving pod). CPU is throttled at 500m to allow the app to burst during heavy DB queries or intensive API rendering without starving adjacent containers on the same node.

### 2. Horizontal Pod Autoscaler (HPA) Configuration
Min Replicas: 2 / Max Replicas: 5: Setting the minimum to 2 ensures High Availability (HA). If one node or pod fails, the application remains online. The maximum is capped at 5 to protect cluster infrastructure from resource exhaustion (e.g., during a DDoS attack).

Metrics (70% CPU and Memory Utilization): The target is set to 70% to leave a safe 30% buffer. Since spinning up a new Django pod and passing liveness/readiness probes takes time (due to the initialDelaySeconds: 60), the existing pods need enough headroom to handle incoming traffic spikes while new replicas are bootstrapping.

### 3. Rolling Update Strategy
Configuration: maxUnavailable: 1 and maxSurge: 1.

Justification: With a base configuration of 2 replicas, maxUnavailable: 1 guarantees that at least 1 pod is always up and running, ensuring zero-downtime deployments. maxSurge: 1 allows Kubernetes to spin up exactly 1 new version pod at a time. This keeps the resource footprint minimal during deployments, which is crucial for cost-efficiency in development environments.

## How to Access the App After Deployment
Since the manifests do not include a Service or Ingress resource, you can access the application using Kubernetes Port-Forwarding:

Run the port-forward command targeting the deployment:

```
kubectl port-forward deployment/todoapp 8000:8080 -n mateapp
```
Open your web browser and navigate to:
http://localhost:8000