# Django ToDo list

This is a todo list web application with basic features of most web apps, i.e., accounts/login, API, and interactive UI. To do this task, you will need:

- CSS | [Skeleton](http://getskeleton.com/)
- JS  | [jQuery](https://jquery.com/)

## Explore

Try it out by installing the requirements (the following commands work only with Python 3.8 and higher, due to Django 4):

```
pip install -r requirements.txt
```

Create a database schema:

```
python manage.py migrate
```

And then start the server (default is http://localhost:8000):

```
python manage.py runserver
```

Now you can browse the [API](http://localhost:8000/api/) or start on the [landing page](http://localhost:8000/).

## Task

Create a kubernetes manifest for a pod which will containa ToDo app container:

1. Fork this repository.
1. Create a `deployment.yml` file with a deployment for the app.
1. Deployment should have
    1. Strategy: RollingUpdate
    1. Resource requests and limits (in idle state you should have 2 pods running)
    1. Pod spec should be same as for pods manifest
1. Createa a `hpa.yml` file with a Horizontal Pod Autoscaler for the app.
1. Autoscaler should define
    1. Minimum number of pods as 2
    2. Maximum number of pods as 5
    3. Autoscale should be triggered by both CPU and Memory
1. Both new manifests should belong to `mateapp` namespace
1. `README.md` should be updated with the instructions on how to deploy the app to k8s
1. `README.md` Should have explained you choice of resources requests and limits
1. `README.md` Should have explained your choice of HPA configuration
1. `README.md` Should have explained your strategy configuration (Why such numbers)
1. `README.md` Should have explained how to access the app after deployment
1. Create PR with your changes and attach it for validation on a platform.


## Deploying the ToDo Application on Kubernetes with Autoscaling
### Prerequisites
1. Kubernetes cluster (Minikube recommended for testing).
2. kubectl installed and configured to interact with your cluster.
3. Ensure that the metrics-server is running for Horizontal Pod Autoscaler (HPA) to work. You can install it using the following command:
```
minikube addons enable metrics-server
```

## Steps to Deploy the Application
### 1. Create the Namespace: 
 Apply the namespace.yml file to create a dedicated namespace for the ToDo application.


```
kubectl apply -f .infrastructure/namespace.yml
```
### 2. Deploy the Application: Apply the deployment.yml file to deploy the ToDo application and the pod.


```
kubectl apply -f .infrastructure/deployment.yml
```
### 3.Set Up ClusterIP and NodePort Services: Apply the clusterip.yml and nodeport.yml to expose the application within the cluster and at the node level.

```
kubectl apply -f .infrastructure/clusterip.yml
kubectl apply -f .infrastructure/nodeport.yml
```

### 4. Configure Horizontal Pod Autoscaler: Apply the hpa.yml file to enable autoscaling based on CPU and memory utilization.

```
kubectl apply -f .infrastructure/hpa.yml
```
### 5. Verify the Setup: Check the status of the Horizontal Pod Autoscaler:

```
kubectl get hpa -n todoapp
```
### 6.Port Forward to Access the Application: You can access the application locally using the port-forward command:

```
kubectl port-forward svc/todoapp-clusterip 8080:8080 -n todoapp
```
Access the application in your browser at http://localhost:8080.


### Resource Requests and Limits Explanation

In the deployment manifest, I  defined resource requests and limits for the `todoapp` container. This is important to ensure efficient resource usage and prevent one container from consuming all available resources on the node.

```
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```
#### Explanation of Values:
**Memory Request** (64Mi): This defines the minimum amount of memory the container will be allocated. By setting a lower memory request, I ensure that our container runs efficiently without wasting unused memory. 64Mi is a reasonable amount for this lightweight ToDo application.

**CPU Request (250m):** This represents the guaranteed minimum CPU allocated for the container (250m = 0.25 CPU cores). This ensures that even if the node is under heavy load, our app will get at least 25% of one CPU core, which is sufficient for a simple application like this.

**Memory Limit (128Mi):** The memory limit defines the maximum memory the container is allowed to use. If the container exceeds this limit, it could be terminated or throttled. 128Mi ensures that the application doesn't consume too much memory on the node but has enough room to handle typical operations.

**CPU Limit (500m):** The CPU limit is set to 500m (0.5 CPU cores). This prevents the container from using more than half a CPU core, ensuring other containers or system processes also have available CPU resources. For a lightweight web app like this, half a core is sufficient for handling typical traffic.


### Horizontal Pod Autoscaler (HPA) Configuration Explanation

In the HPA configuration, we define how the Kubernetes Horizontal Pod Autoscaler should scale our ToDo application based on CPU utilization. Below is the HPA configuration from the manifest:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: todoapp-hpa
  namespace: todoapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: todoapp-deployment
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 50
  ```

### Explanation of Values:
**minReplicas: 2:**
This sets the minimum number of pod replicas to 2. It ensures that even if the load on the application is low, we still have at least 2 running instances. This provides redundancy and ensures availability in case of failures.

**maxReplicas: 5:**
The maximum number of replicas is set to 5. This ensures that the application can scale up to 5 instances when there is a heavy load. Limiting the upper bound helps control resource usage and prevents excessive scaling that might overload the cluster.

**targetCPUUtilizationPercentage: 50:**
The HPA will trigger scaling actions based on the average CPU utilization across all running pods. If the CPU utilization exceeds 50% (the target), HPA will increase the number of pod replicas. If the CPU utilization falls below this threshold, it will reduce the number of replicas.

### Why These Numbers?
**Minimum Replicas (2):**
Setting 2 as the minimum number of replicas ensures that the application has enough redundancy and availability. Even with minimal traffic, having 2 pods ensures fault tolerance.

**Maximum Replicas (5):**
The choice of 5 for maximum replicas is based on anticipated load. Given the nature of the ToDo application, 5 replicas should handle traffic spikes while preventing resource exhaustion on the cluster.

**50% CPU Utilization Target:**
The target CPU utilization of 50% strikes a balance between performance and efficiency. If the average CPU usage across the pods exceeds 50%, the application will scale to handle the increased load, but if CPU utilization stays below this value, fewer resources are used.

### Accessing the ToDo Application After Deployment

Once the ToDo application is successfully deployed on Kubernetes, you can access it using two main methods:

1. **Accessing via NodePort:**

   The application is exposed using a **NodePort** service, which allows access to the application through a specific port on any node in the cluster.

   - **NodePort Configuration:**
     The application is configured to use port `30080` on the node.
     In the `nodeport-service.yml` file, we have the following configuration:
   
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: todoapp-nodeport
     namespace: todoapp
   spec:
     type: NodePort
     selector:
       app: todoapp
     ports:
     - protocol: TCP
       port: 80
       targetPort: 8080
       nodePort: 30080
    ```

### Accessing the App: To access the application, use the following URL format:
```
http://<Node-IP>:30080
```
Replace <Node-IP> with the IP address of any node in your cluster. The node IP can be found by running:

```
kubectl get nodes -o wide
```

**Accessing via Port-Forwarding:**

Alternatively, you can use the port-forward command to temporarily forward a local port to the ClusterIP service in Kubernetes.

Port-Forwarding Command: Run the following command to forward the service port 80 to your local port 8080:

```
kubectl port-forward svc/todoapp-clusterip 8080:80 -n todoapp
```

Accessing the App: After running the port-forward command, you can access the application locally at:

```
http://localhost:8080
```