# ToDo app: Deployment and HPA

## Prerequisites

- `kubectl` configured for a cluster (Minikube, Docker Desktop Kubernetes, kind, etc.)
- **Metrics Server** installed in the cluster (required for CPU/Memory-based HPA). For Minikube:

  ```bash
  minikube addons enable metrics-server
  ```

## Deploy

From the repository root:

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml
```

Check resources in namespace `mateapp`:

```bash
kubectl get deploy,hpa,pods,svc -n mateapp
kubectl wait --for=condition=Available deployment/todoapp -n mateapp --timeout=180s
```

You should see **2** pods in `Running` state with `READY 1/1` when the app is idle (Deployment replicas + HPA `minReplicas`).

## Resource requests and limits

|              | CPU  | Memory |
| ------------ | ---- | ------ |
| **requests** | 64m  | 128Mi  |
| **limits**   | 128m | 256Mi  |

**Why these values**

- The ToDo app is a lightweight Django process; in idle it uses a fraction of one CPU core and well under 128Mi RAM.
- **Requests** reserve minimal capacity on the node so the scheduler can place **2 pods** reliably without overcommitting the cluster on a small local setup.
- **Limits** cap burst usage (e.g. during requests or startup) so a single pod cannot starve others; the 2× ratio between request and limit leaves headroom for short spikes without making HPA metrics meaningless.
- HPA uses utilization **relative to requests**, so requests must reflect realistic baseline usage, not peak load.

## HPA configuration

- **minReplicas: 2** — matches the Deployment default and the task requirement for two pods at idle; improves availability during rollouts and node issues.
- **maxReplicas: 5** — upper bound for load tests or traffic spikes without unbounded scaling on a dev cluster.
- **Metrics: CPU and Memory at 60% average utilization** — scale out when either resource stays above ~60% of its **request** across pods. Using both metrics avoids scaling only on CPU while memory grows (or vice versa). 60% is a common middle ground: not so low that HPA flaps, not so high that pods are already stressed before new replicas appear.

Verify HPA:

```bash
kubectl describe hpa todoapp -n mateapp
kubectl get hpa todoapp -n mateapp -w
```

## Access the app after deployment

### NodePort (browser / curl from your machine)

Service `todoapp-nodeport` exposes port **80** → container **8080** on node port **30007**.

```bash
# Minikube
NODE_IP=$(minikube ip)
curl -sS "http://${NODE_IP}:30007/api/health"

# Or open in browser: http://<NODE_IP>:30007/
```

Replace `<NODE_IP>` with your node IP on Docker Desktop / other clusters.

### Port-forward (any cluster)

```bash
kubectl port-forward -n mateapp service/todoapp-nodeport 8080:80
```

Then: [http://127.0.0.1:8080/](http://127.0.0.1:8080/) or `curl http://127.0.0.1:8080/api/health`.

### ClusterIP (inside the cluster)

From a debug pod in `mateapp`:

```bash
kubectl run -n mateapp curl-test --rm -it --image=curlimages/curl --restart=Never -- \
  curl -sS http://todoapp-clusterip/api/health
```

## Useful commands

```bash
kubectl logs -n mateapp -l app=todoapp --tail=50
kubectl rollout status deployment/todoapp -n mateapp
kubectl top pods -n mateapp   # needs metrics-server
```
