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


# Instructions

## Deploying TodoApp to Kubernetes

Create namespace:

`kubectl create namespace mateapp`

Apply the Kubernetes manifests:

`kubectl apply -f .infrastructure/deployment.yml -n mateapp`
`kubectl apply -f .infrastructure/mysecret.yml -n mateapp`
`kubectl apply -f .infrastructure/hpa.yml -n mateapp`
`kubectl apply -f .infrastructure/nodeport.yml -n mateapp`

## Configuration Explanations

Requests:
CPU: 200m - This ensures that each pod has at least 20% of a CPU core available. It’s chosen to guarantee the app has enough CPU to run under normal conditions.
Memory: 400Mi - This ensures that each pod gets at least 400MB of memory, which is the baseline requirement for the app to function without thrashing or excessive swapping.
Limits:
CPU: 400m - This caps the CPU usage to 40% of a CPU core. It's a safeguard against runaway processes that could starve other pods of CPU time.
Memory: 800Mi - This limits the memory usage to prevent any memory leak or inefficient memory usage from affecting other pods on the same node.

Horizontal Pod Autoscaler (HPA) Configuration
Min replicas: 2 - Ensures that there are at least two instances of the app running to handle the load and provide high availability.
Max replicas: 5 - Caps the number of pods to 5 to prevent over-scaling that could exhaust cluster resources.
CPU average utilization: 70% - Triggers scaling up when the CPU usage goes beyond 70% of the request, ensuring the app scales up before it gets too slow.
Memory average utilization: 70% - Similar to CPU scaling, it helps in handling increases in traffic by adding more pods based on memory usage.

Strategy Configuration
RollingUpdate:
MaxUnavailable: 1 - Allows one pod at a time to be unavailable during the update process. This ensures that there is minimal impact on the app's availability.
MaxSurge: 1 - Allows creating one more pod than the desired number during the update, which helps in keeping the app fully available during the update, as there’s always an extra pod serving traffic.

## Testing the App After Deployment:
NodePort Service: The TodoApp is exposed via a NodePort service on port 30090.

http://localhost:30090