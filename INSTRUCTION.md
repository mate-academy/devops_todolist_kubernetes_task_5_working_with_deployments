1. how to deploy the app to k8s:

kubectl create namespace mateapp

kubectl apply -f todoapp-deployment.yaml -n mateapp
kubectl apply -f todoapp-service.yaml -n mateapp

kubectl apply -f todoapp-hpa.yaml -n mateapp

2. choice of resource requests and limits

app is big, and need a lot of resourses, so I made max 5 pods and gave them more memory

3. choice of HPA configuration

just need to be like this

4. strategy configuration (Why such numbers)

garantees to have working app while updating

5. how to access the app after deployment

use command kubectl port-forward svc/todoapp 8080:80

use browser

http://localhost:8080/
