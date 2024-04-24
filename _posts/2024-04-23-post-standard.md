---
title: "Three-Tier Web Application Deployment with Docker"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

Hey everyone, in this blog post, we're diving into the world of deploying full-stack web applications using Docker, breaking it down into three distinct microservices:

Frontend Container powered by ReactJS
Backend Container running NodeJS
Database Container utilizing MongoDB
But before we delve into the deployment process, let's take a moment to understand what Docker is and why it's a game-changer.

Understanding Docker:
Docker is more than just a platform; it's a paradigm shift in how we develop, test, and deploy applications. At its core, Docker utilizes containers—lightweight, self-contained packages of software containing everything an application needs to run. They ensure consistency across diverse environments, making deployment a breeze.

Docker doesn't stop there; it offers a suite of tools for managing containers. One such tool is Docker Compose, which simplifies the orchestration of multi-container applications. In this blog, we'll harness the power of Docker Compose to streamline our full-stack web application deployment. Let's get started.

1. Building a Full Stack Application:
You have two options here: either develop a new full-stack application or utilize an existing one built on ReactJS, NodeJS, and MongoDB. For demonstration purposes, I've chosen my "Stock Suggestion App," available for cloning here. Ensure your chosen application adheres to the same tech stack.

Here's the directory structure of our application:

bash
Copy code
GFG-HACK/
└── client/
└── server/
2. Setting Up a Docker Hub Repository:
Head over to Docker Hub and create a repository to store your Docker images. Depending on your needs, you can opt for a public or private repository. Once created, take note of the repository name for future reference.

3. Crafting Dockerfiles for Each Service:
3.1 Dockerfile for ReactJS Frontend:

Before creating the Dockerfile, ensure your package.json file includes a build script. If not, add the following script:

json
Copy code
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject"
}
Dockerfile:

Dockerfile
Copy code
FROM node:16

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npx", "serve", "-s", "build"]
3.2 Dockerfile for NodeJS Backend:

Dockerfile:

Dockerfile
Copy code
FROM node:16

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
Building and Pushing Docker Images:

bash
Copy code
docker build -t <DockerHubUsername>/<RepositoryName>:client .
docker build -t <DockerHubUsername>/<RepositoryName>:server .
Pulling MongoDB Image:

bash
Copy code
docker pull mongo:latest
4. Running Our Application with Docker Images:
With our images ready, it's time to fire up our application by running the Docker containers and exposing the necessary ports for seamless communication between microservices.

Running Containers:

bash
Copy code
docker run -dp 3000:3000 <DockerHubUsername>/<RepositoryName>:client
docker run -dp 27017:27017 mongo
Note: We're using the -d flag to ensure containers run in the background without blocking the terminal.

Next, we create a network to connect the server and MongoDB containers, facilitating communication between them.

Creating a Docker Network:

bash
Copy code
docker network create <NETWORK_NAME>
docker network connect <NETWORK_NAME> <MONGODB_CONTAINER_ID>
docker network connect <NETWORK_NAME> <NODEJS_CONTAINER_ID>
Finally, we attach the network to the server container, enabling communication between the server and database. There's no need to connect the network to the client container.

Attaching Network to Server Container:

bash
Copy code
docker run -dp 5000:5000 --network <NETWORK_NAME> <DockerHubUsername>/<RepositoryName>:server
Testing the Application:
Visit http://localhost:3000 to interact with the deployed application.

And voila! We've successfully deployed our three-tier MERN stack application with Docker.

Conclusion:
By containerizing our MERN stack application with Docker and leveraging Docker networks, we've unlocked a portable, efficient, and scalable development environment. This streamlined approach simplifies collaboration and ensures seamless deployment to production.

But remember, this is just the beginning! There are endless possibilities for further optimization, such as volume management for persistent data storage and integration with container orchestration tools like Kubernetes. As your MERN application evolves, Docker will remain an invaluable asset in your development toolkit.