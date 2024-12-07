##  How to deploy the app to k8s

``

    kubectl create ns mateapp
    kubectl apply -f clusterIp.yml 
    kubectl apply -f nodeport.yml 
    kubectl apply -f deployment.yml 
    kubectl apply -f externalName.yml
    kubectl config set-context --current --namespace=mateapp
``

## explain choice of resource requests and limits

Examples of policies that could be created using LimitRange are:
In a 2 node cluster with a capacity of 8 GiB RAM and 16 cores, constrain Pods in a namespace to request 100m of CPU with a max limit of 500m for CPU and request 200Mi for Memory with a max limit of 600Mi for Memory.
Define default CPU limit and request to 150m and memory default request to 300Mi for Containers started with no cpu and memory requests in their specs.

## Choice of HPA

Took it from lesson

##  Strategy configuration

MaxUnavailable specifies the maximum number of pods that are allowed to be unavailable during the rollout.

## How to access the app after deployment

http://localhost:8000/