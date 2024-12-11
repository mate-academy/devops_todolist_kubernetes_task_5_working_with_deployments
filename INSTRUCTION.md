## Deploy the application following the commands:
```
cd ./.infrastructure/  
kubectl apply -f namespace.yml
kubectl apply -f busybox.yml
kubectl apply -f clusterip.yml
kubectl apply -f nodeport.yml
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
kubectl apply -f todoapp-pod.yml
cd ..
```
## Check Pod Status:
```
kubectl get pods -o wide -n todoapp
```

## Go to the pod busybox:
```
kubectl -n todoapp exec -it busybox --sh
```
## Register the command:
```
curl http://todoapp-clusterip.todoapp.svc.cluster.local
```
# Requests and resource limitations.
## Resource Requests

### Memory Request: 64Mi

This value is chosen to ensure the pod has enough memory for basic operations without consuming excessive resources. The application is lightweight and doesn't require much memory during normal operation.

### CPU Request: 250m

This setting ensures the pod gets a quarter of a CPU core during typical operation, which is sufficient for handling routine tasks in a lightweight web application like this.

## Resource Limits

### Memory Limit: 128Mi

The memory limit is set to 128Mi to provide a buffer while preventing excessive memory consumption that could impact other pods running on the node. If the pod exceeds this limit, Kubernetes will terminate it, ensuring stability across the cluster.

### CPU Limit: 500m

The CPU limit is set to half a CPU core, allowing the pod to handle occasional spikes in traffic or processing needs. This limit balances performance and resource availability on the node.

# Horizontal Pod Autoscaler (HPA) configuration.
### Min Replicas: 2

Ensures the application maintains high availability by always running at least two instances.

### Max Replicas: 5

Allows the system to scale up to handle higher loads, accommodating sudden spikes in traffic.

### CPU Utilization: 70%

The HPA will trigger scaling when the average CPU usage exceeds 70%. This setting ensures pods are added before the system becomes overloaded.



# Deployment strategy configuration.

### Type: RollingUpdate
This strategy ensures that updates to the application are performed gradually, minimizing downtime.

### maxUnavailable: 2
The maximum number of replicas that may be unavailable during an update.

### maxSurge: 2
The maximum number of new replicas that can be added at the same time.

# Accessing the Application
After deploying the todoapp in the mateapp namespace, the app can be accessed using the following steps:

### Port Forwarding
Use kubectl to forward the pod’s port to a local port:

```
bash kubectl port-forward -n mateapp deployment/todoapp-deployment 8080:8080
```

#### Once port forwarding is established, access the app via:

Health Check Endpoint: http://localhost:8080/api/health
This verifies if the app is alive.

Readiness Endpoint: http://localhost:8080/api/ready
This verifies if the app is ready to serve requests.