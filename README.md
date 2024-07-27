# Django ToDo list

This is a todo list web application with basic features of most web apps, i.e., accounts/login, API, and interactive UI. To do this task, you will need:

- CSS | [Skeleton](http://getskeleton.com/)
- JS  | [jQuery](https://jquery.com/)

## Explore

Try it out by installing the requirements (the following commands work only with Python 3.8 and higher, due to Django 4):

```
pip install -r requirements.txt
```

Create a database schema:

```
python manage.py migrate
```

And then start the server (default is http://localhost:8000):

```
python manage.py runserver
```

Now you can browse the [API](http://localhost:8000/api/) or start on the [landing page](http://localhost:8000/).

## Task

Create a kubernetes manifest for a pod which will containa ToDo app container:

1. Fork this repository.
1. Create a `deployment.yml` file with a deployment for the app.
1. Deployment should have
    1. Strategy: RollingUpdate
    1. Resource requests and limits (in idle state you should have 2 pods running)
    1. Pod spec should be same as for pods manifest
1. Createa a `hpa.yml` file with a Horizontal Pod Autoscaler for the app.
1. Autoscaler should define
    1. Minimum number of pods as 2
    2. Maximum number of pods as 5
    3. Autoscale should be triggered by both CPU and Memory
1. Both new manifests should belong to `mateapp` namespace
1. `README.md` should be updated with the instructions on how to deploy the app to k8s
1. `README.md` Should have explained you choice of resources requests and limits
1. `README.md` Should have explained your choice of HPA configuration
1. `README.md` Should have explained your strategy configuration (Why such numbers)
1. `README.md` Should have explained how to access the app after deployment
1. Create PR with your changes and attach it for validation on a platform.


### Steps

kubectl create namespace mateapp

kubectl apply -f development.yml

Set up Horizontal Pod Autoscaler (HPA):

kubectl apply -f hpa.yml

Resource Requests and Limits
Memory Requests: 128Mi and Limits: 256Mi\

Chosen to ensure the application has enough memory to function smoothly under typical loads while preventing it from consuming excessive resources.
CPU Requests: 250m and Limits: 500m

Ensures the application has sufficient CPU to handle regular traffic and can scale up under load without overloading the node.

Horizontal Pod Autoscaler (HPA) Configuration
Minimum Replicas: 2
Ensures high availability; if one pod fails, another is available to handle traffic.
Maximum Replicas: 5

Allows scaling up to handle increased load, balancing resource utilization and cost.
CPU and Memory Utilization: 70%
Triggers scaling when average utilization exceeds 70%, ensuring responsive scaling under increased load.

RollingUpdate Strategy Configuration
maxUnavailable: 1
Ensures at least one pod is always available during updates.
maxSurge: 1
Allows one extra pod to be created during updates, ensuring smooth transitions with no downtime.

kubectl get pods -n mateapp

kubectl port-forward <pod-name> 8080:8080 -n mateapp
Replace <pod-name> with the name of one of the running pods.

Access the application in your browser:
Open http://localhost:8080 in your web browser.


kubectl get hpa -n mateapp
kubectl top pods -n mateapp

# Create the namespace
kubectl create namespace mateapp

# Deploy the application
kubectl apply -f development.yml

# Set up Horizontal Pod Autoscaler (HPA)
kubectl apply -f hpa.yml

# Get the list of running pods
kubectl get pods -n mateapp

# Port forward the service to access the application locally
kubectl port-forward <pod-name> 8080:8080 -n mateapp

# Access the application in your browser
# Open http://localhost:8080 in your web browser

# Monitor the application and HPA status
kubectl get hpa -n mateapp
kubectl top pods -n mateapp