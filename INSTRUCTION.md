How to deploy the app to Kubernetes

Create the namespace:

kubectl create namespace mateapp


Apply Deployment and HPA manifests:

kubectl apply -f deployment.yml -n mateapp
kubectl apply -f hpa.yml -n mateapp


Check deployed resources:

kubectl get pods -n mateapp
kubectl get deployments -n mateapp
kubectl get hpa -n mateapp

2. Explanation of resource requests and limits

CPU request: 250m — guarantees the scheduler reserves 0.25 CPU per pod so both pods can run reliably in idle state.

CPU limit: 700m — prevents any single pod from consuming too much CPU.

Memory request: 64Mi — lightweight baseline ensuring pods can start even on limited nodes.

Memory limit: 128Mi — caps memory usage to avoid uncontrolled growth.

These values ensure stable operation with 2 pods always running while preventing resource over-consumption.

3. Explanation of HPA configuration

minReplicas: 2 — required idle state: always keep 2 pods running.

maxReplicas: 5 — allows scaling up while preventing unlimited growth.

CPU & Memory metrics, target 70% — scaling triggers when pods are under noticeable load on either CPU or RAM, ensuring responsiveness under stress.

This configuration matches requirements and provides balanced scaling behavior.

4. Explanation of strategy configuration

Strategy: RollingUpdate

maxSurge: 1 — during updates Kubernetes may create 1 extra pod to maintain capacity.

maxUnavailable: 1 — at most one pod can be unavailable during rollout.

These numbers ensure:

uninterrupted service availability

smooth updates

minimal temporary resource overhead

5. How to access the app after deployment
Option 1 — Port-forward (recommended for testing)
kubectl port-forward deployment/todoapp 8080:8080 -n mateapp


Access the app at:

http://localhost:8080/

health: http://localhost:8080/api/health

readiness: http://localhost:8080/api/ready