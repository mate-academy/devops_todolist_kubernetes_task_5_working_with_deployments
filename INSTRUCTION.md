# Kubernetes deployment instructions for ToDo app

- `deployment.yml` - Deployment with `RollingUpdate` strategy and resource requests/limits
- `hpa.yml` - Horizontal Pod Autoscaler (CPU + Memory)

Both manifests use the `mateapp` namespace as required by the task.

## Deploy the app to Kubernetes

1. Create namespace:

```bash
kubectl create namespace mateapp
```

2. Apply the Deployment:

```bash
kubectl apply -f deployment.yml
```

3. Apply the HPA:

```bash
kubectl apply -f hpa.yml
```

4. Check resources:

```bash
kubectl get deploy,pods,hpa -n mateapp
```

## Resource requests and limits

Deployment uses:

- `requests.cpu: 100m`
- `requests.memory: 128Mi`
- `limits.cpu: 300m`
- `limits.memory: 256Mi`

Reasoning:

- The app is a small Django-based web app and usually stays light in idle mode.
- `100m / 128Mi` lets the scheduler place **at least 2 pods** comfortably on a small training cluster.
- Limits protect the node from one pod consuming too much CPU/RAM.
- HPA uses resource utilization percentages, so requests are required and should be realistic.

## HPA configuration (why min 2 / max 5 and CPU + Memory)

HPA settings:

- `minReplicas: 2`
- `maxReplicas: 5`
- CPU target: `70%`
- Memory target: `75%`

Reasoning:

- `minReplicas: 2` keeps two app instances running in idle state (task requirement) and improves availability.
- `maxReplicas: 5` is a reasonable upper bound for a small app in a learning/staging cluster to avoid overusing cluster resources.
- CPU and Memory metrics together make scaling more stable:
  - CPU catches request/load spikes.
  - Memory catches growth from worker/process usage and cached data.
- 70-75% targets leave headroom before pods hit limits.

Note:

- HPA based on CPU/Memory requires Metrics Server in the cluster.

## RollingUpdate strategy (why these numbers)

Deployment strategy:

- `type: RollingUpdate`
- `maxUnavailable: 0`
- `maxSurge: 1`

Reasoning:

- `maxUnavailable: 0` means no running pod is taken down before a new pod becomes ready, which helps avoid downtime.
- `maxSurge: 1` allows one extra pod during update, which is enough for a small app and limits temporary resource usage.
- With `replicas: 2`, the rollout typically runs with 2-3 pods during update and keeps service available.

## How to access the app after deployment

### Option 1 (fastest): Port-forward to the Deployment

```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

Open:

- `http://localhost:8080/`

### Option 2: Expose with a Service (NodePort or ClusterIP)

The repository already contains service examples in `.infrastructure/`.
To use them with this deployment, create/apply a Service in the `mateapp` namespace (instead of `todoapp`) with selector:

- `app: todoapp`

If using NodePort (for example `30080`), access:

- `http://<node-ip>:30080/`
