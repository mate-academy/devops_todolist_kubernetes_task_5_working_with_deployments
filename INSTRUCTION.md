## Deployment Steps

### Create a infrastructure(how to deploy the app to k8s):
   ```bash
   kubectl apply -k .\.infrastructure\
   ```
### How to access the app after deployment:
```bash
http://localhost:30080/
   ```
## Resource requests and limits
CPU Request:100m (0.1 CPU) - This ensures the app has enough CPU to run in idle state.
CPU Limit: 400m (0.4 CPU) - This limits the CPU usage to prevent excessive consumption.
Memory Request: 128Mi - The app should be allocated at least 128Mi of memory to function correctly.
Memory Limit: 512Mi - The app is limited to 512Mi of memory to avoid memory leaks.
### These values were chosen based on app metrics in work.

## Horizontal Pod Autoscaler (HPA)
Min. pods = 2
Max. pods = 5
### The HPA triggers scaling when CPU or memory usage exceeds 70%.
These numbers were selected to avoid sudden jumps in resource load, so that HPA has time to raise an additional POD.