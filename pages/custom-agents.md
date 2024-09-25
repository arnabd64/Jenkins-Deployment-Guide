# How to write a Dockerfile for custom Jenkins Agents

I am going to deploy an agent with the following runtime configuration:

+ __OS__: Debian 12 Bookworm
+ __Python__: 3.10.*
+ __Python Packages__: `fastapi, uvicorn, sqlalchemy>2.0, pydantic>2.8`

# Quick Follow-up

Before we proceed any further, I want to share some details about my Jenkins deployment so that you, the reader and me are on the same page, this ensures that you can understand the following commands much better.

 I have deployed the Jenkins CI server on a docker container named `jenkins-master` which is attached to a docker network named `jenkins-vnet`.
I have used the commands mentioned on [my guide](./standalone-docker.md#deploy-jenkins-server) to deploy the Jenkins server.

# Configure agent on Jenkins server

1. On your Jenkins dashboard, Create a new Node by following: _Manage Jenkins_ > _Nodes_ > _New Node_
2. Let's name the agent as __Python-SQL__ and click the _Permanent Agent_ checkbox.
3. The _Number of executors_ controls the maximum number of parallel jobs the agent is allowed to handle. Change it accoriding to your needs.
4. Set the _Remote root directory_ to `/home/jenkins/agent`, it's the location where the build artifacts will be stored.
5. Assign a label to the agent, I am going with `python-sql`, remember this label will be useful while writing a `Jenkinsfile`. Also change the `Usage` field to _Only build jobs with label expressions matching this node_.
6. Click on Save to finish the process. You will be redirected to the Nodes dashboard where you can see your newly created agent.
7. Click on the agent to access command to connect the jenkins server to the agent. The command looks like this:
```bash
java -jar agent.jar -url http://localhost:8080/ -secret e71a8f85dcbaab621d0ea0dc647b60b7c496b441fd97c88d12cfa9a4f42049c4 -name python3 -workDir "/home/jenkins/agent"
```
8. Note down the values of the parameters: `secret` and `name`, it will be needed later on.

# Build Docker image for custom agent

> [!IMPORTANT]
> 
> For an agent to function properly, these packages must be installed during runtime:
> 
> * `curl`, `wget` and `git` to download source code and/or other artifacts.
> 
> * `openjdk-17-jre-headless`, a Java runtime package that enables the Jenkins server to control the agent.

1. Create a working directory on your machine where you can store the Dockerfiles for your custom agents. I have created mine at `~/jenkinsAgents`
2. Create subdirectory for individual custom agents. I created a folder `python-310` and move into the directory.
3. In a file named, `packages.txt` add the packages/libraries to be installed by the OS package manager. In our case:
```text
git
wget
curl
openjdk-17-jre-headless
```
4. For installing python libraries, create a `requirements.txt` file:
```text
fastapi
uvicorn[standard]
sqlalchemy>=2.0
pydantic>=2.8
```
5. Download `agent.jar` which connects the agent with the master server. This file can be downloaded from the Jenkins master server by visiting (http://localhost:8080/jnlpJars/agent.jar). or send a GET request:

```bash
curl -o agent.jar http://localhost:8080/jnlpJars/agent.jar
```

6. Write the following `Dockerfile`.

```Dockerfile
# pull base image
FROM python:3.10-slim-bookworm


# install os and python packages
COPY requirements.txt packages.txt /deps/
RUN apt-get update && \
	xargs -a /deps/packages.txt apt-get install -y --no-install-recommends && \
	apt clean && \
	python3 -m pip install -r /deps/requirements.txt --progress-bar off && \
	python3 -m pip cache purge

# create user: jenkins
ENV USER=jenkins
ENV HOME=/home/$USER
ENV PATH=$HOME/.local/bin:$PATH
RUN useradd -m $USER
USER ${USER}
WORKDIR ${HOME}

# copy the agent.jar file
COPY --chown=${USER} agent.jar .

ENTRYPOINT [ "java", "-jar", "agent.jar", "-workDir=/home/jenkins/agent" ]
```

5. Build the image for the custom agent using the following command:
```bash
docker build -t agent-python:3.10 .
```
6. Run the container:
```bash
docker run -d --rm --name=python310 --network=jenkins-vnet agent-python:3.10 -url http://jenkins-master:8080 -secret=e71a8f85dcbaab621d0ea0dc647b60b7c496b441fd97c88d12cfa9a4f42049c4 -name=python
```
7. Check the Nodes dashboard to check if connection was successful. If any error occurs check the logs by running `docker logs python310`

# Sharing my thoughts

In this guide, I have shared my experience on how to create a custom Jenkins agent running on python but the above method can be used to build agents for any runtime as long as you understand how to setup that runtime automatically in the first place.

Here are some tips:

* What every base operating system you use (eg, Debian, Ubuntu, Alpine etc.), make sure to install `openjdk-*-jre-headless` replace the `*` with an appropriate version of `openjdk`. Without the Java runtime you cannot run the `agent.jar`.
* Install simple packages like `git`, `wget` and `curl` otherwise after deploying the agent you can face issues when running a pipeline due to mising dependencies.
* Choose the Docker base image wisely. In this guide, I wanted an agent running Python 3.10 on top of Debian 12 Bookworm so I went with docker image `python:3.10-slim-bookworm`.
* During runtime make sure to attach the agent container to the same docker network as the jenkins master otherwise connection will not be possible.

> [!TIP]
> This completes the deployment process. [Click Here](../README.md#contents) to go back to Homepage.