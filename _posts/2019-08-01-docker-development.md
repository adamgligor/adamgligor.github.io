---
layout: post
title: My development environment with docker
description: 
date: '2019-08-01'
tags: programming
---

A development environment using docker ...

## Goal

I have a python project consisting of some scripts interacting with websites or apis and a database. I want to create a development environment using docker where i'm able to run multiple versions of python hassle free and have my data locally and debugger support.

## Prerequisites

Docker running on host, Pycharm ide (pro version, because the community does not have built in docker support) and MySqlWorkbench, a free management tool for mysql databases

## Setup

![placeholder](/public/docker_pycharm/docker.jpg "docker setup")

Docker running on macbook (host), python code is running in a docker container, and it can be debugged from the ide (pycharm). Mysql running in another container, and I can manage it using a sql ide running on the host. Both containers can talk to each-other (ie. python code can write to the db)

## Implementation

1. Create a dockerfile for the python project so it can run in docker. A simple one looks like this:


```
FROM  python:3.7-alpine

ENV HOME_DIR /app
WORKDIR $HOME_DIR

COPY . $HOME_DIR

RUN pip install -r requirements.txt

```

2. Build the docker image. Run from the folder containing the dockerfile

```
docker build . -t proj_name
``` 

3. Create a docker-compose file. I found this the easiest way to set up the networking. The example defines two containers `mycode` and `mysql1` and a network `mynet`, additionally exposes mysql management on host port 3307

```
version: '3'
services:

 mycode:
   build: ./
   container_name: mycode
   networks:
     - mynet
 mysql1:
   image: mysql/mysql-server:5.7
   container_name: mysql1
   ports:
     - "3307:3306"
   networks:
     - mynet
networks:
 mynet:
```

4. Instruct pycharm to use the docker-compose file. This will have pycharm control the `mycode` container. That is starting, stopping, attaching the debugger and syncing code from ide to container. Go to: pycharm menu > preferences > project interpreter > settings > add > choose docker-compose and configure it like this:

![placeholder](/public/docker_pycharm/pycharm.png "pycharm setup")

5. Create the mysql container. The `mysql1` has to be created and configured, it's a one time thing.

```
docker run --name=mysql1 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=your_pass -d mysql/mysql-server:5.7
```
TIP: for other parameters and settings look up the official mysql docker image on docker-hub 

6. Configure remote access privileges, this is required for mysqlworkbech.

```
Docker start mysql1
docker exec -it mysql1 bin/bash

mysql -u root -p
GRANT ALL PRIVILEGES ON *.* TO 'root' IDENTIFIED BY 'your_pass' WITH GRANT OPTION;
exit
```
.. not production grade security obviously

7. Adjust docker networking, This adds `mysql1` in the `mynet` network ...

```
# find the network that was created by pycharm (reads mycode_mynet)
docker network ls 

# connect mysql1 to the same network
docker network connect <network_name> mysql1

# confirm that both containers are listed  
docker inspect <network_name>
```

8. Test result 

Now the two container will know each other by name, I can use in python code
```
[database]
default = mysql://root:your_pass@mysql1/mydb?charset=utf8 
```

Running and debugging python code inside the container should work thanks to pycharm magic, and code files are synced to the container behind the scene

Mysqlworbehch on host should be able to connect to mysql ( use the port set in the docker-compose)


## Resources 

What helped me figure out this stuff:

- docker networking [here](https://dev.to/mozartted/docker-networking--how-to-connect-multiple-containers-7fl)
- pycharm docker support [here](https://www.jetbrains.com/help/pycharm/docker.html)

