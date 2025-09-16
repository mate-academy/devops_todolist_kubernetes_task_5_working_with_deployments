# INSTRUCTION — Deploy ToDo App to Kubernetes (namespace `mateapp`)

## Prerequisites
- Access to a Kubernetes cluster (`kubectl` configured).
- `metrics-server` must be installed in the cluster (required for HPA).
- Docker image: `ikulyk404/todoapp:3.0.0` (works with built-in SQLite, so no DB/SECRET_KEY env vars are required for a basic setup).

---

## 1) Deployment
Apply the manifests:

```
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```
Check resources:
```
kubectl get ns
kubectl get deployments -n mateapp
kubectl get pods -n mateapp
```
## 2) Verify HPA
Check the Horizontal Pod Autoscaler:
```
kubectl get hpa -n mateapp
kubectl describe hpa todoapp-hpa -n mateapp
```
Verify metrics-server is running:
```
kubectl get deployment -n kube-system metrics-server
```
## 3) Access the App
Option A — Port-forward (local testing)
```commandline
POD=$(kubectl get pods -n mateapp -l app=todolist -o jsonpath='{.items[0].metadata.name}')
kubectl port-forward -n mateapp $POD 8080:8080
```
Now open http://localhost:8080
## 4) Resource Requests & Limits
Requests: 100m CPU, 128Mi Memory — baseline resources guaranteed for each pod. Low values ensure two pods can run on most clusters without over-committing.

Limits: 500m CPU, 512Mi Memory — upper bound to prevent resource hogging. Gives enough room for request spikes but keeps pods under control.
## 5) HPA Configuration
minReplicas: 2 — ensures high availability (always 2 pods running).

maxReplicas: 5 — prevents uncontrolled scaling.

CPU target: 50% — scales up when average CPU usage exceeds 50%.

Memory target: 60% — scales up if memory grows beyond 60% utilization.

Using both CPU and Memory makes scaling more reliable.

## 6) RollingUpdate Strategy
maxSurge: 1 — one extra pod may be created during updates, ensuring no downtime.

maxUnavailable: 1 — only one pod can be unavailable at a time, maintaining availability.

Balanced values for safe rolling updates while minimizing downtime.
## 7) Debugging Commands
```
kubectl get deploy -n mateapp
kubectl get pods -n mateapp -o wide
kubectl get hpa -n mateapp
kubectl describe hpa todoapp-hpa -n mateapp
kubectl top pods -n mateapp   # requires metrics-server
```

