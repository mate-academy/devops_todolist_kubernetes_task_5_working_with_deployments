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



### how to deploy the app to k8s

Deploying the App to Kubernetes (K8s)

Use these commands:

kubectl apply -f namespace.yml # Create namespace
kubectl apply -f deployment.yml # Deploy the app
kubectl apply -f hpa.yml # Configure automatic scaling (HPA)

### Resource Requests and Limits Explanation
Resource requests and limits were chosen based on expected app resource usage. Memory: 64Mi as minimum request and 128Mi as maximum limit. CPU: 250m request and 500m limit to ensure adequate processing power while avoiding excessive resource consumption.

### HPA Configuration Explanation
HPA config scales replicas based on observed app resource metrics from 2 to 5 replicas. 'scaleTargetRef' targets 'todoapp' deployment, 'minReplicas' and 'maxReplicas' set lower and upper scaling bounds. CPU and memory metrics considered with a 70% target average utilization for each to efficiently allocate resources.

### Strategy Configuration Explanation (Why These Numbers)
RollingUpdate strategy chosen to gradually introduce new versions, replacing old pods incrementally. Only one additional pod allowed during updates, ensuring at least one old pod remains available ('maxSurge: 1' and 'maxUnavailable: 1' respectively) for efficient deployment and service availability. These settings align with best practices for continuous delivery and service reliability.


### To access the app:

Start the manifest:

kubectl apply -f nodeport.yml

# Get the service name:

kubectl get services -o wide

# Check the service:External Access:

http://"nodeIP":30080

# On localhost:
http://localhost:30080
