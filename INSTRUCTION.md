# How to deploy the Todoapp to Kubernetes.

## 1 Namespace creating.
    kubectl apply -f .infrastructure/namespace.yml

## 2 Image build.
    docker build . -t timnord1993/todoapp:3.0.0

## 3 Services adding.

### 3.1 Applying clusterIp service.
    cubectl apply -f .infrastructure/services/clusterip.yml -n todoapp

### 3.2 Applying nodePort service.
    cubectl apply -f .infrastructure/services/nodePort.yml -n todoapp

## 4 Applying pods.
    cubectl apply -f .infrastructure/todoapp-pod.yml -n todoapp

## 5. Applying Deployment
- Resource requests specify minimum guaranteed resources for pod to be launched even on node with limited resources.
100m CPU and 128Mi RAM is enough for light python service.

- 250m CPU and 256Mi RAM are reasonable upper-limits that prevent the pod from consuming excessive resources.


    kubectl apply -f deployment.yml -n todoapp

- RollingUpdate strategy was chosen to make updates smoothly and without downtime. 
maxUnavailable: 1  ->  In order to keep the app running only 1 pod can be unavailable during updating.
maxSurge: 3  ->  In order to apply new update faster you can create one additional pod.

## 6. Applying Horizontal Pod Autoscaler
- Since the load on the ToDo service is low, initial quantity minReplicas=1 is enough.
- As the load increases, HPA will increase the number of replicas to 3  ->  maxReplicas=5
- averageUtilization: 70 was chosen as balanced value: it allows to scale automatically and not to keep excessive pods constantly.


    kubectl apply -f hpa.yml -n todoapp