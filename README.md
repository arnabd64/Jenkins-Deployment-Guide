# Jenkins Deployment Guide

A written guide on how to deploy a Jenkins CI server along with various Agent Nodes using Docker.

# Contents

1. [Deploy Jenkins Server](./pages/standalone-docker.md)
2. [Compose file for Standalone Jenkins](./pages/standalone-docker-compose.md)
3. [Jenkins Agents](./pages/jenkins-agents.md)
4. [Dockerfile Template for Custom Jenkins Agents]()
5. Attaching an Agent to Jenkins Deployment
6. Jenkins Agent with Docker Compose
































## Pull the image from [Docker Hub](https://hub.docker.com/r/jenkins/jenkins)
We will be using the official Jenkins docker image hosted on docker hub.
```bash
$ docker pull jenkins/jenkins:lts-jdk17
```
## Create a Docker bridge network
We will be using a custom bridge network for deploying all the containers, this ensures some level of network isolation from other docker containers.

```bash
$ docker network create --driver=bridge --subnet=172.30.0.0/24 jenkins-network
```
## Allocate Persistant Storage
We will create a docker volume to store all the data generated by Jenkins so that in the case that our Jenkins container goes down, we don't suffer any data loss.
```bash
$ docker volume create jenkins-home
```

## Run the Master Node
This container is then master node, it's job is to control all the agent nodes where the actual building jobs will be executed. 
```bash
$ docker run \
  -itd \
  --rm \
  --name jenkins-master \
  --network jenkins-network \
  -v jenkins-home:/var/jenkins_home \
  -p 8080:8080 \
  -p 50000:50000 \
  jenkins/jenkins:lts-jdk17
```

## Setup Jenkins

1. Go to your browser and access Jenkins by visiting: `http://localhost:8080`. Jenkins will ask you to *unlock* it using a secret key that is emitted in the container logs.
```bash
$ docker logs jenkins-master
```
2. Copy the secret key that looks like `c4ded237084847ba9b3d5a22d9efcf6f` and paste it in the textbox on the Jenkins login screen.
3. Click on Install Suggested Plugins to get started with some of the commonly used plugins.
4. Then fill up a form for creating an admin user.
5. Once signup is complete you will be greeted by Jenkins home screen which indicates that setup is complete.

## Start an Agent Node

I will setup a Python 3.10 node

### Obtain the docker IP of Master

Run the command to get the IP address of the Jenkins Master Container:

```bash
$ docker inspect jenkins-master | grep -w IPAddress
```
> [!TIP]
> If you are following this guide then the IP should look like `172.30.0.2`

### Create a Node on Jenkins Dashboard

1. Go to **Manage Jenkins** section on the Jenkins Dashboard. From there go to the **Nodes** page.
2. Create a **New Node**, give it a name and check the **Permanent Agent** tickbox. I will be naming it *Python 3.10*
3. Add `/home/jenkins` to the **Remote root directory** field and give the node a label. I will be using `python310`
4. Change the usage option to *Only build jobs with label expressions matching this node*
5. Click on **Save** to save the Node. You will be redirected to *Nodes* page.

Now we have provisoned a new node on the Jenkins server, it's time to actually create the Node.

### Build a Docker image for jenkins Agent

Although there are several prebuilt Docker Images for Jenkins Agents available on the Docker hub, I suggest that you build your own custom images for having a stronger hold on your runtime environments.

> [!NOTE]
> We will be using [jenkins/agent](https://hub.docker.com/r/jenkins/agent) image repository for our **Base Image**


