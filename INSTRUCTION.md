# Kubernetes ToDo Application

## Deploy application

Create namespace:

```bash id="y2u6pf"
kubectl apply -f .infrastructure/namespace.yml
```

Deploy application:

```bash id="t9e5jk"
kubectl apply -f .infrastructure/deployment.yml
```

Deploy services:

```bash id="k8d3ra"
kubectl apply -f .infrastructure/clusterip.yml
kubectl apply -f .infrastructure/nodeport.yml
```

Deploy HPA:

```bash id="n6m2wy"
kubectl apply -f .infrastructure/hpa.yml
```

Check resources:

```bash id="b5s1cq"
kubectl get pods -n mateapp
kubectl get svc -n mateapp
kubectl get hpa -n mateapp
```

---

# Resource Requests and Limits

The application uses the following resources:

```yaml id="v4p8xd"
requests:
  cpu: 100m
  memory: 128Mi

limits:
  cpu: 500m
  memory: 512Mi
```

## Why these values?

* 100m CPU and 128Mi memory are enough for an idle Django application.
* Limits protect the cluster from excessive resource usage.
* 500m CPU and 512Mi memory allow the application to handle temporary spikes.

Two replicas are always running to provide high availability.

---

# HPA Configuration

HPA configuration:

```yaml id="p7j9lv"
minReplicas: 2
maxReplicas: 5
```

Autoscaling is based on:

* CPU utilization
* Memory utilization

Threshold:

```yaml id="a3h7eu"
averageUtilization: 70
```

## Why these values?

* 70% utilization is a balanced threshold before scaling.
* Minimum 2 replicas guarantee availability.
* Maximum 5 replicas prevents uncontrolled scaling and excessive resource usage.

---

# RollingUpdate Strategy

Deployment strategy:

```yaml id="f2m5qn"
maxUnavailable: 1
maxSurge: 1
```

## Why this strategy?

* Only one pod can be unavailable during update.
* One additional pod may be temporarily created during deployment.
* This minimizes downtime and ensures service availability.

---

# Access Application

## Port Forward

```bash id="m8n1sw"
kubectl port-forward svc/todoapp-clusterip 8080:80 -n mateapp
```

Open:

```text id="v6r4xe"
http://localhost:8080/api/ready
http://localhost:8080/api/health
```

---

# Access Using NodePort

Get node IP:

```bash id="z4k8pc"
kubectl get nodes -o wide
```

Open:

```text id="x9b7ju"
http://NODE_IP:30080/api/ready
http://NODE_IP:30080/api/health
```
