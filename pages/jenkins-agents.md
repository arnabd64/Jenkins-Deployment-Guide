# Jenkins Agent

> [!IMPORTANT]
> The __Jenkins CI Server__ or __Master Node__ holds all configuration and acts like a control server that orchestrates all workflow defined in the pipelines.

A Jenkins Agent is a separate entity which is detached from the __master node__, which can be a _virtual machine_ a _Docker container_, or a _cloud instance_ that connects to the Jenkins master to execute tasks. This flexibility allows Jenkins to effectively distribute workloads and scale as needed, accommodating various environments and tools. When a build job is triggered, the master node delegates the job to the designated agent, which can operate across different runtime environments.


# References

1. [Jenkins Agents: The Ultimate Guide to Scaling Your CI/CD Pipeline | Medium](https://medium.com/@sayalishewale12/jenkins-agents-the-ultimate-guide-to-scaling-your-ci-cd-pipeline-427d94dc1302)
2. [Getting started with Jenkins: Agents](https://benmatselby.dev/post/jenkins-basic-agent/)
3. [Jenkins Agents: Distributed Builds Explained](https://reintech.io/blog/jenkins-agents-distributed-builds-guide)
4. [Difference Between Jenkins Agent And Node | Geeks for Geeks](https://www.geeksforgeeks.org/difference-between-jenkins-agent-and-node/)