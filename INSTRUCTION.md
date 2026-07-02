# Deploying the ToDo App to Kubernetes

This document explains how to deploy the ToDo app `Deployment` and `HorizontalPodAutoscaler`
(`.infrastructure/deployment.yml` and `.infrastructure/hpa.yml`) to a Kubernetes cluster, and
the reasoning behind the configuration choices. Both manifests deploy into the `mateapp`
namespace, as required by the task. Note this is a different namespace from the existing
`todoapp-pod.yml`, `clusterIp.yml`, and `nodeport.yml` manifests in this repo (which use
`todoapp`), so the Deployment's pods are managed independently from those.

## Prerequisites

- A running Kubernetes cluster and `kubectl` configured to talk to it.
- The [metrics-server](https://github.com/kubernetes-sigs/metrics-server) add-on installed and
  running in the cluster. The HPA needs it to read CPU/memory metrics — without it, the
  autoscaler will show `<unknown>` for its targets and will not scale.

## Deploy steps

1. Create the `mateapp` namespace (both manifests deploy into it):

   ```bash
   kubectl create namespace mateapp
   ```

2. Apply the Deployment:

   ```bash
   kubectl apply -f .infrastructure/deployment.yml
   ```

3. Apply the HorizontalPodAutoscaler:

   ```bash
   kubectl apply -f .infrastructure/hpa.yml
   ```

4. Verify everything is up:

   ```bash
   kubectl -n mateapp get deployments
   kubectl -n mateapp get pods
   kubectl -n mateapp get hpa
   ```

   You should see the `todoapp` Deployment with 2/2 pods ready, and the `todoapp` HPA
   reporting current CPU/memory usage against the configured targets.

## Resource requests and limits

Each container requests `cpu: 100m` / `memory: 128Mi` and is limited to `cpu: 250m` /
`memory: 256Mi`.

- **Requests** are set low because this is a small Django app whose idle footprint (serving a
  handful of requests, running health/readiness probes) does not need much CPU or memory.
  Setting requests close to real idle usage lets the scheduler pack pods efficiently and gives
  the HPA a meaningful baseline to calculate utilization percentages against.
- **Limits** are set at 2.5x the CPU request and 2x the memory request, enough headroom to
  absorb short traffic bursts without immediately triggering the OOM killer or CPU throttling,
  while still protecting the node from a single misbehaving pod consuming all its resources.
- Requests/limits are also what make the HPA's percentage-based targets (see below) meaningful:
  without them, "70% utilization" would have nothing to be a percentage of.

## HPA configuration

- **minReplicas: 2** — matches the Deployment's baseline replica count, so the app always runs
  with at least 2 pods for redundancy (a rolling update or a node failure never drops to a
  single point of failure), even when idle/no traffic.
- **maxReplicas: 5** — caps scale-out at 5 pods, comfortably covering traffic spikes for a small
  todo-list app without letting a runaway load (or a bug) scale the app indefinitely and exhaust
  cluster resources or cost budget.
- **CPU and memory metrics, both at 70% average utilization** — the task requires scaling to
  react to both CPU and memory. 70% is used for both because it leaves ~30% burst headroom
  above the trigger point (relative to the container's limit) before pods hit their hard
  limits, while still being high enough to avoid scaling out on minor, temporary blips. When
  either metric crosses 70% of its request on average across pods, the HPA adds replicas (up to
  5); when both drop back below the target, it scales back down (to a minimum of 2).

## RollingUpdate strategy

The Deployment uses:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

- **maxUnavailable: 0** — with only 2 baseline replicas, losing even one pod during a rollout
  means running at 50% capacity. Setting this to 0 guarantees the currently-running pods keep
  serving traffic until their replacements are ready, so a deploy never reduces the app's
  available capacity.
- **maxSurge: 1** — allows exactly one extra pod to be created above the desired replica count
  while a rollout is in progress. Combined with `maxUnavailable: 0`, updates proceed one pod at
  a time (new pod created and passes its readiness probe → one old pod terminated → repeat),
  giving a safe, zero-downtime rollout without over-provisioning the cluster with many surplus
  pods at once.

## Accessing the app after deployment

The Deployment/HPA manifests intentionally only manage the pods (as scoped by this task); no
Service exists yet in the `mateapp` namespace, so to reach the pods you can either create one or
port-forward directly.

- **For a quick local check without creating a Service**, port-forward directly to the
  Deployment:

  ```bash
  kubectl -n mateapp port-forward deployment/todoapp 8080:8080
  ```

  Then open [http://localhost:8080](http://localhost:8080) in a browser, or:

  ```bash
  curl http://localhost:8080/api/health
  ```

- **To expose it more permanently inside/outside the cluster**, create a `ClusterIP` or
  `NodePort` Service in the `mateapp` namespace with a selector of `app: todoapp`, following the
  same pattern as the existing `.infrastructure/clusterIp.yml` / `.infrastructure/nodeport.yml`
  (which target the `todoapp` namespace):

  ```bash
  # example: NodePort, adapted to the mateapp namespace
  kubectl -n mateapp expose deployment todoapp --type=NodePort --port=80 --target-port=8080
  curl http://<node-ip>:<node-port>/api/health
  ```
