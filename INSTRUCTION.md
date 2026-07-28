# ToDo Application Instructions

## Resource requests and limits

For each application pod, CPU request is set to 50m and memory
request is set to 128Mi. These values reserve enough resources for
the lightweight Django application during normal operation.

CPU limit is set to 200m and memory limit is set to 256Mi. The limits
are higher than the requests, allowing the application to temporarily
use additional resources during increased traffic.

The requests are also used by the Horizontal Pod Autoscaler to
calculate CPU and memory utilization. With a target utilization of 70%,
scaling may begin when average usage exceeds approximately 35m CPU or
90Mi of memory per pod.

## Deployment strategy

The Deployment uses the RollingUpdate strategy to update the application
gradually without stopping all pods at the same time. This allows the
application to remain available during deployment of a new version.

The `maxUnavailable` value is set to `1`, which means that no more than
one of the two application pods may be unavailable during the update.
Therefore, at least one pod should remain available to handle requests.

The `maxSurge` value is set to `1`, which allows Kubernetes to create one
additional pod above the desired number of replicas during the update.
With two configured replicas, a maximum of three pods may temporarily
exist.

These values were selected to provide a balance between application
availability and resource usage. The application remains accessible
during the update, while the cluster only needs resources for one
temporary additional pod.

## Accessing the application

After deploying the application, first check that the pods and services are running:

```bash
kubectl get pods -n mateapp
kubectl get services -n mateapp
```

To access the application from the local machine, forward the service port:

```bash
kubectl port-forward service/todoapp-nodeport 8080:80 -n mateapp
```

After that, open the following address in a browser:

```text
http://localhost:8080
```

The `kubectl port-forward` command forwards local port `8080` to port `80` of the Kubernetes service. 
The service then sends the request to port `8080` of one of the application pods.