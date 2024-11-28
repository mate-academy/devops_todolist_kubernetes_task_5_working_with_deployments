# Django ToDo list

This is a to-do list web application with the basic features of most web apps, i.e., accounts/login, API, and interactive UI. To do this task, you will need:

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

You can now browse the [API](http://localhost:8000/api/) or start on the [landing page](http://localhost:8000/).

## Task

Create a Kubernetes manifest for a pod that will contain a ToDo app container:

1. Fork this repository.
1. Create a `deployment.yml` file with a deployment for the app.
1. Deployment should have
    1. Strategy: RollingUpdate
    1. Resource requests and limits (in the idle state you should have 2 pods running)
    1. Pod spec should be same as for pods manifest
1. Create a `hpa.yml` file with a Horizontal Pod Autoscaler for the app.
1. Autoscaler should define
    1. Minimum number of pods as 2
    2. Maximum number of pods as 5
    3. Autoscale should be triggered by both CPU and Memory
1. Both new manifests should belong to `mateapp` namespace
1. `INSTRUCTION.md` should be created with the instructions on how to deploy the app to k8s
1. `INSTRUCTION.md` Should have explained your choice of resource requests and limits
1. `INSTRUCTION.md` Should have explained your choice of HPA configuration
1. `INSTRUCTION.md` Should have explained your strategy configuration (Why such numbers)
1. `INSTRUCTION.md` Should have explained how to access the app after deployment
1. Create PR with your changes and attach it for validation on a platform.

## How to deploy the app to k8s:

Use commands:

```
kubectl apply -f deployment.yml
```

```
kubectl apply -f hpa.yml
```
# Explanation with my choice of resources requests and limits

```
Simply because these boundaries were in theory, and they work well for our learning environment.
```

# Test app with the port-forward

Command:

```
kubectl port-forward service/{service_name} 8000:80
```

## Test app with ClusterIP

Commands:

```
kubectl -n {namespace} exec -it busybox -- sh
```

```
curl http://{service_name}.{namespace}.svc.cluster.local
```