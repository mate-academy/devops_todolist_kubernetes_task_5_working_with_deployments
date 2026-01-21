kubectl apply -f namespace.yml
kubectl config set-context --current --namespace=mateapp
kubectl apply -f clusterIp.yml
kubectl apply -f nodeport.yml
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml

Request and limits resources in deployment are selected for low-load application.
HPA config is selected application requiring 24/7 operation
The scaling configuration is selected to increase the load by 2.5 times. From 2 pods to 5

Go to browser: localhost:30080



