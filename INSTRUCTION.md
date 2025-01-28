# Instructions for deploying and testing ToDo application
# Deployment
# Resource Limits
CPU: Limits are set to 250m for requests and 500m for limits. This ensures stable operation of the application during idle periods and restricts maximum CPU usage.

Memory: Limits are set to 64Mi for requests and 128Mi for limits. This provides sufficient memory for the application to operate and prevents exceeding memory limits.

# RollingUpdate Strategy
maxSurge: 1: Allows one additional pod to be created during the update process.

maxUnavailable: 1: It means that during the application update (replica pods update), no more than 1 pod can be unavailable at any given time.
# Horizontal Pod Autoscaler (HPA)
# HPA Parameters
minReplicas: 2: The minimum number of pods to ensure application availability even under low load.

maxReplicas: 5: The maximum number of pods to allow scaling of the application under increased load.

CPU and Memory Utilization: Autoscaling is triggered when CPU or memory usage reaches 70%.
# How to apply the pods manifests:

kubectl apply -f todoapp-pod.yml -n todoapp
kubectl apply -f busybox.yml -n todoapp

# How to apply the ClusterIP manifest:
kubectl apply -f clusterIP.yml -n todoapp

# How to apply the NodePort manifest:
kubectl apply -f nodeport.yml -n todoapp

# Testing the ToDo application:
kubectl port-forward pod/todoapp 8081:8080 -n todoapp

# After this, open your browser and go to:
# http://localhost:8081

# How to test an app by calling a ClusterIP service DNS from a busybox container:

kubectl -n todoapp  exec -it busybox -- sh
curl http://todoapp-service.todoapp.svc.cluster.local:80 or wget http://todoapp-service.todoapp.svc.cluster.local:80

# How to apply Deployment manifest:
kubectl apply -f deployment.yml -n todoapp
# How to apply Hpa manifest:
kubectl apply -f hpa.yml -n todoapp
# How to access an app using a NodePort service:
kubectl get nodes -n todoapp -o wide
# find the INTERNAL-IP after this, open your browser and go to:
http://<INTERNAL-IP>:30080


