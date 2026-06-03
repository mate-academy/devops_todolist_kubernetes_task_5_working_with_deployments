# ToDo App Kubernetes Deployment Guide

## Prerequisites
- `kubectl` installed and configured
- Access to a Kubernetes cluster
- `metrics-server` installed in the cluster (required for HPA)

## Deployment Steps

### 1. Create the namespace
kubectl apply -f namespace.yml

### 2. Deploy the application
kubectl apply -f deployment.yml

### 3. Apply the services
kubectl apply -f clusterIp.yml
kubectl apply -f nodeport.yml

### 4. Apply the Horizontal Pod Autoscaler
kubectl apply -f hpa.yml

### 5. Verify the deployment
kubectl get pods -n todoapp
kubectl get deployment -n todoapp
kubectl get hpa -n todoapp

## Accessing the Application
The application is accessible via the NodePort service on port **30080**:
http://<NODE_IP>:30080

To get the node IP:
kubectl get nodes -o wide

## Resource Requests and Limits — Explanation

| Parameter | Value | Reason |
|-----------|-------|--------|
| requests.memory | 128Mi | Django + SQLite consumes ~80–100Mi in idle state |
| requests.cpu | 100m | Django is nearly idle on CPU without active traffic |
| limits.memory | 256Mi | 2x buffer over request to handle traffic spikes |
| limits.cpu | 200m | Prevents a single pod from consuming too many shared CPU resources |

QoS class: **Burstable** — pods are allowed to use more resources under load,
while the scheduler can still place them based on their requests.

## HPA Configuration — Explanation

- **minReplicas: 2** — ensures high availability; if one pod fails,
  the second continues serving traffic without interruption
- **maxReplicas: 5** — sufficient to handle peak load without excessive infrastructure cost
- **CPU threshold: 70%** — gives Kubernetes enough time to spin up a new pod
  before the existing ones are overwhelmed
- **Memory threshold: 70%** — prevents OOM kills by scaling out before memory is exhausted

Scaling formula used by HPA:
Desired Replicas = Current Replicas × (Current Metric Value / Target Metric Value)

Example: 2 pods at 90% CPU → 2 × (90 / 70) ≈ 3 pods

## Deployment Strategy — Explanation

Strategy: **RollingUpdate**
- **maxUnavailable: 1** — at most 1 pod can be unavailable during an update,
  meaning at least 1 pod always serves traffic
- **maxSurge: 1** — allows 1 extra pod to be created above the replica count
  to speed up the rollout

**Why RollingUpdate and not Recreate:**
RollingUpdate guarantees zero-downtime deployments — users do not experience
any interruption during updates. Recreate terminates all pods before starting
new ones, which causes full downtime and is not acceptable for a production web application.

**Why these numbers (1 and 1):**
With 2 replicas as the baseline, maxUnavailable: 1 ensures 50% capacity is
always available. maxSurge: 1 allows a temporary total of 3 pods during rollout,
which is a reasonable overhead for a lightweight Django application.