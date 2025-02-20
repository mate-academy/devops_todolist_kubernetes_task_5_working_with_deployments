## Deploying the App to Kubernetes
# 1. Create a deployment.yml file with the necessary configurations.
# 2. Create an hpa.yml file for the Horizontal Pod Autoscaler.
# 3. Apply the Deployment Manifest - kubectl apply -f deployment.yml
# 4. Deploy the Horizontal Pod Autoscaler (HPA) - kubectl apply -f hpa.yml

### Choice of Resource Requests and Limits
These are standard values for a lightweight web app. The minimum CPU (50m) and memory (64Mi) ensure basic performance. The limits (200m CPU, 128Mi memory) prevent excessive resource usage.

### Choice of HPA Configuration
Minimum replicas set to 2 to ensure high availability. Maximum replicas set to 5 to handle peak traffic. Autoscaling triggers when CPU or memory utilization exceeds 70%. This setup ensures the app scales efficiently based on workload.

### Choice of Deployment Strategy
RollingUpdate strategy ensures zero downtime during deployments. maxUnavailable: 1 → At most, one pod can be unavailable during updates. maxSurge: 1 → At most, one extra pod can be created temporarily during updates. This approach keeps the app available 24/7 while using minimal resources.

### How to Access the App After Deployment
You can access the app using: http://localhost:30007