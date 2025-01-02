# Instructions for Deploying the Todo App to Kubernetes

## Deployment Instructions

To deploy the Todo app to Kubernetes, use the following commands:

1. Apply the `deployment.yml` file:
   ```bash
   kubectl apply -f deployment.yml
   ```
2. Apply the `hpa.yml` file:
   ```bash
   kubectl apply -f hpa.yml
   ```

## Resource Requests and Limits

We have set the following resource requests and limits for the containers:
- **CPU Request**: 250m (0.25 CPU) - This ensures the app has enough CPU to run in idle state.
- **CPU Limit**: 500m (0.5 CPU) - This limits the CPU usage to prevent excessive consumption.
- **Memory Request**: 64Mi - The app should be allocated at least 64Mi of memory to function correctly.
- **Memory Limit**: 128Mi - The app is limited to 128Mi of memory to avoid memory leaks.

These values were chosen based on typical requirements for a lightweight Django app.

## Horizontal Pod Autoscaler (HPA)

The HPA is configured to scale the app based on CPU and memory utilization:
- **Min Pods**: 2 - The minimum number of pods ensures redundancy.
- **Max Pods**: 5 - The app can scale up to 5 pods based on load.
- **CPU and Memory Utilization**: The HPA triggers scaling when CPU or memory usage exceeds 80%.

## Accessing the App

After the app is deployed, you can access it using a NodePort or ClusterIP service, depending on your setup.
- For **ClusterIP** service:
  ```bash
  kubectl port-forward svc/todolist-clusterip-service 8000:80
  ```
  Visit the app at `http://localhost:8000`.
- For **NodePort** service, use the Node's IP and port:
  ```bash
  http://<node-ip>:30000
  ```

## Conclusion

The deployment is configured for optimal performance and scalability. The app will automatically scale up or down based on resource usage, ensuring efficiency and cost-effectiveness.
