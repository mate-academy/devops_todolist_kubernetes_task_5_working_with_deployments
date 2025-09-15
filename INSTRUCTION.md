## App deploy to k8s:
 - deploy namespace with command ```kubectl apply -f .infrastructure/namespace.yml```
 - deploy deployment with command ```kubectl apply -f .infrastructure/deployment.yml -n mateapp```
 - deploy hpa with command ```kubectl apply -f .infrastructure/hpa.yml -n mateapp```
 - deploy one of the next services:
   - clusterIp with command ```kubectl apply -f .infrastructure/clusterIp.yml -n mateapp```
   - or nodeport with command ```kubectl apply -f .infrastructure/nodeport.yml -n mateapp```

## Deployment strategy explanation:
Selected strategy is `RollingUpdate` with options `maxUnavailable` - 0, and `maxSurge` - 1. This will prevent reducing number of working pods for better client experience by adding instances of new version on top of existing. 
Since the app is small and maximum number of pods if 5 (explained below) - `maxSurge` - 1 is a good balance between fast updates and cost efficiency. 


## Resources requests and limits explanation:

Since the app is very small and minimalistic, it will not use huge amount resources. Base on this, resources requests set to:
- cpu - 250m
- memory - 256Mi

To handle traffic spikes and high loads without triggering autoscaling resources limits set to:
- cpu - 500m
- memory - 512Mi


## Resources requests and limits explanation:
The purpose of HPA is adding new pods when existing facing high resources utilization. To handle this HPA metrics are set to:
- cpu average utilization - 75%
- memory average utilization - 75%

It allows for existing pods handle traffic until they occupied most of available resources and prevent scaling when resources utilization is low or moderate.

Minimal number of instances is 2. It will allow to balance traffic between them and handle more requests.
Maximum number of instances is 5. It will allow to handle good amount of users but prevent occupation of all host resources.

## How to reach the app:
- if you deployed the nodePort service - open the app on the [localhost:30080](http://localhost:30080)
- if you deployed the clusterIp port - run the command ```kubectl port-forward service/todoapp 8080:80 -n mateapp``` and open [localhost:8080](http://localhost:8080)

## Waring:
Do not apply `todoapp-pod.yml` specification if Deployment/HPA were applied. It will create duplicates of pods that will also be handled by Service and cause unexpected behavior of the k8s
