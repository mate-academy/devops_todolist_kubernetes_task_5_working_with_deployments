# 1) How to deploy the app to k8s:
    
Firstly go to the directory .infrastructure:

    cd ./.infrastructure

And then:

    kubectl apply -f deployment.yml


# 2) Resourse requests and limits:
    
* **Memory (Requests: 128Mi / Limits: 256Mi):** * `128Mi` of RAM is allocated as a guaranteed baseline (`requests`), which is more than enough for the application to boot and handle idle state traffic. 
* The `256Mi` `limit` acts as a safety ceiling. It provides a 2x headroom to handle temporary memory spikes (e.g., heavy API payloads) while preventing memory leaks from exhausting the cluster's node resources (via standard Kubernetes OOMKilled protection).
* **CPU (Requests: 100m / Limits: 200m):** * `100m` (0.1 of a CPU core) ensures that the Pod can be easily scheduled on almost any node while maintaining smooth execution during low-traffic periods. 
* The `200m` limit allows the container to burst up to 20% of a core during startup or intense computation without causing severe CPU throttling.

# 3) HPA configuration:

* **Min Replicas (2):** Matches the assignment requirement to have exactly 2 pods running in the idle state for high availability and redundancy.
* **Max Replicas (5):** Caps the maximum scale-out factor to 5 pods. This accommodates a 2.5x traffic surge while protecting the infrastructure from unexpected cloud billing costs or database connection exhaustion.
* **Triggers (CPU & Memory at 70%):** The 70% threshold is an industry-standard best practice. It triggers scaling *before* the application hits its performance bottlenecks.
* Dual-metric tracking ensures that if the app becomes bound by heavy computation (CPU) OR large data caching/session accu

# 4) Strategy configuration:

With an idle state of 2 replicas, this configuration ensures the safest, most resource-efficient rollout:
* `maxSurge: 1` means during a new deployment, Kubernetes will spin up at most **1 additional Pod** at a time (totaling 3 temporarily), preventing massive sudden spikes in node resource consumption.
* `maxUnavailable: 1` guarantees that even during an active update, **at least 1 Pod is always online** and healthy to serve incoming user traffic, ensuring absolutely **zero-downtime (ZDT)** upgrades.

# 5) How to access the app:

    kubectl port-forward deployment/todoapp-deployment <your_port>:8080 -n mateapp