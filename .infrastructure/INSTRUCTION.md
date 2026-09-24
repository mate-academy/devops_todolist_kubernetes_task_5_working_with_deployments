# Deploying an application in a Kubernetes cluster.
## Works with Docker Desktop
### Installation Instructions:
1. Create a namespace.
```
kubectl apply -f namespace.yml
```
*For convenience, it's safe to say that you should use the default namespace.*
```
kubectl config set-context --current --namespace=todoapp
```

2. Start the Deployment
```
kubectl apply -f deployment.yml
```

3. Start the service ClusterIP.
```
kubectl apply -f clusterIp.yml
```
4. Start the service NodePort
```
kubectl apply -f nodeport.yml
```
5. Start the HorizontalPodAutoscaler
```
kubectl apply -f hpa.yml
```
To test the ToDo program using the `port-forward` command in `service`, run the following command:
```
kubectl port-forward pod/<PodeName> 30080:8080
```
The application should be accessed via the NodePort service at the following address:
http://localhost:30080

or set up port forwarding to the service 

```
kubectl port-forward -n mateapp svc/todoapp 8000:80
```

The application should be accessed at the following address:
http://localhost:8000/

## Resource Requests/Limits
- requests: cpu 100m, memory 128Mi — Minimum requirements for running Django.
- limits: cpu 200m, memory 256Mi — buffer for peak loads, 2x the number of requests.


## HPA Configuration
- minReplicas: 2, maxReplicas: 5
- trigger: 70% CPU та Memory — a balance between response speed and resource efficiency.


## Strategy
- RollingUpdate — Safe updates with no downtime.