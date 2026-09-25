# To deploy:

## Preparation:
To create all the services, nodes, pods, etc. run:
```
# Create namespaces
kubectl apply -f .infrastructure/namespace.yml

# Create pods
kubectl apply -f .infrastructure/deployment.yml

# Create services
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/nodeport.yml
```
## To access:
To accsses website with todoapp run go on:
localhost:30080
or
127.0.0.1:30080

# Resource explanation:

The todoapp resources requests are:
memory: "64Mi"
cpu: "250m"

And limits are:
memory: "128Mi"
cpu: '500m'

This will provide enough resources to run the app and if it will consume to many(500m or 128Mi) the pods going to be killed with OutOfMemory
Deployment also uses the RollingUpdate type of how the replicas should update, so maxUnavailable: 1 and maxSurge: 1 means that its only can be 1 pod unavailable and 1 pod updated. This method will provide continuous connect to app 

To provide autoscaling when resources not enough, so they are on 70% and more to limits hpa going to create up to 5 replicas what is enough to app to work correctly

