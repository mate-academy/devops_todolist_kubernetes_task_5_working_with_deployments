# Kubernetes Deployment Instructions for Django ToDo List

## 1. How to Deploy the App to Kubernetes

1. Ensure you have a running Kubernetes cluster and `kubectl` configured.
2. Build and push your Docker image for the Django app to a container registry accessible by your cluster.
3. Create the namespace:
   ```bash
   kubectl apply -f .infrastructure/namespace.yml
   ```
4. Apply the deployment manifest:
   ```bash
   kubectl apply -f .infrastructure/deployment.yml -n mateapp
   ```
5. Apply the HPA manifest:
   ```bash
   kubectl apply -f .infrastructure/hpa.yml -n mateapp
   ```

## 2. Resource Requests and Limits

- **Requests:**
  - Memory: `128Mi`
  - CPU: `250m` (0.25 core)
- **Limits:**
  - Memory: `256Mi`
  - CPU: `500m` (0.5 core)
  

**Reasoning:**
- The app is lightweight and can run with minimal resources in idle state.
- Requests ensure the pod gets enough resources to start and run reliably.
- Limits prevent a single pod from consuming excessive resources, protecting cluster stability.

## 3. HPA Configuration

- **Min Pods:** 2
- **Max Pods:** 5
- **Metrics:** CPU (target 70%) and Memory (target 70%)

**Reasoning:**
- Minimum 2 pods for high availability and redundancy.
- Maximum 5 pods to control scaling and cost.
- Autoscaling on both CPU and memory ensures responsiveness to real workload spikes.

## 4. Deployment Strategy Configuration

- **Type:** RollingUpdate
- **MaxUnavailable:** 1
- **MaxSurge:** 1

**Reasoning:**
- RollingUpdate ensures zero downtime during updates.
- `maxUnavailable: 1` allows only one pod to be unavailable at a time, maintaining service.
- `maxSurge: 1` allows one extra pod during updates for smooth rollout.

## 5. How to Access the App After Deployment

- Check todoapp pod name:
  kubectl get pods -l app=todoapp -n mateapp
- Execute port-forward to access the app pod:
  kubectl port-forward pod/<pod-name> 8000:8080
- Go to the app page via browser:
  localhost:8000/ 

## 6. List of actions and their commands to track deployment state:
 - kubectl get ns mateapp
 - kubectl get deployments -n mateapp
 - kubectl get pods -n mateapp -l app=todoapp
 - kubectl get hpa -n mateapp
 - kubectl rollout status deployment/todoapp -n mateapp
