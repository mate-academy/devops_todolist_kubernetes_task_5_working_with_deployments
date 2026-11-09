# How to deploy the ToDo app to Kubernetes

## What you need

- A running Kubernetes cluster (minikube, kind or Docker Desktop)
- `kubectl`
- metrics-server (without it, the autoscaler can't see CPU and memory).
  On minikube: `minikube addons enable metrics-server`

## Deploy

```bash
kubectl apply -f namespace.yml
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

Check:

```bash
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
```

You should see 2 `todoapp` pods in `Running` status.

## Open the app

```bash
kubectl port-forward -n mateapp deployment/todoapp 8000:8080
```

Then open http://localhost:8000/ (API: http://localhost:8000/api/).

## Resources

| | Request | Limit |
|---|---|---|
| CPU | 100m | 250m |
| Memory | 256Mi | 512Mi |

- Request = what the pod is guaranteed. The app is small, so this is enough.
- Limit = the most the pod can use. It leaves room for busy moments, but stops one pod from taking everything.
- The autoscaler counts usage as a percentage of the request, so the request must not be too small.

## Autoscaler (HPA)

- **Min 2 pods:** if one pod fails, the other still works.
- **Max 5 pods:** the app can grow under load, but won't use up the whole cluster.
- **CPU 70% / Memory 80%:** add pods before the existing ones are full. Memory is a bit higher because it changes slowly.
- If CPU and memory disagree, the autoscaler picks the one that needs more pods.

## Update strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

- **RollingUpdate:** pods are replaced one by one, so the app stays online.
- **maxUnavailable: 0:** an old pod is removed only after the new one is ready, so there are always 2 working pods.
- **maxSurge: 1:** only 1 extra pod at a time, so the update uses few resources.