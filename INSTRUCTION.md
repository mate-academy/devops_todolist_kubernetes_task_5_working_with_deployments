# Instructions for Deploying the ToDo App to Kubernetes

## Deployment Instructions

1. Ensure you have a Kubernetes cluster set up and `kubectl` configured to interact with it.
2. Create the `mateapp` namespace:
   kubectl apply -f .infrastructure/namespace.yml
3. Apply the deployment manifest:
   kubectl apply -f .infrastructure/deployment.yml
4. Apply the nodeport manifest:
   kubectl apply -f .infrastructure/nodeport.yml
4. Apply the Horizontal Pod Autoscaler (HPA) manifest:
   kubectl apply -f .infrastructure/hpa.yml
5. Verify the deployment:
   kubectl get all -n mateapp
   kubectl get hpa -n mateapp

## Resource Requests and Limits

- **CPU Requests**: 250m (ensures the app gets a guaranteed minimum CPU allocation).
- **Memory Requests**: 64Mi (ensures the app has enough memory to operate in idle state).
- **CPU Limits**: 500m (prevents the app from consuming excessive CPU resources).
- **Memory Limits**: 128Mi (prevents the app from using too much memory).

These values were chosen to balance resource efficiency and application performance under typical workloads.

## HPA Configuration

- **Minimum Pods**: 2 (ensures high availability and redundancy).
- **Maximum Pods**: 5 (limits resource usage during peak loads).
- **Scaling Metrics**:
  - CPU utilization: 70%
  - Memory utilization: 70%

This configuration ensures the application scales dynamically based on demand while maintaining resource efficiency.

## Deployment Strategy

Strategy RollingUpdate
    MaxUnavailable: 1 (ensures at least one pod is always available during updates).
    MaxSurge: 1 (allows one additional pod to be created during updates).

    This strategy minimizes downtime and ensures a smooth update process.

## Accessing the Application

You could access the app by nodeport with: `http://localhost:30080`.
