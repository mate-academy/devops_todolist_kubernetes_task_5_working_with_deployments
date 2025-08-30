# Инструкция по развертыванию ToDo приложения в Kubernetes

## Обзор

Данная инструкция описывает развертывание Django ToDo приложения в Kubernetes кластере с использованием Deployment и Horizontal Pod Autoscaler (HPA).

## Предварительные требования

- Kubernetes кластер (версия 1.19+)
- kubectl настроен для работы с кластером
- Metrics Server установлен в кластере для работы HPA
- Docker образ приложения `todoapp:latest` доступен в реестре

## Структура манифестов

### 1. deployment.yml - Развертывание приложения

#### Выбор ресурсных лимитов и запросов

**Resource Requests:**

- **CPU: 100m** - минимальные требования для Django приложения в idle состоянии
- **Memory: 128Mi** - достаточно для загрузки Django и базовых операций

**Resource Limits:**

- **CPU: 500m** - ограничение для предотвращения монополизации CPU одним подом
- **Memory: 512Mi** - достаточно для обработки множественных запросов без риска OOM

**Обоснование выбора:**

- Django приложения обычно потребляют мало ресурсов в idle состоянии
- Лимиты установлены с запасом для обработки пиковых нагрузок
- Соотношение requests/limits позволяет эффективно использовать ресурсы кластера

#### Стратегия развертывания RollingUpdate

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

**Обоснование параметров:**

- **maxUnavailable: 1** - гарантирует, что минимум 1 под всегда доступен во время обновления
- **maxSurge: 1** - позволяет создать 1 дополнительный под для плавного перехода
- **Replicas: 2** - обеспечивает высокую доступность в idle состоянии

**Преимущества данной конфигурации:**

- Нулевое время простоя при обновлениях
- Контролируемое использование ресурсов
- Быстрые откаты в случае проблем

### 2. hpa.yml - Автоматическое масштабирование

#### Конфигурация HPA

```yaml
minReplicas: 2
maxReplicas: 5
metrics:
  - CPU: 70%
  - Memory: 80%
```

**Обоснование выбора:**

**Минимальное количество подов (2):**

- Обеспечивает высокую доступность
- Позволяет распределить нагрузку
- Защищает от сбоев одного пода

**Максимальное количество подов (5):**

- Ограничивает потребление ресурсов кластера
- Достаточно для обработки значительной нагрузки
- Предотвращает неконтролируемое масштабирование

**Метрики масштабирования:**

- **CPU 70%** - оптимальный уровень для веб-приложений, оставляет резерв для пиков
- **Memory 80%** - предотвращает OOM ошибки, учитывает специфику Python приложений

**Политики масштабирования:**

- **Scale Up**: быстрое увеличение при росте нагрузки (до 2 подов за 60 сек)
- **Scale Down**: осторожное уменьшение для предотвращения флаппинга (стабилизация 5 мин)

## Развертывание приложения

### Шаг 1: Создание namespace

```bash
kubectl create namespace mateapp
```

### Шаг 2: Применение манифестов

```bash
# Развертывание приложения
kubectl apply -f deployment.yml

# Настройка автомасштабирования
kubectl apply -f hpa.yml
```

### Шаг 3: Проверка развертывания

```bash
# Проверка статуса подов
kubectl get pods -n mateapp

# Проверка статуса deployment
kubectl get deployment -n mateapp

# Проверка статуса HPA
kubectl get hpa -n mateapp
```

## Доступ к приложению

### Создание Service

Для доступа к приложению необходимо создать Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: todoapp-service
  namespace: mateapp
spec:
  selector:
    app: todoapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### Варианты доступа

#### 1. Port Forward (для тестирования)

```bash
kubectl port-forward -n mateapp service/todoapp-service 8080:80
```

Приложение будет доступно по адресу: http://localhost:8080

#### 2. Ingress (для production)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todoapp-ingress
  namespace: mateapp
spec:
  rules:
    - host: todoapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: todoapp-service
                port:
                  number: 80
```

#### 3. NodePort (для внешнего доступа без Ingress)

Измените тип Service на NodePort:

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

## Мониторинг и обслуживание

### Проверка логов

```bash
kubectl logs -n mateapp -l app=todoapp -f
```

### Проверка метрик HPA

```bash
kubectl describe hpa todoapp-hpa -n mateapp
```

### Масштабирование вручную (при необходимости)

```bash
kubectl scale deployment todoapp-deployment --replicas=3 -n mateapp
```

## Особенности приложения

- **База данных**: SQLite (встроенная, не требует внешних зависимостей)
- **Порт**: 8080 (настроен в Dockerfile)
- **Health checks**: Настроены liveness и readiness пробы на корневой путь `/`
- **Переменные окружения**: Настроены для корректной работы Django

## Рекомендации по production

1. **Использовать внешнюю БД** (PostgreSQL, MySQL) вместо SQLite
2. **Настроить Persistent Volumes** для хранения данных
3. **Добавить мониторинг** (Prometheus, Grafana)
4. **Настроить логирование** (ELK stack, Fluentd)
5. **Использовать Secrets** для конфиденциальных данных
6. **Настроить Network Policies** для безопасности

## Устранение неисправностей

### Проблемы с запуском подов

```bash
kubectl describe pod -n mateapp -l app=todoapp
```

### Проблемы с HPA

```bash
kubectl describe hpa todoapp-hpa -n mateapp
kubectl top pods -n mateapp
```

### Проблемы с доступом

```bash
kubectl get svc -n mateapp
kubectl get endpoints -n mateapp
```
