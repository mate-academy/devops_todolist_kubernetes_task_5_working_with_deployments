# Instructions to Deploy the App to Kubernetes

This document provides detailed steps to deploy the application to Kubernetes, along with an explanation of resource
configurations, Horizontal Pod Autoscaler (HPA), and deployment strategies.

---

## 1. Prerequisites

Ensure the following tools are installed on your system:

- **kubectl**: To interact with the Kubernetes cluster.
- **kubeconfig**: Access to the cluster must be configured.
- **Docker**: To build container images.
- **A container registry**: To push and pull container images (e.g., DockerHub, AWS ECR, GCP Container Registry).

---

## 2. Steps to Deploy

### Step 1: Build and Push the Docker Image

1. Build the container image:
   ```bash
   docker build -t <your-registry>/<app-name>:<version> .
   ```

2. Push the image to the container registry:
   ```bash
   docker push <your-registry>/<app-name>:<version>
   ```

### Step 2: Apply Kubernetes Manifests

Apply the deployment, service, and other configuration files:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml
```

---

## 3. Resource Requests and Limits Explanation

Resource requests and limits ensure that each pod has sufficient resources for predictable performance, avoiding
over-utilization of the node or interference between pods.

- **Requests**: Define the guaranteed CPU and memory a pod will receive.
- **Limits**: Define the maximum CPU and memory a pod can consume. This prevents a pod from monopolizing node resources.

### Example:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

- **Reasoning**:
    - CPU request: `250m` ensures the app has a minimum baseline of computation power equivalent to 25% of a single-core
      CPU.
    - CPU limit: `500m` restricts the app from consuming more than 50% of a single-core CPU.
    - Memory request: `128Mi` guarantees adequate memory for the app to operate efficiently.
    - Memory limit: `256Mi` prevents excessive memory consumption.

These values are chosen based on application profiling during staging/testing and ensure stability for the app under
expected workloads.

---

## 4. Horizontal Pod Autoscaler (HPA) Configuration

HPA automatically adjusts the number of pod replicas to handle varying loads, improving scalability and resource
efficiency. Below is an example HPA configuration:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-name
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### Explanation:

- **minReplicas**: 2 (minimum pods ensure high availability).
- **maxReplicas**: 10 (supports expected surge in traffic).
- **target CPU utilization**: 70% (maintains an efficient CPU usage threshold).

These values control automatic scaling, ensuring responsiveness to traffic spikes without over-provisioning.

---

## 5. Deployment Strategy Configuration

Use the rolling update strategy to minimize downtime during deployments:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

### Explanation:

- **maxUnavailable: 1**: Ensures at least one pod is always available during an update.
- **maxSurge: 1**: Adds only one additional pod during the rollout to prevent overloading cluster resources.

---

## 6. Accessing the App After Deployment

Once deployed, the service provides access to the application. Example service file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: app-name
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

### Steps to Access:

1. If using a LoadBalancer:
    - Retrieve the external IP:
      ```bash
      kubectl get service app-service
      ```
    - Access the app at `http://<external-ip>`.

2. If using NodePort:
    - Retrieve the cluster's NodePort and IP address:
      ```bash
      kubectl get service app-service
      ```
    - Access the app at `http://<node-ip>:<node-port>`.

---

By following these instructions, you can deploy and manage the app in a Kubernetes cluster while ensuring scalability,
availability, and optimized resource usage.