# Deployment Instructions
## Resource Requests & Limits
- **CPU**: request: 100m, limit: 500m
- **Memory**: request: 128Mi, limit: 256Mi


**Чому так:** Request — гарантує мінімальні ресурси для роботи. Limit — захищає від перевитрат. Idle стан потребує мало ресурсів, тому 100m CPU вистачає для 2 подів.
## HPA Configuration
- minReplicas: 2
- maxReplicas: 5
- CPU trigger: 70%
- Memory trigger: 70%


**Чому так:** 2 пода в idle — мінімум для availability. 5 — максимум щоб не витрачати занадто багато ресурсів. 70% — розумний поріг: не занадто чутливий, не занадто лінивий.
## Strategy Configuration
- Type: RollingUpdate
- maxSurge: 1
- maxUnavailable: 0


**Чому так:** RollingUpdate дозволяє оновлювати без даунтайму. maxSurge: 1 — додається 1 новий под перед видаленням старого. maxUnavailable: 0 — всі пода завжди доступні.
## Access the App
```bash
# Прокинути порт
kubectl port-forward -n mateapp svc/todoapp 8000:8000


## Deployment Commands
```bash
# Apply manifests
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
# Check status
kubectl get pods -n mateapp
kubectl get hpa -n mateapp
