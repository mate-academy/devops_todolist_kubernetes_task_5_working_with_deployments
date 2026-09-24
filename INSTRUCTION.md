# Django ToDo App — Kubernetes Deployment

## 1. Deployment

The Django ToDo application is deployed using a Kubernetes `Deployment` with the `RollingUpdate` strategy.

The application runs with **2 replicas** in the idle state:

```text
todoapp
├── Pod 1
└── Pod 2
```

The Deployment is located in the `mateapp` namespace.

To deploy the application:

```bash
kubectl apply -f deployment.yml
```

To check the Deployment:

```bash
kubectl get deployment -n mateapp
```

To check the running Pods:

```bash
kubectl get pods -n mateapp
```

---

## 2. Resource Requests and Limits

The container has the following resource configuration:

```yaml
requests:
  memory: "64Mi"
  cpu: "250m"

limits:
  memory: "128Mi"
  cpu: "500m"
```

### Requests

The requests define the amount of resources that Kubernetes should reserve for the container.

The application requests:

* **64 MiB of memory**
* **250m CPU**, which is equal to 0.25 CPU core

These values were chosen as a reasonable starting point for a small Django ToDo application that does not require significant resources during normal operation.

### Limits

The limits define the maximum amount of resources that the container can use.

The limits are:

* **128 MiB of memory**
* **500m CPU**, which is equal to 0.5 CPU core

The limits provide additional resources when the application needs them while preventing a single container from consuming an unlimited amount of CPU or memory.

The memory limit is higher than the request, allowing the application to temporarily use more memory during increased activity.

---

## 3. Horizontal Pod Autoscaler

The application uses a `HorizontalPodAutoscaler` configured with Kubernetes `autoscaling/v2`.

To deploy the HPA:

```bash
kubectl apply -f hpa.yml
```

The HPA configuration is:

```text
Minimum replicas: 2
Maximum replicas: 5
CPU target: 50%
Memory target: 50%
```

The HPA monitors both **CPU and memory utilization**.

When the resource utilization increases, Kubernetes can increase the number of application Pods.

For example:

```text
Low load
   ↓
2 Pods

Higher load
   ↓
3–4 Pods

High load
   ↓
up to 5 Pods
```

The number of replicas cannot be reduced below 2 or increased above 5.

### Why 2–5 replicas?

The minimum of **2 replicas** provides redundancy and satisfies the requirement to have two Pods running in the idle state.

The maximum of **5 replicas** limits the amount of resources that the application can consume while still allowing it to handle increased traffic.

### Why 50%?

A target of **50% utilization** provides a resource buffer. The application can be scaled out before the Pods reach their resource limits.

For example, for the CPU request of `250m`, 50% utilization corresponds to approximately:

```text
250m × 50% = 125m
```

The same principle is used for memory utilization.

---

## 4. Deployment Strategy

The Deployment uses the `RollingUpdate` strategy:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

`RollingUpdate` updates the application gradually instead of removing all old Pods at once.

### `maxSurge: 1`

During an update, Kubernetes can temporarily create **one additional Pod** above the desired number of replicas.

For example, when there should normally be 2 Pods:

```text
Before update:

Pod 1
Pod 2

During update:

Pod 1
Pod 2
Pod 3  ← temporary additional Pod
```

This allows a new version of the application to start before an old Pod is removed.

### `maxUnavailable: 1`

During the update, at most **one Pod can be unavailable**.

With two replicas, this allows Kubernetes to update the application gradually while keeping another Pod available.

The values `1` and `1` were chosen as a simple configuration suitable for a small application with two normal replicas.

---

## 5. Checking the HPA

The HPA status can be checked with:

```bash
kubectl get hpa -n mateapp
```

More detailed information can be obtained with:

```bash
kubectl describe hpa todoapp-hpa -n mateapp
```

The current Pods can be checked with:

```bash
kubectl get pods -n mateapp
```

---

## 6. Accessing the Application

After deployment, the application needs to be exposed so that it can be accessed from outside the Pods.

First, check the available Services:

```bash
kubectl get services -n mateapp
```

If the application is exposed through a Kubernetes Service, use the corresponding Service configuration to access the Django application.

For local Kubernetes environments, the application can be accessed using the appropriate method provided by the Kubernetes environment, such as port forwarding.

For example:

```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

After that, open:

```text
http://localhost:8080
```

The application should then be available through the local browser.

---

## 7. Useful Commands

Apply the manifests:

```bash
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

Check Deployment:

```bash
kubectl get deployment -n mateapp
```

Check Pods:

```bash
kubectl get pods -n mateapp
```

Check HPA:

```bash
kubectl get hpa -n mateapp
```

Get detailed HPA information:

```bash
kubectl describe hpa todoapp-hpa -n mateapp
```

Check application logs:

```bash
kubectl logs deployment/todoapp -n mateapp
```

Access the application locally:

```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```
