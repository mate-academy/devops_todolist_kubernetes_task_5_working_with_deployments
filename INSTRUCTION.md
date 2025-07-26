1. Make sure that you have access to the Kubernetes cluster and configured kubectl

2. Verify that namespace `mateapp` does exist, if not - create it:

```bash
kubectl get namespace mateapp || kubectl create namespace mateapp
```

3. Use Deployment:

```bash
kubectl apply -f deployment.yml
```

4. Use HorizontalPodAutoscaler:

```bash
kubectl apply -f hpa.yml
```

5. Verify, that Deployment and HPA were created:

```bash
kubectl get deployments -n mateapp
kubectl get hpa -n mateapp
```

## If you want to get access to the app after deployment, you can use this command:

```bash
kubectl port-forward deployment/todoapp 8080:80 -n mateapp
```