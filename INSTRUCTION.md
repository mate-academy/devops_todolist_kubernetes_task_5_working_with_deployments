1. How to deploy the app to k8s.

    - Create new Namespace "mateapp" from namespace.yml manifest:

        cd .\.infrastructure\ | kubectl apply -f namespace.yml
    
    - Run the app from deployment.yml manifest:

        kubectl apply -f deployment.yml

    - Run Horizontal Pod Autoscaler for the app from hpa.yml manifest:

        kubectl apply -f hpa.yml


2. Explaine your choice of resource requests and limits.

    The specified requests and limits parameter values ​​ensure the use of the minimum number of replicas for running the application with low load.


3. Explained your choice of HPA configuration.

    Since the application is being used in test mode, this HPA configuration will provide a clear example of Horizontal Pod Autoscaler with medium load.


4. Explained your strategy configuration (Why such numbers).

    This configuration enables the application to be updated without stopping it.


5. How to access the app after deployment.

    - Create NodePort service in the mateapp Namespace from nodeport.yml manifest:

        kubectl apply -f nodeport.yml

    - Run the app using TCP port 30007.