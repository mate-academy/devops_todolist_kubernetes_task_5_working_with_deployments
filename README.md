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


---

## Deploying the Application to Kubernetes
First, ensure kubectl is installed and connected to your Kubernetes cluster. Deploy the app with:
```bash
kubectl apply -f deployment.yml
```
## Checking the Deployment
Verify that the pods are running with:
```bash
kubectl get pods -n todoapp
```
## Autoscaler Deployment
Deploy with:
```bash
kubectl apply -f hpa.yml
```
## Resource Requests and Limits
 - Memory and CPU Requests: Set at "128Mi" for memory and "250m" for CPU, ensuring sufficient resources for the basic functionality of the Django application under typical workloads.
 - Memory and CPU Limits: Set at "256Mi" for memory and "500m" for CPU to prevent overuse of cluster resources during peak times.
## Horizontal Pod Autoscaler (HPA)
The HPA adjusts the number of pods based on CPU usage, ensuring the app handles increased traffic while maintaining performance. Set CPU utilization at 80%, with specified minimum and maximum pod counts based on expected traffic.

## Update Strategy
RollingUpdate Strategy: Allows updates with no downtime by controlling:

 - maxSurge (1): One extra pod during updates.
 - maxUnavailable (1): Only one pod unavailable at a time during updates.

## Accessing the Service
### Test app with the port-forward
```
kubectl port-forward service/{service_name} 8000:80
```

### Test app with ClusterIP
```
kubectl -n {namespace} exec -it busybox -- sh
```

```
curl http://{service_name}.{namespace}.svc.cluster.local
```