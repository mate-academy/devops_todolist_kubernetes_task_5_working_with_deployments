kubectl apply -f .infrastructure/namespace.yml

kubectl apply -f .infrastructure/busybox.yml

kubectl apply -f .infrastructure/clusterIp.yml

kubectl apply -f .infrastructure/deployment.yml

kubectl apply -f .infrastructure/nodeport.yml
http://<node-ip>:30080

kubectl config set-context --current --namespace=mateapp

kubectl get pods

kubectl get deployment

kubectl get replicaset

kubectl top node

kubectl get hpa
---
requests is the minimum number of resources that Kubernetes guarantees to a container
limits is the maximum number of resources that the container can use
for small web applications like Flask or Django it is enough:
requests 100m/128Mi what does it mean 100 millicore (10% of one core CPU) 
and 128 mebibytes (134 megabytes) 
limits 500m/512Mi what does it mean 500 millicore (50% of one core CPU) 
and 512 mebibytes (536 megabytes)

kubectl apply -f .infrastructure/hpa.yml
minReplicas: 2 becouse if one Pod goes down, the other will continue to serve requests;
you can update the application without downtime;
the load is distributed between the two Pods
maxReplicas: 5 becouse this is enough to demonstrate autoscaling;
not creating too many Pods;
not overloading a small cluster
averageUtilization: 70 because If you wait until 100%, users may already experience delays. At 70%, there is still a margin, so new Pods are launched early
maxUnavailable: 1 Means that only one Pod can be unavailable at a time during an update
maxSurge: 1 Means that Kubernetes can create one additional Pod beyond the number of replicas
maxUnavailable: 1, maxSurge: 1 - This is the simplest and safest strategy

In general, these values ​​provide a balance between baseline resource consumption, peak loads, availability, and deployment speed