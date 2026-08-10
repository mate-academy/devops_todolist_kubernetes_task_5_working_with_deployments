# TodoApp Kubernetes Deployment

## 1. Create the namespace

Create the `mateapp` namespace:

```bash
kubectl create namespace mateapp
```

## 2. Deploy the application

Apply the Deployment:
```bash
kubectl apply -f deployment.yml
```

Apply the Horizontal Pod Autoscaler:
```bash
kubectl apply -f hpa.yml
```

## 3. Check the deployment

Check the Deployment:
```bash
kubectl get deployment -n mateapp
```

Check the Pods:
```bash
kubectl get pods -n mateapp
```

## 4. Check the HPA
```bash
kubectl get hpa -n mateapp
```

## Resource requests and limits
The requests are intentionally relatively small because the TodoApp is a lightweight application. 
They allow Kubernetes to schedule two pods without requiring excessive cluster resources.

The limits prevent a single application pod from consuming an unlimited amount of CPU or memory.

The CPU limit of 500m and memory limit of 128Mi provide enough resources for normal application operation while keeping resource usage controlled.

## HPA configuration

1. The minimum number of replicas is 2 - this ensures that two application pods are available even when the application is idle and provides basic redundancy.


2. The maximum number of replicas is 5 - five pods should provide enough capacity for increased traffic while preventing uncontrolled resource consumption.


3. The target utilization for both CPU and memory is 70%.
When resource utilization increases above the target, Kubernetes can increase the number of pods. When the load decreases, Kubernetes can scale the application back down, but never below 2 pods.

## Deployment strategy

The Deployment uses the RollingUpdate strategy:

- maxUnavailable 0 means that Kubernetes should not intentionally make existing pods unavailable during an update.

- maxSurge 1 allows Kubernetes to create one additional pod during the update.

This configuration provides a safer update process because the new version can start before an old pod is removed. It also helps keep the application available during deployment.

## How to access the application

First, check the running pods:

```bash
kubectl get pods -n mateapp
```

To access the application locally, use port forwarding:
```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

Then open http://localhost:8080