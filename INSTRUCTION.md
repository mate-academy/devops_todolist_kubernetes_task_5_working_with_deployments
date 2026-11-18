### To deploy the app to K8s:
- Prepare a new cluster.
- Inside the project's folder, run:
```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/nodeport.yml
kubectl apply -f .infrastructure/clusterIp.yml
```

#### Resource requests and limits
Resource requests in the deployment are set to:
```
requests:
  memory: "64Mi"
  cpu: "250m"
limits:
  memory: "128Mi"
  cpu: "500m"
```
These values are sufficient to cover basic functionality.

#### HPA configuration
For app scalability, HPA min replicas is `2`, with up to `5` max, and CPU and memory average utilisation targets of `70%`.

#### Strategy configuration
To provide high availability, the strategy is `type: RollingUpdate` with `maxUnavailable: 1` and `maxSurge: 1`.

#### To access the app after deployment
To access the app, visit [localhost:30007](http://localhost:30007)
> If it's not available, try port forwarding.
> ```bash
> kubectl port-forward service/todoapp 30007:80 -n mateapp
> ```