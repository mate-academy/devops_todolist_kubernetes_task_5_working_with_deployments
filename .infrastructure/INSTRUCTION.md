# INSTRUCTION.md

## 🚀 Deploying the TodoApp to Kubernetes

### 1. Deployment Steps
* Create namespace:
   ```bash
   kubectl create namespace mateapp

* Apply manifests
    ```bash
    kubectl apply -f deployment.yml
    kubectl apply -f nodeport.yml
    kubectl apply -f hpa.yml


Memory: 64Mi — the minimum guaranteed memory for running the application.

CPU: 250M — a quarter of the CPU, sufficient for processing basic requests.

---

minReplicas: 2 — ensures high availability (one pod will always be running even if another fails).

maxReplicas: 5 — limits scaling to avoid overloading the cluster.

Metrics:

CPU Utilization: 80%

Memory Utilization: 80%

Rationale: When the load exceeds 80%, new replicas are created, balancing performance and resource savings.

---
RollingUpdate:

maxUnavailable: 1 — при обновлении максимум один под может быть недоступен.

maxSurge: 1 — создаётся максимум один дополнительный под сверх реплик.

Обоснование: стратегия обеспечивает плавное обновление без простоя, сохраняя доступность приложения.
---
Go to http://127.0.0.1:30080/ in your browser to check your app.