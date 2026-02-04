# Deployment instructions

## 1) Create namespace

```bash
kubectl create namespace mateapp
```

## 2) Deploy the app and HPA

```bash
kubectl apply -f deployment.yml
kubectl apply -f hpa.yml
```

Check rollout:

```bash
kubectl -n mateapp rollout status deployment/todoapp
```

Note: HPA requires `metrics-server` to be installed in the cluster.

## 3) Access the app

Simplest option is port-forwarding:

```bash
kubectl -n mateapp port-forward deployment/todoapp 8080:8080
```

Open http://localhost:8080 in your browser

If you prefer a Service, you can copy `.infrastructure/nodeport.yml`, change the namespace to `mateapp`, and apply it.

---

# Rationale

## Resource requests and limits

- **Requests:** `cpu: 100m`, `memory: 128Mi`. These are conservative for a small Django app and ensure the scheduler can place at least 2 pods on modest nodes (for example, a 1 vCPU / 2 GiB node would still have plenty of headroom).
- **Limits:** `cpu: 500m`, `memory: 512Mi`. This allows short bursts (page loads, API spikes) without throttling immediately, while still preventing a single pod from consuming excessive resources.

This sizing keeps two replicas running in idle state while leaving room for rollout surge and HPA scale-out.

## HPA configuration

- **Min 2 / Max 5 replicas** meets the requirement and provides a small but useful burst capacity.
- **CPU 70% / Memory 75% utilization** targets keep headroom for traffic spikes while avoiding frequent scale-up on minor fluctuations. Scaling on both CPU and memory covers both compute-heavy and memory-heavy load patterns.

## RollingUpdate strategy

- **maxUnavailable: 0** keeps all existing replicas available during updates (no downtime), which preserves the “2 pods running” baseline.
- **maxSurge: 1** allows one extra pod during rollout for faster replacement while keeping resource usage bounded.
