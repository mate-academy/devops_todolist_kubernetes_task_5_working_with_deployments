In order to deploy the application do the following:
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/clusterIp.yml

The "todoapp" pods launched are going to have the following requests and limits:

requests:
    cpu: 300m
    memory: 64Mi
limits:
    cpu: 300m
    memory: 64Mi

That is to ensure that these pods are the last ones to be evicted due to resource constraints, according to the QoS classifications Guaranteed pods should have an equal amount of requests and limits resources.

The HorizontalPodAutoscaler has a minimum number of replicas set to "2", that is to ensure high availability and redundancy and to ensure that rolling updates are possible.
The CPU / memory target utilization for scaling is set to 70% for both types of resources, this way the pods can be scaled out in advance before they reach their memory limits. 

The deployment "strategy" is set to "rollingUpdate" with a maxSurge of 1 and maxUnavailable of 1. Rolling updates ensure that we are always going to have pods running, which means that the users would always be able to access our application even when updating pods, maxUnavailable ensures that there is always going to be at least one pod that is accessible to the users, maxSurge 1 ensures that we can scale beyond the desired amount of replicas when updating.

After applying the manifest files the application can be accessed by using port-forwarding in the following way:

kubectl port-forward service/todoapp 7080:80 (now accessible from localhost:7080)