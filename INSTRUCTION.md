# Deploying the ToDo app to Kubernetes

## Deploy

Requires a running cluster, `kubectl`, and metrics-server (needed by the HPA; on Minikube: `minikube addons enable metrics-server`).

```
kubectl create namespace mateapp
kubectl apply -f infrastructure/deployment.yml
kubectl apply -f infrastructure/hpa.yml
kubectl get deployment,pods,hpa -n mateapp
```

In the idle state there should be 2 `todoapp` pods in `Running` state.

## Access the app

No Service is defined, so use port-forwarding (the container listens on 8080):

```
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

Then open http://localhost:8080/ (landing page) or http://localhost:8080/api/ (API).

## Resource requests and limits

- **Requests: 125m CPU, 256Mi memory.** A Django app uses little CPU when idle but needs a fair amount of memory. Two idle pods reserve only 250m CPU and 512Mi, so they fit on a small cluster. The memory request is high enough that idle usage stays below the HPA target.
- **Limits: 250m CPU, 512Mi memory** (2x the requests). This leaves room for bursts while the HPA reacts, and a leaking pod is OOM-killed instead of exhausting the node.

## HPA configuration

- **Min 2 / max 5 pods:** 2 matches the idle requirement and gives high availability. 5 caps cost and cluster usage (2.5x the baseline capacity).
- **CPU and memory at 70% average utilization:** with several metrics the HPA uses the one that requires the most pods, so the app scales if either is under pressure. 70% leaves about 30% headroom for new pods to start before the existing ones are saturated. Utilization is measured against the requests (about 88m CPU and 179Mi memory per pod).

## Strategy configuration

`RollingUpdate` with `maxUnavailable: 1` and `maxSurge: 1`:

- With 2 replicas, at least 1 pod always serves traffic during an update (no downtime).
- At most 3 pods exist at once, so the rollout is fast and extra resource use is small.
- `0` is not valid for both values, and larger numbers would either take all pods down or double the resource usage.
