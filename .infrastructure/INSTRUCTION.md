# Deployment Instructions

## Prerequisites

- A running Kubernetes cluster (e.g. `minikube start`)
- `kubectl` configured to talk to the cluster
- The `metrics-server` addon enabled (required for HPA to read CPU/Memory usage):

```bash
minikube addons enable metrics-server
```

## How to deploy

1. Make sure the `mateapp` namespace exists:

```bash
kubectl create namespace mateapp
```

(skip this step if the namespace already exists)

2. Apply the Deployment:

```bash
kubectl apply -f deployment.yml
```

3. Apply the Horizontal Pod Autoscaler:

```bash
kubectl apply -f hpa.yml
```

4. Verify everything is running:

```bash
kubectl get deployment -n mateapp
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
```

You should see 2 pods running in the idle state, and the HPA showing current CPU/Memory
utilization against the configured targets.

## How to access the app after deployment

The Deployment itself does not expose the app outside the cluster — a Service is
required for that. The simplest way to access the app for testing purposes is
`kubectl port-forward`:

```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

Then open:

```
http://localhost:8080
```

This works because `port-forward` tunnels traffic through the Kubernetes API server
directly to the pod, without requiring a Service or any additional network routes to be
configured on the host machine.

If a Service (e.g. `ClusterIP`, `NodePort`) is created separately for this app, it can
be used instead, following the same `app: todoapp` label selector used by this
Deployment.

## Explanation of resource requests and limits

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

- **Requests (100m CPU / 128Mi memory)** — this is the guaranteed minimum the scheduler
  reserves for each pod when deciding which node to place it on. The values are set
  low, reflecting a lightweight app (a simple to-do list backend) that is mostly idle
  under normal load, so we don't want to over-reserve cluster capacity for something
  that rarely needs it.
- **Limits (250m CPU / 256Mi memory)** — this is the hard ceiling a single pod is
  allowed to consume. The limit is set to roughly 2.5x the request, giving the app
  enough headroom to handle short traffic spikes without being immediately throttled
  or OOMKilled, while still preventing a single pod from starving other workloads on
  the same node if something misbehaves (e.g. a memory leak).
- The specific numbers were chosen for a small, low-traffic demo/test application. In a
  real production environment these values should be tuned based on actual observed
  usage (e.g. via `kubectl top pods` or historical metrics), rather than assumed
  upfront.

## Explanation of HPA configuration

```yaml
minReplicas: 2
maxReplicas: 5
metrics:
  - cpu:    averageUtilization: 70
  - memory: averageUtilization: 80
```

- **minReplicas: 2** — matches the required idle-state replica count from the
  Deployment. Having at least 2 replicas at all times also gives basic availability:
  if one pod is being restarted (e.g. during a rolling update or a node issue), the
  other can still serve traffic.
- **maxReplicas: 5** — caps the autoscaler at 5 pods, which is enough to absorb a
  significant traffic spike (2.5x the baseline) for a small application, without
  letting a runaway load (or a misconfigured/looping client) scale the app unbounded
  and consume the whole cluster's resources.
- **Both CPU and Memory triggers** — the HPA is configured with two independent
  metrics; Kubernetes will scale up if *either* one crosses its target, and only scale
  down when *both* are back below target. This is safer than a CPU-only or
  memory-only setup, because it also covers scenarios where the app becomes
  memory-bound (e.g. handling more concurrent requests/sessions) without a
  corresponding rise in CPU usage.
- **70% CPU / 80% memory thresholds** — CPU is set slightly more conservatively
  (70%) since CPU usage can spike quickly and briefly, so scaling a bit earlier avoids
  latency spikes. Memory is set a bit higher (80%) since memory usage tends to grow
  more gradually and predictably, so there's less risk of a sudden overload if the
  autoscaler reacts a little later.

## Explanation of the RollingUpdate strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

- **RollingUpdate** (instead of `Recreate`) was chosen because the app should remain
  available during updates — with `Recreate`, all pods would be terminated before new
  ones are created, causing a full outage during every deploy. `RollingUpdate`
  replaces pods incrementally, so the service keeps responding throughout the rollout.
- **maxUnavailable: 1** — with only 2 replicas in the idle state, allowing at most 1
  pod to be unavailable during an update still guarantees that at least 1 pod is always
  serving traffic. Setting it higher (e.g. 2) would risk taking down the entire app
  during a rollout, which is unacceptable even for a small deployment.
- **maxSurge: 1** — allows at most 1 extra pod to be created above the desired replica
  count during the rollout. This lets Kubernetes bring up a new pod *before* removing
  an old one, keeping capacity stable (or even slightly higher) during the transition,
  rather than dropping capacity first and then recovering it.
- Together, `maxUnavailable: 1` / `maxSurge: 1` produce a smooth, one-pod-at-a-time
  rollout: a new pod comes up and becomes Ready, then one old pod is removed — repeated
  until all pods are on the new version. This is a conservative, safe default suitable
  for a small replica count like this one.