## Deployment steps
```
# Apply the Namespace
kubectl apply -f .infrastructure/namespace.yml

# Apply the Deployment (Application)
kubectl apply -f .infrastructure/deployment.yml

# Apply the Horizontal Pod Autoscaler
kubectl apply -f .infrastructure/hpa.yml
```

## Configuration explanations
Requests (memory: 64Mi, cpu: 250m):
These ensure the pod has enough "guaranteed" resources to start the Django server without being killed immediately.
64Mi is a lean baseline for a Python application.

Limits (memory: 128Mi, cpu: 500m):
Limits prevent a single pod from consuming all node resources (memory leaks or heavy processing).
The 128Mi limit allows for occasional spikes during request processing or media handling.

Min Replicas (2): Ensures high availability even if one pod fails or is being updated.

Max Replicas (5): Provides a safety ceiling to prevent infinite scaling that could exhaust the cluster's resources.


Metrics (CPU 70% / Memory 70%):

CPU at 70%: This threshold is chosen to maximize resource utilization. We trigger scaling only when the existing pods are significantly busy, avoiding "flapping" (unnecessary scaling up and down).

Memory at 70% triggers scaling when the application starts holding significant data in RAM, ensuring we add more pods before hitting the 128Mi hard limit.

## Deploymnt strategy:
RollingUpdate was chosen to ensure zero downtime.

## Strategy Configuration (RollingUpdate)
maxUnavailable: 1: This ensures that during an update, only one pod can be taken offline at a time. Since we start with 2 replicas, we always have at least 1 pod (50% capacity) serving traffic.

maxSurge: 1: This allows the cluster to temporarily create one extra pod (total of 3) during the update process. It ensures the new version is "Ready" before we kill the old version, achieving zero downtime.

## Hot to access the app

The application is exposed via a NodePort service.
Run ``` kubectl get svc -n mateapp ``` to find the assigned Port (range 30000-32767).
Access the app in your browser at: http://localhost:<YOUR_NODEPORT>.
