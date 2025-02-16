To deploy the app to k8s run the following commands:
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml

My choice of resource requests and limits are:
Requests: 64 Mi (approximately 67Mb) for memory and 200m for CPU (1/5 of CPU)
Limits: 64 Mi (approximately 67Mb) for memory and 500m for CPU (1/2 of CPU)

For Horizontal pod Autoscaler I set 
minimal number of pods 2
maximum 5

As the minimal number of pods set to 2, the maxUnavailable is set to 1, in this case at least 1 pod remains accessible during update
maxSurge is set to 1 to optimise system load on update

To access the app using a NodePort Service run 
kubectl apply -f .infrastructure/nodeport.yml
and open in a browser http://localhost:30080
