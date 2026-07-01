1. To deploy the app to k8s, you need to follow these steps:

Need to run the following commands in your terminal:

```bash
kubectl apply -f namespace.yml
kubectl apply -f clusterIp.yml
kubectl apply -f nodeport.yml
kubectl apply -f busybox.yml
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

2. I choose 70% CPU and 70% Memory as the threshold for the HPA configuration because it allows the application to scale up when the resource usage is high, ensuring that the application can handle increased load without performance degradation. The minimum number of pods is set to 2 to ensure that there is always a baseline level of availability, while the maximum number of pods is set to 5 to prevent over-provisioning and excessive resource  consumption.

The resource requests and limits are set to `64Mi`/`128Mi` for memory and `250m`/`500m` for CPU because this is a small Django ToDo application with a lightweight expected idle footprint. The requests reserve enough resources for each pod to start and run reliably, while the limits prevent a single pod from consuming too much CPU or memory under higher load. These values also work well with the HPA configuration, because Kubernetes can schedule two baseline pods and still scale the Deployment when CPU or memory utilization reaches the configured threshold.


3. The strategy configuration is set to RollingUpdate to ensure that the application can be updated without downtime. This strategy allows for a gradual replacement of old pods with new ones, ensuring that there is always a minimum number of pods available to handle requests.

4. To access the app after deployment, you can use the NodePort service that exposes the application on a specific port of the node's IP address. You can find the NodePort by running the following command:

```bash
kubectl get svc -n todoapp
```
