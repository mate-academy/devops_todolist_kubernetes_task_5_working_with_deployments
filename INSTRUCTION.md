# Deploying the ToDo App to Kubernetes

These instructions describe how to deploy the app to a Kubernetes cluster using the provided manifests

## Prerequisites

- Kubernetes cluster created and running
- Namespace 'mateapp' created
- Metrics Server installed and running in the cluster

## Attached files

- cluster.yml
- metricsserver.yml
- deployment.yml
- hpa.yml
- INSTRUCTION.md

## Deployment

- Install Kind on Linux

```bash
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
    chmod +x ./kind
    sudo mv ./kind /usr/local/bin/kind
```

- Create the namespace
```bash
    kubectl create namespace mateapp
```

- Apply the Deployment

```bash
    cd .infrastructure
    kubectl apply -f metricsserver.yml
    kubectl apply -f deployment.yml
    kubectl apply -f hpa.yml
```

## Explanation of Configuration Choices

### Resource Requests and Limits
```
resources:
    requests:
        memory: "128Mi"
        cpu: "100m"
    limits:
        memory: "256Mi"
        cpu: "500m"
```

### Horizontal Pod Autoscaler (HPA)
```
minReplicas: 2
maxReplicas: 5
```
- Minimum Replicas = 2
- Maximum Replicas = 5

### RollingUpdate Strategy
```
strategy:
  type: RollingUpdate
```
