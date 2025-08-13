# INSTRUCTIONS

## 1. Create namespace

```bash
kubectl create namespace mateapp
```

## 2. Deploy the app

```bash
kubectl apply -f .infrastructure/deployment.yml
```

## 3. Deploy Autoscaler

```bash
kubectl apply -f .infrastructure/hpa.yml
```
## 4. Access the app

```bash
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp
```

In this deployment, the resource requests and limits were chosen to balance stability and efficiency. The application requests 64Mi of memory and 250m of CPU, which is enough for a small Python web service to run reliably without wasting cluster resources. The memory limit is set to 128Mi to prevent excessive consumption in case of memory leaks, and the CPU limit is set to 500m, allowing the application to handle short bursts of load while still protecting other workloads in the cluster.

The Horizontal Pod Autoscaler (HPA) is configured with a minimum of two replicas to ensure high availability, so that if one pod fails, another can continue serving traffic. The maximum number of replicas is set to five to handle temporary traffic spikes without overloading the cluster. Scaling is triggered when CPU or memory utilization exceeds 70% of the requested resources, which strikes a balance between performance and resource efficiency.

The deployment strategy uses a rolling update with maxSurge set to 1 and maxUnavailable set to 1. This ensures that during updates, only one extra pod is temporarily created, and no more than one pod becomes unavailable at a time. This approach minimizes downtime and maintains service availability throughout the deployment process.
