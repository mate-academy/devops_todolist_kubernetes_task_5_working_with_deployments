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

Instruction: 

1. Apply the app config files:

```
kubectl apply -f .infrastructure/namespace.yml
```
```
kubectl apply -f .infrastructure/busybox.yml
```
```
kubectl apply -f .infrastructure/clusterIp.yml 
```
```
kubectl apply -f .infrastructure/deployment.yml
```
```
kubectl apply -f .infrastructure/nodeport.yml
```
```
kubectl apply -f .infrastructure/hpa.yml
```

2. Check the deployment:
```
kubectl get pods -n todoapp
```
1. `README.md` Should have explained you choice of resources requests and limits

The choice of resources and limits was made using the example from the course. The chosen resources and limits are 
pretty enough for the todo app for education purposes.

1. `README.md` Should have explained your choice of HPA configuration

HPA configuration choice was made as the resources and limits choice - using examples from the course. 
The HPA configuration fully meets the requirements and guarantees high fault tolerance 

1. `README.md` Should have explained your strategy configuration (Why such numbers)

A deployment strategy was implemented using the example from the course and the task requirements. Using 
such a configuration we can be sure that the application is accessible during the deployments

1. `README.md` Should have explained how to access the app after deployment

Using clusterIP:
```
kubectl port-forward service/todoapp 8080:80 -n todoapp 
```

Using nodePort:
```
kubectl -n todoapp exec -it busybox -- sh
```
```
curl http://todoapp.todoapp.svc.cluster.local
```


1. Create PR with your changes and attach it for validation on a platform.
