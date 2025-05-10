# TodoApp Kubernetes Deployment Instructions

## Deployment Steps

1. Create the namespace:
```bash
kubectl apply -f .infrastructure/namespace.yml
```

2. Deploy the application:
```bash
kubectl apply -f .infrastructure/deployment.yml
```

3. Create the services:
```bash
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml
```

4. Deploy the Horizontal Pod Autoscaler:
```bash
kubectl apply -f .infrastructure/hpa.yml
```

## Configuration Explanations

### Resource Requests and Limits
The application is configured with the following resource specifications:

- CPU:
  - Request: 100m (0.1 CPU core)
  - Limit: 200m (0.2 CPU core)
- Memory:
  - Request: 128Mi
  - Limit: 256Mi

These values were chosen based on the following considerations:
- The application is a lightweight Python web service
- 100m CPU request ensures the pod gets enough CPU time for basic operations
- 200m CPU limit prevents any single pod from consuming too much CPU
- 128Mi memory request provides enough RAM for the Python application and its dependencies
- 256Mi memory limit prevents memory leaks from affecting the entire node

### Horizontal Pod Autoscaler Configuration
The HPA is configured with:
- Minimum replicas: 2 (ensures high availability)
- Maximum replicas: 5 (prevents excessive resource consumption)
- CPU target utilization: 70%
- Memory target utilization: 80%

This configuration:
- Maintains at least 2 pods for redundancy
- Scales up when CPU or memory usage exceeds 70-80%
- Limits maximum scaling to 5 pods to prevent cluster resource exhaustion
- Uses both CPU and memory metrics for more accurate scaling decisions

### Deployment Strategy
The deployment uses RollingUpdate strategy with:
- maxSurge: 1
- maxUnavailable: 0

This configuration:
- Ensures zero-downtime deployments
- Allows for gradual rollout of new versions
- Maintains service availability during updates
- Prevents resource spikes by limiting surge to one pod

## Accessing the Application

The application can be accessed in two ways:

1. **ClusterIP Service** (Internal access):
   - Service name: todoapp
   - Port: 80
   - Target port: 8080
   - Accessible only within the cluster

2. **NodePort Service** (External access):
   - Port: 80
   - NodePort: 30080
   - Target port: 8080
   - Accessible from outside the cluster using any node's IP address

To access the application externally:
```bash
# Get any node's IP address
kubectl get nodes -o wide

# Access the application using:
http://<node-ip>:30080
```

## Monitoring the Deployment

To monitor the deployment status:
```bash
# Check deployment status
kubectl get deployment -n todoapp

# Check pod status
kubectl get pods -n todoapp

# Check HPA status
kubectl get hpa -n todoapp

# Check services
kubectl get svc -n todoapp
```