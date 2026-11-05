# the instructions on how to deploy the app to k8s
cd /path/to/project/.infrastructure
kubectl apply -f namespace.yml
kubectl apply -f busybox.yml
kubectl apply -f todoapp-pod.yml
kubectl apply -f clusterip.yml
kubectl apply -f nodeport.yml
kubectl config set-context --current --namespace=mateapp

# P.S. if busybox.yml doesn't work use
kubectl apply -f alpine.yml

# instructions on how to test an app by calling a ClusterIP service DNS from a busybox container
kubectl exec -it -n mateapp busybox -- sh
# if busybox doesn't work
kubectl exec -it -n mateapp alpine -- sh
# run command 
curl http://todoapp-service.mateapp.svc.cluster.local
# you'll see html code 

# my choice of resource requests and limits:
was chosen by analyzing requirements of application while it run with load and without it

# my choice of HPA configuration:
-Minimum number of pods as 2
-Maximum number of pods as 5
this parameters due to task
Autoscale should be triggered by both CPU and Memory
70% of load to give some time to start new pods if load will increase

# my strategy configuration:
RollingUpdate was chosen cause I have no significat restrictions in hardware resources

# how to access the app after deployment
url for local browser: http://127.0.0.1:30080/