# ToDo app: Deployment and HPA

## 1. Deploy the app

```bash
kubectl create namespace mateapp
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
```

Check the result:

```bash
kubectl get deployment,pods,hpa -n mateapp
```

In the idle state you should see `2/2` pods ready and the HPA with `MINPODS 2`, `MAXPODS 5`, `REPLICAS 2`.

### Metrics Server

HPA needs Metrics Server to read CPU and memory usage. If `kubectl top pods -n mateapp` returns an error, install it:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

On local clusters (Docker Desktop / kind) kubelet uses a self-signed certificate, so add the `--kubelet-insecure-tls` flag:

```bash
kubectl patch deployment metrics-server -n kube-system --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

After a minute `kubectl get hpa -n mateapp` shows real percentages in the `TARGETS` column instead of `<unknown>`.

## 2. Resource requests and limits

| | CPU | Memory |
| --- | --- | --- |
| requests | 100m | 256Mi |
| limits | 250m | 512Mi |

Measured in the idle state with `kubectl top pods -n mateapp`: about **15m CPU** and **63Mi memory** per pod, which is 14% of the CPU request and 24% of the memory request. Both are far below the HPA targets (70% / 80%), so the app stays at 2 pods when idle.

- **CPU request 100m (0.1 core).** In the idle state the Django dev server uses only a few millicores, so 100m is enough to schedule the pod on a small local node and leaves room for other pods.
- **CPU limit 250m.** Allows short spikes (startup, migrations check, a burst of requests) without one pod taking the whole node. Under constant load the pod hits the HPA threshold earlier than the limit, so new pods are added instead of throttling one.
- **Memory request 256Mi.** `runserver` starts two Python processes (the server and the autoreloader). Together they use noticeably less than 256Mi in idle, so idle memory stays below the 80% HPA threshold and the app stays at 2 pods.
- **Memory limit 512Mi.** Twice the request: enough headroom for load, while a memory leak ends with the container being restarted (OOMKilled) instead of eating the node's memory.

## 3. HPA configuration

- **minReplicas: 2.** Two pods are always running: if one pod restarts or its node fails, the other still serves traffic. This is also the required idle state.
- **maxReplicas: 5.** Enough to absorb a load spike on a local cluster without exhausting it: 5 × 512Mi = 2.5Gi memory at the limits.
- **CPU target 70%** of the request. Scaling starts before pods become saturated, leaving 30% headroom while a new pod passes its readiness probe (about 30 seconds for this app).
- **Memory target 80%** of the request. Memory is less spiky than CPU, so a higher threshold avoids extra pods caused by normal fluctuations. Python rarely gives memory back, so a lower threshold could keep extra pods running for no reason.

HPA scales up when **either** CPU or memory crosses its target and scales down only when both are below their targets.

## 4. Strategy configuration

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

- **RollingUpdate** replaces pods one by one, so the app is never fully stopped during an update (unlike `Recreate`).
- **maxUnavailable: 0.** No old pod is removed until a new one is ready. With only 2 replicas, losing one would cut capacity in half.
- **maxSurge: 1.** Only one extra pod is created at a time: the update uses at most one pod's worth of additional resources, which matters on a small local cluster. The readiness probe (`api/ready`, ready after ~30 s) guarantees traffic goes to the new pod only when it can handle it.

Check an update:

```bash
kubectl rollout status deployment/todoapp -n mateapp
kubectl rollout history deployment/todoapp -n mateapp
```

## 5. Access the app

Forward a local port to the deployment:

```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

Then open http://localhost:8080 in a browser, or check from another terminal:

```bash
curl http://localhost:8080/api/health
```

Expected answer: `Health OK`. The API is available at http://localhost:8080/api/.
