# Deployment Instructions for MateApp

## 1. Prerequisites
- Kubernetes cluster with `metrics-server` installed.
- Access via `kubectl`.
- Namespace `mateapp` must exist (create it with `kubectl create ns mateapp`).

## 2. Deployment

### Step 1: Apply the deployment
```bash
kubectl apply -f deployment.yml
```

Step 2: Apply the autoscaler

kubectl apply -f hpa.yml

3. Resource Configuration Explained

Requests:
	•	CPU: 100m — достатньо для базового обслуговування в простої.
	•	Memory: 128Mi — для легкого сервісу без інтенсивних обчислень.

Limits:
	•	CPU: 500m — обмеження на півядра для контрольованого масштабування.
	•	Memory: 512Mi — щоб уникнути OOM при піках навантаження.

4. HPA Configuration Explained
	•	minReplicas = 2: мінімальний рівень доступності (включаючи високодоступність).
	•	maxReplicas = 5: достатній верхній поріг для стійкого масштабування без перевитрат.
	•	CPU 60% & Memory 70%: збалансовані метрики, які враховують як обчислювальне, так і RAM навантаження.

5. Deployment Strategy Explained

RollingUpdate:
	•	maxUnavailable: 1: забезпечує, що при оновленні завжди залишається принаймні один под активним.
	•	maxSurge: 1: дозволяє створити 1 додатковий под під час оновлення для безперервності сервісу.
