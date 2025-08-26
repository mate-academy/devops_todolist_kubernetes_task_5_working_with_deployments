# Instructions: Deploying Django ToDo App to Kubernetes

## 1. Prerequisites

- Kubernetes cluster (local or cloud)
- `kubectl` configured for your cluster

## 2. Namespace Creation

First, create the namespace for the app:

```sh
kubectl apply -f .infrastructure/namespace.yml
```

## 3. Deploy the Application

Apply the deployment manifest:

```sh
kubectl apply -f .infrastructure/deployment.yml
```

### Strategy Configuration

- **Type:** RollingUpdate
- **maxUnavailable:** 1
- **maxSurge:** 1

**Reasoning:**  
RollingUpdate ensures zero downtime during updates. Setting `maxUnavailable: 1` and `maxSurge: 1` allows only one pod to be unavailable or added at a time, minimizing resource spikes and ensuring availability.

### Resource Requests and Limits

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

**Reasoning:**  
- **Requests:** The app is lightweight; 64Mi memory and 250m CPU ensure it runs smoothly even under low load.
- **Limits:** 128Mi memory and 500m CPU prevent resource overuse, protecting cluster stability.

## 4. Expose the Application

Choose one of the provided service manifests:

- For internal access (ClusterIP):  
  ```sh
  kubectl apply -f .infrastructure/clusterIp.yml
  ```
- For external access (NodePort):  
  ```sh
  kubectl apply -f .infrastructure/nodeport.yml
  ```

## 5. Enable Autoscaling

Apply the Horizontal Pod Autoscaler (HPA):

```sh
kubectl apply -f .infrastructure/hpa.yml
```

### HPA Configuration

- **minReplicas:** 2
- **maxReplicas:** 5
- **CPU & Memory Utilization Target:** 50%

**Reasoning:**  
- **minReplicas: 2** ensures high availability.
- **maxReplicas: 5** limits resource usage.
- **CPU/Memory at 50%** triggers scaling when pods are moderately loaded, balancing performance and cost.

## 6. Health and Readiness Probes

- **Liveness Probe:** Checks `/api/health` endpoint.
- **Readiness Probe:** Checks `/api/ready` endpoint.

These endpoints ensure Kubernetes only routes traffic to healthy and ready pods.

## 7. Accessing the App

- **NodePort:**  
  Access via `http://<NodeIP>:30080/`
- **ClusterIP:**  
  Use port forwarding:  
  ```sh
  kubectl port-forward svc/todoapp -n todoapp 8080:80
  ```
  Then open `http://localhost:8080/`

## 8. Verify Deployment

Check pod status:

```sh
kubectl get pods -n mateapp
```

Check service endpoints:

```sh
kubectl get svc -n todoapp
```

## 9. Additional Notes

- The app image is `ikulyk404/todoapp:3.0.0`
- Database is SQLite (no external DB required)
- For troubleshooting, check pod logs:
  ```sh
  kubectl logs <pod-name> -n mateapp
  ```

---
**Summary:**  
This configuration ensures the Django ToDo app is highly available, resource-efficient, and scalable. Rolling updates, resource limits, and autoscaling provide reliability and cost control. Access is available via NodePort or port-