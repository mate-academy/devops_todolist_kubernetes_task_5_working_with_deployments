# Instructions

## Deploying the app to Kubernetes

1. Enable the metrics-server addon (required for HPA to read CPU/memory metrics):

```bash
minikube addons enable metrics-server
```

2. Apply the manifests:

```bash
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

3. Verify everything is running:

```bash
kubectl get deploy,hpa,pods -n mateapp
```

## Resource requests and limits

The application is lightweight (Django + SQLite) and doesn't consume many resources, so I set modest requests — 100m CPU and 128Mi memory. Limits are a bit higher (250m / 256Mi) so a pod can handle short spikes without taking over all the node's resources. The modest values were also chosen because the HPA can scale up to 5 pods — with large requests they simply wouldn't fit in the cluster.

## HPA configuration

A minimum of 2 pods provides fault tolerance: if one fails, the other keeps serving traffic. A maximum of 5 is a reasonable upper bound for the load, beyond which there's no benefit (and cluster resources are limited). The 80% threshold lets the app scale up preventively — before it starts lagging or hitting OOM — rather than reacting at 100%. CPU and memory metrics cover both main types of load. Note: 80% is measured against requests — with cpu 100m, the threshold triggers at 80m.

## Strategy configuration

RollingUpdate with maxUnavailable 1 and maxSurge 1 provides zero-downtime updates. With two pods, maxUnavailable 1 guarantees at least one pod is always available, and maxSurge 1 allows temporarily adding one extra pod (3 at peak) so capacity isn't lost during the update.

## Accessing the app

For quick access, forward a port to the deployment:

```bash
kubectl port-forward deployment/todoapp-deployment 8080:8080 -n mateapp
```

The app is then available at `http://localhost:8080`. This is a debugging method; for persistent access you'd create a Service (ClusterIP or NodePort), as in the previous task.