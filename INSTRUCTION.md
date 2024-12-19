## Deployment Instructions

### Steps to Deploy

1. **Create the Namespace:**
   ```sh
   kubectl create namespace mateapp
   ```

2. **Apply the Deployment:**
   ```sh
   kubectl apply -f deployment.yml
   ```

3. **Apply the Horizontal Pod Autoscaler (HPA):**
   ```sh
   kubectl apply -f hpa.yml
   ```

4. **Verify the Deployment:**
   Check the status of pods and ensure they are running:
   ```sh
   kubectl get pods -n mateapp
   ```

5. **Verify the HPA:**
   Confirm that the HPA is configured and scaling is enabled:
   ```sh
   kubectl get hpa -n mateapp
   ```

### Accessing the Application
- By default, the application will not have an external service to expose it.
- To access it, you can:
  - Create a Kubernetes service of type `LoadBalancer` or `NodePort`.
  - Port-forward the application to your local machine for testing:
    ```sh
    kubectl port-forward -n mateapp deployment/todoapp 8080:8080
    ```
  - Open `http://localhost:8080` in your browser.

## Configuration Choices Explained

### Resource Requests and Limits
- **Requests:** Ensure that each pod has at least `125m` CPU and `64Mi` memory allocated.
- **Limits:** Prevent overuse of resources by capping at `125m` CPU and `64Mi` memory.

### HPA Configuration
- **Minimum Pods:** Set to 2 to maintain availability during idle times.
- **Maximum Pods:** Capped at 5 to balance scaling and resource costs.
- **Scaling Triggers:** Autoscaling is based on 85% utilization for both CPU and memory.

### RollingUpdate Strategy
- **maxUnavailable: 1:** Ensures at least one pod remains available during updates.
- **maxSurge: 1:** Allows for one additional pod to be created during updates for smooth transitions.
