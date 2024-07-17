how to deploy the app to k8s
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
choice of resources requests and limits
The resource requests and limits are set to ensure that the application has the necessary resources to run efficiently without overloading the cluster. The chosen values are:
Requests: 64Mi memory and 250m CPU
Limits: 128Mi memory and 500m CPU
choice of HPA configuration
Minimum and Maximum Pods: The HPA will maintain a minimum of 2 pods and can scale up to a maximum of 5 pods based on resource usage.
Autoscaling Triggers: The autoscaling is triggered by:
CPU utilization exceeding 80%
Memory utilization exceeding 70%
strategy configuration (Why such numbers)
The RollingUpdate strategy ensures zero downtime during updates. 
The configuration parameters:
maxUnavailable: 1: Ensures that at least one pod remains available during updates. This value is chosen to maintain high availability of the application. If a pod becomes unavailable during an update, there will still be at least one other pod running to handle user requests.
maxSurge: 1: Allows one additional pod to be created during updates. This value is chosen to balance resource utilization and update speed. By limiting the number of additional pods to one, we prevent excessive resource consumption while still allowing the update process to proceed efficiently.
how to access the app after deployment
kubectl apply -f .infractructure/
how to test
kubectl -n todoapp exec -it busybox -- sh
curl http://{service-name}.{namespace}.svc.cluster.local
kubectl port-forward service/{service_name} 8081:80
open a web browser and navigate to http://localhost:30080