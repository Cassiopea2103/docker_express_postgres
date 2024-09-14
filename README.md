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
            
    ![AWS dashboard](./0.aws_dashboard_ec2_created.png)

### Connecting to Docker VM via SSH 
<small>_For SSH connection , we use the app MobaXterm_</small>  
  
1. Here is by default the MobaXterm interface with connection configurations :       
![MobaXterm_Interface](./1.xterm_interface.png)
2. On a successful connection , we have :       
![Connection_successful](./2.ssh_connected.png)


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
      
![docker_running](./3.docker_running_state.png)

After which , we have to add the ubuntu user to the docker group so it can run docker cmds : 
```bash
sudo usermod -aG docker ubuntu
```
      
![user_ubuntu_added_to_docker_group](./4.user_ubuntu_groups.png)

### Postgres configuration : 
<small>Our backend consumes data from Postgres</small>

For setting up postgres on our EC2 server : 
1. We pull the image from Docker Hub : 
    ```docker
    docker pull postgres
    ```
        
    ![postgres_image_pull](./5.postgres_pull.png)

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
   
    ![postgres_container](./6.running_postgres_container.png)
   
4. We can check next running containers in docker with : 
    ```bash 
    docker ps
    ```   
    ![running_containers](./7.running_container_postgres.png)
    
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
    ![node_project_setup](./8.initialize_express_app.png)

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
![server_content](./9.express_code.png)  


5. Next we create our Dockerfile for creating express app image : 
    ```bash
    touch Dockerfile
    ```

    The content of the Dockerfile :    
    ![Dockerfile_content](./10.dockerfile.png)

6. After which we run the docker build command to create our express image : 
    ```bash
    docker build -t my-express-app .
    ```      
    ![docker_image_build](./11.1.express_image_build.png)
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
![create_user](./13.1create_user2_api.png)

* Retrieving users :     
![retrieve_users](./14.get_users.png)

* <small>_The same goes for UPDATE and DELETE users endpoints_</small>

* Using the web interface to retrieve users : 
![retrieve_users](./15.get_users_from_web.png)


### Publishing image to Docker Hub : 
We tag the image for before pushign it to docker hub:
```docker
docker tag my-express-app cassiopea21/my-express-app:v1.0.0
```   

![docker_tagging_image](./16.tagging%20image.png)

Logging into docker : 
```docker
docker login
```      
![docker_login](./17.docker%20login.png)

Pushing the image to Docker Hub : 
```docker 
docker push cassiopea21/my-express-app:v1.0.0
```   
![docker_image_push](./18.image%20push.png)


On Docker Hub , we can check the image is effectively there :   
![docker_hub_interface](./19.docker-hub-image.png)




