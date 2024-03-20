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

## how to deploy the app to k8s:

```
kubectl apply -f deployment.yml
```

```
kubectl apply -f hpa.yml
```

## Explaining the choice of resources requests and limits

I choose values from the theory video for studding purpose.
By setting these requests and limits, we ensure that no more than two pods (as specified) can run simultaneously without overloading the system. These values are tailored to the characteristics of the PC where we deploy the app

## Explaining the choice of HPA configuration

- minReplicas to 2 and maxReplicas to 5 - app can dynamically scale up or down based on demand
- averageUtilization: 70 - utilization for both CPU and memory are set at 70%.If the utilization exceeds 70%, additional pods will be created, and if it falls below, excess pods will be terminated

## Explaining the strategy configuration

- RollingUpdate - strategy ensures a gradual rollout of new versions. It replaces old pods with new ones incrementally, minimizing service disruption. By allowing only one pod to be updated at a time (maxSurge: 1), it maintains a smooth transition
- maxUnavailable: 1 - setting ensures that at least one old pod remains available during the update process

## How to access the app after deployment

```
kubectl apply -f .infractructure/
```

## Test app with the port-forward

```
kubectl port-forward service/{service_name} 8081:80
```

## Test app with ClusterIP

```
kubectl -n {namespace} exec -it busybox -- sh
```

```
curl http://{service_name}.{namespace}.svc.cluster.local
```
