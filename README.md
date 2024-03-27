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


## How to apply all manifests
Use command:
```
kubectl apply -f .infractructure/
```

## How to test application using the port-forward command
Use this command:
```
kubectl port-forward service/{service_name} 8081:80
```
Content will appear at: 
```
localhost:8081
```

## How to test application using ClusterIP service DNS from a busybox container
Use this command to connect a 'busybox' pod
```
kubectl -n {namespace} exec -it busybox -- sh
```

Now we can HHHT GET push to the additional CURL tool:
```
curl http://{service_name}.{namespace}.svc.cluster.local
```

## How to test application using a NodePort Service
Use this command to see port your nodePorts
```
kubectl get svc -n {namespace}
```
For testing use address like
```
http://{NodeIP}:{NodePort}
```
Example:
```
http://localhost:30080
```
