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


In order to deploy the app to k8s you should execute the next steps:
Create namespace
kubectl create namespace mateapp
Then apply the next yaml files
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml

I decided to set the next values for cpu and memory
cpu = from 500m to 1000m
memory = from 512Mi to 1Gi
This is base parameters for the standard application. After starting needs to analyze working of the application and then increase values if it has need
Using resources depends on:
Traffic and load. Amount of users and amount of requests
Database size
I/O Operations
Caching and optimization

I decided to start from 2 replicas. It will be enough for starting and analyzing working. And then will increase amount of the replicas if it has need. Maximize values of replicas is 5

I choosed RollingUpdate strategy configuration with next parameters
maxUnavailable: 1
maxSurge: 1

This strategy works well when we need to have constant access to the application
maxUnavailable: 1 - will provide constant work the application
maxSurge: 1 - will provide saving resources

In order to check the application in your browser you should execute the next command
kubectl get pods
kubectl port-forward pod/name_of_pod_from_result_previous_command 8080:8080 -n mateapp
Now, the application is available for viewing in your browser by url
http://localhost:8080/

