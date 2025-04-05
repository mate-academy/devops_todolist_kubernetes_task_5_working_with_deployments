# Django ToDo list
## Resource Requests and Limits
```
requests:
  memory: '128Mi'
  cpu: '30m'
limits:
  memory: '256Mi'
  cpu: '60m'
```
This ensures efficient resource usage without overloading the node.
## HPA Configuration
```
minReplicas: 2
maxReplicas: 5
averageUtilization: 70
```
This provides responsive auto-scaling under load.
## Rolling Update Strategy
```
maxUnavailable: 1
maxSurge: 1
```
This allows zero-downtime deployments.
## Testing the application
### Testing the application by calling a ClusterIP service DNS from a busybox container
To get started, you need to connect to the pod using the command
1. `kubectl exec -it -n todoapp busybox -- sh`

Inside the Busybox shell, test the connectivity using `curl`
2. `curl http://todoapp-service.todoapp.svc.cluster.local`
### Testing ToDo Application Using Port-Forward
To test te application run:
1. `kubectl port-forward svc/todoapp-service 8081:80`
Open the browser add write:
2. `http://localhost:8081`
### Testing the Application using a NodePort service
Open the browser add write:
1. `http://localhost:30080`