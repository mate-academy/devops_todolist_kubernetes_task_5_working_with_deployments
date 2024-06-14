# Django ToDo list

This is a todo list web application with basic features of most web apps, i.e., accounts/login, API, and interactive UI. To do this task, you will need:

- CSS | [Skeleton](http://getskeleton.com/)
- JS  | [jQuery](https://jquery.com/)

## Explore

Try it out by installing the requirements (the following commands work only with Python 3.8 and higher, due to Django 4):

```
pip install -r requirements.txt
```

Create a database schema:

```
python manage.py migrate
```

And then start the server (default is http://localhost:8000):

```
python manage.py runserver
```

Now you can browse the [API](http://localhost:8000/api/) or start on the [landing page](http://localhost:8000/).

## Task

Create a kubernetes manifest for a pod which will containa ToDo app container:

1. Fork this repository.
1. Create a `deployment.yml` file with a deployment for the app.
1. Deployment should have
    1. Strategy: RollingUpdate
    1. Resource requests and limits (in idle state you should have 2 pods running)
    1. Pod spec should be same as for pods manifest
1. Createa a `hpa.yml` file with a Horizontal Pod Autoscaler for the app.
1. Autoscaler should define
    1. Minimum number of pods as 2
    2. Maximum number of pods as 5
    3. Autoscale should be triggered by both CPU and Memory
1. Both new manifests should belong to `mateapp` namespace
1. `README.md` should be updated with the instructions on how to deploy the app to k8s
1. `README.md` Should have explained you choice of resources requests and limits
1. `README.md` Should have explained your choice of HPA configuration
1. `README.md` Should have explained your strategy configuration (Why such numbers)
1. `README.md` Should have explained how to access the app after deployment
1. Create PR with your changes and attach it for validation on a platform.

#
# Solution

### I. Deploying the app

*terminal is opened in project root folder

1. **created mateapp namespace**

    ```
    kubectl apply -f .\.infrastructure\namespace.yml
    ```
2. **created the deployment service**

    1. requested such recources becouse it is enough for our dead simple todoapp yet not triggering autoscaling in idle and minimum load.
    2. set strategy's `maxUnavailable: 1` because we have default idle 2 containers, in order to guarantee the service running at all times we can not set bigger number
    3. set strategy's `maxSurge: 1` in we do not expect sudden huge increase in resources consumption

    ```
    kubectl apply -f ..infrastructure\deployment.yml
    ```
3. **created horizontal autoscaler**

    I've set `averageUtilization` 70% of *requested* resources

    1. however I could achive same with `averageValue` of `9.6Mi` for memory & `35m` for cpu,
        but 70% `averageUtilization` is more reliable since `requests` could be changed in future
        this option makes our autoscaler not independent controller ( it should be adjusted for appropriate changes )
    2. option with exact `value` is not necessary in our case

    ```
    kubectl apply -f ..infrastructure\hpa.yml
    ```
4. **created nodeport service**

    ```
    kubectl apply -f ..infrastructure\nodeport-svc.yml
    ```
5. **created curlimage pod**

    ```
    kubectl apply -f .\.infrastructure\busybox.yml
    ```

### II. Accessing the app

1. accessed our app through `localhost:30001` since we have active nodeport service
2. accessed our app through curlimage

    ```
    kubectl exec -it -n mateapp curlpod -- sh
    curl http://nodeportsvc.mateapp.svc.cluster.local/
    curl http://nodeportsvc.mateapp.svc.cluster.local/api/health
    ```
3. accessed our app through port-forwarding

    ```
     port-forward svc/nodeportsvc 8081:80 -n mateapp
    ```

    `localhost:8081`

    ```
    exit
    ```
