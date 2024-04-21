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

## Deploying the App to Kubernetes
Ensure you have kubectl installed and configured to interact with your Kubernetes cluster.
kubectl apply -f deployment.yml
## Verify the Deployment
Confirm that the pods are running:
kubectl get pods -n mateapp
## Resource Requests and Limits Explanation
- Memory and CPU Requests: We have set the memory request at "128Mi" and CPU request at "500m" because this is the baseline amount of resources needed for our application to run smoothly under normal conditions.
- Memory and CPU Limits: The limits are set at "256Mi" for memory and "700m" for CPU to ensure that our application does not consume excessive cluster resources under peak load, which could potentially affect other applications running in the same cluster.
## Horizontal Pod Autoscaler (HPA) Configuration
The HPA is configured to automatically scale the number of pods based on CPU usage, ensuring that the application can handle increases in traffic while maintaining performance.
Explain the specific metrics and thresholds set (e.g., CPU utilization at 80%, minimum and maximum pod counts), and why these were chosen based on the application's performance profile and traffic expectations.
## Strategy Configuration Explanation
RollingUpdate Strategy: This strategy allows us to update the application with zero downtime by controlling the maxSurge and maxUnavailable parameters.
- maxSurge (1): Allows one extra pod during the update, which helps in maintaining service availability.
- maxUnavailable (1): Ensures that only one pod at a time is unavailable during the rolling update, minimizing the impact on availability.
## Service Access
kubectl get svc -n mateapp
If using NodePort, you might access your app via http://<NodeIP>:<NodePort>.
