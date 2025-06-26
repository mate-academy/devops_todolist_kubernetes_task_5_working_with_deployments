1) Create a namespace `mateapp` in your Kubernetes cluster.
```bash
kubectl apply -f ./.infrastructure/namespace.yml
```
2) Run all the manifests in the `./.infrastructure` directory.
```bash
kubectl apply -f ./.infrastructure/
```
3) Access the app via node port forwarding.
```bash
http://localhost:30080/
```