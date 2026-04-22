# Django ToDo list

This is a to-do list web application with the basic features of most web apps, i.e., accounts/login, API, and interactive UI. To do this task, you will need:

- CSS | [Skeleton](http://getskeleton.com/)
- JS  | [jQuery](https://jquery.com/)

## Explore

Try it out by installing the requirements (the following commands work only with Python 3.8 and higher, due to Django 4):

```sh
pip install -r requirements.txt
```

Create a database schema:

```sh
python manage.py migrate
```

And then start the server (default is <http://localhost:8000>):

```sh
python manage.py runserver
```

You can now browse the [API](http://localhost:8000/api/) or start on the [landing page](http://localhost:8000/).

## Deployment

to deploy app use following command:

```sh
kubectl create ns mateapp && \
kubectl apply -f .infrastructure/deployment.yml && \
kubectl apply -f .infrastructure/hpa.yml && \
kubectl apply -f .infrastructure/clusterIp.yml
```

## Choice explanation

1. Resource limits and request for deployment \
    Small Django app doesn't need too much resources for initial start, but need more that usual in high usage due to low event loop engine optimization.

1. Resource utilization for horizontal scaling in horizontal pod autoscaler \
    Optimal replicas in small utilization is 2 and in high load (+-70%) up to 5 replicas

1. Update strategy configuration \
    for strategy maximum unavailbe 1 pod and max surge 1 for easy app access during rolling update

## Easy access to app

To access the app use ```kubectl port-forward``` function, just enter

```sh
kubectl port-forward svc/todoapp -n mateapp 8080:80
```

and follow [the link](http://localhost:8080/)
