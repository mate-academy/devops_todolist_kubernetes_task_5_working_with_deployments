# Kubernetes Deployment Instructions
## Prerequisites

#### Before deploying the application, make sure you have:

* A running Kubernetes cluster
* kubectl installed and configured
* Metrics Server installed in the cluster (required for the Horizontal Pod Autoscaler)
* Deployment

### Apply the manifests in the following order:

`kubectl apply -f deployment.yml`
`kubectl apply -f hpa.yml`

##### Verify that the resources were created successfully:

`kubectl get deployments -n mateapp`
`kubectl get pods -n mateapp`
`kubectl get hpa -n mateapp`
`Resource Requests and Limits`

#### The Deployment defines both resource requests and limits for CPU and memory.

_Example values:_

* CPU request: 250m
* CPU limit: 500m
* Memory request: 64Mi
* Memory limit: 128Mi

#### Horizontal Pod Autoscaler Configuration

##### The Horizontal Pod Autoscaler is configured with:

* Minimum replicas: 2
* Maximum replicas: 5
* CPU utilization target: 50%
* Memory utilization target: 70%

### RollingUpdate Strategy

#### The Deployment uses the RollingUpdate strategy.

**Example configuration:**

* maxUnavailable: 1
* maxSurge: 1

#### After deployment, verify that the pods are running:

`kubectl get pods -n mateapp`

#### If the application is exposed through a Service, access it using:

`kubectl get svc -n mateapp`



**You can also use port forwarding for local access:**

`kubectl port-forward deployment/todoapp-deployment 8080:8080 -n mateapp`

**Then open:**

http://localhost:8080