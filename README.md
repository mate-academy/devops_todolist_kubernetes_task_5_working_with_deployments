# Kubernetes TodoApp Deployment

## 1. Apply the manifests to create the namespace, BusyBox pod, NodePort and ClusterIP services, Horizontal Pod Autoscaler, and app pods through Deployment manifest:

Run the following command to apply each manifest individually as follows:
```bash
kubectl apply -f ./.infrastructure/namespace.yml
kubectl apply -f ./.infrastructure/clusterip.yml
kubectl apply -f ./.infrastructure/nodeport.yml
kubectl apply -f ./.infrastructure/busybox.yml
kubectl apply -f ./.infrastructure/deployment.yml
kubectl apply -f ./.infrastructure/hpa.yml
```

## 2. Check status of the deployed pods:
```bash
kubectl get pods -o wide -n todoapp
```

## 3. Resource Requests and Limits
- Memory:
    - Request: 64Mi — The pod gets at least 64MB of memory to run.
    - Limit: 128Mi — The pod can use up to 128MB of memory. If it goes over, it will be restarted.

- CPU:
    - Request: 250m — The pod is guaranteed to get a quarter of a CPU core.
    - Limit: 500m — The pod can use up to half a CPU core.

These settings make sure the pod runs smoothly without using too many resources.

## 4. HPA configuration
- minReplicas: 2 — The app will always have at least 2 pods running.
- maxReplicas: 5 — The app can scale up to 5 pods if needed.
- CPU and Memory: The app will scale if CPU or memory usage goes over 70%. This ensures the app can handle more traffic without performance issues, but won't use too many resources.

## 5. Deployment Strategy
- RollingUpdate: This strategy ensures that the app is updated without downtime by updating a few pods at a time.
    - maxUnavailable: 1 — Only one pod will be unavailable during an update, keeping the app running with at least one pod.
    - maxSurge: 1 — One extra pod can be created during the update, speeding up the process without causing downtime.

## 6. Accessing the App
You can access the app through the NodePort service in your web browser at: http://[localhost:30080](http://localhost:30080/)

Or you can test the app using busybox and curl command:

1. Connect to BusyBox pod: 
```bash
kubectl -n todoapp exec -it busybox -- sh
```
2. Execute curl command:
```bash
curl http://todoapp-clusterip.todoapp.svc.cluster.local 
```

This should return a response from the TodoApp if it’s running correctly.
