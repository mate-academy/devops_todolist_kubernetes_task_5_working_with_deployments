
## 1. Create Namespace

Create the required namespace for your app:

```sh
kubectl create namespace mateapp
```

---

## 2. Deploy the Application

Apply the deployment manifest:

```sh
kubectl apply -f .infrastructure/deployment.yml -n mateapp
```

**Why RollingUpdate strategy?**  
The deployment uses `RollingUpdate` to ensure zero downtime during updates.  
- `maxUnavailable: 1` means at most one pod can be unavailable during the update.
- `maxSurge: 1` means at most one extra pod can be created above the desired number during the update.
This balances availability and resource usage.

---

## 3. Apply Horizontal Pod Autoscaler (HPA)

Apply the HPA manifest:

```sh
kubectl apply -f .infrastructure/hpa.yml -n mateapp
```

**Why set both CPU and memory requests/limits?**  
- Resource requests guarantee minimum resources for each pod, preventing resource starvation.
- Limits prevent a pod from consuming excessive resources ("noisy neighbor" problem).
- HPA v2 can scale based on both CPU and memory utilization, providing more accurate and flexible scaling.

---

## 4. Verify Deployment and Scaling

Check deployment, HPA, and pods:

```sh
kubectl get deploy -n mateapp
kubectl get hpa -n mateapp
kubectl get pods -n mateapp
```

Describe HPA for details:

```sh
kubectl describe hpa todoapp-hpa -n mateapp
```

---

## 5. Access the Application

If you have a Service manifest (e.g., ClusterIP, NodePort, or Ingress), use it to access the app.  
If not, use port-forward:

```sh
kubectl port-forward deploy/todoapp 8080:8080 -n mateapp
```

Then open in your browser or use curl:

```sh
curl http://localhost:8080/
```

---

## 6. Health Checks

Verify health endpoints:

```sh
curl http://localhost:8080/api/health
curl http://localhost:8080/api/ready
```

---