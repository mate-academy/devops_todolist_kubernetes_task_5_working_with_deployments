# Todoapp Deployment to Kubernetes

This guide provides instructions on deploying the Todoapp to Kubernetes, including setting up a Horizontal Pod Autoscaler (HPA) and explaining the configurations.

## Prerequisites

- Kubernetes cluster
- kubectl configured to communicate with your cluster
- Docker

## Deployment Instructions

### Step 1: Clone the Repository

Clone the repository to your local machine:

```bash
git clone [repository URL]
cd [repository directory]
```

Replace [repository URL] and [repository directory] with your repository's URL and local directory name, respectively.

### Step 2: Create the Namespace

Ensure the mateapp namespace exists in your Kubernetes cluster:

```bash
kubectl create namespace mateapp
```

If the namespace already exists, this command will simply return a message stating that the namespace is unchanged.

### Step 3: Apply Kubernetes Manifests

Deploy the application and its resources to the Kubernetes cluster:
```bash
kubectl apply -f deployment.yml -n mateapp
kubectl apply -f hpa.yml -n mateapp
```

## Configuration Explanations

### Deployment Configuration
1. **Strategy**: 
   - We use the `RollingUpdate` strategy to update the pods with zero downtime. By setting `maxUnavailable` to 1, we ensure that only one pod at a time is taken down during the update process. Similarly, `maxSurge` is set to 1, allowing only one extra pod to be created above the desired number of pods. This approach helps in maintaining service availability while updating or scaling the application.
   
2. **Resources**:
   - **Requests**: These are the minimum resources required by each pod. Setting the CPU request at `100m` means that each pod is guaranteed at least 100 millicores of CPU. Similarly, a memory request of `200Mi` ensures at least 200 MiB of memory is allocated to each pod. These values are crucial for the Kubernetes scheduler to place pods on nodes with sufficient resources.
   - **Limits**: These values define the maximum amount of resources a pod can use. A CPU limit of `500m` prevents any single pod from using more than half a CPU core, while a memory limit of `500Mi` caps the memory usage. This is important to prevent any single application from monopolizing the cluster's resources, ensuring stability and availability of other services in the cluster.

### Horizontal Pod Autoscaler (HPA) Configuration
1. The HPA dynamically adjusts the number of replicas based on real-time metrics. It ensures that the deployment can handle varying loads efficiently without manual intervention.
2. The HPA is configured to scale the number of pods between a minimum of 2 and a maximum of 5. It targets a resource utilization of 70% for both CPU and memory. When the average CPU or memory usage across all pods exceeds 70%, the HPA will start to create additional pods, up to a maximum of 5. Conversely, if the usage drops below this threshold, the HPA will reduce the number of pods, ensuring resource efficiency.

### Accessing the Application
Apply the Kubernetes manifests by running the following command in the terminal:
```bash
kubectl apply -f .infrastructure/
```

## Testing

### Using Port Forwarding
1. Check your services
   ```bash
    kubectl get svc -n todoapp
   ```
2. Forward the port from the todoapp pod to your local machine:
    ```bash
    kubectl port-forward service/{service_name} 8081:80 -n todoapp
    ```
3. Open your browser and visit ```http://localhost:8081``` to access the Todoapp.

### Test ClusterIP using Busybox Container

1. Access the shell of the busybox pod:
    ```bash
    kubectl -n todoapp exec -it busybox -- sh
    ```
2. Test the connection to the ToDo app:
    ```bash
   curl http://{service_name}.todoapp.svc.cluster.local
    ```

### Test app with NodePort

1. Use the following command to find out the NodePort assigned to your service:
    ```bash
    kubectl get svc -n todoapp
    ```
2. Note the NodePort next to your service. For testing, use the address format http://{NodeIP}:{NodePort}
3. Open a browser and navigate to the NodePort URL, for example:
    ```bash
    http://localhost:30080
    ```

### Confirm the deployment, service, and HPA are functioning correctly:

- Check the deployment status:
```bash
  kubectl get deployments -n mateapp
```
- Verify the HPA is active:s:
```bash
 kubectl get hpa -n mateapp
```
