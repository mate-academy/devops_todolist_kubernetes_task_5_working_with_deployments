# RollingUpdate allows the application to be updated without stopping all existing pods at the same time.

# The minimum number of replicas is 2 to provide basic availability. If one pod becomes unavailable, another pod can continue serving requests.

# The maximum number of replicas is 5. This provides enough capacity for increased traffic without allowing unlimited pod creation.

# The CPU target is 70%. When the average CPU utilization of the pods exceeds this level, the HPA can increase the number of replicas.

# The application is a small Django ToDo application and does not require significant CPU or memory resources under normal load.

# The CPU request of 100m guarantees that the Kubernetes scheduler reserves a small amount of CPU for each pod. The CPU limit of 500m allows the application to temporarily use more CPU when required.

# The memory request of 128Mi is sufficient for the application to start and operate under normal conditions. The memory limit of 512Mi provides additional memory for temporary increases in application usage while preventing a single pod from consuming an excessive amount of node memory.


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