## preparing pods and deployment

1. Create the namespace and check that it exists: 
```
kubectl apply -f .infrastructure/namespace.yml &&
kubectl get namespace
```

2. Create the Deployment (pods and controller). Next check its creation: 
```
kubectl apply -f .infrastructure/deployment.yml &&
kubectl get deployment
```

3. Create horizontal pod autoscaler: 
```
kubectl apply -f .infrastructure/hpa.yml
```

## !Important. Do you have metrics server:
```
kubectl top nodes
```

 **If you don't see metrics for your pods, it means you don't have it installed. You can install it with the next steps:**

4. Check if the metrics-server is installed in the cluster (required for the HPA to read CPU/Memory metrics):
```
kubectl get deployment metrics-server -n kube-system
```

5. If it is not installed, install it:
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

6. Check that the metrics-server is running and reporting metrics:
```
kubectl get deployment metrics-server -n kube-system
kubectl top nodes
```

### explanation
- The limits and requests were chosen for a minimal working setup. If you need more, adjust the memory and cpu values in .infrastructure/deployment.yml (increase cpu if the app runs slowly, increase memory if it's sometimes killed by the OOM killer — memory is in MiB). The HPA triggers scaling once usage reaches 70% of the values written in deployment.yml: if the app needs more, k8s creates new pods, up to a maximum of 5 (minimum 2). For non-stop availability during updates, the RollingUpdate strategy creates 1 new pod before removing an old one (maxSurge: 1, maxUnavailable: 1).

## accessing the app after deployment

This task's manifests (deployment.yml, hpa.yml) don't include a Service, so the simplest way to reach the app is port-forwarding straight to the Deployment:

```
kubectl port-forward -n mateapp deployment/todoapp-deploy 8080:8080
```

Then open the app at:
```
http://localhost:8080
```
