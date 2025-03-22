# Інструкції з розгортання ToDo застосунку в Kubernetes

## Передумови

*   `kubectl` налаштовано.
*   Кластер Kubernetes доступний.
*   Docker образ застосунку завантажено.

## Кроки розгортання

1.  Форкніть репозиторій.
2.  Створіть `deployment.yml`:

    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: todoapp-deployment
      namespace: mateapp
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: todoapp
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 0
      template:
        metadata:
          labels:
            app: todoapp
        spec:
          containers:
            - name: todoapp-container
              image: vhurana/todoapp:3.0.0 
              ports:
                - containerPort: 8000
              resources:
                requests:
                  cpu: "100m"
                  memory: "256Mi"
                limits:
                  cpu: "200m"
                  memory: "512Mi"
    ```

3.  Створіть `hpa.yml`:

    ```
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: todoapp-hpa
      namespace: mateapp
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: todoapp-deployment
      minReplicas: 2
      maxReplicas: 5
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 50
        - type: Resource
          resource:
            name: memory
            target:
              type: Utilization
              averageUtilization: 70
    ```

4.  Застосуйте маніфести:

    ```
    kubectl apply -f .infrastructure/deployment.yml
    kubectl apply -f .infrastructure/hpa.yml
    ```

## Пояснення конфігурації

### Resource Requests/Limits

*   `requests`: Гарантує мінімальні ресурси (CPU: 100m, Memory: 256Mi).
*   `limits`: Обмежує максимальне споживання ресурсів (CPU: 200m, Memory: 512Mi).
*   **Чому:** Початкові значення, налаштуйте за результатами моніторингу.

### HPA

*   `minReplicas: 2`: Забезпечує мінімальну доступність.
*   `maxReplicas: 5`: Обмежує споживання ресурсів.
*   CPU 50%, Memory 70%: Почніть масштабування при досягненні цих значень.
*   **Чому:** Баланс між продуктивністю та споживанням ресурсів.

### Strategy

*   `RollingUpdate`: Плавне оновлення без простою.
*   `maxSurge: 1`: Максимум 1 додатковий под під час оновлення.
*   `maxUnavailable: 0`: Забезпечує доступність під час оновлення.

## Доступ до застосунку

Використовуйте:

1.  **NodePort:** (Розробка/тестування). `kubectl apply -f .infrastructure/nodeport.yml`
2.  **LoadBalancer:** (Хмара). `kubectl apply -f .infrastructure/clusterlp.yml`
3.  **Ingress:** (Production, рекомендовано). Потребує Ingress Controller.
    *   Створіть Ingress Resource.

## Моніторинг

Моніторте застосунок за допомогою Kubernetes Dashboard, Prometheus, Grafana.
