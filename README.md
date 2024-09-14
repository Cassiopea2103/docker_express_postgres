# Project Report
## Dockerizing Express/Node.js backend with Postgres on AWS EC2

---
This project involves setting up a Dockerized Express/Node.js backend application with a PostgreSQL database, running on an AWS EC2 instance. The project demonstrates how to deploy and manage a simple web application and its database using Docker.

<small>_Since we had issues with local Docker installation on our computer , we decided to do the project on an Amazon AWS EC2 instance_</small>

--- 

###  Setting up an AWS EC2 instance :
On AWS EC2 dashboard , we created a new instance with the parameters : 
* Image : Ubuntu LTS 22.04 
* Type : t2.micro
* With key/pair 
* Network with allowed PORTS 22 ( SSH ) , 3000 ( express ) , 5432 ( Postgres )
            
    <img width="932" alt="0 aws_dashboard_ec2_created" src="https://github.com/user-attachments/assets/703e6390-7008-487e-8701-d7d4600ecf72">


### Connecting to Docker VM via SSH 
<small>_For SSH connection , we use the app MobaXterm_</small>  
  
1. Here is by default the MobaXterm interface with connection configurations :       
<img width="960" alt="1 xterm_interface" src="https://github.com/user-attachments/assets/9106e575-ceff-4130-ae44-df48a4052ce2">

2. On a successful connection , we have :       
<img width="960" alt="2 ssh_connected" src="https://github.com/user-attachments/assets/81c127f9-821b-47ba-9dd4-c998ddffa034">



### Installing and configuring Docker on the host server 
On the EC2 instance SSH terminal , we lauch the following commands to install docker : 
```bash
cd /opt
mkdir docker && cd docker 
sudo chown -R ubuntu:ubuntu docker 
sudo apt update 
sudp apt install docker.io 
```

Next to check for installation status , we can do : 
```bash
sudo systemctl start docker
sudo systemctl enable docker 
sudo systemctl status docker
```
      
<img width="960" alt="3 docker_running_state" src="https://github.com/user-attachments/assets/36d85065-8087-4c94-8878-b94964c75c5c">


After which , we have to add the ubuntu user to the docker group so it can run docker cmds : 
```bash
sudo usermod -aG docker ubuntu
```
      
<img width="956" alt="4 user_ubuntu_groups" src="https://github.com/user-attachments/assets/a4e8826b-5f24-4e8e-9025-42c2ce4170a7">


### Postgres configuration : 
<small>Our backend consumes data from Postgres</small>

For setting up postgres on our EC2 server : 
1. We pull the image from Docker Hub : 
    ```docker
    docker pull postgres
    ```
        
    <img width="788" alt="5 postgres_pull" src="https://github.com/user-attachments/assets/c9a36a83-4aaa-4d70-9b8c-795875c82f16">


2. Create a docker network : 
    ```docker
    docker network create my-network
    ```

3. Run the postgres image to create the container:
    ```docker
    docker run -d --name postgres-container \
    --network my-network \
    -e POSTGRES_USER=myuser \
    -e POSTGRES_PASSWORD=mypassword 
    -e POSTGRES_DB=mydb \
    -p 5432:5432 \
    postgres 
    ``` 
   
   <img width="679" alt="6 running_postgres_container" src="https://github.com/user-attachments/assets/7f6dc88f-3132-454d-9bf5-164e85fab14c">

   
4. We can check next running containers in docker with : 
    ```bash 
    docker ps
    ```   
   <img width="959" alt="7 running_container_postgres" src="https://github.com/user-attachments/assets/183ebf66-54d4-4b9e-a6b4-e481fd31907c">

    
5. Next we manually log into postgres and add extra configurations like database mydb creation , and table users setup : 
    ```docker
    docker exec -it postgres-container psql -U myuser -d postgres
    ```

    ```postgres
    CREATE DATABASE mydb;
    CREATE DATABSE

    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100),
        email VARCHAR(100) UNIQUE NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
    CREATE TABLE
    ```


Once done , we can stop and restart the postgres container for precautions : 

```docker
docker stop postgres-container
docker start postgres-container
```

### Express backend 
Inside the VM `/opt` folder , we can a folder `express_postgres_app` 


1. Initializing the node project : 
    ```node
    npm init -y
    ```       
    <img width="593" alt="8 initialize_express_app" src="https://github.com/user-attachments/assets/eaa209b3-3c8b-45e9-b7b6-302d92672091">


2. Install dependencies : 
    ```node 
    npm install express pg nodemon
    ```

3. Create a `server.js` file : 
    ```bash
    touch server.js
    ```

4. The content of `server.js` is as follows :  
<small><i>It's a simple node backend with users CRUD operations interacting with postgres database</i></small>       
<img width="960" alt="9 express_code" src="https://github.com/user-attachments/assets/9299a903-e6f1-4707-ba38-13e41475cfe6">



5. Next we create our Dockerfile for creating express app image : 
    ```bash
    touch Dockerfile
    ```

    The content of the Dockerfile :    
    <img width="726" alt="10 dockerfile" src="https://github.com/user-attachments/assets/ce729795-00d3-43cf-801c-2083ca82c8ab">


6. After which we run the docker build command to create our express image : 
    ```bash
    docker build -t my-express-app .
    ```      
    <img width="796" alt="11 1 express_image_build" src="https://github.com/user-attachments/assets/0d4cfcc8-3de3-42c7-849a-d5fec7675345">

7. Running the express image to build the container: 
    ```docker
    docker run -d \
    --name express-app \
    --network my-network
    -p 3000:3000 \
    my-express-app
    ```

### Testing application : 
<small>_With the containers ( both Postgres and express backend app ) running , we can test our backned endpoints to see if everything is working appropriately_</small>


<small>_On any API test platform ( we used Insomnia ) , we create our endpoints routes for testing the app_</small>

* Creating a user :       
<img width="960" alt="13 1create_user2_api" src="https://github.com/user-attachments/assets/81e8c6c5-8841-49ff-9199-67989de0af73">


* Retrieving users :     
<img width="960" alt="14 get_users" src="https://github.com/user-attachments/assets/34bd9c22-b73d-409c-9a0b-094c0f1d69df">


* <small>_The same goes for UPDATE and DELETE users endpoints_</small>

* Using the web interface to retrieve users : 
<img width="953" alt="15 get_users_from_web" src="https://github.com/user-attachments/assets/2c5b5fd7-23ea-4d47-a3fb-c22e13079a5f">



### Publishing image to Docker Hub : 
We tag the image for before pushign it to docker hub:
```docker
docker tag my-express-app cassiopea21/my-express-app:v1.0.0
```   

<img width="911" alt="16 tagging image" src="https://github.com/user-attachments/assets/a688a846-4c4a-40af-9143-230dd7b240a1">


Logging into docker : 
```docker
docker login
```      
<img width="959" alt="17 docker login" src="https://github.com/user-attachments/assets/0b0f3ab6-f369-4b9c-bf9c-0ab3d3a2485e">


Pushing the image to Docker Hub : 
```docker 
docker push cassiopea21/my-express-app:v1.0.0
```   
<img width="955" alt="18 image push" src="https://github.com/user-attachments/assets/82c0f7b9-c8e7-409a-bab9-3dd4cf11c8bd">



On Docker Hub , we can check the image is effectively there :   
<img width="951" alt="19 docker-hub-image" src="https://github.com/user-attachments/assets/52136908-de22-4136-a53b-8121b4b613fb">





