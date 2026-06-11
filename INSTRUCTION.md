# Instructions

## How to deploy

kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml

## Check deployment

kubectl get pods -n mateapp
kubectl get hpa -n mateapp

## Resource requests and limits

- requests.cpu: 100m — минимум CPU для старта пода
- requests.memory: 128Mi — минимум памяти для Django приложения
- limits.cpu: 200m — максимум CPU, не даёт поду монополизировать ноду
- limits.memory: 256Mi — максимум памяти, защита от утечек

## HPA configuration

- minReplicas: 2 — минимум 2 пода для отказоустойчивости
- maxReplicas: 5 — максимум 5 подов, достаточно для нагрузки
- cpu: 70% — триггер масштабирования по CPU
- memory: 70% — триггер масштабирования по памяти
- Порог 70% выбран чтобы новые поды успевали стартовать до достижения 100%

## RollingUpdate strategy

- maxSurge: 1 — поднимаем 1 новый под перед удалением старого
- maxUnavailable: 1 — допускаем 1 недоступный под во время обновления
- Это обеспечивает zero-downtime деплой — приложение всегда доступно

## How to access the app

kubectl port-forward service/todoapp-clusterip 8080:80 -n mateapp
# Открой: http://localhost:8080