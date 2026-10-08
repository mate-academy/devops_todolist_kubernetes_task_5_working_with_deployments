# Instructions for Deploying and Testing Deployment with HPA

## 1. How to Apply Manifests
```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
```

## 2. Technical Decisions & Explanations

### Resource Requests and Limits Configuration
* **Requests (CPU: 100m, Memory: 64Mi):** Django is lightweight in an idle state. 64Mi of RAM and 0.1 CPU core are enough to initialize structures.
* **Limits (CPU: 200m, Memory: 128Mi):** Set to double request values to handle short-term spikes without triggering an OOM kill or throttling.

### Strategy Configuration (RollingUpdate)
* **maxUnavailable: 0** — Guarantees zero downtime by never reducing healthy pods below the target count during deployments.
* **maxSurge: 1** — Allows exactly 1 additional temporary pod during upgrades for efficient resource management.

### HPA Configuration Choice
* **Min Replicas: 2 / Max Replicas: 5:** Matches requirements to scale automatically up to 5 pods under load.
* **CPU Target (50%) & Memory Target (70%):** Triggers scaling early enough before applications hit hard resource ceilings.

## 3. How to Access the App After Deployment
```bash
kubectl port-forward deployment/todoapp-deployment 8000:8000 -n mateapp
```
URL: http://127.0.0.1:8000
