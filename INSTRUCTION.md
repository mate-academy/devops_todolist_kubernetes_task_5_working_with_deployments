# Deploying Django ToDo App to Kubernetes

## Prerequisites

- `kubectl` configured against the target cluster
- [Metrics Server](https://github.com/kubernetes-sigs/metrics-server) installed — required for HPA to work:
  ```bash
  kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
  ```
- Docker image `ikulyk404/todoapp:3.0.0` available (pulled from the registry)

## Deployment Steps

```bash
# 1. Apply the namespace
kubectl apply -f .infrastructure/mateapp-namespace.yml

# 2. Apply the Deployment
kubectl apply -f .infrastructure/deployment.yml

# 3. Apply the Horizontal Pod Autoscaler
kubectl apply -f .infrastructure/hpa.yml
```

Verify everything is running:

```bash
kubectl get all -n mateapp
```

## Resource Requests & Limits

| Resource | Request | Limit | Rationale                                                                                                                                                                                                                         |
|----------|---------|-------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CPU      | 100m    | 500m  | Django with SQLite is lightweight. 100m (0.1 core) covers idle request processing; 500m allows burst to 0.5 core during traffic spikes before HPA kicks in.                                                                       |
| Memory   | 120Mi   | 512Mi | Python + Django + SQLite baseline is ~80–100Mi. 120Mi provides a small safety margin. 512Mi is a conservative upper bound — this app rarely exceeds 200Mi even under moderate load. Prevents a single pod from starving the node. |

## HPA Configuration

| Setting       | Value | Rationale                                                                                                                                                              |
|---------------|-------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Min replicas  | 2     | Matches the Deployment's replica count — ensures high availability even at idle.                                                                                       |
| Max replicas  | 5     | Enough to handle moderate traffic without over-provisioning (cost-conscious).                                                                                          |
| CPU target    | 70%   | Scales out before CPU becomes a bottleneck, keeping response times low. 70% leaves headroom for sudden request bursts without immediate scaling.                       |
| Memory target | 80%   | Scales out when memory pressure builds, preventing OOM kills. Memory doesn't spike as quickly as CPU, so a higher target avoids thrashing (rapid scale-up/scale-down). |

**Verification:** Check if Metrics Server is working:

```bash
kubectl top pods -n mateapp
```

## Strategy Configuration

| Setting        | Value         | Rationale                                                                                                                      |
|----------------|---------------|--------------------------------------------------------------------------------------------------------------------------------|
| Type           | RollingUpdate | Standard strategy for zero-downtime deployments.                                                                               |
| maxSurge       | 1             | During an update, one extra pod is created before old pods are terminated. Ensures capacity is maintained.                     |
| maxUnavailable | 0             | No pod is terminated until a replacement is fully ready (passes readiness probe). Guarantees zero downtime during deployments. |

**Trade-off:** Slightly more transient resource usage during updates (one extra pod for the duration of the rollout),
but the user experience is uninterrupted.

## Pod Spec Parity

The Deployment's pod spec exactly matches the original pod manifests (see `.infrastructure/todoapp-pods-reference.yml`):

| Field | Pod Spec | Deployment Pod Spec | Status |
|-------|----------|---------------------|--------|
| Image | `ikulyk404/todoapp:3.0.0` | `ikulyk404/todoapp:3.0.0` | ✅ Match |
| Container port | `8080` | `8080` | ✅ Match |
| Liveness probe path | `api/health` | `api/health` | ✅ Match |
| Liveness probe port | `8080` | `8080` | ✅ Match |
| Liveness initialDelaySeconds | `60` | `60` | ✅ Match |
| Readiness probe path | `api/ready` | `api/ready` | ✅ Match |
| Readiness probe port | `8080` | `8080` | ✅ Match |
| Readiness initialDelaySeconds | `5` | `5` | ✅ Match |
| Env `PYTHONUNBUFFERED` | `"1"` | `"1"` | ✅ Match |

The Deployment adds `resources` (requests/limits) which were not in the original pod manifests — this is intentional for production readiness.

## Accessing the App

### Option 1: Port-forward (recommended for local testing)

```bash
kubectl port-forward svc/todoapp 8080:80 -n mateapp
```

Then open: **http://localhost:8080**

### Option 2: NodePort (for cluster-external access)

The existing `.infrastructure/nodeport.yml` exposes the app on port **30080** of each node:

```
http://<node-ip>:30080
```

### Option 3: ClusterIP (for internal cluster access)

```bash
kubectl get svc -n mateapp   # find the ClusterIP
kubectl port-forward svc/todoapp 8080:80 -n mateapp
```

## Useful Commands

```bash
# Check pod status
kubectl get pods -n mateapp

# View deployment status
kubectl get deployment -n mateapp

# View HPA status
kubectl get hpa -n mateapp

# View logs
kubectl logs -l app=todoapp -n mateapp

# Scale manually (overrides HPA temporarily)
kubectl scale deployment todoapp --replicas=3 -n mateapp
```
