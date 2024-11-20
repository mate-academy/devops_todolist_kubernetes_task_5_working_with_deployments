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


### Deploy app:
```
kubectl apply -f deployment.yml
```
```
kubectl apply -f hpa.yml
```
### Resources requests and limits:
**Requests**: 
CPU: 250m, Memory: 64Mi

**Limits**:
CPU: 500m, Memory: 128Mi

These values are chosen based on the observed resource usage of the Todo application. Setting low requests and high limits ensures the application gets the resources it needs while preventing excessive resource usage.

### HPA Configuration:
The Horizontal Pod Autoscaler (HPA) automatically scales the number of pods in a deployment based on resource utilization. This helps ensure your application has enough pods to handle traffic without running out of resources. The minimum replicas ensure at least two pods are always running, even if utilization is low. The maximum replicas define the upper limit for scaling. We target 70% average CPU and memory utilization to balance resource usage and cost-efficiency.

### Strategy Configuration:
We've configured a RollingUpdate deployment strategy with _`maxSurge`_ and `maxUnavailable` set to 1. This ensures a smooth and controlled deployment process.
`maxSurge`: Limits the number of additional pods that can be created during a deployment.
`maxUnavailable`: Limits the number of pods that can be unavailable during a deployment.
By setting both values to 1, we minimize downtime and ensure the application remains available during updates.

### Accessing the App:
```
kubectl apply -f .infractructure/
```
```
kubectl port-forward service/{service_name} 8080:8080
```
```
curl http://{service_name}.{namespace}.svc.cluster.local
```