# Django ToDo List
### Resource requests and limits
This resourse limits will ensure usage without overloading the node.
```
 resources:
          requests:
            memory: "64Mi"
            cpu: "20m"
          limits:
            memory: "128Mi"
            cpu: "100m"
```
### HPA configuration
This provides responsive autoscaling under load
```
  minReplicas: 2
  maxReplicas: 5
  averageUtilization: 70
```
### Strategy choice
RollingUpdate will provide zero downtime deployment 
```
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```
## Testing the app
### Testing access to the app using port-forward
1. Run the command below:
```bash
kubectl port-forward svc/todoapp-service 8081:80
```
2. After follow the link:
```
http://localhost:8081
```
### Testing access to the app through ClusterIp
1. Connect to busybox:
```bash
kubectl exec -it -n todoapp busybox -- sh
```
2. Execute this coomand inside the shell:
```bash
curl http://todoapp.service.todoapp.svc.cluster.local
```
### Testing access to the app using NodePort
Follow the link:
```
http://localhost:30080
```