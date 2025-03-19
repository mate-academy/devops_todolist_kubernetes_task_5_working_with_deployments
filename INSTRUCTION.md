## Apply the deployment manifest
```
kubectl apply -f .infrastructure/deployment.yml
```

## Apply the Horizontal Pod Autoscaler
```
kubectl apply -f .infrastructure/hpa.yml
```

## Verify the deployment
```
kubectl get deployments -n todoapp
kubectl get pods -n todoapp
```

## Check the autoscaler
```
kubectl get hpa -n todoapp
```

## Accessing the Application

```
kubectl port-forward -n mateapp deployment/todo-app 8080:8080
```
