# Work with Kubernetes

## Prerequisites

Before you start, create a required namespace and set it as a default
(for convenience):

```bash
kubectl apply -f .infrastructure/namespace.yml 
kubectl config set-context --current --namespace mateapp
```

## Deploy the app to k8s

To deploy the app, run the following command

```bash
kubectl apply -f .infrastructure/deployment.yml
```

## Resource requests and limits explained

Here is the default resources specified:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

The `requests` values were obtained from the output of the `docker stats`
command after the containerized app had been run.

The `requests.cpu` value was calculated by the following formula:

```
requests.cpu = (containerized_app_cpu_usage * maximum_available_cpu) / 100
```

## HPA configuration explained

This Kubernetes manifest defines a **HorizontalPodAutoscaler (HPA)** for the
`todoapp` Deployment in the `mateapp` namespace. The HPA dynamically adjusts
the number of pod replicas based on CPU and memory utilization.

### Key Sections:

- **`scaleTargetRef`**: Specifies the target resource for scaling.
In this case, the `todoapp` Deployment in the `apps/v1` API group.
- **`minReplicas`**: The minimum number of pod replicas the HPA will maintain
(2 replicas).
- **`maxReplicas`**: The maximum number of pod replicas the HPA can scale up to
(5 replicas).
- **`metrics`**: Defines resource utilization thresholds for scaling:
  - **cpu**: Pods scale if the average utilization exceeds 70% of the requested
    CPU.
  - **memory**: Pods scale if the average utilization exceeds 70% of the
    requested memory.

## Configuration strategy for Horizontal Pod Autoscaler explained

This HPA manifest was designed with a focus on ensuring the `todoapp`
Deployment can handle variable workloads effectively while maintaining resource
efficiency.

### Strategy Breakdown

1. **`minReplicas: 2`**:
   - Ensures that at least two replicas of the `todoapp` are always running,
   providing redundancy and improving fault tolerance.
   - Useful for maintaining availability during updates or in the event of a
   single pod failure.

2. **`maxReplicas: 5`**:
   - Limits the scaling to a maximum of five replicas to balance resource usage
   and cost considerations.

3. **`cpu` and `memory` utilization (`averageUtilization: 70`). Why 70%?**
   - This value strikes a balance between performance and efficiency:
      - If utilization exceeds 70%, scaling up ensures pods have sufficient
      resources to handle increased demand, avoiding performance degradation.
      - Keeps pods operating below their limits, reducing the risk of
      throttling or out-of-memory (OOM) errors.
      - Prevents underutilization of resources, optimizing for cost and
      performance.
   - Both CPU and memory metrics are included to ensure balanced scaling:
      - Applications may be constrained by either CPU or memory usage,
      so monitoring both prevents bottlenecks.

## Access the app after deployment

To access the app after deployment, apply the `nodeport.yml` manifest:

```bash
kubectl apply -f .infrastructure/nodeport.yml
```

Now, follow the URL to open the website: `http://localhost:30080/`
