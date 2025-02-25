## instructions on how to deploy the app to k8s
``` bash
cd .infrastructure
kubectl create namespace mateapp
kubectl set-context --current --namespace=mateapp
kubectl apply -f busybox.yml
kubectl apply -f clusterIp.yml
kubectl apply -f nodeport.yml
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

## choice of resource requests and limits
The resource requests and limits are set to 5m CPU and 10Mi memory. This is because the 
app is a simple to-do list app that does not require many resources to run. The app is 
also not expected to have a lot of traffic, so the resource requests and limits are set to 
a minimum to save resources and be able to test deployment and hpa configs with additional
loads.

## choice of HPA configuration
The HPA configuration is set to a minimum of 2 pods and a maximum of 5 pods. Metrics are
set to CPU utilization of 70% and memory utilization and memory utilization of 70%. The
metrics are set to 70% because it is the normal working load for the app.

## strategy configuration (Why such numbers)
The strategy configuration is set to RollingUpdate because it is the most common strategy 
used in production environments. The strategy is set to minimum 2 pods and maximum 5 pods.
The strategy is set to 2 pods because it is the minimum number of pods required to run the
app. The strategy is set to 5 pods because it is the maximum number of pods that can be
scaled up to handle additional loads.

## how to access the app after deployment
The app can be accessed by running the following command:
``` bash
kubectl port-forward service/todoapp 8081:80 -n mateapp
```
and then opening http://localhost:8081/api in a web browser.
