# Інструкція: HPA за CPU та Memory

## 7. Додайте resource requests до Deployment

HPA з типом цілі `Utilization` обчислює відсоток використання відносно
`resources.requests`. У `deployment.yml` для контейнера мають бути задані
обидва ресурси:

```yaml
resources:
  requests:
    cpu: "150m"
    memory: "256Mi"
  limits:
    cpu: "200m"
    memory: "512Mi"
```

## 8. Налаштуйте HPA для CPU і пам’яті

У `hpa.yml` додайте дві resource metrics. Ціль `70` означає 70% від
відповідного request на pod.

```yaml
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

## 9. Застосуйте Deployment і HPA

```powershell
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

## 10. Перевірте стан автоскейлера

```powershell
kubectl get hpa -n my-namespace
kubectl describe hpa todoapp -n my-namespace
kubectl top pods -n my-namespace
```

`kubectl top` вимагає встановленого Metrics Server. У колонці `TARGETS`
будуть показані поточні значення CPU і пам’яті відносно заданих цілей.

## 11. Як працює масштабування

HPA розраховує потрібну кількість реплік для CPU та memory окремо. Для
масштабування вгору він застосовує більшу з двох рекомендацій: якщо CPU або
пам’ять перевищує 70%, кількість pod’ів зросте в межах `minReplicas: 2` і
`maxReplicas: 5`. Зменшення відбувається лише коли обидві метрики дозволяють
зменшити кількість реплік.
