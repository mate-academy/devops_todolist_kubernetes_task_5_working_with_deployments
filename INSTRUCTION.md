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

Requests (memory: 64Mi, cpu: 100m):
These ensure the pod has enough "guaranteed" resources to start the Django server without being killed immediately.
64Mi is a lean baseline for a Python application.

Limits (memory: 128Mi, cpu: 500m):
Limits prevent a single pod from consuming all node resources (memory leaks or heavy processing).
The 128Mi limit allows for occasional spikes during request processing or media handling.

Min Replicas (2): Ensures high availability even if one pod fails or is being updated.

Max Replicas (5): Provides a safety ceiling to prevent infinite scaling that could exhaust the cluster's resources.


Metrics (CPU 50% / Memory 70%):

CPU at 50% is a standard threshold to trigger scaling before the user notices latency.

Memory at 70% triggers scaling when the application starts holding significant data in RAM, ensuring we add more pods before hitting the 128Mi hard limit.

## Deploymnt strategy:

RollingUpdate was chosen to ensure zero downtime.

## Hot to access the app

The application is exposed via a NodePort service.
Run ``` kubectl get svc -n mateapp ``` to find the assigned Port (range 30000-32767).
Access the app in your browser at: http://localhost:<YOUR_NODEPORT>.
