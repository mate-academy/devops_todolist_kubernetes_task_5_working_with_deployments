# ToDo App Kubernetes Deployment

## 1. Deployment
Deployment використовує стратегію **RollingUpdate**, щоб забезпечити безперервну доступність.  
`maxUnavailable: 1` — не більше одного поду може бути недоступним під час оновлення.  
`maxSurge: 1` — допускається створення одного додаткового поду для плавного оновлення.

## 2. Ресурси
Контейнер має:
- `requests`: 64Mi, 250m — мінімальні ресурси для стабільного запуску Django у стані очікування.
- `limits`: 128Mi, 500m — обмеження при навантаженні, щоб уникнути OOM Killer та перевантаження CPU.

## 3. Автоматичне масштабування (HPA)
HPA реагує на 70% CPU і 75% пам’яті:
- `minReplicas: 2` — забезпечує базову доступність навіть при мінімальному навантаженні.
- `maxReplicas: 5` — запобігає перевитраті ресурсів при пікових навантаженнях.

## 4. Розгортання у кластері
```bash
kubectl create ns mateapp
kubectl apply -f deployment.yml
kubectl apply -f nodeport.yml
kubectl apply -f hpa.yml
