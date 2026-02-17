# Instruction: Deployment of TodoApp to Kubernetes

This guide provides instructions on how to deploy the **todoapp** to a Kubernetes cluster and explains the configuration choices.

---

## How to Deploy the App

At first: 
```bash
    cd .infrastructure
```

1.  **Create the Namespace:**
    ```bash
    kubectl apply -f namespace.yml
    ```
2.  **Deploy the Application:**
    ```bash
    kubectl apply -f deployment.yml
    ```
3.  **Configure Horizontal Pod Autoscaler (HPA):**
    ```bash
    kubectl apply -f hpa.yml
    ```
4.  **Expose the Application:**
    ```bash
    kubectl apply -f nodeport.yml
    ```

---

## Configuration Explained

### 1. Resource Requests and Limits
* **Requests (cpu: 250m, memory: 64Mi):** ✅ These values represent the guaranteed minimum resources allocated to each pod, ensuring efficient node utilization during idle states.
* **Limits (cpu: 500m, memory: 128Mi):** ✅ These define the maximum resource usage allowed, preventing a single pod from consuming all node resources and protecting against memory leaks.

### 2. RollingUpdate Strategy
* **maxUnavailable: 1:** ✅ Only one pod can be down during the update process, ensuring that at least one pod is always available to handle traffic.
* **maxSurge: 1:** ✅ This allows Kubernetes to temporarily spin up one extra pod (total of 3) during a version rollout to ensure a smooth transition with zero downtime.

### 3. Horizontal Pod Autoscaler (HPA)
* **Min: 2 / Max: 5:** ✅ The app starts with 2 replicas for high availability and can scale up to 5 pods during high-load periods.
* **Metrics (70% CPU/Memory):** ✅ Scaling is triggered when average utilization exceeds 70%, leaving a 30% buffer to handle request spikes while new pods are being initialized.

### 4. Health Probe Verification & Pod Spec Parity
✅ The application endpoints `/api/health` and `/api/ready` have been verified. The standalone Pod manifest and the Deployment spec are now perfectly aligned.

#### Verified Endpoints:
* **Liveness Probe:** `http://<pod-ip>:8080/api/health` — Verified via `curl`, returns `200 OK`. ✅
* **Readiness Probe:** `http://<pod-ip>:8080/api/ready` — Verified via `curl`, returns `200 OK`. ✅

#### Infrastructure Requirements for HPA:
* **Metrics Server:** The cluster must have `metrics-server` installed for CPU/Memory scaling. ✅
* **API Version:** Uses `autoscaling/v2` for multi-metric support. ✅

### 5. Image Availability & Reproducibility
✅ The deployment uses the image `ikulyk404/todoapp:3.0.0`. 

* **Public Access:** This image is hosted on Docker Hub and is publicly available. No `imagePullSecrets` or authentication is required to pull the image. ✅
---

## How to Access the App

Once deployed, the application is accessible via **NodePort**:

1.  **Docker Desktop / Minikube:**
    * Open your browser at: `http://localhost:30080` ✅
2.  **Remote Cluster:**
    * Find your Node IP: `kubectl get nodes -o wide`
    * Access via: `http://<NODE_IP>:30080` ✅

You can verify the status of all components using:
`kubectl get all -n mateapp`