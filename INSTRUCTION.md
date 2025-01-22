# Deployment Instructions

## Namespace Creation
First, create the `mateapp` namespace:
```bash
kubectl create namespace mateapp
```

## Deployment
 Apply the deployment.yml file to deploy the ToDo app:
```bash
kubectl apply -f deployment.yml
```
# Autoscaling
Apply the Horizontal Pod Autoscaler configuration:
```bash
kubectl apply -f hpa.yml
```
# Explanation of Choices
## Resource Requests and Limits
    Requests: Ensure the app always has minimum resources (128Mi memory, 250m CPU) to run.
    Limits: Prevent resource overconsumption with upper limits (256Mi memory, 500m CPU).
## HPA Configuration
    Min Pods: 2 pods ensure redundancy and availability during low traffic.
    Max Pods: 5 pods can handle high traffic spikes efficiently.
## CPU/Memory Thresholds: Autoscaling triggers when CPU usage exceeds 75% or memory usage exceeds 70%.
    RollingUpdate Strategy
    MaxUnavailable: 1 pod ensures high availability during updates.
    MaxSurge: 1 pod speeds up updates by adding new pods before terminating old ones.

## Accessing the App
Once deployed, the app can be accessed through its service (modify as needed):
```bash
kubectl expose deployment todo-app --type=NodePort --name=todo-app-service --namespace=mateapp
```
Find the NodePort and access the app:
```bash
kubectl get service todo-app-service -n mateapp
```
Open <NODE_IP>:<NODE_PORT> in your browser.


