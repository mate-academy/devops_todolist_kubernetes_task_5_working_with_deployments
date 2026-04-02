# ToDo App Deployment Instructions

## 1. Deployment Steps
To deploy the application to your Kubernetes cluster using the provided manifests, run the following commands in order:

1.  **Create the Namespace**:
    ```bash
    kubectl apply -f namespace.yml
    ```
2.  **Deploy the Application**:
    ```bash
    kubectl apply -f deployment.yml
    ```
3.  **Create the Service (NodePort)**:
    ```bash
    kubectl apply -f nodeport.yml
    ```
4.  **Enable Autoscaling**:
    ```bash
    kubectl apply -f hpa.yml
    ```

---

## 2. Resource Requests and Limits
* **Requests (CPU: 250m, Memory: 64Mi):** These represent the guaranteed resources for each pod. 250m CPU and 64Mi RAM are sufficient for the Python-based ToDo app to handle baseline requests and ensure stable startup.
* **Limits (CPU: 500m, Memory: 128Mi):** These are "hard limits" to prevent a single pod from consuming excessive node resources. This protects the cluster from potential memory leaks or unexpected CPU spikes.

## 3. HPA Configuration
* **Min Replicas (2) / Max Replicas (5):** A minimum of 2 pods ensures high availability (HA) across nodes. A maximum of 5 pods allows the app to scale during traffic peaks while staying within resource budgets.
* **Target Utilization (70%):** Thresholds for both CPU and Memory are set to 70%. This provides a buffer, allowing the HPA to spin up new pods before the existing ones become fully saturated and potentially slow down or crash.

## 4. Strategy Configuration (RollingUpdate)
* **maxUnavailable: 1:** Guarantees that at least one pod is always running and ready to serve traffic during an update process.
* **maxSurge: 1:** Allows the deployment to create one temporary additional pod. This ensures the new version is healthy before the old version is terminated, resulting in zero-downtime deployments.

## 5. How to Access the App
The application is exposed via a **NodePort** service on port **30080**. Note that the `ClusterIP` (internal IP) is only accessible within the cluster network and will not work in a browser on your host machine.
- **Via Localhost:** If your cluster environment (e.g., Docker Desktop, Kind, or Minikube) maps node ports to your host, access the app at: `http://localhost:30080`
- **Manual Port-Forward (Fallback):** If the port is not already reachable from your host machine via the control-plane container, use the following command:
    ```
    kubectl port-forward svc/todoapp 8080:8080 -n mateapp
    ```
    Then, open `http://localhost:8080` in your browser.
