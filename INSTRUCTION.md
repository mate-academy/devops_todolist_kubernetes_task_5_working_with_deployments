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
```
requests:
  memory: "64Mi"
  cpu: "250m"
limits:
  memory: "128Mi"
  cpu: "500m"
```
The app is small and stateless, so these values are enough for basic functionality. Requests reserve what a pod normally needs, and limits are about 2x that to absorb spikes without letting one pod hog the node.

#### HPA configuration
Min replicas is `2`, max is `5`, with CPU and memory average utilisation targets of `70%`.
- **2 min:** the app stays available if one pod fails or is being updated.
- **5 max:** allows scaling under load while capping resource use on a small cluster.
- **70%:** leaves headroom for traffic while new pods start, without scaling too eagerly.

#### Strategy configuration
`type: RollingUpdate` with `maxUnavailable: 1` and `maxSurge: 1`.
- **`maxUnavailable: 1`:** at least one pod keeps serving during an update, so there is no downtime.
- **`maxSurge: 1`:** a new pod starts before an old one is removed, so capacity doesn't drop, and extra resource use is limited to one pod.

#### To access the app after deployment
To access the app, visit [localhost:30007](http://localhost:30007)
> If it's not available, try port forwarding.
> ```bash
> kubectl port-forward service/todoapp 30007:80 -n mateapp
> ```