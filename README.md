# Django ToDo list

This is a todo list web application with basic features of most web apps, i.e., accounts/login, API, and interactive UI. To do this task, you will need:

- CSS | [Skeleton](http://getskeleton.com/)
- JS | [jQuery](https://jquery.com/)

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

1. run following commands from .infrastructure directory: -`kubectl apply -f namespace.yml` -`kubectl apply -f nodeport.yml` -`kubectl apply -f clusterip.yml` -`kubectl apply -f deployment.yml` -`kubectl apply -f hpa.yml`
1. Amount of resources from the lesson was not enough so i had to add some, because containers were stuck in termination loop

1. 70% seems more or less default value, new instances will be launched if cpu or ram goes abowe 70%

1. strategy is described in deployment file and it's values are ignored when you create autoscaler

1. app can be accesed using address `http://localhost:30080` or by running command `http://localhost:30080` and accessing address `http://localhost:8001`

to do the cleanup run `kubectl delete namespace todoapp` and `kubectl delete deployment todoapp`
