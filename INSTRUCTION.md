How to deploy the app to k8s:
  kubectl apply -f namespace.yml
  kubectl apply -f clusterIp.yml
  kubectl apply -f nodeport.yml
  kubectl apply -f deployment.yml
  kubectl apply -f hpa.yml
  
Choice of resource requests and limits:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
  Set the both requests and limits parameters for CPU and memory to avoid overcommitment or underutilization.
  Requests reflected the minimum resources needed for stable operation.
  Limits capped usage to prevent noisy neighbor issues or runaway processes.

Choice of HPA configuration:
  minReplicas: 2
  maxReplicas: 5
  Set the minReplicas and maxReplicas thoughtfully to avoid thrashing or under-provisioning.
  
  CPU Utilization: 70
  Memory Utilization: 70
  Target CPU and Memory utilization started with 70% as a baselines.

Strategy configuration:
  rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  RollingUpdate is the default and safest for most cases.
  Set the maxUnavailable and maxSurge to balance availability and rollout speed.

How to access the app after deployment:
  port-forward service/todoapp-svc-cip 8081:80 -n mateapp
  https://localhost:30080 - by use NodePort