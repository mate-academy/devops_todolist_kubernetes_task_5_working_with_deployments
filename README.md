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

##### How to deploy
1.Clone repository to your machine
2.To create namespace mateapp run
```
kubectl create namespace mateapp
```
3.To apply manyfests run
```
kubectl apply -f deployment.yml -n mateapp
kubectl apply -f hpa.yml -n mateapp
```
##### Access to the app
Use forward-port to get app from browser
run 
```
kubectl apply -f nodeport.yml -n mateapp
```
and then visit http://localhost:30080 to open app.

##### Resources explanation
Requests: These are the minimum resources required by each pod. Setting the CPU request at 100m means that each pod is guaranteed at least 100 millicores of CPU. Similarly, a memory request of 200Mi ensures at least 200 MiB of memory is allocated to each pod. These values are crucial for the Kubernetes scheduler to place pods on nodes with sufficient resources.
Limits: These values define the maximum amount of resources a pod can use. A CPU limit of 500m prevents any single pod from using more than half a CPU core, while a memory limit of 500Mi caps the memory usage. This is important to prevent any single application from monopolizing the cluster's resources, ensuring stability and availability of other services in the cluster.

##### HPA configuration explanation
The HPA dynamically adjusts the number of replicas based on real-time metrics. It ensures that the deployment can handle varying loads efficiently without manual intervention.
The HPA is configured to scale the number of pods between a minimum of 2 and a maximum of 5. It targets a resource utilization of 70% for both CPU and memory. When the average CPU or memory usage across all pods exceeds 70%, the HPA will start to create additional pods, up to a maximum of 5. Conversely, if the usage drops below this threshold, the HPA will reduce the number of pods, ensuring resource efficiency.

##### Strategy configuration explanation
RollingUpdate strategy is used to update the pods with zero downtime. By setting maxUnavailable to 1, we ensure that only one pod at a time is taken down during the update process. Similarly, maxSurge is set to 1, allowing only one extra pod to be created above the desired number of pods. This approach helps in maintaining service availability while updating or scaling the application.