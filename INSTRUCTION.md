# RollingUpdate allows the application to be updated without stopping all existing pods at the same time.

# The minimum number of replicas is 2 to provide basic availability. If one pod becomes unavailable, another pod can continue serving requests.

# The maximum number of replicas is 5. This provides enough capacity for increased traffic without allowing unlimited pod creation.



# Apply al manifests
kubectl create namespace mateapp
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
# Confirm pods rolling out
kubectl get pods -n mateapp -w
kubectl get deployment todoapp -n mateapp
kubectl get hpa todoapp -n mateapp
# Acces the app after deployment
kubectl port-forward -n mateapp deployment/todoapp 8080:8080