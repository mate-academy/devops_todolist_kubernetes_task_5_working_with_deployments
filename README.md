# how to deploy the app to k8s:

```
kubectl apply -f deployment.yml
```
```
kubectl apply -f hpa.yml
```

I choose values from the theory video for studding purpose.
By setting these requests and limits, we ensure that no more than two pods (as specified) can run simultaneously without overloading the system. These values are tailored to the characteristics of the PC where we deploy the app

## Explaining the choice of HPA configuration

- minReplicas to 2 and maxReplicas to 5 - app can dynamically scale up or down based on demand
- averageUtilization: 70 - utilization for both CPU and memory are set at 70%.If the utilization exceeds 70%, additional pods will be created, and if it falls below, excess pods will be terminated

## How to access the app after deployment

```
kubectl apply -f .infractructure/
```

## Test app with the port-forward

```
kubectl port-forward service/{service_name} 8081:80
```

## Test app with the port-forward

```
kubectl port-forward service/{service_name} 8081:80
```
```
curl http://{service_name}.{namespace}.svc.cluster.local
```