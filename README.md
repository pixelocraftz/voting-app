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

 
![image](https://user-images.githubusercontent.com/82562140/114856477-c8bfc180-9e04-11eb-88e1-c0d76cd9cba6.png)
