# Kubernetes Deployment Instructions

## 1. Project Overview

This project is based on Django and is deployed to a Kubernetes cluster.

The application uses the training Docker image:

`ikulyk404/todoapp:3.0.0`

The Deployment and HorizontalPodAutoscaler are deployed to the `mateapp` namespace, as required by the task.

The application services and other related Kubernetes resources are also deployed to the same `mateapp` namespace. Keeping related resources in one namespace simplifies service discovery, resource management, and communication between application components.

The Kubernetes configuration uses:

- Deployment
- ClusterIP Service
- NodePort Service
- HorizontalPodAutoscaler (HPA)
- RollingUpdate deployment strategy
- CPU and memory resource requests and limits

---

## 2. Deployment Steps

Go to the `.infrastructure` directory and apply the manifests in the following order.

Create the namespace:

```bash
kubectl apply -f namespace.yml
```

Set `mateapp` as the default namespace for the current Kubernetes context:

```bash
kubectl config set-context --current --namespace=mateapp
```

Create the Deployment and application pods with configured CPU and memory resource requests and limits:

```bash
kubectl apply -f deployment.yml
```

Create the ClusterIP service:

```bash
kubectl apply -f clusterIp.yml
```

Create the NodePort service:

```bash
kubectl apply -f nodeport.yml
```

Create the Horizontal Pod Autoscaler:

```bash
kubectl apply -f hpa.yml
```

Check the deployed Kubernetes resources:

```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get hpa
```

The application can be accessed through the NodePort service in a browser:

`http://localhost:30080/`

---

## 3. RollingUpdate Strategy

The `RollingUpdate` strategy is used to update the application without stopping all application pods at the same time.

The Deployment uses the following configuration:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

`maxUnavailable: 1` means that during an update, no more than one pod may be unavailable at a time.

This is suitable for an application that normally runs at least two replicas because at least one pod can remain available while another pod is being replaced.

`maxSurge: 1` means that Kubernetes may temporarily create one additional pod above the desired number of replicas during the rollout.

This provides a balance between application availability and resource consumption. Pods are replaced gradually, allowing the application to remain available during deployment while avoiding the creation of too many temporary pods.

---

## 4. Horizontal Pod Autoscaler Configuration

According to the task requirements, the application should run between 2 and 5 pods.

The HPA configuration uses:

```yaml
minReplicas: 2
maxReplicas: 5

metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

`minReplicas: 2` ensures that at least two application pods are running.

`maxReplicas: 5` limits automatic scaling so that the application cannot grow beyond five replicas.

The autoscaler monitors both CPU and memory utilization. When one of the configured metrics requires more replicas, the HPA increases the desired replica count, up to the configured maximum of five pods.

The target value of `70%` leaves some resource headroom before the pods become heavily loaded while still allowing the cluster to use resources efficiently.

The HPA changes the desired replica count of the Deployment. The Deployment and its ReplicaSet then create or remove pods as necessary.

---

## 5. Accessing the Application

The application is exposed through the NodePort service on port `30080`.

Open the following address in a browser:

`http://localhost:30080/`

The application can also be accessed by forwarding a local port to the NodePort service:

```bash
kubectl port-forward service/todoapp-nodeport-service 8083:80
```

Then open:

`http://localhost:8083/`

In this case, local port `8083` is forwarded to service port `80`.

---

## 6. Verification

Check the running pods:

```bash
kubectl get pods
```

Check the Horizontal Pod Autoscaler:

```bash
kubectl get hpa
```

View detailed HPA information:

```bash
kubectl describe hpa todoapp
```

Check the Deployment:

```bash
kubectl get deployment
```
