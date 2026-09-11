Use the following instructions below:

1. All instructions:
    - how to deploy the app to k8s:
        kubectl create namespace mateapp
        kubectl apply -f .infrastructure/deployment.yml
        kubectl apply -f .infrastructure/hpa.yml
        kubectl get deployments -n mateapp
        kubectl get replicaset -n mateapp
        kubectl get pods -n mateapp
        kubectl get hpa -n mateapp

    - explained my choice of resource requests and limits:
        I choose those requests and limits as I have Python apllication (todoapp) as test application and those configuration are recommended and acceptable for k8s.

         1) Requests (cpu: 100m, memory: 128Mi):

            CPU: 100m (0.1 vCPU core) provides sufficient compute power for a lightweight web application during normal/idle operations.

            Memory: 128Mi ensures the container has enough room for standard runtime startup and memory overhead without reserving excessive cluster resources.

            Idle Pod Count: Setting replicas: 2 in combination with these requests guarantees that in an idle state, 2 pods run smoothly while maintaining high availability across cluster nodes.

        2) Limits (cpu: 250m, memory: 256Mi):

            CPU: 250m (0.25 vCPU core) allows short bursts of processing power under temporary spikes without monopolizing node CPU.

            Memory: 256Mi acts as a guardrail. If an unexpected memory leak or spike occurs, the container is bounded to prevent node-level memory exhaustion, triggering a controlled container restart (OOMKilled) if breached.

    - explained my choice of HPA configuration:
        Min Replicas (minReplicas: 2): Matches the idle requirement of 2 pods to ensure basic fault tolerance and high availability even under low traffic.

        Max Replicas (maxReplicas: 5): Sets an upper ceiling to prevent runaway scaling from consuming all available cluster capacity during heavy load spikes.

        Metrics Targets (averageUtilization: 70 for CPU & Memory):

        Scaling triggers when average resource consumption across pods exceeds 70% of requested resources (70m CPU or ~89.6Mi Memory per pod).

        A 70% threshold leaves a safety buffer (30%) so existing pods can handle incoming traffic spikes while new pods are being scheduled and initialized.

    - explained my strategy configuration (Why such numbers):
        Strategy Type: RollingUpdate

        Parameters:

        maxUnavailable: 1: Guarantees that during updates, at least 1 pod (50% of the desired replica count) remains running and serving traffic.

        maxSurge: 1: Allows Kubernetes to spawn at most 1 temporary extra pod during deployment (bringing total pods to 3 temporarily).

        Why these numbers?

        This configuration balances zero-downtime updates with resource efficiency.

        It prevents cluster resource overload during deployments while ensuring the application never drops completely offline during a release.

    - explained how to access the app after deployment:
        kubectl port-forward deployment/todoapp 8081:8080 -n mateapp
        Make sure http://localhost:8081/ URL is available and working. 