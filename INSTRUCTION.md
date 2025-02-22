
### To deploy the app to Kubernetes, run the following commands:
```bash
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
```

### Horizontal Pod Autoscaling:
- For horizontal pod scaling, I chose the minimum of `2` and the maximum of `5` replicas.

### Resource Allocation for the Application:
- I tested the application deployment on a separate server with the following specs:
  - `6GB` of RAM
  - `Intel i5` 3rd generation processor.
  
Based on the available resources and the expected load, I decided to allocate the following resources for each pod:

#### Requested Resources:
- **Memory**: `64Mi`
- **CPU**: `128m`

#### Limit Resources:
- **Memory**: `128Mi`
- **CPU**: `250m`

### Reasoning:
- The `request` values specify the minimum amount of resources required by the pod, ensuring it has enough capacity to start and run the app.
- The `limit` values set the maximum amount of resources a pod can use. This helps avoid overloading the node.