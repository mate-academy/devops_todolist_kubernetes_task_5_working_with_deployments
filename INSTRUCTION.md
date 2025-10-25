All resources are deployed into the mateapp namespace

Apply the Deployment:
kubectl apply -f .infrastructure/deployment.yml

This manifest deploys the application todoapp with 2 replicas:

replicas: 2


The container uses the image:

image: ikulyk404/todoapp:3.0.0


and exposes port 8080.

Health checks are defined:

livenessProbe: /api/health

readinessProbe: /api/ready

Deployment Strategy:
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1


RollingUpdate ensures zero-downtime updates.

maxUnavailable: 0 — because 1 should be alive

maxSurge: 1 — allows creating one extra pod during deployment to speed up rollout.
This setup balances reliability and rollout speed.


Resource Requests and Limits:
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"

Requests — minimum guaranteed resources.

Limits — maximum allowed resources.

These values are optimal for lightweight Python/Django web apps.

Horizontal Pod Autoscaler:
kubectl apply -f .infrastructure/hpa.yaml

configuration:

minReplicas: 2
maxReplicas: 5
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        averageUtilization: 70

minReplicas: 2 — keeps at least 2 pods for redundancy.

maxReplicas: 5 — prevents over-scaling beyond reasonable limits.

averageUtilization: 70 — new pods are created when CPU or memory usage exceeds 70%.


Verification and Access:

kubectl get pods -n mateapp
kubectl get hpa -n mateapp
kubectl get deployments -n mateapp


This is the simplest and works in all environments:

kubectl port-forward deployment/todoapp-deployment 8080:8080 -n mateapp


Now open your browser and visit:

http://localhost:8080