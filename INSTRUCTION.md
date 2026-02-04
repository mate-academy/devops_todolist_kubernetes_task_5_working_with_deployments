# ToDo App Deployment Instructions

## 1. How to deploy
Run the following commands to set up the environment and deploy the application:

```bash
# Apply the Namespace
kubectl apply -f .infrastructure/namespace.yml

# Apply the Deployment (Application)
kubectl apply -f .infrastructure/deployment.yml

# Apply the Service (for access)
kubectl apply -f .infrastructure/service.yml

# Apply the Horizontal Pod Autoscaler
kubectl apply -f .infrastructure/hpa.yml
```
## 2. Configuration Explanations

## Resource Requests and Limits strategy
I configured resources based on the requirement to have stability and prevent resource starvation:

Requests (CPU: 100m, Memory: 128Mi): This guarantees that the pod gets the minimum required resources to start the Django application properly.

Limits (CPU: 500m, Memory: 256Mi): These limits prevent a single pod from consuming all node resources in case of a memory leak or infinite loop, ensuring the cluster remains stable.

## HPA Configuration
Min Replicas (2): Ensures High Availability. If one pod fails, the other continues to serve traffic.

Max Replicas (5): Allows the application to handle traffic spikes up to 2.5x the normal load.

Triggers (CPU 70%, Memory 80%): Scaling is triggered before the pod hits 100% of its limits. This proactive scaling prevents the application from becoming slow or crashing (OOMKilled) before new pods are ready.

## Deployment Strategy
I used RollingUpdate with:

MaxSurge: 1 (Creates new pods before deleting old ones)

MaxUnavailable: 0 (Ensures 100% availability during updates) This strategy ensures zero downtime deployments.

## 3. Accessing the App
The application is exposed via a NodePort service. You can access it at: http://localhost:30008