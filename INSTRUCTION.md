# 🚀 Deploying the ToDo App to Kubernetes

These instructions describe how to deploy the `todoapp` to a Kubernetes cluster using the provided manifests.

## 🧾 Prerequisites

- A Kubernetes cluster up and running.
- `kubectl` configured to interact with your cluster.
- `mateapp` namespace created (`kubectl create ns mateapp` if not already present).
- Metrics Server installed and running in the cluster.

## 📦 Files Provided

- `deployment.yml`: Contains the Deployment manifest for the `todoapp`.
- `hpa.yml`: Defines a Horizontal Pod Autoscaler for the `todoapp`.
- `INSTRUCTION.md`: You're reading it!

## 📄 Step-by-Step Deployment

1. **Create the namespace (if it doesn't exist):**
   ```bash
   kubectl create namespace mateapp
   ```
2.	**Apply the Deployment:**
    ```bash
    kubectl apply -f deployment.yml
    ```
3. **Apply the HPA configuration:**
    ```bash
    kubectl apply -f hpa.yml
    ```
4. **Verify resources are running:**
    ```bash
    kubectl get deployments -n mateapp
    kubectl get pods -n mateapp
    kubectl get hpa -n mateapp
    ```

## ⚙️ Explanation of Configuration Choices

### 🧠 Resource Requests and Limits
```
resources:
  requests:
    cpu: 100m
    memory: 100Mi
  limits:
    cpu: 500m
    memory: 500Mi
```

- Requests define the minimum guaranteed resources each pod will get. I chose low values (100m CPU, 100Mi Memory) to ensure the app starts and runs under idle load.
- Limits cap the maximum resources a pod can consume to avoid overwhelming the node (500m CPU, 500Mi Memory), giving room to scale within reasonable bounds.

This balance ensures the app runs efficiently and can trigger autoscaling when needed.

---

### 📈 Horizontal Pod Autoscaler (HPA)
```
minReplicas: 2
maxReplicas: 5
```
- Minimum Replicas = 2: Ensures high availability; the app stays online even if one pod fails.
- Maximum Replicas = 5: Provides room to scale during increased demand, avoiding over-provisioning.

The HPA triggers on:
- CPU usage > 70%
- Memory usage > 70%

This dual trigger makes scaling more responsive to actual application workload and memory footprint.

### 🔁 RollingUpdate Strategy
```
strategy:
  type: RollingUpdate
```
- Rolling updates ensure zero downtime deployments by gradually updating pods.
- I use default settings which roll out updates one pod at a time while keeping the service available.

This is ideal for production environments and safer for upgrades compared to Recreate strategy.

## 🌐 Accessing the App

The app is exposed via a Kubernetes Service using both ClusterIP and NodePort types.

### 🔹 ClusterIP
This service is only accessible within the cluster—useful for internal communication or if you’re routing traffic through an Ingress controller.

To forward traffic locally, you can use:
```sh
kubectl port-forward svc/todoapp-service 8080:80 -n mateapp
```
Then open the app in your browser at:
http://localhost:8080

---

### 🔹 NodePort
If you want to access the app externally, the NodePort service exposes it on a specific port of each node.

To get the NodePort:
```sh
kubectl get svc todoapp-service -n mateapp
```
Look for the PORT(S) column (e.g., 80:31234/TCP).
Then visit the app in your browser using the node’s IP address:
```
http://<NodeIP>:<NodePort>
```
If you’re using Minikube, you can get the node IP with:
```sh
minikube ip
```