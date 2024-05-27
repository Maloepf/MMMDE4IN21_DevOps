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