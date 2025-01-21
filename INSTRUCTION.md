## default parameters in `.infrastructure\deployment.yml`:
1. resources: 
You should chose resources:
    resources:
        requests:
        memory: "64Mi"
        cpu: "250m"
        limits:
        memory: "128Mi"
        cpu: "500m"
2. strategy:
    strategy:
    type: RollingUpdate
    rollingUpdate:
        maxUnavailable: 1
        maxSurge: 1
        
But you can change them if you need

## to apply manifests execute commands:
1. namespace: `kubectl apply -f .\.infrastructure\namespace.yml`
2. busybox: `kubectl apply -f .\.infrastructure\busybox.yml`
3. deployment: `kubectl apply -f .\.infrastructure\deployment.yml`
4. clusterip: `kubectl apply -f .\.infrastructure\clusterip.yml`
5. nodeport: `kubectl apply -f .\.infrastructure\nodeport.yml`
6. hpa: `kubectl apply -f .\.infrastructure\hpa.yml`



## to configure port forwarding execute `kubectl port-forward service/todoapp-service -n todoapp 8081:80`
To get to API open `http://127.0.0.1:8081` in browser. 

## to curl api from CLI:
1. execute command `kubectl exec -n todoapp -it busyboxplus -- sh`
2. execute command `curl http://todoapp-service.todoapp.svc.cluster.local`