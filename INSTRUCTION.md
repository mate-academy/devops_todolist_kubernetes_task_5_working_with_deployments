First of all create a namespace if not yet created:
```
kubectl apply -f .infrastructure/namespace.yml
```

Then start deployment by:
```
kubectl apply -f .infrastructure/deployment.yml
```

The next step is to create an ip service to balance working load:
```
kubectl apply -f .infrastructure/clusterIp.yml
```

Then start the port forwarding from our cluster using:
```
kubectl apply -f .infrastructure/nodeport.yml
```

Now you can connect to our app via http://localhost:30080/