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


# ---------------------------------------

# Deploy the application following the commands:

kubectl apply -f ./.infrastructure/namespace.yml
kubectl apply -f ./.infrastructure/busybox.yml
kubectl apply -f ./.infrastructure/clusterip.yml
kubectl apply -f ./.infrastructure/nodeport.yml
kubectl apply -f ./.infrastructure/deployment.yml
kubectl apply -f ./.infrastructure/hpa.yml
kubectl apply -f ./.infrastructure/todoapp-pod.yml

# Check Pod Status:

kubectl get pods -o wide -n todoapp


# Go to the pod busybox:

kubectl -n todoapp exec -it busybox --sh

# Register the command:

curl http://todoapp-clusterip.todoapp.svc.cluster.local

# Explanations
# 1. Requests and resource limitations.
    Requests define the minimum resources that the container will receive.

    Limits limit the maximum amount of resources a container can use.

    Requests: memory: 64Mi, cpu: 250m - the minimum resources required for the application to operate under normal conditions.

    Limits: memory: 128Mi, cpu: 500m - the maximum resources that the container can consume to avoid unnecessary resource usage.

# 2. Horizontal Pod Autoscaler (HPA) configuration.
    CPU: If average CPU usage exceeds 70%, the number of replicas will increase.

    Memory: If the average memory usage exceeds 70%, the number of replicas will also increase.

# 3. Deployment strategy configuration.
    maxUnavailable: 2 - the maximum number of replicas that may be unavailable during an update.

    maxSurge: 2 - the maximum number of new replicas that can be added at the same time.

These values ​​were chosen to ensure smooth application updates without sudden crashes.
