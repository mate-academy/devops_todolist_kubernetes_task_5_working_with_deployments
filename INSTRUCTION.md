# Deployment Instructions

## 1. Deploy the application

Create the namespace:

    kubectl apply -f .infrastructure/namespace.yml

Create the Deployment:

    kubectl apply -f .infrastructure/deployment.yml

Create the Horizontal Pod Autoscaler:

    kubectl apply -f .infrastructure/hpa.yml

Check that the resources are running:

    kubectl get deployments -n mateapp
    kubectl get pods -n mateapp
    kubectl get hpa -n mateapp

## 2. Resource requests and limits

The following resources are configured for each application container:

- CPU request: `100m`
- CPU limit: `500m`
- Memory request: `128Mi`
- Memory limit: `256Mi`

The values were chosen based on the application's resource usage in the idle state. Resource usage was checked using `kubectl top pods`.

The CPU usage was approximately `10-13m`, while memory usage was approximately `62Mi`.

Requests define the amount of resources Kubernetes takes into account when scheduling the Pod on a Node. Limits define the maximum amount of resources the container is allowed to use.

The selected values provide enough resources for normal application operation while leaving additional capacity for increased load.

## 3. Horizontal Pod Autoscaler

The HPA is configured with:

- Minimum replicas: `2`
- Maximum replicas: `5`
- CPU target utilization: `70%`
- Memory target utilization: `70%`

The HPA keeps at least two replicas running and can scale the Deployment up to five replicas depending on resource utilization.

CPU and memory utilization are calculated relative to the resource requests of the containers.

The HPA uses metrics provided by Metrics Server and changes the desired number of replicas of the Deployment when necessary.

## 4. Deployment strategy

The Deployment uses the `RollingUpdate` strategy with:

- `maxSurge: 1`
- `maxUnavailable: 1`

`maxSurge: 1` allows Kubernetes to create at most one additional Pod above the desired number of replicas during an update.

`maxUnavailable: 1` allows at most one of the required Pods to be unavailable during an update.

This allows the application to be updated gradually while keeping it available during the deployment process.

## 5. Accessing the application

For local access to the application, use port forwarding:

    kubectl port-forward deployment/todoapp 8080:8080 -n mateapp

This forwards local port `8080` to port `8080` of one of the Pods managed by the Deployment.

The application can then be accessed at:

    http://localhost:8080
