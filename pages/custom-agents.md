# How to write a Dockerfile for custom Jenkins Agents

I am going to deploy an agent with the following runtime configuration:

+ __OS__: Debian 12 Bookworm
+ __Python__: 3.10.*
+ __Python Packages__: `fastapi, uvicorn, sqlalchemy>2.0, pydantic>2.8`

# Quick Followup

Before we proceed any further, I want to share some details about my Jenkins deployment so that you, the reader and me are on the same page, this ensures that you can understand the following commands much better.

 I have deployed the Jenkins CI server on a docker container named `jenkins-master` which is attached to a docker network named `jenkins-vnet`.
I have used the commands mentioned on [my guide](./standalone-docker.md#deploy-jenkins-server) to deploy the Jenkins server.

# Setup

1. Create a working directory on your machine where you can store the Dockerfiles for your custom agents. I have created mine at `~/jenkinsAgents`

2. Create subdirectory for individual custom agents. I created a folder `python-310-sql` and move into the directory.

3. Download `agent.jar` which connects the agent with the master server. This file can be downloaded from the jenkins master server by visiting (http://localhost:8080/jnlpJars/agent.jar). or send a GET request:

```bash
$ curl -o agent.jar http://localhost:8080/jnlpJars/agent.jar
```

3. In this step we create a file that stores the packages and thier versions needed to build the environment we need. Since I am building a python runtime, I'll be creating a `requirements.txt` file for all python packages and a `packages.apt` to store all the packages to be installed via the `apt` package manager.

4. Write the following `Dockerfile`.

```Dockerfile
FROM python:3.10-slim-bookworm

COPY . .

RUN apt update && \
    # install apt packages
    xargs -a packages.apt apt install -y --no-install-recommends && \
    apt clean && \
    # install python packages
    pip install --progress-bar off -r requirements.txt && \
    pip cache purge

RUN useradd -m jenkins -s /bin/bash
ENV USER=jenkins
ENV HOME=/home/${USER}
ENV PATH=${HOME}/.local/bin:${PATH}
ENV JENKINS_HOME=${HOME}/agent
USER ${USER}
WORKDIR ${HOME}

ENTRYPOINT [ "java", "-jar", "${HOME}/agent.jar" ]
```

