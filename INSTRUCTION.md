# ToDo App — Kubernetes Deployment & Autoscaling Instructions

## Docker Hub Repository

- App image: https://hub.docker.com/r/olehkovalievskyi/todoapp

## Prerequisites

- A running Kubernetes cluster (e.g., Kind, Minikube, or Docker Desktop Kubernetes)
- `kubectl` installed and configured to talk to your cluster
- Metrics Server installed in the cluster (required for HPA to read CPU/Memory metrics)

## Project Structure

All Kubernetes manifests are located in the `.infrastructure` folder:

- `namespace.yml` — creates the `mateapp` namespace
- `deployment.yml` — creates a Deployment managing the ToDo app pods
- `hpa.yml` — creates a Horizontal Pod Autoscaler for the Deployment

## 1. Apply the manifests

Apply the manifests in the following order:

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
```

Check that the Deployment created 2 pods (idle state):

```bash
kubectl get pods -n mateapp
```

Check the Deployment status:

```bash
kubectl get deployment todoapp -n mateapp
```

Check the HPA status:

```bash
kubectl get hpa -n mateapp
```

## 2. Access the app after deployment

Forward a local port to the Deployment's pods (via a Service, if one is
configured, or directly via a pod for quick testing):

```bash
kubectl port-forward -n mateapp deployment/todoapp 8000:8080
```

Keep this command running in a terminal, then open your browser and
navigate to:

http://localhost:8000

To stop port-forwarding, press `Ctrl+C` in the terminal.

## Design decisions

### Resource requests and limits

The container is configured with the following resources:

```yaml
resources:
  requests:
    cpu: "256m"
    memory: "64Mi"
  limits:
    cpu: "500m"
    memory: "128Mi"
```

**Why these values:** the ToDo app is a lightweight Django application
serving a small number of API requests, so it doesn't need large
CPU/memory reservations. The `requests` values (256m CPU, 64Mi memory)
reflect the typical baseline load of a single Django process serving
light traffic, ensuring the scheduler reserves just enough resources
per pod. The `limits` (500m CPU, 128Mi memory) give the container
headroom for short traffic spikes without allowing a single pod to
consume excessive cluster resources, which keeps the cluster stable
if multiple pods scale up simultaneously.

### RollingUpdate strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

**Why these values:** with a minimum of 2 replicas, `maxUnavailable: 1`
ensures that at least 1 pod is always serving traffic during an update,
so the app stays available with zero downtime. `maxSurge: 1` allows
Kubernetes to briefly create 1 extra pod above the desired count while
rolling out a new version, which speeds up the rollout without over
provisioning the cluster.

### Horizontal Pod Autoscaler configuration

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

**Why these values:**
- `minReplicas: 2` matches the idle-state requirement and guarantees
  basic redundancy — if one pod fails or is being updated, the other
  keeps serving traffic.
- `maxReplicas: 5` caps scaling at a level that comfortably handles
  traffic spikes without risking uncontrolled resource consumption on
  the cluster.
- A `70%` average utilization target for both CPU and memory is a
  common, conservative threshold: it triggers scaling early enough to
  absorb rising load smoothly, while avoiding unnecessary scale-up
  from brief, harmless usage spikes.

## Cleaning up

To remove all resources created by these manifests:

```bash
kubectl delete -f .infrastructure/hpa.yml
kubectl delete -f .infrastructure/deployment.yml
kubectl delete -f .infrastructure/namespace.yml
```