Create Deployment
``` k apply -f deployment.yml ```  
Create Horizontal Pod Autoscaler
``` k apply -f hpa.yml  ```

Access the app
``` http://localhost:8080/ ```

Resource requests and limits are specified in deployment.yml will set a resoures up and bottom CPU and Memory utilization so the pod cant use more that specified in request and do not overuse the limits to avoid crashing all pods in a node in case of failure of one pod becouse of using all the resources.

HPA configuration we are using to set a limits and by reaching them HPA will create more pods to reduce latency and scale app for better performance

Deployment strategy is created to manage our application easier and by the label deployment know which pods to manage and template setup is from what new pods will be created

How to access the app after deployment - we would need to make sure our deployment is running and all required pods are created abd running. Use ``` kubectl describe deployments ```
If everything is working as expected you can access you app by using ``` http://localhost:8080/ ```
