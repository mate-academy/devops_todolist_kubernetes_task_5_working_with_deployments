# INSTRUCTION.md

## Overview

This guide describes how to deploy the ToDo application to a Kubernetes cluster using the provided `deployment.yml` and `hpa.yml` manifests. The application will run in the `mateapp` namespace and utilize resource requests and limits, a rolling update strategy, and horizontal pod autoscaling based on CPU and memory usage.

---

## Prerequisites

- Access to a Kubernetes cluster (v1.23 or newer is recommended)
- `kubectl` configured to access your cluster
- Metrics Server installed in the cluster (required for HPA to work)
- Namespace `mateapp` created:

```bash
kubectl create namespace mateapp
```

---

## Deployment Instructions

1. Apply the deployment manifest:

```bash
kubectl apply -f deployment.yml
```

2. Apply the autoscaler manifest:

```bash
kubectl apply -f hpa.yml
```

3. Confirm that everything is working:

```bash
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
kubectl get deployment -n mateapp
```

---

## Resource Requests and Limits Explanation

The application is lightweight and doesn’t require many resources. That’s why the resource requests are set to 64Mi of memory and 250m of CPU, which is enough for normal operation under low to medium load. The limits are set to 128Mi of memory and 500m of CPU to provide some headroom during short bursts of activity.

This configuration ensures efficient resource usage while preventing the pod from overloading the cluster in case of unexpected load.

---

## HPA Configuration Explanation

The Horizontal Pod Autoscaler is configured to maintain at least 2 replicas at all times and scale up to 5 replicas when necessary. Autoscaling is triggered by both CPU and memory utilization — scaling up occurs when CPU usage goes above 75% or memory usage above 70%.

This ensures better responsiveness to different types of load: CPU-bound tasks (like computations) and memory-bound tasks (like caching or large responses).

Setting 2 as the minimum ensures high availability, even if one pod goes down. The limit of 5 prevents the app from consuming excessive cluster resources.

---

## Deployment Strategy Explanation

The deployment uses the `RollingUpdate` strategy with `maxUnavailable` set to 1 and `maxSurge` also set to 1. This means during an update:

- A maximum of one pod can be unavailable at any time.
- A maximum of one additional pod can be created temporarily.

This allows for zero-downtime deployments with minimal risk: there's always at least one pod running during an update.

---

## Accessing the App After Deployment

To access the app, you can expose it using a Kubernetes Service of type `NodePort` or set up an Ingress resource if you're using an ingress controller like NGINX. Here’s a quick way using `NodePort`:

1. Create a service (you can name it `service.yml`) that targets port 8080 and exposes it via NodePort (e.g., port 30080).
2. Apply the service:

```bash
kubectl apply -f service.yml
```

3. Open the app in your browser using:

```
http://<NodeIP>:30080
```

To get the IP address of a node, run:

```bash
kubectl get nodes -o wide
```

If you're using a cloud provider, you can instead expose the app using a `LoadBalancer` type service or Ingress controller for production.

---

## Summary

- The ToDo app is deployed using Kubernetes manifests with RollingUpdate strategy and autoscaling.
- Resource usage is limited and optimized for lightweight operation.
- Autoscaling is triggered by both CPU and memory usage.
- The app runs in the `mateapp` namespace and is accessible via NodePort or Ingress.
- The setup ensures both high availability and efficient scaling under load.
