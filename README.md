Voting App Demo by Ajay Kumar
=============================

A simple distributed application running across multiple Docker containers.

Getting started
---------------

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/). 


## Linux Containers

The Linux stack uses Python, Node.js, .NET Core (or optionally Java), with Redis for messaging and Postgres for storage.

> If you're using [Docker Desktop on Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows), you can run the Linux version by [switching to Linux containers](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers), or run the Windows containers version.

Run in this directory:
```
$docker-compose up
```
The app will be running at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001).



The vote interface is then available on port 5000 on each host of the cluster, the result one is available on port 5001.

Architecture
------------

![Architecture diagram](architecture.png)

* A front-end web app in [Python](/vote) or [ASP.NET Core](/vote/dotnet) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) or [NATS](https://hub.docker.com/_/nats/) queue which collects new votes
* A [.NET Core](/worker/src/Worker), [Java](/worker/src/main) or [.NET Core 2.1](/worker/dotnet) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) or [TiDB](https://hub.docker.com/r/dockersamples/tidb/tags/) database backed by a Docker volume
* A [Node.js](/result) or [ASP.NET Core SignalR](/result/dotnet) webapp which shows the results of the voting in real time


Notes
-----

The voting application only accepts one vote per client. It does not register votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple 
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to 
deal with them in Docker at a basic level. 



#Demo voting app deployment in AWS Cloud using docker and docker composed:
**************************************************************************

Requirement:
------------
a)Supported OS
b)Navigate to https://docs.docker.com/install
c)Docker community
d)OS requirement

Requirements:
-------------
-t2.micor EC2 inctnace.
-Configure to connect thorugh Putty/gitbash.
-Install docker on EC2 instance.
-Start the docker service.
-git
-python
-.net framwork
-node js
-Enable ports under SG i.e 5000,5001 and 80.


Login to AWS Account:
======================

-Create a Ubuntu EC2 t2.micro machine ,create a key pair .pem and covert to .ppk file.
-Login to EC2 instance with default user "ubuntu" via putty using private key.

#switch to root user: (sudo command can be use with sudo user )

$sudo su -

#install the lates packages and dependenacies:

$apt update 

or 

$apt-get update

#check git version (Default git will be installed in the system)

$git --version

#Install Docker:

$docker version 

or 

$docker --version

Install docker in ubuntu:
-------------------------------------------------------------------------

Documentation: https://docs.docker.com/install/linux/docker-ce/ubuntu/

Install using the convenience script (login as root):

$curl -fsSL https://get.docker.com -o get-docker.sh
$sh get-docker.sh


#check docker engine/service is running or not with below command:

$service docker status

******************************************************
LAB Practice:
=============
*Demo voting app deploy in docker using docker container and docker compose in AWS Cloud.


#Source code Github repo:
https://github.com/ajaykumarsahoo/voting-app-demo



#Clone the repo from github:

$git clone https://github.com/ajaykumarsahoo/voting-app-demo.git

Running in AWS EC2 instance:

-Expose ports in SG
-5000 as TCP
-5001 as TCP
-80 as  http
-443 as https

#Check OS version:

$cat /etc/os-release


#Deploy the app using docker containers:
=======================================

Example1:

1. create image and run vote app (python).

-build image as it's not available in docker hub:
a)$docker build -t voting-app .

b)$docker images

d)$docker run -d --name=vote -p 5000:80 --link redis:redis voting-app 


#check if app is running inside container:

$curl http://localhost:5000

NOte: If we see html o/p means app is running fine.

2. Run image for redis from docker hub.

c)$docker run -d --name=redis redis

3. RUn image for postgres:9.4 from docker hub.

Note: default need to provide DB username/pwd for postgres.

e) $docker run -d --name=db -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres postgres:9.4


4. Create image and run worker(.net).

build image first (cutome)

h)$docker build -t worker-app .

i)$docker images

j)$docker run -d --name=worker --link redis:redis --link db:db worker-app

5. create image and run result app(node js).

-build image as it's not available in docker hub:

f)$docker build -t result-app .

g)$docker run -d --name=result -p 5001:80 --link db:db result-app


$curl http://localhost:5001

--------------------------------------


#Deploy the app using docker-compose:
=====================================

Example2:

#Using docker compose:

vi docker-compose.yml

version: "3"
services:
 redis:
  image: redis
 db:
  image: postgres:9.4
  environment:
   POSTGRES_USER: postgres
   POSTGRES_PASSWORD: postgres
 voting-app:
  image: voting-app
  ports:
   - 5000:80
 result-app:
  image: result-app
  ports:
   - 5001:80
 worker:
  image: worker-app
  


#Install docker compose on Ubuntu: #
====================================
#Download and install docker-compose:

$curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#Change file permission to executable :

$chmod +x /usr/local/bin/docker-compose

#Check the docker-compose version:

$docker-compose --version 

or

$docker-compose version

#Start the containers:

$docker-compose up   


************************************************************************************************************************************


#Need to login to dockerhub as images are stored in my public repo:

$docker login
un:ajaykumar143
pwd:xxxxxxxxxxxxxx

Push all Images to docker hub:public

Docker hub login:

$docker login
us:ajaykumar143
pwd:xxxxxxxxxxx

$docker images ls

#tag a docker image first and then push:

$docker image tag worker:latest ajaykumar143/example-voting-app:worker
$docker image tag voting-app:latest ajaykumar143/example-voting-app:voting-app
$docker image tag result-app:latest ajaykumar143/example-voting-app:result-app
$docker image tag redis:latest ajaykumar143/example-voting-app:redis
$docker image tag postgres:9.4 ajaykumar143/example-voting-app:postgres

#push image:

$docker image push ajaykumar143/example-voting-app:worker
$docker image push ajaykumar143/example-voting-app:voting-app
$docker image push ajaykumar143/example-voting-app:result-app
$docker image push ajaykumar143/example-voting-app:redis
$docker image push ajaykumar143/example-voting-app:postgres

#Docker pull image from Dockerhub:

https://hub.docker.com/r/ajaykumar143/example-voting-app/tags


$docker pull ajaykumar143/example-voting-app

======================================================================================
