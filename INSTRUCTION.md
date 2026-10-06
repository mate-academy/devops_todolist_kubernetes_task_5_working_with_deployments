# Deploying the ToDo app to Kubernetes

All manifests are in the `.infrastructure` folder and belong to the `mateapp` namespace.

| File | Purpose |
|------|---------|
| `namespace.yml` | `mateapp` namespace |
| `deployment.yml` | Deployment of the ToDo app (2 replicas, RollingUpdate) |
| `hpa.yml` | Horizontal Pod Autoscaler (2–5 pods, CPU + memory) |
| `clusterIp.yml` | Internal service (`ClusterIP`) |
| `nodeport.yml` | External access service (`NodePort`, port 30080) |

## Prerequisites

- A running cluster (Minikube, Docker Desktop, kind, ...) and `kubectl` configured for it.
- **metrics-server** installed. The HPA cannot read CPU/memory usage without it.
  On Minikube: `minikube addons enable metrics-server`.

## Deployment steps

```bash
# 1. Namespace first, everything else lives in it
kubectl apply -f .infrastructure/namespace.yml

# 2. Deployment (creates 2 pods)
kubectl apply -f .infrastructure/deployment.yml

# 3. Autoscaler
kubectl apply -f .infrastructure/hpa.yml

# 4. Services
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml
```

Or all at once: `kubectl apply -f .infrastructure/namespace.yml && kubectl apply -f .infrastructure/`
(the first command guarantees the namespace exists before the rest is applied).

Verify:

```bash
kubectl get deployment,pods,hpa,svc -n mateapp
kubectl rollout status deployment/todoapp -n mateapp
```

In the idle state you should see 2 `todoapp` pods in `Running` / `READY 1/1`, and the HPA showing `MINPODS 2`, `MAXPODS 5`.

## Accessing the app

The app listens on port 8080 in the container; the services map it to port 80.

- **NodePort** (`nodeport.yml`): the app is available on port `30080` of any node.
  - Minikube: `minikube service todoapp -n mateapp --url` (or `http://$(minikube ip):30080`).
  - Docker Desktop: `http://localhost:30080`.
- **Port-forward** (works on any cluster):
  ```bash
  kubectl port-forward svc/todoapp 8080:80 -n mateapp
  ```
  then open <http://localhost:8080>. The API is at <http://localhost:8080/api/>.
- **From inside the cluster**: `http://todoapp.mateapp.svc.cluster.local` (ClusterIP service).

## Why these resource requests and limits

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

- The app is a Django app run with `manage.py runserver` (SQLite). At idle it uses very little CPU and memory, but the Python/Django process needs some headroom at startup and under load, so `250m` CPU / `256Mi` memory is reserved per pod. 2 idle pods need `500m` / `512Mi` in total, which still fits a small local cluster.
- Limits are 2x the requests. That leaves room for short bursts (migrations, request spikes) while preventing one pod from starving the node. A pod that goes over the memory limit is OOM-killed and restarted; over the CPU limit it is only throttled.
- Requests are also required for the HPA: percentage utilization is calculated relative to the **request**, so without requests the autoscaler cannot work.

## Why this HPA configuration

- `minReplicas: 2` keeps the app available if one pod fails or is being updated, and matches the 2 pods expected in the idle state.
- `maxReplicas: 5` caps the cost and protects a small cluster from being exhausted by a traffic spike or a runaway loop. At the maximum, 5 pods request 1.25 CPU / 1.25 GiB and can use up to 2.5 CPU / 2.5 GiB (limits), which a typical dev cluster can handle.
- Two metrics, CPU and memory, both with `averageUtilization: 70`. The HPA scales when **either** one is exceeded. 70% of the request leaves 30% headroom, so new pods have time to start (the liveness probe delay is 60s) before existing ones become saturated. A lower value would scale too eagerly; a higher one would react too late.

## Why this rollout strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

- `RollingUpdate` replaces pods gradually, so the app stays available during an update (unlike `Recreate`).
- `maxSurge: 1`: at most 1 extra pod (3 in total with 2 replicas) is created during an update. This keeps the extra resource usage small.
- `maxUnavailable: 1`: at most 1 of 2 pods can be down during an update, so at least 1 pod always serves traffic, and the update does not wait for extra capacity.
- With 2 replicas, the rollout goes one pod at a time. The readiness probe makes sure traffic is only sent to a new pod once it is ready.

## Cleanup

```bash
kubectl delete -f .infrastructure/
```
