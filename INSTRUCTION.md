# Todo App Kubernetes Deployment

## 1. Deploy the application

The Todo App is deployed to the `mateapp` namespace.

First, create the namespace:

```
kubectl create namespace mateapp
```

Apply the Deployment:

```
kubectl apply -f deployment.yml
```

Apply the Horizontal Pod Autoscaler:

```
kubectl apply -f hpa.yml
```

Check the deployed resources:

```
kubectl get pods -n mateapp
kubectl get deployment -n mateapp
kubectl get hpa -n mateapp
```

The application starts with 2 replicas.

---

## 2. Resource requests and limits

The Deployment uses the following resources:

```
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

The resource requests were selected for the normal idle state of the Todo App.

Each pod requests:

* `100m` CPU
* `128Mi` memory

This allows Kubernetes to schedule the required 2 pods without reserving excessive resources.

The resource limits allow each pod to temporarily use more resources when the workload increases:

* `500m` CPU
* `512Mi` memory

The limits prevent a single pod from consuming an unlimited amount of CPU or memory.

---

## 3. Deployment strategy

The Deployment uses the `RollingUpdate` strategy:

```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

`RollingUpdate` was selected because it allows the application to be updated without intentionally stopping all existing pods at the same time.

The Deployment has 2 replicas in the idle state.

### `maxSurge: 1`

During an update, Kubernetes can temporarily create 1 additional pod.

Therefore, while updating from the old version to the new version, there can temporarily be up to 3 pods.

### `maxUnavailable: 0`

No existing pod should be intentionally unavailable during the update.

This configuration was chosen to keep the application available during a rolling update while still limiting the number of additional pods created during the update.

---

## 4. Horizontal Pod Autoscaler

The application uses a Horizontal Pod Autoscaler configured in `hpa.yml`.

The HPA configuration is:

```
minReplicas: 2
maxReplicas: 5
```

The number of pods can therefore automatically increase from 2 to 5.

The HPA monitors both CPU and memory utilization.

### CPU

The CPU target is:

```
averageUtilization: 70
```

When the average CPU utilization reaches approximately 70% of the requested CPU resources, the HPA can increase the number of pods.

### Memory

The memory target is:

```
averageUtilization: 80
```

When the average memory utilization reaches approximately 80% of the requested memory resources, the HPA can increase the number of pods.

Both CPU and memory are monitored because increased application load can affect either resource.

The minimum of 2 pods ensures that the application has two running instances during normal load.

The maximum of 5 pods provides additional capacity during higher load while preventing unlimited scaling.

---

## 5. Access the application

The Todo App container listens on port `8080`.

To access the application locally, use port forwarding:

```
kubectl port-forward deployment/todoapp-deployment 8080:8080 -n mateapp
```

Then open the application in a browser:

```
http://localhost:8080
```

The port-forward command must remain running while the application is being accessed.

---

## 6. Verify the HPA

To check the current HPA status:

```
kubectl get hpa -n mateapp
```

To see detailed HPA information:

```
kubectl describe hpa todoapp-hpa -n mateapp
```

The HPA will maintain at least 2 pods and can scale the Deployment up to 5 pods depending on CPU and memory utilization.
