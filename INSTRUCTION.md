# Інструкція: HPA за CPU та Memory

## RollingUpdate strategy explanation

The Deployment uses `maxUnavailable: 0`, so Kubernetes keeps every existing Pod
available while rolling out a new version. This avoids a reduction in the number
of ready application replicas during the update. `maxSurge` is intentionally not
set, which uses Kubernetes' default value of `25%`; Kubernetes may temporarily
create up to 25% more Pods than the desired replica count. Together, these values
prioritize uninterrupted availability while allowing a controlled rollout.

## Access instructions

After applying the Deployment, access the ToDo app locally with port forwarding:

```powershell
kubectl port-forward -n mateapp deployment/todoapp 8080:8080
```

Then open `http://localhost:8080` in a browser. Port forwarding targets the
Deployment directly, so it works independently of any Service configuration.

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
kubectl get hpa -n mateapp
kubectl describe hpa todoapp -n mateapp
kubectl top pods -n mateapp
```

`kubectl top` вимагає встановленого Metrics Server. У колонці `TARGETS`
будуть показані поточні значення CPU і пам’яті відносно заданих цілей.

## 11. Як працює масштабування

HPA розраховує потрібну кількість реплік для CPU та memory окремо. Для
масштабування вгору він застосовує більшу з двох рекомендацій: якщо CPU або
пам’ять перевищує 70%, кількість pod’ів зросте в межах `minReplicas: 2` і
`maxReplicas: 5`. Зменшення відбувається лише коли обидві метрики дозволяють
зменшити кількість реплік.
