<!-- how to deploy the app to k8s -->
kubectl apply -f ./.infrastructure/deployment.yml
kubectl apply -f ./.infrastructure/hpa.yml  

<!-- resource requests and limits explanation-->

<!-- minimal memomry and cpu allovated to pod on node is 64Mi and 250m accordigly. 
if pod recources are overused by memory 128Mi and cpu 500m - pod will be killed -->

<!--  HPA configuration explanation-->
<!-- if 75% memory or 75% of cpu is used - this will trigger autoscaling 
once resources dropped - scaling back to lower resources ( >= minReplicas) will be done -->
<!--  strategy configuration explanation -->
<!-- RollingUpdate will create new po and take one down till all resources are recreated -->
<!-- how to access the app after deployment -->