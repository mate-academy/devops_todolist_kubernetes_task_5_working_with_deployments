# Deployment Instructions — todoapp on Kubernetes

## Prerequisites

- `kubectl` installed and configured to point at your cluster
- Access to the target cluster with sufficient permissions to create namespaces, deployments, and HPAs
- Metrics Server installed in the cluster (required for HPA to work)

---

## 1. Create the Namespace

```bash
kubectl create namespace mateapp
```

---

## 2. Apply the Manifests

```bash
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

Verify everything is running:

```bash
kubectl get all -n mateapp
```

Expected output: 2 pods in `Running` state, 1 deployment, 1 HPA.

---

## 3. Accessing the App

The deployment exposes port `8080` inside the cluster. To access the app from outside you need a Service. Apply a LoadBalancer or NodePort service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: todoapp
  namespace: mateapp
spec:
  selector:
    app: todoapp
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

```bash
kubectl apply -f service.yml
kubectl get svc -n mateapp
```

Once the external IP is assigned:

```
http://<EXTERNAL-IP>/api/health
```

For local development without a LoadBalancer, use port-forward:

```bash
kubectl port-forward -n mateapp deployment/todoapp 8080:8080
# App available at http://localhost:8080
```

---

## 4. Resource Requests and Limits — Rationale

| | CPU | Memory |
|---|---|---|
| **Request** | 250m | 64Mi |
| **Limit** | 500m | 128Mi |

**Requests** define the guaranteed resources the scheduler reserves per pod. `250m` CPU and `64Mi` memory reflect the typical idle consumption of a lightweight Node.js/Python todo app — enough for the scheduler to place pods on nodes without over-provisioning.

**Limits** are set to `2×` the request values (`500m` CPU, `128Mi` memory). This asymmetry is intentional:

- Memory limit is doubled (64Mi → 128Mi) to give headroom for occasional spikes (e.g. a burst of requests, GC pressure) without risking OOMKill under normal load.
- CPU limit is doubled (250m → 500m) so the pod can briefly burst for request handling without being throttled immediately. CPU throttling degrades latency; a generous limit prevents this.

Setting requests equal to limits (as in the original manifest) creates a `Guaranteed` QoS class, which is overly strict for a web app with variable traffic. The chosen values give `Burstable` QoS — a better fit for real-world workloads.

**HPA interaction:** The HPA calculates utilization as `actual usage / request`. With `request: 250m`, a pod using 175m CPU registers as 70% — exactly the trigger threshold. This makes the autoscaling math predictable and responsive.

---

## 5. HPA Configuration — Rationale

```yaml
minReplicas: 2
maxReplicas: 5
averageUtilization: 70  # for both CPU and memory
```

**minReplicas: 2** — Two pods are always running. This ensures high availability: if one pod crashes or is evicted during a node issue, the app stays online without waiting for a scale-up event.

**maxReplicas: 5** — For a todo app with modest traffic expectations, 5 pods (2.5× idle capacity) is a reasonable ceiling. It limits infrastructure cost while still handling meaningful traffic spikes (roughly 2.5× baseline load).

**70% utilization threshold** — 70% is the industry-standard sweet spot:
- Low enough that scaling triggers before pods become saturated (avoiding latency spikes).
- High enough that you are not wasting resources by scaling too eagerly.
- Triggering on both CPU and memory means the HPA reacts to whichever resource becomes the bottleneck first, which is important for apps that may be CPU-bound under heavy computation or memory-bound during large payload handling.

---

## 6. RollingUpdate Strategy — Rationale

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

**RollingUpdate** replaces pods incrementally rather than all at once, achieving zero-downtime deployments.

**maxUnavailable: 1** — At most one pod can be unavailable during the update. With `replicas: 2`, this means one pod is always serving traffic throughout the rollout. Choosing `0` would require a surge pod to be ready before any old pod is removed — slower but safer if you need strict minimum capacity. `1` is appropriate here since 1 remaining pod can handle idle traffic while the update proceeds.

**maxSurge: 1** — One extra pod (above the desired 2) can be created temporarily during the rollout. This means at peak the cluster runs 3 pods briefly: the new pod starts, becomes ready, then the old pod is terminated. This keeps rollout fast (no waiting for a pod to terminate before starting the next) at the cost of slightly higher resource usage for a short window.

**Together:** with 2 replicas, `maxUnavailable: 1` + `maxSurge: 1` results in the simplest safe rolling update — one new pod comes up, one old pod goes down, one cycle, done. For larger deployments you might increase both values to speed up rollouts, but for 2 replicas this is optimal.