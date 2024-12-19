Create namespace - kubectl create namespace mateapp
Get ready dockerfile and save it on DockerHub
Create deployment.yml and write manifest on it kubectl apply -f deployment.yml
Create hpa.yml and write manifest on it kubectl apply -f hpa.yml
Check state pods, hpa, deployment
About choise of resourse request i take cpu: "200m" and memory: "256Mi" and double limits for more load
HPA configuration will take the instruction from task :   1. Minimum number of pods as 2
                                                          2. Maximum number of pods as 5
                                                          3. Autoscale should be triggered by both CPU and Memory"
and for averageUtilization for cpu will be 80% and memory 70% 
rollingUpdate.maxUnavailable: 1 -- That will be 100% what one runing pod always work even if there are only two
rollingUpdate.maxSurge: 1 -- This allows you to make sure the new pod is working properly before stopping the old pod.