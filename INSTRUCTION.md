# Deployment Instructions for MateApp

## Deployment Strategy
The deployment uses a `RollingUpdate` strategy with the following parameters:
- `maxUnavailable: 1`: Ensures at least one pod is always available during updates.
- `maxSurge: 1`: Allows one additional pod to be created during updates for smoother transitions.

This strategy minimizes downtime and ensures high availability during updates.

## Resource Requests and Limits
- **Requests**:
  - CPU: `250m`
  - Memory: `128Mi`
- **Limits**:
  - CPU: `500m`
  - Memory: `256Mi`

These values ensure the application has sufficient resources to operate efficiently while preventing over-provisioning.

## Horizontal Pod Autoscaler (HPA)
The HPA is configured to:
- Maintain a minimum of 2 pods and a maximum of 5 pods.
- Autoscale based on:
  - CPU utilization: Target average of 50%.
  - Memory utilization: Target average of 50%.

This configuration ensures the application scales dynamically based on load, optimizing resource usage and maintaining performance.

## Namespace
Both the deployment and HPA belong to the `mateapp` namespace. Ensure the namespace exists before deploying.

## Deployment Steps
1. **Create the Namespace**:
   ```bash
   kubectl create namespace mateapp
   ```

2. **Apply the Deployment Manifest**:
   ```bash
   kubectl apply -f .infrastructure/deployment.yml
   ```

3. **Apply the HPA Manifest**:
   ```bash
   kubectl apply -f .infrastructure/hpa.yml
   ```

4. **Verify the Deployment**:
   ```bash
   kubectl get deployments -n mateapp
   ```

5. **Verify the HPA**:
   ```bash
   kubectl get hpa -n mateapp
   ```

## Accessing the Application
- Ensure a service is configured to expose the application.
- Use `kubectl get svc -n mateapp` to retrieve the service details.
- Access the application via the service's external IP or NodePort.