# TP 1: Discover Docker

## Goals

3-tiers application:

- HTTP server
- Backend API
- Database

For each of those applications, we will follow the same process: choose the appropriate docker base image, create and configure this image, put our application specifics inside and at some point have it running. Our final goal is to have a 3-tier web API running.

### Base images

- [HTTP server](https://hub.docker.com/_/httpd)
- [Backend API](https://hub.docker.com/_/openjdk)
- [Database](https://hub.docker.com/_/postgres)

## DataBase
### Basics

We will use the image: postgres:14.1-alpine.

Let’s have a simple postgres server running.
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

```
Build this image and start a container properly.

```
docker build -t  my_db:v1.0 .
```
```
docker run --name my_db -p 8081:80 -d my_db:v1.0
```
http://localhost:8081/

Re-run the database and adminer with --network app-network to enable adminer/database communication. We use -–network instead of -–link because the latter is deprecated (the previous container need to be stoped and removed).
```
docker network create app-network
docker run --name adminer --network app-network -p 8080:8080 -d adminer
docker run --name my_db --network app-network -p 8081:80 -d my_db:v1.0
```

### Init database

It would be nice to have a [database](./DataBase/) structure initialized with the docker image as well as some initial data.

### Persist data

Let's use [volumes](/Lab1/volume_my_db/) to persist data on the host disk. Here the colume is set in the same file as Dockerfile.

```
docker run --name my_db -v .:/var/lib/postgresql/data --network app-network -p 8081:80 -d my_db:v2.0
```

1-1 Document your database container essentials: commands and [Dockerfile](/DataBase/Dockerfile).


## Backend API

### Basics

For starters, we will simply run a [Java hello-world](/API/) class in our containers, only after will we be running a jar.

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

## Multistage build


1-2 Why do we need a multistage build? And explain each step of this dockerfile.

We are using 2 different technologies, 17-jdk-alpine is just to use javac, then the resul is used with 17-jre-alpine to use the use the runtime technologie.


```
docker run --name simpleapi -p 8083:8080 -d maven:v1.0
```
```
{
    "id": 2,
    "content": "Hello, World!"
}
```

### Backend API

Let’s now build and run the backend API connected to the database. You can get the zipped source code here: simple-api.

## Http server

### Basics

Choose an appropriate base image.

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

### Configuration

You are using the default apache configuration, and it will be enough for now, you use yours by copying it in your image.

Use docker exec to retrieve this default configuration from your running container /usr/local/apache2/conf/httpd.conf.

See [Dockerfile](/Http%20server/Dockerfile).

### Reverse proxy
Let's configure the http server as a simple reverse proxy server in front of our application, this server could be used to deliver a front-end application, to configure SSL or to handle load balancing.

See [httpd.conf](/Http%20server/httpd.conf)

## Link application
### Docker-compose

It is quite painful to orchestrate manually the start, stop and rebuild of our containers. Thankfully, a useful tool called docker-compose comes in handy in those situations.


1-3 Document docker-compose most important commands.

Services : *all the containers that are going to be needed*</br>
&emsp;name *(of the service - container)*</br>
&emsp;&emsp;build: *path to the dockerfile*</br>
&emsp;&emsp;networks: *name of the network to connect to* </br>
&emsp;&emsp;depends: *dependancy on db, services,...*</br>
&emsp;&emsp;ports: *wich port to connect to for external communication*</br>


1-4 Document your docker-compose file

See [docker_compose.yml](/Link_App/docker_compose.yml).


# TP 2: Discover Github Action

## Target Application
Complete pipeline workflow for testing and delivering your software application.


## Setup GitHub Actions
The first tool we are going to use is [GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions). GitHub Actions is an online service that allows you to build pipelines to test your application. Keep in mind that GitHub Actions is not the only one on the market to build integration pipelines.

Historically many companies were using Jenkins (and still a lot continue to do it), it is way less accessible than GitHub Actions but much more configurable. You will also hear about Gitlab CI and Bitbucket Pipelines during your work life.

## Build and test your Application

2-1 What are testcontainers?</br>
Testcontainers is a Java library that provides lightweight, disposable containers for integration testing. It allows developers to easily spin up and manage containers, such as databases, message brokers, or other services, within their test environments, providing a consistent and isolated testing environment without the need for external dependencies.They simply are java libraries that allow you to run a bunch of docker containers while testing. Here we use the postgresql container to attach to our application while testing. 

2-2 Document your Github Actions configurations.</br>
See .github/workflows/main.yml.

## Setup Quality Gate

### What is quality about?
Quality is here to make sure your code will be maintainable and determine every unsecured block. It helps you produce better and tested features, and it will also prevent having dirty code pushed inside your main branch.

For this purpose, we are going to use SonarCloud, a cloud solution that makes analysis and reports of your code. This is a useful tool that everyone should use in order to learn java best practices.

# SonarCloud GitHub Actions Workflow

This document provides the setup and configuration for a GitHub Actions workflow to build and analyze your project using SonarCloud.

## Workflow Configuration

The workflow is defined in a YAML file as follows:

```yaml
name: SonarCloud

on:
  push:
    branches:
      - master
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  build:
    name: Build and analyze
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Ensures full history is fetched for accurate analysis

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: 'zulu'  # You can use other distributions like 'adopt', 'oracle', etc.

      - name: Cache SonarCloud packages
        uses: actions/cache@v3
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar  # Allows cache restoration to speed up the workflow

      - name: Cache Maven packages
        uses: actions/cache@v3
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2  # Uses the hash of the pom.xml file for cache key

      - name: Build and analyze
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Required to access GitHub API
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}  # SonarCloud token for authentication
        run: mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=devopsmalolepf_malolepf
          # Runs the Maven build and SonarCloud analysis
```

# TP 3: Discover Ansible

Inventories
By default, Ansible's inventory is saved in the location /etc/ansible/hosts where you already defined your server.

The headings between brackets (eg: [webservers]) are used to group sets of hosts together, they are called, surprisingly, groups. You could regroup them by roles like database servers, front-ends, reverse proxies, build servers…

Let’s create a project specific inventory, in your project create an ansible directory, then create a new directory called inventories and in this folder a new file (my-project/ansible/inventories/setup.yml).

Test your inventory with the ping command:

```
ansible all -i inventories/setup.yml -m ping
```
```
malo.lajous.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
Facts
Let’s get information about hosts: these kinds of variables, not set by the user but discovered are called facts.

Facts are prefixed by ansible_ and represent information derived from speaking with your remote systems.

You will request your server to get your OS distribution, thanks to the setup module.

```
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```
```
malo.lajous.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "7",
        "ansible_distribution_release": "Core",
        "ansible_distribution_version": "7.9",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```
Earlier you installed Apache httpd server on your machine, let’s remove it:
```
ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
```
```
malo.lajous.takima.cloud | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "changes": {
        "removed": [
            "httpd"
        ]
    },
    "msg": "",
    "rc": 0,
    "results": [
        "Loaded plugins: fastestmirror\nResolving Dependencies\n--> Running transaction check\n---> Package httpd.x86_64 0:2.4.6-99.el7.centos.1 will be erased\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package      Arch          Version                       Repository       Size\n================================================================================\nRemoving:\n httpd        x86_64        2.4.6-99.el7.centos.1         @updates        9.4 M\n\nTransaction Summary\n================================================================================\nRemove  1 Package\n\nInstalled size: 9.4 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Erasing    : httpd-2.4.6-99.el7.centos.1.x86_64                           1/1 \n  Verifying  : httpd-2.4.6-99.el7.centos.1.x86_64                           1/1 \n\nRemoved:\n  httpd.x86_64 0:2.4.6-99.el7.centos.1                                          \n\nComplete!\n"
    ]
}
```

3-1 Document your inventory and base commands

Playbooks
First playbook
Let’s create a first very simple playbook in my-project/ansible/playbook.yml.
```
ansible-playbook -i inventories/setup.yml playbook.yml
```
```
PLAY [all] ***************************************************************************************************************************************************

TASK [Test connection] ***************************************************************************************************************************************
ok: [malo.lajous.takima.cloud]

PLAY RECAP ***************************************************************************************************************************************************
malo.lajous.takima.cloud   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Advanced Playbook

Let’s create a playbook to install docker on your server.

Good news, we now have docker installed on our server. One task was created to be sure docker was running, you could check this with an ad-hoc command or by connecting to the server until you really trust ansible.

Using roles
Our docker install playbook is nice and all but it will be cleaner to have in a specific place, in a role for example. Create a docker role and move the installation task there:

3-2 Document your playbook


Deploy your App
Time has come to deploy your application to your Ansible managed server.

Create specific roles for each part of your application and use the Ansible module: docker_container to start your dockerized application. Here is what a docker_container task should look like: