# Django ToDo list 
The choice of resource requests and limits is based on the expected resource utilization of the application.
These values are chosen to ensure that each pod in the Deployment has enough resources to operate efficiently

`Horizontal Pod Autoscaler (HPA)` Configuration is configured to automatically adjust the number of replicas based on resource utilization metrics.
This configuration ensures that the application can handle varying levels of load efficiently. When the average resource utilization exceeds 70%,
the HPA will automatically scale up the number of replicas, up to a maximum of 5, to handle the increased demand. 

`RollingUpdate` strategy was chosen to ensure the availability of our service 24/7.
Values for "Max Unavailable" and "Max Surge" was chosen to the service load of the app.


## App deployment
1. Get into the directory `.infrastructure` using `cd` command
2. Use `kubectl apply -f deployment.yml`
3. Use `kubectl apply -f hpa.yml`


### App should be available at `http://localhost:30007/`