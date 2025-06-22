# Інструкції до розгортання ToDo App у Kubernetes

## 1. Застосування маніфестів

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml

2. Стратегія розгортання
Ми використовуємо RollingUpdate, щоб оновлення відбувалося без простою:

maxSurge: 1 — допускаємо додатковий pod під час оновлення

maxUnavailable: 1 — дозволено, щоб лише один pod був недоступним

3. Запити та обмеження ресурсів
yaml
Копировать
Редактировать
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
Це забезпечує базовий рівень продуктивності з контрольованим споживанням ресурсів.

4. Конфігурація HPA
yaml
Копировать
Редактировать
minReplicas: 2
maxReplicas: 5
Автомасштабування працює на основі CPU та памʼяті:

Якщо використання перевищує 50%, створюється новий pod

Мінімум — 2 pod, максимум — 5

5. Перевірка доступності
bash
Копировать
Редактировать
kubectl get pods -n mateapp
kubectl get deployment -n mateapp
kubectl get hpa -n mateapp
6. Доступ до застосунку
Використайте port-forward:

bash
Копировать
Редактировать
kubectl port-forward deployment/todoapp-deployment 8000:8080 -n mateapp
Відкрийте у браузері: http://localhost:8000