# Deploy ToDo Application to Kubernetes

## Prerequisites

* Kubernetes cluster
* kubectl configured
* Docker image pushed to Docker Hub

## Deployment

Create namespace:

```bash
kubectl create namespace mateapp
```

Apply manifests:

```bash
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

Verify deployment:

```bash
kubectl get deployments -n mateapp
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
```

## Resource Requests and Limits

The application is a lightweight Django web application.

Requests:

* CPU: 100m
* Memory: 128Mi

These values guarantee that Kubernetes reserves enough resources for stable operation even under low load.

Limits:

* CPU: 500m
* Memory: 512Mi

These limits prevent a single pod from consuming excessive cluster resources while still allowing the application to handle traffic spikes.

The deployment starts with 2 replicas to provide high availability and satisfy the task requirement that the application should run with two pods in idle state.

## HPA Configuration

Minimum replicas: 2

Maximum replicas: 5

Metrics:

* CPU utilization: 70%
* Memory utilization: 80%

The autoscaler increases the number of pods when CPU or memory usage becomes high. A minimum of two replicas ensures availability, while a maximum of five replicas prevents uncontrolled resource consumption.

## RollingUpdate Strategy

Configuration:

```yaml
maxSurge: 1
maxUnavailable: 1
```

Reasoning:

* One additional pod can be created during updates.
* At most one pod can be unavailable during rollout.
* This ensures near-zero downtime while avoiding excessive resource consumption.

## Accessing the Application

After deployment, expose the Deployment using a Service:

```bash
kubectl expose deployment todoapp-deployment \
  --type=NodePort \
  --port=80 \
  --target-port=8000 \
  -n mateapp
```

Get service details:

```bash
kubectl get svc -n mateapp
```

Access the application through the Node IP and assigned NodePort.
