# Kubernetes ToDo Application — Deployment Instructions

This document describes how to deploy the ToDo application to Kubernetes and explains the resource settings, Horizontal Pod Autoscaler (HPA), RollingUpdate strategy, and how to access the application after deployment.

The manifests used for this task are:

- `.infrastructure/namespace.yml`
- `.infrastructure/deployment.yml`
- `.infrastructure/hpa.yml`

The Deployment and HPA belong to the `mateapp` namespace.

---

## 1. Deploy the application to Kubernetes

Create the namespace first:

```bash
kubectl apply -f .infrastructure/namespace.yml
```

Check that the namespace exists:

```bash
kubectl get namespaces
```

Expected result should include:

```text
mateapp
```

Apply the Deployment:

```bash
kubectl apply -f .infrastructure/deployment.yml
```

Apply the Horizontal Pod Autoscaler:

```bash
kubectl apply -f .infrastructure/hpa.yml
```

Check the Deployment:

```bash
kubectl get deployment -n mateapp
```

Check the Pods:

```bash
kubectl get pods -n mateapp
```

In the idle state, two application Pods should be running:

```text
NAME                       READY   STATUS    RESTARTS
todoapp-xxxxxxxxxx-xxxxx    1/1     Running   0
todoapp-xxxxxxxxxx-yyyyy    1/1     Running   0
```

Check the HPA:

```bash
kubectl get hpa -n mateapp
```

For more details:

```bash
kubectl describe deployment todoapp -n mateapp
kubectl describe hpa todoapp -n mateapp
```

---

## 2. Resource Requests and Limits

The application container uses:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "64Mi"
  limits:
    cpu: "500m"
    memory: "128Mi"
```

`requests` define the minimum resources Kubernetes should reserve for the Pod:

- `100m` CPU = 0.1 CPU core.
- `64Mi` memory = 64 MiB RAM.

These values are suitable for a small training Django application in an idle state and allow two Pods to run without reserving excessive resources.

`limits` define the maximum resources the container may use:

- up to `500m` CPU = 0.5 CPU core;
- up to `128Mi` memory.

The limits protect the cluster from one container consuming too many resources.

Resource requests are also important for HPA because CPU and memory utilization percentages are calculated relative to the requested resources.

---

## 3. Horizontal Pod Autoscaler configuration

The HPA uses:

```yaml
minReplicas: 2
maxReplicas: 5
```

This means Kubernetes keeps at least 2 Pods running and may scale up to 5 Pods under load.

CPU metric:

```yaml
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 50
```

Target CPU utilization is `50%`.

Memory metric:

```yaml
- type: Resource
  resource:
    name: memory
    target:
      type: Utilization
      averageUtilization: 70
```

Target memory utilization is `70%`.

Why these values were chosen:

- `minReplicas: 2` provides basic availability.
- `maxReplicas: 5` prevents uncontrolled scaling in the local/test cluster.
- `50%` CPU leaves headroom before Pods become CPU-saturated.
- `70%` memory leaves a safety margin before reaching the memory limit.

Check HPA:

```bash
kubectl get hpa -n mateapp
kubectl describe hpa todoapp -n mateapp
```

If CPU or memory values are shown as `<unknown>`, verify that Metrics Server is installed and working:

```bash
kubectl top pods -n mateapp
```

---

## 4. Deployment strategy: RollingUpdate

The Deployment uses:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

`RollingUpdate` updates the application gradually instead of stopping all Pods at once.

### maxUnavailable: 1

During an update, Kubernetes may have at most one Pod unavailable.

Because the normal state contains two Pods, this allows the application to remain available while another Pod is updated.

### maxSurge: 1

Kubernetes may temporarily create one extra Pod above the desired replica count.

For example, when the desired replica count is 2, Kubernetes may temporarily run 3 Pods during an update.

Why these values were chosen:

- the application remains available during deployment;
- Pods are updated gradually;
- only one additional Pod is created temporarily;
- resource usage stays predictable.

Watch a rollout:

```bash
kubectl rollout status deployment/todoapp -n mateapp
```

Show rollout history:

```bash
kubectl rollout history deployment/todoapp -n mateapp
```

---

## 5. Access the application after deployment

The application container listens on port `8080`.

For a local Kubernetes cluster, the simplest access method is `port-forward`:

```bash
kubectl port-forward -n mateapp deployment/todoapp 8080:8080
```

Expected output:

```text
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

Open in a browser:

```text
http://localhost:8080
```

Or test with curl:

```bash
curl http://localhost:8080/
```

Test readiness:

```bash
curl http://localhost:8080/api/ready
```

Test liveness:

```bash
curl http://localhost:8080/api/health
```

Stop port-forward with:

```text
Ctrl+C
```

---

## 6. Useful verification commands

Show all resources:

```bash
kubectl get all -n mateapp
```

Show Pods with IP addresses:

```bash
kubectl get pods -n mateapp -o wide
```

Show Deployment details:

```bash
kubectl describe deployment todoapp -n mateapp
```

Show HPA details:

```bash
kubectl describe hpa todoapp -n mateapp
```

Show application logs:

```bash
kubectl logs -n mateapp deployment/todoapp
```

Check resource usage:

```bash
kubectl top pods -n mateapp
```

---

## 7. Remove the application

Remove the HPA:

```bash
kubectl delete -f .infrastructure/hpa.yml
```

Remove the Deployment:

```bash
kubectl delete -f .infrastructure/deployment.yml
```

Remove the namespace and all resources inside it:

```bash
kubectl delete namespace mateapp
```

---

## 8. Task checklist

- `deployment.yml` created for the application.
- Deployment strategy is `RollingUpdate`.
- Resource `requests` and `limits` are configured.
- Idle state uses 2 replicas.
- Pod template contains the application container configuration.
- `hpa.yml` uses `autoscaling/v2`.
- HPA minimum replicas: `2`.
- HPA maximum replicas: `5`.
- HPA scales by both CPU and memory utilization.
- Both new manifests belong to the `mateapp` namespace.
- Resource requests and limits are explained.
- HPA configuration is explained.
- RollingUpdate strategy and its values are explained.
- Application access after deployment is explained using `kubectl port-forward`.
