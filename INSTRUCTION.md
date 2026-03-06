To deploy the app to k8s use commands:

    cd ./.infrastructure

    kubectl apply -f deployment.yml
    kubectl apply -f hpa.yml
    
As the app will be the only app deployed on server, we can allocate to the app a lot of resources. In hpa.yml, I put the both utilizations to 80 percents because it is the highest point after which there is a big chance of using all allowed resources. In deployment.yml, I put such maxUnavailable because Deployment deploys 2 replicas and I need 2 replical always be running and such maxSurge to create every new pod while old pod is being deleted.

To access the app use a command:

    kubectl port-forward deployment/todoapp 8080:8080