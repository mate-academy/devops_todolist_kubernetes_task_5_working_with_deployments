# Task 5: Working with Deployments

## 1. Deploy the application to Kubernetes

Apply the manifests in this order (the namespace must exist first):

```
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
```

All new resources belong to the `mateapp` namespace. Check the result:

```
kubectl get pods,deploy,hpa -n mateapp
```

In the idle state there are 2 pods in `Running` (1/1) and the deployment shows `2/2`.

## 2. Access the application after deployment

There is no Service in this task, so use port-forward to the deployment:

```
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

Then open `http://localhost:8080/` in the browser (the ToDo landing page,
"Dead simple Todolists"). The health endpoint `http://localhost:8080/api/health`
returns `Health OK`. Stop the forwarding with `Ctrl + C`.

## 3. Resource requests and limits

```
requests: cpu 100m, memory 128Mi
limits:   cpu 250m, memory 256Mi
```

- The ToDo app is a small Django application that is mostly idle, so a small
  guaranteed amount is enough: `requests` is what the scheduler reserves for each pod.
  Two idle pods reserve only 200m CPU and 256Mi memory, which is fine for a local cluster.
- `limits` are the ceiling: 2.5x the CPU request and 2x the memory request leave room for
  short load peaks, but protect the node from a single pod taking all resources.
  CPU above the limit is throttled; memory above the limit kills the pod (OOMKilled),
  so the memory limit has a reasonable headroom.
- The HPA calculates utilization as a percentage of the `requests`, so small requests
  make the autoscaler react earlier.
- These numbers are an estimate for a small app. In a real project they should be
  checked with `kubectl top pods` under real load and adjusted.

## 4. Horizontal Pod Autoscaler configuration

```
minReplicas: 2, maxReplicas: 5
CPU: averageUtilization 70%, memory: averageUtilization 80%
```

- Minimum 2 pods equals the idle state from the task and keeps the app available
  if one pod fails or is being updated. Maximum 5 pods limits the resources the
  autoscaler can use.
- Autoscaling is triggered by both CPU and memory (the task requirement). The HPA scales
  up when any of the two metrics exceeds its target.
- The thresholds are below 100% so that new pods start before the existing ones are
  saturated. The memory threshold is higher (80%) because memory usage of a running
  app is usually stable and does not drop quickly after a peak.
- The HPA needs metrics-server to read CPU and memory. In a plain Docker Desktop cluster
  it is not installed, so `TARGETS` shows `<unknown>` until metrics-server is added.

## 5. Update strategy

```
strategy: RollingUpdate, maxSurge: 1, maxUnavailable: 0
```

- With 2 replicas an update first creates 1 extra pod (3 in total, `maxSurge: 1`),
  waits until it is Ready (readinessProbe), and only then removes an old pod.
- `maxUnavailable: 0` means no pod is removed before its replacement is ready, so there
  are always at least 2 ready pods and the application has no downtime.
- The price is slower rollout and one extra pod of resources during the update.

To update the app change the image tag in `deployment.yml`, run `kubectl apply -f`
again and watch the rollout:

```
kubectl rollout status deployment/todoapp -n mateapp
```
