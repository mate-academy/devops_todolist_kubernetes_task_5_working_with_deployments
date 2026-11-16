# Django ToDo — Kubernetes Deployment and Horizontal Pod Autoscaler

## 1. Prerequisites

- A running Kubernetes cluster (tested locally with Docker Desktop).
- `kubectl` configured to access the cluster.
- Metrics Server providing CPU and memory metrics through the Kubernetes Metrics API.
- Run the commands below from the repository root.

The application uses the `ikulyk404/todoapp:3.0.0` image and listens on port `8080` inside the container. Port `8000` in the README is the default Django port for local development, not the port used by this image.

## 2. Deploy the application

Create or configure the namespace and deploy the application:

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl rollout status deployment/todoapp -n mateapp
```

Deploy the two Services:

```bash
kubectl apply -f .infrastructure/clusterIP.yml
kubectl apply -f .infrastructure/nodeport.yml
```

Check the resources:

```bash
kubectl get deployment,pods,svc -n mateapp
kubectl get pods -n mateapp -l app=todoapp
```

The Deployment creates two application Pods through a ReplicaSet. Both should become `1/1 Running`. The Deployment and Services use the `app: todoapp` label to identify the application Pods. Do not apply the old standalone Pod manifest; the Deployment now manages the application replicas.

## 3. Metrics Server and HPA

Check whether the Metrics API is available:

```bash
kubectl top nodes
kubectl top pods -n mateapp -l app=todoapp
```

If Metrics Server is not installed, install a release compatible with the cluster. For the local cluster used in this task, the official installation command was:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Check its status and logs:

```bash
kubectl get pods -n kube-system -l k8s-app=metrics-server
kubectl logs -n kube-system deployment/metrics-server --tail=30
```

On the local Docker Desktop cluster, the kubelet certificate did not contain the required IP SAN. For this local testing environment only, the following patch was used to disable kubelet certificate verification:

```bash
kubectl patch deployment metrics-server -n kube-system --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

Apply this patch only if the certificate error occurs and the argument is not already present. In production, use properly trusted kubelet certificates instead of disabling verification. The patch affects the shared Metrics Server, not the application Deployment.

Wait for Metrics Server and verify metrics again:

```bash
kubectl rollout status deployment/metrics-server -n kube-system
kubectl top nodes
```

Create the Horizontal Pod Autoscaler:

```bash
kubectl apply -f .infrastructure/hpa.yml
kubectl get hpa todoapp -n mateapp -w
```

Allow time for the first metrics to appear. Press Ctrl+C to stop watching. If targets remain `<unknown>`, inspect the HPA and Metrics Server:

```bash
kubectl describe hpa todoapp -n mateapp
kubectl top pods -n mateapp -l app=todoapp
```

## 4. Resource requests and limits

The Deployment defines these resources per application container:

| Resource | Request | Limit |
| --- | --- | --- |
| CPU | 250m | 500m |
| Memory | 256Mi | 512Mi |

The CPU request of 250m (0.25 CPU cores) gives the scheduler a resource baseline for placement. The 500m limit (0.5 CPU cores) allows additional CPU usage when needed while bounding consumption. CPU use above the limit may be throttled.

The memory request of 256Mi provides scheduling headroom for Django, while the 512Mi limit bounds memory consumption. Exceeding the memory limit can cause an OOM kill. Requests also provide the baseline for HPA utilization calculations; limits do not define the HPA percentage targets.

During local idle testing, `kubectl top pods` reported:

| Application Pod | CPU | Memory |
| --- | --- | --- |
| todoapp-6d8c8c46-qfl2x | 16m | 62Mi |
| todoapp-6d8c8c46-qmdpk | 17m | 62Mi |

Average CPU usage was 16.5m (about 6.6% of the request) and average memory usage was 62Mi (about 24.2% of the request). These observations are below the HPA targets, so two replicas are expected at idle. The original 64Mi memory request would have produced about 97% utilization at the same idle usage, potentially triggering unnecessary scaling. The chosen values are conservative starting points for this learning application, not production capacity measurements. They should be reviewed under realistic load.

## 5. HPA configuration and rationale

The HPA uses `autoscaling/v2` and targets the `todoapp` Deployment in `mateapp`.

