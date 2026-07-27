## 1. How to deploy the app to Kubernetes
To deploy the application, apply the manifests in the following order. First, create the namespace, then deploy the application and its autoscaler:

```bash
kubectl apply -f namespace.yml

kubectl apply -f deployment.yml

kubectl apply -f hpa.yml
```

**To verify the deployment status:**

```bash
kubectl get hpa -n mateapp
```

```bash
kubectl get pods -n mateapp
```

## 2. Choice of resource requests and limits

- Requests (Memory: 64Mi, CPU: 250m): This is the guaranteed minimum resource allocation required for the scheduler to place the Pod on a Node. It acts like reserving a minimum chunk of RAM/CPU for a virtual machine on a hypervisor.

- Limits (Memory: 128Mi, CPU: 500m): This is the hard cap (similar to Linux cgroups). It prevents a single container with a memory leak or high load from consuming all physical node resources and affecting neighboring services.

## 3. Choice of HPA configuration
The Horizontal Pod Autoscaler is configured to scale between 2 and 5 replicas based on a 70% average utilization of CPU or Memory.

- Why 70%? Waiting for 90-95% utilization is risky because initializing a new container and passing readiness probes takes time. A 70% threshold provides a "buffer," allowing new Pods to spin up and start receiving traffic before the existing instances become overloaded and unresponsive.

## 4. Strategy configuration
We use a RollingUpdate strategy with maxSurge: 1 and maxUnavailable: 1.

- Why these numbers? This ensures zero downtime during updates. It works like taking one backend server out of a load balancer's rotation, updating it, and putting it back before touching the next one. The system will never drop below 1 working replica (2 desired - 1 unavailable) and will not overload the cluster with too many temporary pods (2 desired + 1 surge).

## 5. How to access the app after deployment.

To access the application from a web browser, we use a NodePort service. While a `ClusterIP` provides only internal access within the cluster, `NodePort` opens a specific port on the cluster nodes to accept external traffic.
Apply the service manifest:
```bash
kubectl apply -f nodeport.yml
```

Find the allocated external port:

```Bash
kubectl get service -n mateapp
```

You can now access the application at http://Node-IP:Node-Port, where Node-Port is the external port (typically in the 30000-32767 range) shown in the command output.