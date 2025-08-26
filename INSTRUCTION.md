# 1. Namespace (створити або оновити)
kubectl create namespace mateapp || true

# 2. Застосувати Deployment
kubectl -n mateapp apply -f .infrastructure/deployment.yml

# 3. Застосувати HPA
kubectl -n mateapp apply -f .infrastructure/hpa.yml

# 4. Перевірити розгортання
kubectl -n mateapp rollout status deploy/todoapp

# 5. Перевірити стани
kubectl -n mateapp get deploy,rs,pods
kubectl -n mateapp get hpa
