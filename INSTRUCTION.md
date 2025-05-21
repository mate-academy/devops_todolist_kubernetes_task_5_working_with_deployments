# 📝 INSTRUCTION.md — Deploy Django ToDo App to Kubernetes

## 📁 Files
All Kubernetes manifests are located in the `.infrastructure` directory:
- `namespace-mateapp.yml` — Namespace `mateapp`
- `deployment.yml` — Deployment for the app using image `ikulyk404/kube2py:1.2.0`
- `hpa.yml` — Horizontal Pod Autoscaler (HPA)

---

## 🚀 Deployment Steps

1. **Create the namespace:**
```bash
kubectl apply -f .infrastructure/namespace-mateapp.yml
```

2. **Set context to use the `mateapp` namespace by default (optional but recommended):**
```bash
kubectl config set-context --current --namespace=mateapp
```

3. **Deploy the application:**
```bash
kubectl apply -f .infrastructure/deployment.yml
```

4. **Apply the HorizontalPodAutoscaler:**
```bash
kubectl apply -f .infrastructure/hpa.yml
```

5. **Expose the application via NodePort (make sure the containerPort is 8000):**
```bash
kubectl expose deployment todo-app --type=NodePort --port=8000
```

6. **Check the NodePort:**
```bash
kubectl get svc
```
```
NAME       TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
todo-app   NodePort   10.106.202.103   <none>        8000:31353/TCP   5s
```

7. **Access the application in your browser:**
   Use the NodePort shown in the output.  
   For example, if the output shows `8000:31353/TCP`, then open:
```
http://localhost:31353
```

---

## 🔍 Configuration Explanations

### 🎯 Deployment Strategy: RollingUpdate
- Ensures zero downtime during updates
- `maxSurge: 1`, `maxUnavailable: 1` allow safe gradual rollouts

### 💾 Resource Requests and Limits
- **Requests**: 250m CPU / 128Mi memory — enough for idle Django usage
- **Limits**: 500m CPU / 256Mi memory — to prevent resource overuse

### 📈 Horizontal Pod Autoscaler (HPA)
- Based on both CPU and Memory utilization
- CPU target: 50%
- Memory target: 60%
- Scales between 2 and 5 pods based on load

---

## 📉 HPA Metrics Note

If you see `<unknown>` under CPU or memory when checking HPA status:

```bash
kubectl get hpa
```
```
NAME           REFERENCE             TARGETS                                     MINPODS   MAXPODS   REPLICAS   AGE
todo-app-hpa   Deployment/todo-app   cpu: <unknown>/50%, memory: <unknown>/60%   2         5         0          54s
```
---

## ✅ Useful Commands for Validation

```bash
kubectl get pods
kubectl get hpa
kubectl get svc
kubectl describe hpa todo-app-hpa
kubectl logs <pod-name>
```

---

## 🧹 Cleanup Instructions

### ❌ Delete all resources in `mateapp` namespace:
```bash
kubectl delete all --all -n mateapp
```

This will remove all: Pods, Deployments, Services, HPAs, etc.

### ❌ Delete the namespace itself:
```bash
kubectl delete namespace mateapp
```

This will also remove all remaining resources in that namespace.

### ✅ Confirm everything is gone:
```bash
kubectl get all -A | grep mateapp
```
