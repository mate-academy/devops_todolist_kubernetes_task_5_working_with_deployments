## Creating all resources:
 From ./.infrastructure workdir run commands in this order:
 ```
 kubectl apply -f ./namespace.yml
 kubectl apply -f ./clusterIp.yml
 kubectl apply -f ./nodeport.yml
 kubectl apply -f ./deployment.yml
 kubectl apply -f ./busybox.yml
 kubectl apply -f ./hpa.yml
 ```

### I request 256Mi memory and 100m cpu because it will be enough to normal function for little python web server, and limit it to 512Mi and 200m for successful startup.

### In autoscaling I choose average utilization for memory and cpu 70% of requested because it will be enough  for autoscaler to react on increasing load on server.

### MinUnavailable i choose 1, because min replicas is 2, and it'll be right if one of server during rolling update will be working. And surge 1 additional pod for balance if 1 pod is updating.

## Running app:

Testing NodePort service:

Get ip from one of your node and use it in browser:

http://ip-node:30080
