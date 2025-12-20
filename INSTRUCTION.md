# ToDo App Deployment Instructions

## 1. How to deploy the app to Kubernetes

Prerequisites: Ensure you have `kubectl` configured and connected to your cluster.

1.  **Create the namespace** (if it does not exist yet):
    ```bash
    kubectl create namespace mateapp
    ```

2.  **Apply the Deployment manifest:**
    ```bash
    kubectl apply -f deployment.yml
    ```

3.  **Apply the Horizontal Pod Autoscaler (HPA) manifest:**
    ```bash
    kubectl apply -f hpa.yml
    ```

4.  **Verify the deployment:**
    Check if the pods are up and running (you should see 2 pods initially):
    ```bash
    kubectl get pods -n mateapp -l app=todolist
    ```

---

## 2. Resource Requests and Limits Strategy

We have configured resource boundaries to ensure stability and fair scheduling:

* **Requests (`cpu: 100m`, `memory: 128Mi`):**
    * These values guarantee that the Kubernetes scheduler will only place the pod on a node that has at least this amount of resources available.
    * **Choice:** 128Mi RAM and 0.1 CPU is a sufficient baseline for this lightweight Node.js/Java application to run in an idle state without starvation.

* **Limits (`cpu: 250m`, `memory: 256Mi`):**
    * These are the hard caps. The container is not allowed to exceed these values.
    * **Choice:** Setting the memory limit to 256Mi prevents memory leaks from consuming all available RAM on the node (protecting against "Noisy Neighbor" issues). If the app exceeds this, it will be OOMKilled and restarted automatically.

---

## 3. HPA (Horizontal Pod Autoscaler) Configuration

The autoscaler ensures the application handles traffic spikes efficiently.

* **Min Replicas: 2**
    * Ensures **High Availability**. If one pod crashes or a node becomes unavailable, the second pod continues to serve traffic.
* **Max Replicas: 5**
    * Prevents the application from consuming unlimited cluster resources during a potential bug or extreme load, controlling costs.
* **Triggers (70% CPU / Memory utilization)**
    * The autoscaler adds new pods when the average usage exceeds 70% of the requested resources.
    * **Why 70%?** This leaves a 30% "buffer zone." It takes time for new pods to boot up. Scaling at 70% ensures that by the time traffic hits 100%, the new pods are already ready to handle the load.

---

## 4. Deployment Strategy Configuration

We utilize the **RollingUpdate** strategy to ensure **Zero Downtime** during updates (e.g., changing image versions).

* **`maxUnavailable: 0`**
    * This ensures that Kubernetes will **never** terminate an old pod until a new replacement pod is fully `Ready` and passing health checks. The service capacity never drops below the desired replica count (2).
* **`maxSurge: 1`**
    * This allows Kubernetes to temporarily create 1 extra pod above the desired count (total 3) during the update process to facilitate a smooth transition.

---

## 5. How to access the app after deployment

Since this task focuses on Deployments and HPA (without an external Service), we use `port-forward` to access the application securely from the local machine.

1.  **Establish a port-forward tunnel:**
    This maps port `8080` on your local machine to port `8080` of the deployment in the cluster.
    ```bash
    kubectl port-forward deployment/todoapp-deployment 8080:8080 -n mateapp
    ```

2.  **Access the application:**
    Open your web browser or use curl to check the endpoints:
    * Health Check: http://localhost:8080/api/health
    * Main App: http://localhost:8080/