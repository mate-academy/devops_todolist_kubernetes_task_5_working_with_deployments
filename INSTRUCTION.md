# Todo App Kubernetes Instructions

## Prerequisites

Before deploying the application, make sure that:

* Kubernetes cluster is running.
* `kubectl` is installed and configured.
* The `mateapp` namespace exists.
* Metrics Server is installed in the cluster.
* The cluster supports the `autoscaling/v2` API.
* CPU and memory metrics are available for the pods.

Create the namespace if it does not exist:

```bash
kubectl create namespace mateapp
```

Verify that CPU and memory metrics are available:

```bash
kubectl top pods -n mateapp
```

---

## Deployment

Apply the Deployment:

```bash
kubectl apply -f .infrastructure/deployment.yml
```

The Deployment starts with **2 replicas**.

Check the pods:

```bash
kubectl get pods -n mateapp
```

The expected idle state is **2 running pods**.

---

## Resource Requests and Limits

The Deployment uses the following resources for each pod:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

The requests are set to `100m` CPU and `128Mi` memory because the Todo App has relatively low resource requirements during normal and idle operation. These values allow Kubernetes to run the required 2 pods without reserving unnecessary resources.

The limits are set to `500m` CPU and `512Mi` memory to provide additional resources when the application workload increases while preventing a pod from consuming an unlimited amount of cluster resources.

The resource requests are also required for CPU and memory utilization calculations used by the HPA.

---

## Horizontal Pod Autoscaler

Apply the HPA:

```bash
kubectl apply -f .infrastructure/hpa.yml
```

The HPA configuration is:

* Minimum replicas: **2**
* Maximum replicas: **5**
* CPU target: **70%**
* Memory target: **80%**

The HPA scales the application based on both CPU and memory utilization.

The CPU threshold of **70%** provides some resource headroom before the pods become CPU-constrained.

The memory threshold of **80%** allows the application to use most of its requested memory while leaving some capacity before additional pods are required.

The minimum of **2 replicas** ensures that two pods are available during normal and idle operation.

The maximum of **5 replicas** provides additional capacity during increased load while limiting the total number of pods and resource consumption.

The HPA requires Metrics Server to provide CPU and memory metrics. The Kubernetes cluster must also support the `autoscaling/v2` API.

---

## Deployment Strategy

The Deployment uses the `RollingUpdate` strategy:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

`RollingUpdate` allows a new version of the application to be deployed gradually without stopping all existing pods at once.

`maxUnavailable: 0` means that no existing replica should be intentionally unavailable during the update. This prioritizes application availability.

The trade-off is that additional resources may be required during the update because existing pods remain available until new pods are ready.

`maxSurge: 1` allows Kubernetes to create one additional pod during the update. With 2 desired replicas, the Deployment can temporarily have 3 pods.

This provides a balance between availability and resource consumption. Only one additional pod is created during the rollout instead of creating several additional pods at once.

---

## Application Port and Health Probes

The Todo App listens on **port 8080**.

The Deployment uses:

```yaml
ports:
  - containerPort: 8080
```

The health probes also use port `8080`:

```yaml
livenessProbe:
  httpGet:
    path: api/health
    port: 8080

readinessProbe:
  httpGet:
    path: api/ready
    port: 8080
```

Therefore, the configured container port matches the application port.

---

## Accessing the Application

To access the application locally, use port forwarding:

```bash
kubectl port-forward deployment/todoapp-deployment 8080:8080 -n mateapp
```

Then open:

```text
http://localhost:8080
```

---

## Verification

Check the Deployment:

```bash
kubectl get deployment -n mateapp
```

Check the pods:

```bash
kubectl get pods -n mateapp
```

Check the HPA:

```bash
kubectl get hpa -n mateapp
```

Check CPU and memory metrics:

```bash
kubectl top pods -n mateapp
```
