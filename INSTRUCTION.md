# Deployment Instructions for ToDo App in Kubernetes

## How to Deploy the Application

1. **Create the namespace:**
   ```bash
   kubectl create namespace mateapp
   ```

2. **Apply the Deployment manifest:**
   ```bash
   kubectl apply -f deployment.yml
   ```

3. **Apply the HPA manifest:**
   ```bash
   kubectl apply -f hpa.yml
   ```

---

## Configuration Explained

### 1. Resource Requests and Limits

- **Requests:** `cpu: 100m`, `memory: 128Mi`
  These are the baseline resources required to run a lightweight Django/Python application in idle state. They are sufficient to keep the framework running in the background without any active load, and ensure the pod gets scheduled on a node with enough available resources.

- **Limits:** `cpu: 250m`, `memory: 256Mi`
  Limits are set to prevent a single pod from monopolizing node resources. Django can consume more memory during request processing, so 256Mi provides a safe headroom while avoiding `OOMKilled` (Out of Memory) errors. The CPU limit of 250m allows short bursts of processing without impacting other workloads on the same node.

---

### 2. HPA (Horizontal Pod Autoscaler) Configuration

- **Minimum replicas (2):**
  Guarantees high availability. If one pod crashes or becomes unavailable, the second one continues handling requests without any downtime.

- **Maximum replicas (5):**
  Caps infrastructure costs in case of a sudden traffic spike (e.g., DDoS attack or peak load). For a lightweight ToDo app, 5 pods is more than enough to handle significant load.

- **Triggers (CPU: 70%, Memory: 80%):**
  Scaling begins before a pod reaches its resource limits. This gives new pods time to start up and become ready before the existing ones become overloaded. A CPU threshold of 70% is a widely accepted balance between responsiveness and efficiency. Memory threshold is set slightly higher at 80% because memory usage in Django is generally more stable and short-lived spikes are common.

---

### 3. Deployment Strategy (RollingUpdate)

- **`maxSurge: 1`** — during an update, Kubernetes is allowed to create only 1 additional pod above the desired replica count. This keeps cluster resource usage under control during rollouts.

- **`maxUnavailable: 1`** — during an update, at least 1 pod out of the 2 baseline replicas is always guaranteed to be available to users. This ensures zero downtime during new version releases.

Together, these settings mean that during a rollout Kubernetes will bring up 1 new pod first, wait for it to be ready, then terminate 1 old pod — keeping the app live throughout the entire process.

---

### 4. How to Access the Application

To access the application locally without configuring an Ingress, you can use port-forwarding directly to the Deployment:

```bash
kubectl port-forward deployment/todo-app-deployment 8080:8000 -n mateapp
```

After running this command, open your browser and navigate to:

```
http://localhost:8080
```