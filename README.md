## how to deploy the app to k8s:

kubectl apply -f deployment.yml
kubectl apply -f hpa.yml


## Explaining the choice of resources requests and limits

By configuring these requests and limits, we guarantee that no more than two pods can operate simultaneously without straining the system. These values are customized according to the specifications of the PC where the application will be deployed.

## Explaining the choice of HPA configuration

minReplicas set to 2 and maxReplicas set to 5 - This allows the application to scale up or down dynamically based on demand.
averageUtilization set at 70% - Both CPU and memory utilization thresholds are established at 70%. If usage exceeds this limit, additional pods will be created; conversely, if it drops below, unnecessary pods will be terminated.
## Explaining the strategy configuration

RollingUpdate strategy - This approach facilitates a gradual deployment of new versions. It replaces old pods with new ones incrementally, which minimizes service interruptions. By permitting only one pod to be updated at a time (maxSurge: 1), it ensures a smooth transition.

maxUnavailable set to 1 - This setting guarantees that at least one old pod remains accessible during the update process.

## How to access the app after deployment

kubectl apply -f .infractructure/

## Test app with the port-forward

kubectl port-forward service/{service_name} 8081:80

## Test app with ClusterIP

kubectl -n {namespace} exec -it busybox -- sh

curl http://{service_name}.{namespace}.svc.cluster.local
