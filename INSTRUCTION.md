Clone repository
git clone repository
go to folder repository

## DEPLOY
create namespace
kubectl apply -f namespace.yml
check namespaces
kubectl get ns
create deployment
kubectl apply -f deployment.yml
check replica
kubectl get pods -n mateapp -o wide

## DEPLOY FOR CHECKING
create busybox
kubectl apply -f busybox.yml
create service clusterip
kubectl apply -f clusterIp.yml
create node port
kubectl apply -f nodeport.yml

## Deploy HPA
create horizontal autoscaler
kubectl apply -f hpa.yml
check replica
kubectl get pods -n mateapp -o wide

## TESTING
test clusterip
kubectl port-forward service/todoapp-service 8080:80 -n mateapp
go to page http://localhost:8080
or use busybox app
kubectl -n mateapp exec -it busybox -- sh
curl http://todoapp-service.todoapp.mateapp.svc.cluster.local
test nodeport
go to page http://localhost:30080

## DATA EXPLANE
min replica 2 need for stability work and  high availability, zero-downtime, 
max replica 5 for managing resources with stress activity, more no needed if was more expensive  
maxUnavailable: 1, for rolling update with zero downtime one app always available, maxSurge: 1 for create 1 more app if it`s needed
and 70% resources for efective scale it`s middle chuse no 50% (more replica) and no 90% (time to reaction)
0.25 CPU and 64MB RAM full for our app it`s minimal for pod
