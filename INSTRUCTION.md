# Deployment instructions

## Deploy

1. Create the namespace

```
kubectl apply -f .infrastructure/namespace.yml
```

2. Deploy the app and service:

```
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/nodeport.yml
```

3. Enable autoscaling:

```
kubectl apply -f .infrastructure/hpa.yml
```

4. Verify:

```
kubectl get all -n mateapp
kubectl get hpa -n mateapp
```

## Resource requests/limits choice

- Requests are set to 250m CPU and 256Mi memory to reserve enough resources for a small Django app while keeping scheduling efficient.
- Limits are set to 500m CPU and 512Mi memory to allow short spikes without letting a single pod starve the node.
- With `replicas: 2`, the idle state keeps two pods running as required.

## HPA configuration choice

- `minReplicas: 2` keeps the baseline capacity matching the requirement and avoids cold start latency.
- `maxReplicas: 5` caps resource usage while still allowing burst handling.
- CPU and memory utilization targets at 70% balance responsiveness and stability for a web workload.

## Strategy choice

- `RollingUpdate` provides zero/low downtime during updates.
- `maxUnavailable: 1` ensures at least one pod stays available.
- `maxSurge: 1` allows one extra pod during rollout to speed updates without large resource spikes.

## Access the app

- The service is `NodePort` on port `30080`.
- Access it at `http://localhost:30080`.
