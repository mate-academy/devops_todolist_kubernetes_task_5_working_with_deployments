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

### how to deploy the app to k8s

Use these commands:

1) kubectl apply -f deployment.yml

2) kubectl apply -f hpa.yml

### Explanation of resources requests and limits

I chose the resource requests and limits based on the expected resource usage of the application. For memory, I allocated 64Mi as the minimum request and 128Mi as the maximum limit, considering the application's memory requirements. Similarly, for CPU, I set a request of 250m and a limit of 500m to ensure adequate processing power for the application's expected workload, while preventing excessive resource consumption that could lead to contention with other pods on the cluster. Additionally, these values were selected based on the principles outlined in the topic.

### Explanation of HPA configuration

HPA configuration was chosen to dynamically scale the number of pod replicas based on the observed resource metrics of the application from 2 to 5 Replicas. The 'scaleTargetRef' specifies the Deployment to scale, 'todoapp', while 'minReplicas' and 'maxReplicas' define the lower and upper bounds for scaling, respectively. For metrics, both CPU and memory utilization are considered, with a target average utilization of 70% for each. This ensures that the cluster efficiently allocates resources to meet demand while preventing over-provisioning or under-utilization. By utilizing both CPU and memory metrics, the HPA reacts to different types of resource bottlenecks, optimizing the application's performance and resource utilization in dynamic environments.

### Explanation of strategy configuration (Why such numbers)

The RollingUpdate strategy was chosen to gradually introduce new versions, replacing old pods incrementally to minimize disruptions. By allowing only one additional pod during updates and ensuring at least one old pod remains available ('maxSurge: 1' and 'maxUnavailable: 1' respectively), the strategy balances efficient deployment with maintaining service availability. These settings align with best practices for continuous delivery and service reliability.

### how to access the app after deployment


1. start manifest:

- kubectl apply -f nodeport.yml

You should got such answer: 

service/todoapp-nodeport-service created

2. Get name of service:

- kubectl get services -o wide

3. Check service:

Externall Access:

http://"nodeIP":30080
```
on localhost:
http://localhost:30080 
