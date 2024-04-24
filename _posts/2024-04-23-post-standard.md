---
title: "Deploying a Three-tier Web Application using Docker"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

Hey everyone, in this blog post, we're diving into the world of deploying full-stack web applications using Docker, breaking it down into three distinct microservices:

1. Frontend Container powered by ReactJS
2. Backend Container running NodeJS
3. Database Container utilizing MongoDB


But before we delve into the deployment process, let's take a moment to understand what Docker is and why it's a game-changer.

## Understanding Docker:
Docker is more than just a platform; it's a paradigm shift in how we develop, test, and deploy applications. At its core, Docker utilizes containers—lightweight, self-contained packages of software containing everything an application needs to run. They ensure consistency across diverse environments, making deployment a breeze.

Docker doesn't stop there; it offers a suite of tools for managing containers. One such tool is Docker Compose, which simplifies the orchestration of multi-container applications. In this blog, we'll harness the power of Docker Compose to streamline our full-stack web application deployment. Let's get started.

#### 1. Crafting a Full Stack Application or Utilizing an Existing One based on React.js, Node.js, and MongoDB

I've chosen an application forked from my GitHub, or feel free to leverage any existing application you possess. Just ensure it aligns with the specified tech-stack.

Here's the directory structure of our application:

```bash
keeper-MERN/
└── client/
└── server/
```

#### 2. Establishing a Repository on Docker Hub for Storing Docker Images

Navigate to Docker Hub and establish a new repository to house your Docker images. Depending on your needs, opt for either a public or private repository. After creating the repository, jot down its name as we'll require it later for pushing our Docker images.

Below is an illustration of your repository on Docker Hub:
<br>
![Docker Hub](/assets/images/Screenshot 2024-04-23 235842.png)  
<br>


#### 3. Crafting a Dockerfile for Each Service

A Dockerfile is a textual blueprint containing instructions to build a Docker image. It delineates directives such as the base image, working directory, dependencies, and commands essential for running the application.  
<br>


**3.1 Dockerfile for React.js Server located at client/Dockerfile**

Before creating the Dockerfile for the ReactJs server, make sure you have a build script in your package.json file. If not, add the following script to your package.json file:

```json
"scripts": {
"start": "react-scripts start",
"build": "react-scripts build",
"test": "react-scripts test",
"eject": "react-scripts eject"
}
```
<br>

```Dockerfile
FROM node:16

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npx", "serve", "-s", "build"]
```  
<br>


**Now, let's build the Docker image from the created Dockerfile and push it to the Docker Hub repository. Follow the subsequent steps meticulously:**

Build the Docker image using the following command:
  
  ```bash 
  docker build -t <DockerHubUsername>/<RepositoryName>:client .
  ```  
  <br>
  ![docker build for client](/assets/images/Screenshot 2024-04-24 000056.png)  
  <br>

  Replace ```<DockerHubUsername>``` with your Docker Hub username and ```<RepositoryName>``` with the repository name of the created repository in step 2.   
  <br>
  This command will build the Docker image for the ReactJs server using the Dockerfile in the client folder.

2. Lets check if the image is created successfully by running the following command:
  
  ```bash 
  docker images
  ```  

  This command will list all the Docker images on your system. You should see the Docker image for the ReactJs server in the list.

3. Now we will see if the image is running successfully by running the following command:
  
  ```bash 
  docker run -p 3000:3000 <DockerHubUsername>/<RepositoryName>:client
  ```  
  <br>

  ![docker run](/assets/images/Screenshot 2024-04-24 001449.png)    
  <br>
  ![frontend-browser](/assets/images/Screenshot 2024-04-24 002500.png)  
  
  <br>
  This command will run the Docker image for the ReactJs server on port 3000. You should be able to access the ReactJs server at http://localhost:3000.


4. Push the Docker image to the Docker Hub repository using the following command:
  
  ```bash 
  docker push <DockerHubUsername>/<RepositoryName>:client
  ```  
  <br>

  ![docker frontend](/assets/images/Screenshot 2024-04-24 002913.png)  
  <br>

  This command will push the Docker image to the Docker Hub repository. You can check the Docker Hub repository to see if the image has been pushed successfully.  
  <br>

  ![frontend_push_check](/assets/images/Screenshot 2024-04-24 002924.png)



