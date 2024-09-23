# Dockerfile Template for Custom Jenkins Agents

Jenkins agents are recommended to be deployed using docker. There are several agents available on [Jenkins | Docker Hub](https://hub.docker.com/u/jenkins) under the repository names: `jenkins/jnlp-agent-*`. These preconfigured agents are good for a lot of worlflows but sometimes we need an agent that has a very specific runtime packages, for example we might need an python agent that runs `python-3.10` specifically. In such scenarios we must build and attach a custom Jenkins agent.

Before proceeding, lets look at how to attach a pre-configured agent from Docker Hub to the master node.

## Configure agent on Master Node

> [!NOTE]
> I will be using the [__Python3__](https://hub.docker.com/r/jenkins/jnlp-agent-python3) agent which comes with the latest version of Python _(3.12)_.

1. Login to Jenkins Server
2. Create a new Node, _Manage Jenkins_ > _Nodes_ > _New Node_
3. Now you will have to fill up the details for setting up the Node.
4. I will name the Node _python3_ and select it as a __Permanent Agent__.
5. Change the _Number of executors_ according to your needs, it controls the number of parallel jobs the node is allowed to handle.
6. Change the _Remote root directory_ to `/home/jenkins/agent` and give it a label _python3_.

> [!TIP]
> The label will be used when building Jenkins pipelines

7. Change the _Usage_ to _Only build jobs with label expressions matching this node_. 
8. All necessary settings have been changed and now create the Node.
9. In the _Nodes_ page, you will see you newly created node but the system metrics will show `N/A` indicating that the master cannot connect with the agent.

> [!IMPORTANT]
> Click on the newly created node to get the following command which Jenkins recommends to run an agent.
> 
> ```java -jar agent.jar -url http://localhost:8080/ -secret 75781d5f63eee7fc5c88fc7ab38999a91da9f33b05df5fb6a78e5ba2a38ceea5 -name python3 -workDir "/home/jenkins"```
>
> Please note down the 2 parameters, `secret` and `name` which will be needed later on.

## Launch Agent

1. Pull the docker image for the python agent. ([Docker Repository](https://hub.docker.com/r/jenkins/jnlp-agent-python3))
```bash
$ docker pull jenkins/jnlp-agent-python3
```
2. Run the following docker command:
```bash
$ docker run -d --rm --name=python3 --network=<jenkins-network> --init jenkins/jnlp-agent-python3 -url http://jenkins-server:port -workDir=/home/jenkins <secret> <agent name>
```
> [!TIP]
> Replace:
> 1. `<jenkins-network>` with the name of docker network on which jenkins master node is hosted.
> 2. `http://jenkins-server:port` with `http://jenkins-master:8080`
> 3. `<secret>` and `<name>` with the values of the parameters that you have noted down earlier.
> The final command for me turns out to be:
> ```bash docker run -d --rm --name=python3 --network=jenkins-network --init jenkins/jnlp-agent-python3 -url http://jenkins-master:8080 -workDir=/home/jenkins 75781d5f63eee7fc5c88fc7ab38999a91da9f33b05df5fb6a78e5ba2a38ceea5 python3```

3. Refresh the _Nodes_ page, you should see system metrics for the agent if the launch was successful, otherwise run `docker logs python3` to check the logs.
