1. Create namespace for application

```
kubectl apply -f .infrastructure/namespace.yml
```

2. Update default config namespace for more comfortable running application

```
kubectl config set-context --current --namespace=mateapp
```

3. Create deployment

```
kubectl apply -f .infrastructure/deployment.yml
```

4. Check for successfully created deployment

```
kubectl get deployment -o wide
kubectl get replicaset -o wide
kubectl get pods -o wide
```

5. Add and check pod and services from previous task

```
kubectl apply -f .infrastructure/clusterIp.yml
kubectl apply -f .infrastructure/nodeport.yml
kubectl apply -f .infrastructure/curl.yml
kubectl get svc -o wide
```

6. Create Metrics Server service to

```
kubectl apply -f .infrastructure/metricsServer.yml
```

7. Check for successfully created Metrics Server service

```
kubectl get pods -A
```

8. Create Horizontal Pod Autoscaler

```
kubectl apply -f .infrastructure/hpa.yml
```

9. Now you could monitor used resources by executing

```
kubectl get hpa -A -w
```

10. Deployment was configured with requested 64Mi memory and 250m of CPU, and 512Mi memory and 500m of CPU limits in high loading, after start testing and got memory overloading even with 4Gi in limit of memory.
11. HPA configuration checking loading of working pods and if loading of memory or cpu increase over 70% HPA will run extra pod, due to task requirements minimum amount of replicas is 2 and maximum 5
12. Deployment has RollingUpdate strategy type, it ensure 24/7 working application with 1 extra working replica and 1 unavailable replica during updating
13. Now you can enjoy fantastic todo application on http://localhost:30007/ or http://127.0.0.1:30007/