| Parameter | Value |
| --- | --- |
| Minimum replicas | 2 |
| Maximum replicas | 5 |
| CPU average utilization | 70% |
| Memory average utilization | 70% |

Two minimum replicas meet the idle-state requirement and provide more than one application instance. Five maximum replicas limit scaling in the local cluster. The 70% targets provide headroom before the resource limits are reached.

Utilization is calculated against resource requests. For the selected requests, 70% corresponds to approximately 175m CPU and 179.2Mi memory per Pod on average. The HPA calculates a desired replica count for each metric and chooses the larger recommendation, bounded by the minimum and maximum. Therefore, high memory utilization can trigger scaling even when CPU utilization is low.

The HPA initially reported unknown metrics and then reported CPU at 8% and memory at 24%. These values are consistent with the idle measurements. To confirm the current number of replicas, use:

```bash
kubectl get hpa todoapp -n mateapp
kubectl get deployment todoapp -n mateapp
```

The expected idle result is two replicas. Scaling decisions also depend on controller timing, metric availability and stabilization behavior. The default scale-down stabilization window is normally five minutes. No sustained load test is claimed in this document.

## 6. RollingUpdate strategy and rationale

The Deployment uses the following configuration:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

`maxSurge: 1` permits one additional non-terminating Pod above the desired replica count during an update. With two desired replicas, the rollout may use three non-terminating Pods temporarily. Terminating Pods may still consume resources until they exit.

`maxUnavailable: 1` allows one desired replica to be unavailable during an update. With two desired replicas, the rollout is designed to preserve at least one available replica under normal conditions. These values balance application availability and resource usage in a local cluster. A stricter availability requirement could use `maxUnavailable: 0`, provided enough capacity is available for the additional Pod.

The readiness probe helps prevent traffic from being sent to new Pods before they become ready. The liveness probe checks container health. The configured endpoints are `/api/ready` and `/api/health`, both on port 8080.

To observe a rollout:

```bash
kubectl rollout status deployment/todoapp -n mateapp
kubectl get replicasets -n mateapp -l app=todoapp
```

## 7. Access the application

### NodePort (Docker Desktop)

Open http://localhost:30080/ in a browser. The API is available at http://localhost:30080/api/.

The `todoapp-nodeport` Service exposes NodePort `30080`, Service port `80`, and forwards traffic to container port `8080`.

For a different Kubernetes environment, use a reachable node IP and port 30080. Check node addresses with `kubectl get nodes -o wide`. Network rules must allow the connection.

### ClusterIP with port-forward

The `todoapp` ClusterIP Service exposes port 80 and targets port 8080. Start a local port-forward:

```bash
kubectl port-forward service/todoapp 8086:80 -n mateapp
```

Open http://localhost:8086/ or run `curl -i http://localhost:8086/` in another terminal. Press Ctrl+C to stop forwarding. Port-forward selects one backing Pod for the session; it is not a load-balancing test.

### Internal Service DNS

From a Pod with an HTTP client inside the cluster, use:

```bash
curl -i http://todoapp.mateapp.svc.cluster.local/
```

For example, create a temporary curl-enabled test Pod (requires image pull access):

```bash
kubectl run todoapp-test --rm -it --restart=Never -n mateapp --image=curlimages/curl --command -- curl -i http://todoapp.mateapp.svc.cluster.local/
```

This is optional; the NodePort and port-forward access methods do not require a test Pod.

## 8. Troubleshooting and notes

```bash
kubectl get pods -n mateapp -l app=todoapp
kubectl describe deployment todoapp -n mateapp
kubectl describe hpa todoapp -n mateapp
kubectl logs -n mateapp -l app=todoapp --tail=30
kubectl get endpointslice -n mateapp -l kubernetes.io/service-name=todoapp -o wide
kubectl get svc -A
```

If NodePort 30080 is already allocated in another namespace, resolve the conflict before applying the Service. Do not delete unrelated Services or the shared `mateapp` namespace. If old standalone Pods exist in `mateapp` with the same label, check their ownership and remove only those obsolete resources if necessary.

The application uses SQLite in this learning project. Each replica has its own local database; the replicas do not share application data. A production deployment would need a shared database and appropriate persistence.
