# Deployment Instructions & Architecture Specification

This document provides step-by-step instructions on deploying the Django ToDo application to Kubernetes (`k8s`) along with detailed explanations for the configuration decisions made in `deployment.yml` and `hpa.yml`.

---

## 1. Deployment Instructions

Follow these steps to deploy the application to your Kubernetes cluster:

### Step 1: Create the Namespace
Ensure the target namespace `mateapp` exists in your cluster:
```bash
kubectl create namespace mateapp
```
*(Alternatively, if using `namespace.yml`, run `kubectl apply -f .infrastructure/namespace.yml`)*

### Step 2: Deploy the Application Deployment
Apply the Deployment manifest:
```bash
kubectl apply -f .infrastructure/deployment.yml -n mateapp
```

### Step 3: Deploy the Horizontal Pod Autoscaler (HPA)
Apply the Horizontal Pod Autoscaler manifest:
```bash
kubectl apply -f .infrastructure/hpa.yml -n mateapp
```

### Step 4: Verify Deployment Status
Check that the deployment and pods are up and running:
```bash
kubectl get deployment -n mateapp
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
```

---

## 2. Resource Requests and Limits Rationale

In `deployment.yml`, container resources are defined as follows:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
```

### Rationale:
* **Resource Requests (`cpu: 100m`, `memory: 128Mi`)**:
  - Sets the baseline resource requirement guaranteed by the Kubernetes scheduler for each pod in an idle/low-load state.
  - Python/Django applications typically require ~80MB–100MB of RAM at idle. Requesting `128Mi` ensures stable operation without over-committing cluster memory.
  - CPU request of `100m` (0.1 CPU core) ensures sufficient baseline compute power for idle threads while allowing high density on cluster nodes.

* **Resource Limits (`cpu: 500m`, `memory: 256Mi`)**:
  - Defines the maximum resources a container can consume before being throttled (CPU) or killed (OOM - Memory).
  - Setting CPU limit to `500m` allows temporary CPU bursts during heavy web requests or database operations without starving other pods on the node.
  - Setting Memory limit to `256Mi` prevents memory leaks from exhausting node resources, ensuring pod isolation while giving double headroom above the request level.

---

## 3. Horizontal Pod Autoscaler (HPA) Configuration Rationale

In `hpa.yml`, the autoscaler is configured as follows:

```yaml
spec:
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
        averageUtilization: 80
```

### Rationale:
* **`minReplicas: 2`**:
  - Guarantees high availability (HA). Having at least 2 pods running in idle state prevents single-point-of-failure issues and allows zero-downtime rolling updates.

* **`maxReplicas: 5`**:
  - Limits scaling out to a maximum of 5 pods under heavy traffic load. This cap prevents cluster-wide resource exhaustion and runaway costs while comfortably handling traffic spikes.

* **Dual-Metric Triggers (CPU @ 70%, Memory @ 80%)**:
  - **CPU Utilization (70%)**: Scaling triggers before CPU saturates (100%), ensuring web requests remain fast and responsive during sudden load increases.
  - **Memory Utilization (80%)**: Django application memory usage increases with concurrent requests and session caching. Scaling at 80% prevents pods from exceeding their 256Mi limits and triggering OOM kills.

---

## 4. Deployment Strategy Configuration Rationale

In `deployment.yml`, the deployment strategy is configured as follows:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

### Rationale:
* **`type: RollingUpdate`**:
  - Ensures seamless updates by incrementally updating pod instances with zero downtime.

* **`maxSurge: 1`**:
  - Specifies that at most 1 pod above the desired replica count can be created during an update.
  - Controls resource usage during rollouts by adding 1 pod at a time (totaling 3 pods temporarily), avoiding cluster resource overload.

* **`maxUnavailable: 0`**:
  - Ensures that **0 pods can be unavailable** during the update process.
  - Keeps all 2 minimum baseline pods active and servicing traffic at all times during rollouts, guaranteeing 100% availability and constant service capacity.

---

## 5. Accessing the Application After Deployment

Once deployed, the application can be accessed using any of the following methods:

### Method 1: Port-Forwarding (Recommended for local testing & debugging)
Forward a local port directly to the deployment or pod in the `mateapp` namespace:
```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```
Access the app in your browser or via `curl`:
```bash
curl http://localhost:8080/
```

### Method 2: Access via Kubernetes Service (ClusterIP / NodePort)
If a Service manifest (e.g. NodePort) is deployed:
```bash
kubectl apply -f .infrastructure/nodeport.yml -n mateapp
```
You can access the application using your Kubernetes cluster node IP and NodePort:
```bash
http://<NODE-IP>:30080/
```

### Method 3: Verification via Busybox Pod inside Cluster
To test internal DNS resolution and service routing from within the cluster:
```bash
kubectl run busybox --image=busybox:1.28 --restart=Never -n mateapp -- rm -f out && kubectl exec busybox -n mateapp -- wget -qO- http://todoapp:8080/
```
