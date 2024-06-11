1. instructions on how to deploy the app to k8s

```
    kubectl apply -f deployment.yml -f hpa.yml
```

1. Resources requests and limits
   `I have no idea, just used value from lesson, there was no information on how to understand how many resources u need`

1. HPA configuration
   `I have no idea, just used value from lesson, i suppose it is minimal requirements for app`

1. Strategy configuration (Why such numbers)
   `RollingUpdate was choose because it is a brain of application and should be always available`

1. Access the app after deployment visit `http://localhost:30007`
