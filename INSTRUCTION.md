From the repository root:

```
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
```

Check that it works:

```
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
```

HPA needs metrics-server in the cluster. Without it, `TARGETS` shows `<unknown>`.

## Access the app

Pods are not exposed outside the cluster, so I use port-forward:

```
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

Then open http://localhost:8080 in the browser.

## Requests and limits

- requests: 100m CPU, 128Mi memory
- limits: 250m CPU, 256Mi memory

The app is small, so 100m CPU and 128Mi are enough when it is idle. Limits are higher so the app can handle short load peaks. If a pod uses more memory than the limit, it is killed and restarted. If it uses too much CPU, it is only slowed down.

## HPA

- min pods: 2, because the task needs 2 pods when idle
- max pods: 5, so the app does not take too many cluster resources
- CPU target: 70% of requests
- memory target: 80% of requests

The HPA adds pods when either CPU or memory goes above its target. CPU has a lower target because it grows faster under load, while memory usually stays high after a peak.

## Strategy

``` yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

- `maxSurge: 1`: during an update, one extra pod can be created (3 pods in total).
- `maxUnavailable: 0`: an old pod is stopped only after the new one is ready.

With 2 replicas this means the app is never fully down during an update. The update is a bit slower, but safer.