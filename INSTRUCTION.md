# Kubernetes Deployment Instructions for the ToDo App

This project is prepared for deployment in the `mateapp` namespace with a Deployment, a NodePort Service, and a Horizontal Pod Autoscaler (HPA).

## 1. Prerequisites

Before deploying, make sure you have:

- A working Kubernetes cluster
- `kubectl` configured to talk to the cluster
- Access to a namespace where you can deploy resources

## 2. Deploy the namespace

Apply the namespace first:

```bash
kubectl apply -f .infrastructure/mateapp-namespace.yml
```

## 3. Deploy the application

Apply the Deployment and Service:

```bash
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/nodeport.yml
```

If you want to verify the created workload:

```bash
kubectl get deploy,pods,svc -n mateapp -o wide
```

Expected result:

- 2 application pods running in the `mateapp` namespace
- A Service of type `NodePort` exposing the app on a port in the 30000-32767 range

## 4. Verify the app is healthy

Check pod readiness and HPA status:

```bash
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
kubectl describe deployment todoapp -n mateapp
```

The app is considered healthy when all pods are `Running` and `READY` shows the expected number of replicas.

## 5. Access the app after deployment

The app is exposed through the NodePort service defined in `.infrastructure/nodeport.yml`.

Use one of the following methods:

### Option A: Local browser access

- If you are running Kubernetes locally, open:

```text
http://localhost:30080
```

- If you are using a remote cluster, use the worker node IP or load balancer address instead of `localhost`:

```text
http://<node-ip>:30080
```

### Option B: Port-forward for local testing

```bash
kubectl port-forward svc/todoapp -n mateapp 8080:80
```

Then open:

```text
http://localhost:8080
```

## 6. Resource requests and limits explanation

The Deployment config uses:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### Why these values?

- `cpu: 250m` request: this is the baseline CPU each pod needs to run normally. It is low enough to keep the cluster efficient while still providing sufficient headroom for the Django app under light concurrent traffic.
- `memory: 64Mi` request: this keeps the app affordable in the idle state while allowing the process to maintain its runtime memory without being evicted unnecessarily.
- `cpu: 500m` limit: this prevents a single pod from consuming too much CPU and starving other workloads in the cluster.
- `memory: 128Mi` limit: this caps memory usage and prevents a noisy or faulty pod from consuming all the node memory.

This is a practical conservative configuration for a small web application that is expected to run with 2 replicas in the normal case and scale up when load grows.

## 7. HPA configuration explanation

The HPA is configured as follows:

```yaml
minReplicas: 2
maxReplicas: 5
averageUtilization: 70
```

### Why this configuration?

- `minReplicas: 2`: keeps two instances available at all times so the app remains highly available and avoids downtime during updates or brief spikes.
- `maxReplicas: 5`: limits scale-out to a reasonable range for a small deployment while still allowing the app to absorb traffic bursts.
- `averageUtilization: 70`: the HPA scales when CPU or memory usage remains above 70% of the requested capacity, which is a common threshold to avoid reacting too early to tiny fluctuations while still scaling before the app becomes slow or unstable.
- CPU and memory both included: this makes the app responsive to both compute pressure and memory pressure, which is important for a Python/Django application using background workers, request handling, and memory-heavy page rendering.

This creates a balance between responsiveness and stability: the app scales out under load, but not aggressively enough to cause unnecessary resource churn.

## 8. Deployment strategy explanation

The Deployment uses:

```yaml
strategy:
  type: RollingUpdate
```

### Why `RollingUpdate`?

A rolling update replaces pods gradually instead of taking all replicas down at once. This ensures:

- no full outage during a new version rollout
- users can keep using the app while the new version is being deployed
- the app keeps serving traffic with a reduced number of instances during the transition

With `replicas: 2`, the app remains available during updates while still having enough redundancy for a small service. This is a good default for web applications where uptime matters.

## 9. Useful Kubernetes commands

Check deployment status:

```bash
kubectl get deployment todoapp -n mateapp
kubectl rollout status deployment/todoapp -n mateapp
```

View logs:

```bash
kubectl logs -n mateapp deployment/todoapp
```

Delete the deployment if needed:

```bash
kubectl delete -f .infrastructure/deployment.yml
kubectl delete -f .infrastructure/nodeport.yml
kubectl delete -f .infrastructure/mateapp-namespace.yml
```

## 10. Summary

This setup gives the project:

- a stable namespace (`mateapp`)
- a small but resilient Deployment with 2 replicas by default
- resource constraints to protect the cluster
- autoscaling to 5 replicas when CPU or memory pressure rises
- a rolling update strategy for safe deployments
- external access through a NodePort service on port `30080`
