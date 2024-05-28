TP 1 : Discover Docker

Goals

Target application
3-tiers application:

HTTP server
Backend API
Database
For each of those applications, we will follow the same process: choose the appropriate docker base image, create and configure this image, put our application specifics inside and at some point have it running. Our final goal is to have a 3-tier web API running.

Base images

HTTP server
Backend API
Database

DataBase
Basics

We will use the image: postgres:14.1-alpine.

Let’s have a simple postgres server running, here is what would be a minimal Dockerfile:
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

```
Build this image and start a container properly, you should be able to access your database depending on the port binding you choose: localhost:PORT.

```
docker build -t  my_db:v1.0 .
```
```
docker run --name my_db -p 8081:80 -d my_db:v1.0
```
Your Postgres DB should be up and running. Connect to your database and check that everything is running smoothly.
Don’t forget to name your docker image and container.

http://localhost:8081/

Re-run your database and adminer with --network app-network to enable adminer/database communication. We use -–network instead of -–link because the latter is deprecated (the previous container need to be stoped and removed).
```
docker network create app-network
docker run --name adminer --network app-network -p 8080:8080 -d adminer
docker run --name my_db --network app-network -p 8081:80 -d my_db:v1.0
```

Init database

It would be nice to have our database structure initialized with the docker image as well as some initial data. Any sql scripts found in /docker-entrypoint-initdb.d will be executed in alphabetical order.

Rebuild your image and check that your scripts have been executed at startup and that the data is present in your container.

Persist data

You may have noticed that if your database container gets destroyed then all your data is reset, a database must persist data durably. Use volumes to persist data on the host disk.

```
docker run --name my_db -v .:/var/lib/postgresql/data --network app-network -p 8081:80 -d my_db:v2.0
```
Backend API

Basics

For starters, we will simply run a Java hello-world class in our containers, only after will we be running a jar. In both cases, choose the proper image keeping in mind that we only need a Java runtime.

Here is a complex Java Hello World implementation:

Main.java

```
public class Main {

   public static void main(String[] args) {
       System.out.println("Hello World!");
   }
}
```

1- Compile with your target Java: javac Main.java.
2- Write dockerfile.

```
FROM openjdk:17-oracle
COPY Main.class /app/
WORKDIR /app

CMD ["java","Main"]
```
3- Now, to launch app you have to do the same thing that Basic step 1.

```
docker run --name java-hello-world -p 8082:80  java
-hello-world:v1.0
```

Multistage build
In the previous section we were building Java code on our machine to have it running on a docker container. Wouldn’t it be great to have Docker handle the build as well? You probably noticed that the default openjdk docker images contain... Well... a JDK! Create a multistage build using the Multistage.

Your Dockerfile should look like this:

```
# Build
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```

1-2 Why do we need a multistage build? And explain each step of this dockerfile.

We are using 2 different technologies, 17-jdk-alpine is just to use javac, then the resul is used with 17-jre-alpine to use the use the runtime technologie.

Check
```
docker run --name simpleapi -p 8083:8080 -d maven:v1.0
```
``` (json)
{
    "id": 2,
    "content": "Hello, World!"
}
```

Backend API

Let’s now build and run the backend API connected to the database. You can get the zipped source code here: simple-api.

Http server

Basics

Choose an appropriate base image

Create a simple landing page: index.html and put it inside your container.
```
FROM httpd:2.4
COPY ./index.html /usr/local/apache2/htdocs/
```

It should be enough to now, start your container and check that eveything is working as expected.

Here are commands that you may want to try to do so:

- docker stats
- docker inspect
- docker logs

Configuration

You are using the default apache configuration, and it will be enough for now, you use yours by copying it in your image.

Use docker exec to retrieve this default configuration from your running container /usr/local/apache2/conf/httpd.conf .


Reverse proxy
We will configure the http server as a simple reverse proxy server in front of our application, this server could be used to deliver a front-end application, to configure SSL or to handle load balancing.

So this can be quite useful even though in our case we will keep things simple.

Link application
Docker-compose
1- Install docker-compose if the docker compose command does not work .

You may have noticed that this can be quite painful to orchestrate manually the start, stop and rebuild of our containers. Thankfully, a useful tool called docker-compose comes in handy in those situations.

2- Let’s create a docker-compose.yml file with the following structure to define and drive our containers:


1-3 Document docker-compose most important commands.

Services : all the containers that are going to be needed
    name of the service (container)
        build: path to the dockerfile
        networks: name of the network to connect to 
        depends: dependancy on db, services,...
        ports: wich port to connect to for external communication

```
version: '3.7'

services:
    backend: #definition of the backend container
        container_name: "backend" #fixing the name of the container
        build: #instructions to follow to build it
          context: "C:\\Users\\malol\\OneDrive - Fondation EPF\\EPF\\4A\\S2\\DevOps\\MMMDE4IN21_DevOps\\Lab1\\API\\simple-api-student-main"
        networks: #wich to connect to
          - app-network
        depends_on: 
          - database

    database: #definition of the backend container
        container_name: "database"
        build: 
          context: "C:\\Users\\malol\\OneDrive - Fondation EPF\\EPF\\4A\\S2\\DevOps\\MMMDE4IN21_DevOps\\Lab1\\DataBase"
        networks:
          - app-network

    httpd: #instructions to follow to build it
        build:
          context: "C:\\Users\\malol\\OneDrive - Fondation EPF\\EPF\\4A\\S2\\DevOps\\MMMDE4IN21_DevOps\\Lab1\\Http server"
        ports: #ports of the container to discuss with 
          - "8080:80"
        networks:
          - app-network
        depends_on:
          - backend

1-4 Document your docker-compose file

networks: #definition of the network
    app-network:
```


Discover Github Action

Target Application
Complete pipeline workflow for testing and delivering your software application.

We are going to use different useful tools to build your application, test it automatically, and check the code quality at the same time.

Setup GitHub Actions
The first tool we are going to use is GitHub Actions. GitHub Actions is an online service that allows you to build pipelines to test your application. Keep in mind that GitHub Actions is not the only one on the market to build integration pipelines.

Historically many companies were using Jenkins (and still a lot continue to do it), it is way less accessible than GitHub Actions but much more configurable. You will also hear about Gitlab CI and Bitbucket Pipelines during your work life.

2-1 What are testcontainers?
Testcontainers is a Java library that provides lightweight, disposable containers for integration testing. It allows developers to easily spin up and manage containers, such as databases, message brokers, or other services, within their test environments, providing a consistent and isolated testing environment without the need for external dependencies.