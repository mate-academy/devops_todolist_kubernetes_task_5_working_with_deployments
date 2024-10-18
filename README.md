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
1. Create a `hpa.yml` file with a Horizontal Pod Autoscaler for the app.
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

## Deployment

To deploy the app to k8s, you will need to have a running cluster and `kubectl` installed. You can use the following commands to deploy the app:

```bash
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
```

## Resources requests and limits

I have tested the app locally and found that it uses around 70Mi of memory and 70Mi of CPU at startup. So I decided to set such requests to ensure that the app has enough resources to run smoothly.
Then I have checked the app in idle state and found that it still uses around ≈70Mi of memory but only ≈20m of CPU.
So I decided to set limits a bit higher than requests to ensure that the app can handle more users if needed.

## HPA configuration

I have chosen the following HPA configuration for the app based on the following assumptions:

- Minimum number of pods as 2: To ensure that the app is always available even if one pod fails.
- Maximum number of pods as 5: To ensure that the app can scale up to handle more users if needed without using too many resources.

[//]: # (`README.md` Should have explained your strategy configuration (Why such numbers))

## Strategy configuration

I have set the strategy to RollingUpdate with maxSurge and maxUnavailable set to 1 to ensure that the app is always available even during updates.

## Accessing the app

After deploying the app, you can access it by running the following command:

```bash
kubectl port-forward deployment/todoapp 8080:8080 
```

Then you can access the app at http://localhost:8080.