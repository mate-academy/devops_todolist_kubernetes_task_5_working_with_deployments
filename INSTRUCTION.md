# INSTRUCTION.md

## Деплой додатку TodoApp на Kubernetes

Цей документ описує, як розгорнути додаток `todoapp` у кластері Kubernetes з використанням Deployment та Horizontal Pod Autoscaler (HPA),
а також пояснює вибір ресурсів, стратегій та конфігурацій.

---

## 1. Створення Namespace

Створіть namespace для додатку:
```
kubectl create namespace mateapp
```

## 2. Застосування Deployment

```
kubectl apply -f deployment.yml -n mateapp
```

Пояснення ключових налаштувань:

* replicas: 2 — стартова кількість реплік для забезпечення мінімальної доступності додатку.
* strategy: RollingUpdate
* maxUnavailable: 0 — жодна поточна репліка не буде недоступною під час оновлення.
* maxSurge: 1 — створюється максимум один додатковий под для безперервної роботи.
* livenessProbe перевіряє, чи под "живий" і перезапускає його при зависанні.
* readinessProbe перевіряє, чи под готовий обслуговувати запити.

* resources: requests (cpu: 250m, memory: 256Mi) — мінімальні ресурси, які гарантуються поду.
limits (cpu: 500m, memory: 512Mi) — максимальні ресурси, які под може використати. Це допомагає уникнути перевантаження вузлів.

## 3. Деплой Horizontal Pod Autoscaler (HPA)

Застосуйте HPA:
```
kubectl apply -f hpa.yml -n mateapp
```
Пояснення ключових налаштувань:
* minReplicas: 2 — мінімальна кількість подів для забезпечення доступності.
* maxReplicas: 5 — максимальна кількість подів для уникнення надмірного навантаження на кластер.
* metrics:
  * CPU utilization target: 80% — дозволяє автоматично масштабувати поди при збільшенні навантаження.
  * Memory utilization target: 80% — допомагає контролювати використання пам'яті та уникати проблем з продуктивністю.

## 4. Перевірка статусу

Перевірте, що Deployment та HPA працюють:

```
kubectl get pods -n mateapp
kubectl get deployment todoapp -n mateapp
kubectl get hpa -n mateapp
```

## 5. Доступ до додатку

Щоб отримати доступ до додатку локально:
```
kubectl port-forward svc/todoapp-service 8000:80 -n mateapp
```
Потім відкрийте браузер і перейдіть за адресою: `http://localhost:8000`