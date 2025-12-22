how to deploy the app to k8s:

kubectl apply -f ./.infrastructure/deployment.yml
kubectl apply -f ./.infrastructure/hpa.yml  

Resource requests and limits explanation:

64Mi memory and 250m CPU were chosen based on the application’s observed idle and baseline load, where the app runs stably without garbage collection pressure or CPU throttling.
128Mi memory and 500m CPU define a safe upper bound for short traffic spikes while preventing a single Pod from exhausting node resources.

HPA configuration explanation:

The 75% utilization target was chosen to trigger scaling before CPU or memory saturation, accounting for HPA reaction delay and avoiding latency degradation near 100% usage.
minReplicas: 2 ensures baseline availability and redundancy, while maxReplicas: 5 caps resource usage to prevent uncontrolled scaling on limited cluster capacity.

strategy configuration explanation

maxSurge: 1 allows one extra Pod during updates to maintain availability without causing CPU or memory spikes on the node.
maxUnavailable: 1 ensures at least one Pod remains available while keeping the rollout fast and resource-efficient.

how to access the app after deployment:

kubectl apply -f ./.infrastructure/nodeport.yml

and then access site as http://localhost:30080/