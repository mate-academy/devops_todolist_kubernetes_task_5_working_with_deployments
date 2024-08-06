# Create namespace and apply deployment and hpa files
kubectl create namespace mateapp

kubectl apply -f deployment.yml

kubectl apply -f hpa.yml

# The following resource requests and limits are set to ensure that the application has enough resources to run efficiently without overloading the cluster:
Requests: 64Mi memory and 250m CPU

Limits: 130Mi memory and 500m CPU

# The HPA configuration ensures that the application can scale to handle varying loads while maintaining resource efficiency. 
The target averageUtilization: 70 was chosen to balance performance and cost, ensuring that the application can respond to increased demand without over-provisioning resources.

# Choice of Strategy Configuration
The RollingUpdate strategy with maxUnavailable: 1 and maxSurge: 1 ensures a smooth update process with minimal impact on application availability. 

This configuration balances the need for continuous availability with the ability to deploy updates efficiently.

# How to access and test the application after deployment
kubectl apply -f .infractructure/

kubectl -n todoapp exec -it busybox -- sh

curl http://{service-name}.{namespace}.svc.cluster.local

kubectl port-forward service/{service_name} 8081:80

# Or accessing via browser
http://localhost:30080
