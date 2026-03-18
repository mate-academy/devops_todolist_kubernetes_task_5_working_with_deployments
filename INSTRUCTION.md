# ToDo App Deployment to Kubernetes

This document describes how to deploy the ToDo application to Kubernetes and explains the chosen Deployment, resource limits, rolling update strategy, Horizontal Pod Autoscaler configuration, and how to access the application after deployment.

## Prerequisites

Before deployment, make sure you have:

- a running Kubernetes cluster
- `kubectl` configured to work with your cluster
- the `mateapp` namespace created

Create the namespace if it does not exist yet:

```bash
kubectl create namespace mateapp
```

## Project Files

This deployment uses the following manifests:

* `deployment.yml`

* `hpa.yml`

___

## How to Deploy the Application

Apply the Deployment manifest and HPA (Horizontal Pod Autoscaler manifest):
```bash
kubectl apply -f deployment.yml

kubectl apply -f hpa.yml
```

## How to Verify the Deployment

Check the Deployment:
```bash
kubectl get deployment -n mateapp
```

Check the Pods:
```bash
kubectl get pods -n mateapp
```

Check the HPA:
```bash
kubectl get hpa -n mateapp
```

You can also get more detailed information with:
```bash
kubectl describe deployment todoapp -n mateapp
kubectl describe hpa todoapp -n mateapp
```

---

## Resource Requests and Limits

The following resource requests and limits were configured for the application container:

* CPU request: `100m`
* CPU limit: `250m`
* Memory request: `128Mi`
* Memory limit: `256Mi`

### Why these values were chosen

The requests define the minimum amount of CPU and memory that Kubernetes reserves for each Pod. 
This helps the scheduler place Pods on nodes with enough available resources.

The limits define the maximum amount of CPU and memory that the container can use. 
This protects the cluster from one container consuming too many resources.

The selected values were chosen as a balanced starting point for a lightweight web application. 
The requests are not too high, so the cluster can run multiple Pods, while the limits still provide enough room for the container to handle normal load. 
These values are also suitable for local learning environments such as Docker Desktop, Kind, or Minikube, where cluster resources are limited.

The application should have 2 Pods running in the idle state, so the selected requests and limits 
allow the cluster to run at least two replicas without unnecessary pressure on the node.

---

## HPA Configuration

The Horizontal Pod Autoscaler was configured with:

* minimum replicas: `2`
* maximum replicas: `5`
* target CPU utilization: `70%`
* target memory utilization: `70%`

### Why this HPA configuration was chosen

The minimum number of replicas is set to 2 to ensure that the application always has at least two running Pods, even under low load. This improves availability and matches the task requirement.

The maximum number of replicas is set to 5 to allow the application to scale under increased load, while still keeping resource usage under control in a small training cluster.

CPU and memory were both selected as scaling metrics because they represent two important types of pressure on the application. CPU reflects processing load, while memory reflects the container’s memory consumption. Using both metrics allows the autoscaler to react to different workload patterns.

The target value of 70% is a reasonable middle ground. It allows the application to use resources efficiently but still leaves headroom before the Pods become overloaded.

---

## Rolling Update Strategy

The Deployment uses the RollingUpdate strategy with:

* maxUnavailable: `1`
* maxSurge: `1`

### Why this strategy was chosen

RollingUpdate allows the application to be updated gradually without stopping the whole service at once.

_**maxUnavailable:**_ 1 means that during the update Kubernetes can temporarily make only one Pod unavailable. This helps keep the application running during the rollout.

**_maxSurge:_** 1 means Kubernetes can create one additional Pod above the desired number of replicas during the update. This helps replace old Pods with new ones smoothly and reduces downtime.

These values were chosen because they provide a safe and simple update process for a small application. They allow updates to happen step by step while keeping the service available.

---

## How to Access the Application After Deployment

There are two ways to access and test the application after deployment.

* ### Option 1: Using port-forward

Run:

```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

After that, open the application in your browser:

http://localhost:8080

_**You can also check the health and readiness endpoints:**_

http://localhost:8080/api/health

http://localhost:8080/api/ready


* ### Option 2: Using a busybox pod (via manifest)

Apply the busybox manifest:
```bash
kubectl apply -f busybox.yml
```

Then connect to the pod:
```bash
kubectl exec -it busybox -n mateapp -- sh
```

Inside the container, use wget to test the application:
```bash
wget -qO- http://todoapp:8080/api/health
```
or:
```bash
wget -qO- http://todoapp:8080/api/ready
```

This method allows testing the application from inside the Kubernetes cluster using a dedicated test pod.