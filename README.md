# voting-app
A Kubernetes deployment training application using Docker containers. Postgres nd Redis images are available in Docker Hub.

Training Application Overview:
![image](https://user-images.githubusercontent.com/82562140/114845014-fe5ead80-9df8-11eb-9f39-fee1f5cc7ed2.png)

There is a Vote web app running on Python which allows user to vote.
The choice of the user is collected in redis queue. There is a .NET worker which connects the Redis queue to App Postgres Database to store the user vote.
Node.js web app fetches the user vote option from db and displays it in a web page.

Screenshot of Python Web application which allows user to vote. Runs on public IP of cluster node.
![image](https://user-images.githubusercontent.com/82562140/114832131-c2711b80-9deb-11eb-91cd-865d384052cf.png)

Screenshot of Node.js Web application which allows users to view voting result. Runs on public IP of cluster node.
![image](https://user-images.githubusercontent.com/82562140/114832273-ee8c9c80-9deb-11eb-938e-e8fd57fb49a3.png)

#Pre-requisites for running this project
1) VM/Cloud Cluster Nodes (1 Master Node and any number of worker Nodes) with Kubernetes components installed on it. Ensure that all worker nodes are connected to master node.
2) I used pre-setup environment to execute this app, Strigo.io/events with 1 master node and 2 worker nodes setup in AWS cloud.

#Exceution Sequence [Cluster nodes are Linux machines]

1) Make new directory in home/ubuntu/
ubuntu@master:~$ mkdir mylabproject
ubuntu@master:~$ cd ~/mylabproject
   
2) Create a custom namespace 
ubuntu@master:~/mylabproject$ kubectl create -f vote-namespace.yaml   #this command will create a new namespace. If not created all our pods are created in default Kubernetes namespace.
ubuntu@master:~/mylabproject$ kubectl get namespace  #this command is to verify namespace creation
 

  
3)Create a Persistent Volume with type hostPath and Storage size 1Gi
   ubuntu@master:~/mylabproject$ kubectl create -f vote-pv.yaml 
   ubuntu@master:~/mylabproject$ kubectl get pv #this command is to verify PV creation. At this stage, Claim field will be empty.


4.) Create a Persistent Volume Claim with access mode Read & write and link it with our PV. Linking happens through StorageClass Name yaml parameter.
ubuntu@master:~/mylabproject$ kubectl create -f vote-pv-claim.yaml 
ubuntu@master:~/mylabproject$ kubectl get pv #this command is to verify PV creation & linking to PV claim


5) Create a deployment for postgres with image postgres:9.4 
ubuntu@master:~/mylabproject$ echo "postgres" | base-64  #encrypt Postgres Username and Password and use in Kubernetes secret file.
ubuntu@master:~/mylabproject$ kubectl create -f db-secret.yaml #create a Kubernetes Secret file which stores all the encrypted auth creds, tokens, certs. 
ubuntu@master:~/mylabproject$ kubectl create -f vote-db-deployment.yaml #run the db deployment
ubuntu@master:~/mylabproject$ kubectl get all  --namespace vote  #displays all the running instances inside namespace vote

Any status other than Running is considered as error.

6.) Create a postgres db clusterIP service 
ubuntu@master:~/mylabproject$ kubectl create -f vote-db-service.yaml #run the db service
ubuntu@master:~/mylabproject$ kubectl describe service db #verify the creation of service db


7.) Create a deployment for redis with image redis:alpine
ubuntu@master:~/mylabproject$ kubectl create -f vote-redis-deployment.yaml #run the redis deployment
ubuntu@master:~/mylabproject$ kubectl get all  --namespace vote  #displays all the running instances inside namespace vote

Any status other than Running is considered as error.

8.) Create a redis cluster IP service
ubuntu@master:~/mylabproject$ kubectl create -f vote-redis-service.yaml #run the redis service
ubuntu@master:~/mylabproject$ kubectl describe service redis #verify the creation of service redis


9.) Create a Deployment with 1 replica set using the image mirantistraining/voting-worker:v1
ubuntu@master:~/mylabproject$ kubectl create -f vote-worker-deployment.yaml #run the worker deployment

10.) Create a Deployment with 2 replicas set using the image mirantistraining/voting-app:v1
ubuntu@master:~/mylabproject$ kubectl create -f vote-app-deployment.yaml #run the app deployment

11.) Create a Deployment with one replica using the image mirantistraining/voting-result:v1
ubuntu@master:~/mylabproject$ kubectl create -f vote-result-deployment.yaml #run the result deployment

12.) Create a nodeport service running on Port 31002 for voting-app
ubuntu@master:~/mylabproject$ kubectl create -f vote-app-service.yaml #create a nodeport service on port 31002

13.) Create a nodeport service running on Port 31003 for voting-app
ubuntu@master:~/mylabproject$ kubectl create -f vote-result-service.yaml #create a nodeport service on port 31003
ubuntu@master:~/mylabproject$ kubectl get all  --namespace vote  #displays all the running instances inside namespace vote. To verify step 9 to 13.


14.) Run the voting application on http://<publicIP of master node or worker node>:31002 & Run the voting result application on http://<publicIP of master node or worker node>:31003 
![image](https://user-images.githubusercontent.com/82562140/114856199-74b4dd00-9e04-11eb-8048-ebd7f4dfce2a.png)


   
   
