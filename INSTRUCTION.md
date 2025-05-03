## 1. Create handy aliases and switch to yml files directory:  
`alias kub=kubectl`  
`alias cls=clear`  
`cd /.infrastructure`  

## 2. Create namespace:  
`kub apply -f namespace.yml`  
set it as default:  
`kub config set-context --current --namespace=mateapp`  

## 3. Start services:  
`kub apply -f nodeport.yml`  
`kub apply -f clusterIp.yml`  

## 4. Start busybox pod:  
`kub apply -f busybox.yml`  

## 5. Start deployment:  
`kub apply -f deployment.yml`  

## 6. Strategy configuration, HPA configuration  
Are defined by the task:
    > Autoscaler should define
    > 1. Minimum number of pods as 2
    > 2. Maximum number of pods as 5
    > 3. Autoscale should be triggered by both CPU and Memory  

    > Deployment should have
    > 1. Strategy: RollingUpdate
    > 1. Resource requests and limits (in the idle state you should have 2 pods running)

## 7. Access the app after deployment  
Wait until `kub get pods` shows that all pod are 1/1 Running.
You can access app via node port by utilizing the link:  
http://127.0.0.1:30080  
