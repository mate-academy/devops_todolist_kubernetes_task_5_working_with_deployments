## Deployment Instruction: TodoApp
This guide explains how to deploy the TodoApp to a Kubernetes cluster and provides a breakdown of the architectural decisions made for reliability and scalability.

### 1. How to Deploy the App
To deploy the application, follow these steps in order:
1. Create the namespace:
`kubectl create namespace mateapp`
2. Apply Deployment and HPA:

`kubectl apply -f deployment.yml`

`kubectl apply -f hpa.yml`
3. Apply Service (NodePort):
`kubectl apply -f nodeport.yml`
### 2. Resource Requests and Limits
We have configured the following resources for each pod:
- `Requests: memory: 130Mi, cpu: 200m`
- `Limits: memory: 220Mi, cpu: 400m`
#### Why these numbers?
- Requests: Set based on the actual baseline consumption of the Flask/Python app (~30-60Mi). We added a "safety buffer" to ensure the pod always has enough resources to start without being throttled.
- Limits: The memory limit is set to roughly double the request to handle occasional spikes in traffic. If the pod hits 220Mi, it will be terminated (OOMKilled) to protect the node from crashing.

### 3. HPA Configuration
The Horizontal Pod Autoscaler (HPA) is configured with:
- Min Replicas: 2 (Ensures High Availability).
- Max Replicas: 5 (Protects cluster resources from infinite scaling).
- Target Utilization: 70% for both CPU and Memory.

Why these choices? We chose 70% as a threshold to allow the app to scale before it becomes dangerously slow. Using both CPU and Memory metrics ensures that if the app is either calculation-heavy or data-heavy, the cluster will respond by adding more pods.

### 4. Deployment Strategy
We use the RollingUpdate strategy with the following settings:
- maxUnavailable: 1
- maxSurge: 1

Why these numbers? This ensures Zero Downtime.
- maxSurge: 1 means during an update, Kubernetes will spin up one extra pod before killing an old one.
- maxUnavailable: 1 ensures that at least 1 pod is always running and serving traffic while the update is in progress.

### 5. How to Access the App
Since the application is running in a Kind (Kubernetes-in-Docker) environment on a local host, a standard NodePort might not be directly reachable via localhost.

Port Forwarding (Recommended)

Run the following command to create a tunnel from your machine to the service:
`kubectl port-forward svc/todoapp 8081:80 -n mateapp`