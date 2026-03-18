## Kubernetes task 5 — Deployments + HPA

### Apply manifests

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/busybox.yml
```

Verify:

```bash
kubectl -n mateapp get deploy,rs,pods,svc,hpa
kubectl -n mateapp describe deploy todoapp
kubectl -n mateapp describe hpa todoapp
```

### Why these resource requests/limits
- **requests cpu=50m, memory=128Mi**: small baseline so 2 pods can run in idle on typical local clusters and the scheduler can place them easily.
- **limits cpu=250m, memory=256Mi**: prevents a single pod from consuming too much CPU/RAM on a small node while still allowing short spikes.

### Why this HPA configuration
- **minReplicas=2**: ensures high availability (two pods) even in idle state.
- **maxReplicas=5**: caps scaling to avoid exhausting small clusters.
- **CPU + Memory utilization**: scales on both common bottlenecks; requires Metrics Server in the cluster.
- **Targets**: CPU 70% and Memory 75% are moderate thresholds to scale before the pods become overloaded.

### Why this RollingUpdate strategy
- **maxUnavailable=0**: keeps all existing pods available during update (no downtime).
- **maxSurge=1**: adds at most one extra pod during rollout to keep resource usage predictable.

### How to access the app

#### Option A: Port-forward the ClusterIP service

```bash
kubectl -n mateapp port-forward svc/todoapp 8080:80
```

Then:

```bash
curl -i http://localhost:8080/
curl -i http://localhost:8080/api/ready
curl -i http://localhost:8080/api/health
```

#### Option B: Call the service DNS from busybox

```bash
kubectl -n mateapp exec -it pod/busybox -- sh
```

Inside:

```sh
curl -i http://todoapp/
curl -i http://todoapp/api/ready
curl -i http://todoapp/api/health
```

#### Option C: Use NodePort
- NodePort is set to `30080`
- Open: `http://<NODE_IP>:30080/`

Get the node IP:

```bash
kubectl get nodes -o wide
```

