Change working dir
```bash
cd .infrastructure
```
Start multi-node cluster using kind and config file `cluster.yml`
Create namespace `mateapp`
Start deployment
```bash
kubectl apply -f deployment.yml
```
Start horizontal planning autoscale
```bash
kubectl apply -f hpa.yml
```

1. I've chosen such config for requests and limits because it's suitable for testing of small web-app
2. HPA config limits are 80% and seems logical
3. I'm using RollingUpdate deployment strategy with only 1 max unavailable nodes
