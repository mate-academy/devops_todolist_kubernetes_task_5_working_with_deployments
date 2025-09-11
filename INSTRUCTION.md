
## 1. Kubernetes Manifests

This deployment uses two primary manifest files: `deployment.yml` to define the application's state and `hpa.yml` to
configure its autoscaling behavior

### 3.1. `deployment.yml` - Application Deployment

This manifest defines the desired state for the ToDo application, including resource allocation and update strategies

*   **Resource Requests:**
    *   **CPU: `100m`** - This is the minimum CPU required for the Django application to run in an idle state
    *   **Memory: `128Mi`** - Sufficient memory for Django to load and perform basic operations
*   **Resource Limits:**
    *   **CPU: `500m`** - Prevents a single pod from consuming excessive CPU resources
    *   **Memory: `256Mi`** - Provides enough headroom to handle multiple concurrent requests without risking Out Of Memory errors

### 1.2. `hpa.yml` - Horizontal Pod Autoscaler (Rolling Update Deployment Strategy)

A `RollingUpdate` strategy ensures zero-downtime deployments by incrementally updating pods with the new version.
This configuration provides a resilient and highly available setup with seamless updates and quick rollbacks.
The HPA automatically adjusts the number of running pods based on observed CPU and memory usage

*   **`minReplicas: 2`**: Maintains a minimum of two pods for high availability and load distribution
*   **`maxReplicas: 5`**: Caps the total number of pods to prevent uncontrolled resource consumption
*   **Target Metrics:**
    *   **CPU Utilization at 70%**: A common threshold for web applications that triggers a scale-up event, leaving sufficient headroom for sudden traffic spikes
    *   **Memory Utilization at 80%**: A safe threshold to prevent OOM errors, considering the memory management characteristics of Python applications

The HPA is configured with scaling policies to react quickly to increased load and cautiously when load decreases

## 2. Deployment Steps

Follow these steps to deploy the application to your cluster:

### Step 1: Create a Namespace and set it as current

Isolate the application resources in a dedicated namespace

```bash
kubectl create namespace mateapp
```

```bash
kubectl config set-context --current --namespace=mateapp
```

### Step 2: Apply the Manifests

Apply the deployment and HPA configurations to the cluster

```bash
# Apply the deployment configuration
kubectl apply -f deployment.yml

# Apply the Horizontal Pod Autoscaler configuration
kubectl apply -f hpa.yml
```

### Step 3: Verify the Deployment

Check the status of the created resources to ensure everything is running correctly

```bash
# Check the status of the pods
kubectl get pods

# Check the status of the deployment
kubectl get deployment

# Check the status of the HPA
kubectl get hpa
```

## 3. Accessing the Application

#### **Option 1: `port-forward` (For Development & Testing)**

Forward traffic from your local machine to the service

```bash
kubectl port-forward service/todoapp-service 8080:80
```

The application will be accessible at `http://localhost:8080`

#### **Option 2: `NodePort` (For Simple External Access)**

Expose the application directly on a static port on each cluster node. Modify the service type to `NodePort`

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080 # Example port
```

## 4. Monitoring and Maintenance

### Checking Logs

Stream the logs from all application pods

```bash
kubectl logs -l app=todoapp -f
```

### Describing HPA Status

Get detailed information about the HPA's current state and events

```bash
kubectl describe hpa todoapp-hpa
```

### Manual Scaling

If needed, you can manually override the replica count of the deployment with:

```bash
kubectl scale deployment todoapp-deployment --replicas=3
```