**3.2 Creating Dockerfile for NodeJs server**  
  
<br>
**This is what your final Dockerfile should look like:**

```Dockerfile
FROM node:16

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
```  
<br>


**Now we will build the Docker image from the created Dockerfile for the server and push it to the Docker Hub repository, follow the below steps carefully**

- Build the Docker image using the following command:
  
```bash 
docker build -t <DockerHubUsername>/<RepositoryName>:server .
```  
<br>
![docker build server](/assets/images/Screenshot 2024-04-24 003116.png)    
<br>

Replace ```<DockerHubUsername>``` with your Docker Hub username and ```<RepositoryName>``` with the repository name of the created repository in step 2.   
<br>
This command will build the Docker image for the Nodejs server using the Dockerfile in your local repository.


**Note**: We will not run the server image as it requires a connection to the MongoDB database. We will run the server image after creating the MongoDB image.

- Push the Docker image to the Docker Hub repository using the following command:
  
```bash 
  docker push <DockerHubUsername>/<RepositoryName>:server
```  
<br>

![docker server push](/assets/images/Screenshot 2024-04-24 003230.png)  
<br>

This command will push the Docker image to the Docker Hub repository. You can check the Docker Hub repository to see if the image has been pushed successfully.  
<br>  

![server_push_check](/assets/images/Screenshot 2024-04-24 003245.png)  
<br>


Now we have our client and server images pushed to the Docker Hub repository and ready to run. We now will pull an already created MongoDB image from the Docker Hub and run it.

- **3.3 Pulling MongoDB image from Docker Hub**

- Pull the MongoDB image from the Docker Hub using the following command:
  
```bash
docker pull mongo:latest
```
![mongo pull](/assets/images/Screenshot 2024-04-24 003507.png)  
<br>


### 4. Run our application with the created docker images

Now that we have all our images ready, we can run our application by running all the docker containers and exposing the ports on which they will run so all three microservices can communicate with each other.
    
Follow the below commands to run your containers:  
<br>

```bash
docker run -dp 3000:3000 <DockerHubUsername>/<RepositoryName>:client
docker run -dp 27017:27017 mongo
```
<br>
Note: we have used -dp tag so that the containers can run in the background without blocking the terminal 

Now that we have created the containers, we need to create a network that we will connect to the server and the mongo container. We will do this so that we can get the IP of the mongo container and use it in the connection string as ````mongoose.connect('mongodb:<CONTAINER_IP>:27017:<DB_NAME>') ```` 

This is how your create a docker network:  
<br>

![create network](/assets/images/Screenshot%202024-04-24%20003942.png)  
<br>

Now you attach this network to the mongodb container:
```` docker network connect <NETWORK_NAME> <CONTAINER_ID>````  
<br>   

![mongo-connect-net](/assets/images/Screenshot%202024-04-24%20004017.png)
<br>

You can get the container IP using  ```` docker inspect <CONTAINER_NAME>````  
<br>

![network inspect](/assets/images/Screenshot%202024-04-24%20004052.png)  
<br>

After this, use this container IP or container ID in the the mongoose connection string and build your server image and run it again. Attach the network to the server container too to allow communication between server and database. Note: You do not have to connect the network to the client container 
````bash
docker run -dp 5000:5000 <DockerHubUsername>/<RepositoryName>:server
````  
<br>

![server-net](/assets/images/Screenshot%202024-04-24%20004429.png)  

**Once all the containers are up and running, you can test the app at** ```` http://localhost:3000 ````

This is how the website will look like:  
<br>

![home](/assets/images/Screenshot%202024-04-24%20072656.png)  
<br>


**And voila! We've successfully deployed our three-tier MERN stack application with Docker.**

#### Conclusion:
By containerizing our MERN stack application with Docker and leveraging Docker networks, we've unlocked a portable, efficient, and scalable development environment. This streamlined approach simplifies collaboration and ensures seamless deployment to production.

But remember, this is just the beginning! There are endless possibilities for further optimization, such as volume management for persistent data storage and integration with container orchestration tools like Kubernetes. As your MERN application evolves, Docker will remain an invaluable asset in your development toolkit.
