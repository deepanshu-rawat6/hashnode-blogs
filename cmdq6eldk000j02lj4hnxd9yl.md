---
title: "Mastering Jenkins Cost Efficiency: Scalable CI/CD with AWS ECS Master-Agent Architecture"
seoTitle: "Cut Jenkins Costs 70% with AWS ECS CI/CD Agents"
seoDescription: "Learn how to scale Jenkins efficiently using AWS ECS master-agent setup, cutting costs and boosting CI/CD performance with dynamic agents."
datePublished: Wed Jul 30 2025 16:24:19 GMT+0000 (Coordinated Universal Time)
cuid: cmdq6eldk000j02lj4hnxd9yl
slug: mastering-jenkins-cost-efficiency-scalable-cicd-with-aws-ecs-master-agent-architecture
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1744542855284/4883bd51-bdcf-432e-bbc7-c58e1fe3a1d0.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1753892219481/458832bf-97d3-4ef5-9d45-6dfafd0385e3.png
tags: cloud, aws, devops, jenkins, finops

---

So, as we know `Jenkins` is one of the most battle-tested automation tool in the market. But, it comes with an overhead of managing the server. Sometimes, in order to run our automation and CI/CD workloads, we might end-up choosing a bigger server, which might look ideal for an shorter analysis, but would cost a hefty bill annually.

How to avoid such costs? How to improve?

Enter……drum…roll…please……`Master-Slave` architecture, but wait we need to discuss the caveats of `traditional` Jenkins setup.

---

# The Traditional Jenkins Trap: Why Your Current Setup Bleeds Money

Traditional Jenkins deployments typically involve a single, powerful server that handles all build processes. This approach has several drawbacks:

* Over-provisioning: Teams often choose larger servers to handle peak loads, leading to underutilized resources during off-peak hours
    
* Resource waste: A single server running 24/7 consumes resources even when no builds are running
    
* Scalability limitations: Scaling requires manual intervention and often involves downtime
    
* Cost inefficiency: Fixed costs regardless of actual usage patterns
    

---

# Solution: Master-Slave Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753614912121/bc163038-474a-447e-afd9-ba274fad78b3.png align="center")

The Master-Slave architecture addresses these challenges by separating the Jenkins master (controller) from the build execution environment (agents). For this we would be using AWS ECS for running our Jenkins agents and following that we can right size our master Jenkins instance to maybe a lower config, just for scheduling our workloads. Our agents will be utilizing the jenkins’ `inbound agent` image, which would run on ECS Fargate and EC2 based ECS. This approach offers:

* **Dynamic scaling**: Agents can be provisioned on-demand and terminated when not needed
    
* **Cost optimization**: Pay only for the compute resources you actually use
    
* **Workload distribution**: Different types of builds can use appropriately sized agents
    
* **Improved reliability**: Master server remains lightweight and stable
    
* **Infrastructure as Code**: Reproducible, version-controlled infrastructure
    

---

# Impact: What We Actually Did (and How You Can Too!)

So, after implementing our Master-Slave architecture with ECS across multiple environments, and surprisingly we were able to achieve **60-70% cost reduction + way better scalability (and yes, these numbers can be true based on the fine tuning)**  
  
I’ll cover how we were able to drive these savings and the actual ROI analysis in the next blog.  

---

# Prerequisites for ECS Fargate and EC2 Launch Types

We're implementing a hybrid approach using AWS ECS with Infrastructure as Code principles, featuring two distinct agent types:

* **AWS ECS Cluster:** Set up both Fargate and EC2 capacity types.
    
* **Terraform:** Automated, reproducible, version-controlled infrastructure.
    
* **Custom Docker images:** Tailor your Jenkins agent with only the tools you need.
    
* **AWS ECS Plugin v1.49 for Jenkins:** Enables seamless integration.
    
* **Equal focus on security groups, IAM roles, and networking.** Don’t skip this; 90% of issues are misconfigurations here!
    

## Choosing the Right Jenkins Agents on AWS ECS: Fargate, EC2, and Fargate Spot Explained

### 1\. **ECS Fargate Agents**

* **Best for**: Moderate tasks and builds that don't require Docker daemon access
    
* **Benefits**: Serverless, no infrastructure management, cost-effective for standard workloads
    
* **Use cases**: Code compilation, testing, lightweight deployments
    

### 2\. **ECS EC2 Launch Type Agents**

* **Best for**: High-compute tasks and jobs requiring Docker daemon access
    
* **Benefits**: Full control over the underlying infrastructure, support for privileged operations
    
* **Use cases**: Docker-in-Docker (DinD) builds, resource-intensive compilations, complex deployments
    

### 3\. **ECS Fargate Spot Agents**

* **Best for**: Non-critical builds and cost optimization
    
* **Benefits**: Up to 50% cost savings compared to regular Fargate
    
* **Use cases**: Development builds, testing, non-time-critical deployments
    

## Complete Infrastructure as Code Implementation

For the complete IaC setup follow the steps provided in the [BLOG.md](https://github.com/deepanshu-rawat6/Jenkins-Master-Slave-infra-tf/blob/master/BLOG.md), also the source code for all terraform files in present under this repository, click [here](https://github.com/deepanshu-rawat6/Jenkins-Master-Slave-infra-tf).

---

## Job-Specific Docker Images

You can craft your own Docker Images for the Jenkins agents based on the type of dependencies in the Jenkins jobs or if you want a minimal image to start with choose the `inbound-agent:latest` or alpine variants of `inbound-agent` image.

### General purpose dockerfile

```dockerfile
ARG DOCKER_DEFAULT_PLATFORM=linux/amd64
ARG JDK_VERSION=jdk21
ARG JAVA_VERSION=21
ARG ALPINE_VERSION=3.22

FROM jenkins/inbound-agent:alpine${ALPINE_VERSION}-${JDK_VERSION}

USER root

# Update and install dependencies
RUN apk update && apk add --no-cache \
    <add in the packages required for the jobs>
    aws-cli \
    docker-cli

# Add in your dependencies required for running Jenkins jobs

USER jenkins

ENTRYPOINT ["/usr/local/bin/jenkins-agent"]
```

Build these docker image and push them into a private ECR.

---

# Setup: ECS Fargate as Jenkins agents

I hope, you’ve implemented the whole infra for following the steps provided in the [BLOG.md](https://github.com/deepanshu-rawat6/Jenkins-Master-Slave-infra-tf/blob/master/BLOG.md)

So….before setting up the agents, we need to install the most important plugin, which would allow us to harness ECS tasks and that is the [**ECS Fargate Plugin**](https://plugins.jenkins.io/amazon-ecs/). We’d be choosing  1.49 version of this plugin.  
  
Then, we need to allow a port for the inbound agents aka Jenkins to ping the Master node, so for that go to `Manage Jenkins > Security`, scroll down for a bit and find `Agents`, you can allowing any port you want, make sure you’re whitelisting it in the security group of the Master Jenkins instance, and in the docker-compose file for our Master Jenkins container.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753805551849/b3282536-f5f7-4a7f-aee9-4cf8bdab452d.png align="center")

Moving on, using ECS Fargate as Jenkins agents, require a few steps in `AWS Console` and `Jenkins`. I’ve broken down into hierarchical steps.

* **Step 1: Configuration of Cloud in** `Jenkins`
    
    * After installation of the plugin, go to the `Settings > Clouds > New Cloud`, give a name for the Cloud and select `Amazon EC2 Container Service Cloud`.
        
    * Go to the configure option in the newly created cloud, adding in your region and search from the your ECS-Cluster from the list of ECS-Cluster in your region. Also, leave the assumed role ARN to be empty as the Jenkins container would assume the IAM role attached to the EC2 instance.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753635298610/a2adad95-a7f6-4c34-845b-f8ea0ed750a7.png align="center")
        
    * Now, under the `Advanced` option, we having more configurations:
        
        * Allowing declarative settings, set this field to be `all`, or if being specific add just the right settings inside our task definition to be overridden.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753797477021/bfadb58e-7b70-4c29-a25d-3cf2b0bc01a6.png align="center")
            
        * Restricting the number of agents, can also help us reduce costs, as leaving the field to zero, can provision unlimited agents.
            
        * Then, the rest of the advanced settings are as follows.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798339402/22638282-0c6b-470c-b669-48c5c93e6cb4.png align="center")
            
            * In the `tunnel connection through`: Add the private IP of the Jenkins Master
                
            * `Alternative Jenkins URL`: Private IP of Jenkins Master along with port(:8080)
                
    * The following configurations for `label` , `task-defination`, and `image`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798708815/d98c3556-7409-4e9f-ac60-9a011a318505.png align="center")
        
        Note: Here, we’re using `jenkins/inbound-agent` image as the default image(ps: we don’t really care about this image as we’ll be overriding images as per our Jobs dependencies) or just add one of your own custom Jenkins agent image.
        
    * For the launch type, select `Fargate`, and rest of the settings as in the image given above:
        
        Note: For `Fargate` use `awsvpc` in the Network mode.
        
        Also, for `Operating System Family` most commonly `Linux` would be your choice, but there are other `Windows` related options as well
        
        And for `CPU Architecture`, you can either go with `X86_64` or `ARM64`
        
    * Define the `soft/hard Memory Reservations` , `CPU units`, `ephemeral storage`, `subnets` and `security groups`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798817666/23c9cc3f-2249-439a-a098-00c219b6b4cb.png align="center")
        
    * Now, under Advanced, give the `task role ARN` and `task execution role ARN`, and enable `Command Execution`.
        
    * Also, don’t forgot to check the `Enable Command Execution` , as this would allow our Jenkinsfile to be executed on the agent node.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798848874/9879f0b4-119e-4437-9ad4-eb0ebf8e26c6.png align="center")
        
    * Configure the logs driver, by choosing `awslogs` , and adding the following `Logging Configuration`(don’t forget to add in your region):
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753799831218/e80171f8-c638-48cc-8d11-33c50bf97ef3.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753799847936/b1b7362b-a53f-4d92-b3d9-1149df206313.png align="center")
        
* **Step 3: Using agents in our pipelines**
    
    * The change to be made in the pipeline:
        
        * inheritFrom: Label of Cloud as mentioned above
            
        * image: Docker image to be used
            

---

## Setup: ECS EC2 launch type as Jenkins agents

For heavy workloads, or in case of requirement of `root` access(in case of DinD - Docker in Docker), our `fargate` approach lags behind because of security reasons, as containers run on fargate without `privileged` mode.

* **Step 1: Configuration of Cloud in** `Jenkins`
    
    * Installation: We’ve already installed this while setting up Fargate nodes.
        
    * After installation of the plugin, go to the `Settings > Clouds > New Cloud`, give a name for the Cloud and select `Amazon EC2 Container Service Cloud`.
        
    * Go to the configure option in the newly created cloud, adding in your region and search from the your ECS-Cluster from the list of ECS-Cluster in your region. Also, leave the assumed role ARN to be empty as the Jenkins container would assume the IAM role attached to the EC2 instance.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753635298610/a2adad95-a7f6-4c34-845b-f8ea0ed750a7.png align="center")
        
    * Now, under the `Advanced` option, we having more configurations:
        
        * Allowing declarative settings, set this field to be `all`, or if being specific add just the right settings inside our task definition to be overridden.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753797477021/bfadb58e-7b70-4c29-a25d-3cf2b0bc01a6.png align="center")
            
        * Restricting the number of agents, can also help us reduce costs, as leaving the field to zero, can provision unlimited agents.
            
        * Then, the rest of the advanced settings are as follows.
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798339402/22638282-0c6b-470c-b669-48c5c93e6cb4.png align="center")
            
            * In the `tunnel connection through`: Add the private IP of the Jenkins Master
                
            * `Alternative Jenkins URL`: Private IP of Jenkins Master along with port(:8080)
                
    * The following configurations for `label` , `task-defination`, and `image`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753799139237/c2478d99-6353-44d8-925f-4e0e0d854d47.png align="center")
        
        Note: Here, we’re using `jenkins/inbound-agent` image as the default image(ps: we don’t really care about this image as we’ll be overriding images as per our Jobs dependencies) or just add one of your own custom Jenkins agent image.
        
    * For the launch type, select `EC2` , and rest of the settings as in the image given above
        
        Also, for `Operating System Family` most commonly `Linux` would be your choice, but there are other `Windows` related options as well
        
        And for `CPU Architecture`, you can either go with `X86_64` or `ARM64`
        
        Heads-up: For `EC2` based ECS, don’t forget to check the `Default Capacity Provider` , in our case it is the ASG we’ve made using terraform. And the network-mode will be `default`, instead of `awsvpc` as we did in `Fargate` profile.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753799447168/4978c131-2ed8-4fe1-a553-148ecba406b3.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753799499053/c8d9bdee-8e76-4da4-8a0c-8f093ec9ae46.png align="center")
        
    * Define the `soft/hard Memory Reservations` , `CPU units`, `ephemeral storage`, `subnets` and `security groups`.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798817666/23c9cc3f-2249-439a-a098-00c219b6b4cb.png align="center")
        
    * Now, under Advanced, give the `task role ARN` and `task execution role ARN`, and enable `Command Execution`.
        
    * Also, don’t forgot to check the `Enable Command Execution` , as this would allow our Jenkinsfile to be executed on the agent node.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798848874/9879f0b4-119e-4437-9ad4-eb0ebf8e26c6.png align="center")
        
    * Configure the logs driver, by choosing `awslogs` , and adding the following `Logging Configuration` (don’t forget to add in your region):
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753799879933/ec2dadb4-0581-4bd2-a951-bf42d19c98cd.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753799885447/ec8278eb-2873-4f5f-952e-0815ab907934.png align="center")
        
    * But, wait our `EC2` based approach was for heavy operations and especially, using `DinD` (Docker in Docker), for that we need to use the `Container Mount Points`, and map your `docker.sock` in order to access docker engine from Jenkins agent container.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753800496996/6e11efd3-d498-4866-8e34-b2b9147bff20.png align="center")
        
* **Step 3: Using agents in our pipelines**
    
    * The change to be made in the pipeline:
        
        * inheritFrom: Label of Cloud as mentioned above
            
        * image: Docker image to be used
            

---

# Implementation: **Write Your Pipelines to Use ECS Agents**

So, in-order to test our setup, we’ll be using a `fargate` type Jenkins agent. So we’ll be using the normal (no-override) and overriding task definition. First of all, let’s create a new job of type `pipeline` in Jenkins.

## Normal configurations

* After creating the job, go to `Configure` and under `pipeline`, use the sample Jenkinsfile given below:
    

```java
pipeline {
    agent {
        ecs {
            inheritFrom 'testing'
        }
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

* Save and Apply the Jenkinsfile, and then hit `Build Now`
    
* Let’s check the console output for the just started build.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753810476200/4e6bc4f2-22ef-448c-9b69-fafc8c3fde15.png align="center")
    
* So, in the above image, as you can a line, **‘testing-testing-agent-17-2d1cc-v99bh’ is offline**, as stated in the plugin documentation the following syntax is followed for creating an agent name: {label}-{job-name}-{job-run-number}-{5-random-chars}. So, in our case **‘testing-testing-agent-17-2d1cc-v99bh’,** first `testing` is the label provided, followed by Job Name(testing-agent), then build number(17) and then some random characters, and some more characters.  
      
    Note: Don’t worry if you see suspended written under Build Executor Status, it is normal behavior of the plugin
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753810926315/a3ec90eb-2f69-4df1-866e-fc47acfb794a.png align="center")
    
* As we see in the AWS Console as well, our task after the fargate container is `provisioned`, comes under `running`
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753811728662/35e48d9d-10ee-4660-8d93-7f38a6d8f587.png align="center")
    
* After successfully, executed our “hello world” message gets printed in the console.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753811150965/96d039c6-12a9-4490-9116-5975218ab511.png align="center")
    

## Override configurations

So, everything would remain the same, just in the Jenkinsfile itself, we can add in few more declarative, like as shown here I’ve added `image` in order to use a custom image over the parent task definition(which I’ve provided in the Cloud Configs)

```java
pipeline {
    agent {
        ecs {
            inheritFrom 'testing'
            image '<IMAGE_URI>'
        }
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

---

# Challenges and solutions: Roadbumps (And GOTCHAS)

So, implementing the AWS ECS approach, wasn’t bread and butter, we did face challenges related to Jenkins Cloud configurations, security groups, and many more. I’ll list a few challenges here:

* Improper documentation and lack of online resources for implementing this approach, although the documentation is pretty apt, but some places it are left to be for try and find out like the `Allowing declarative settings` to be set `all`, which lets us override the parent task definition.  
      
    Fun Fact: The whole Cloud config acts like a task definition, so using `Allowing declarative settings`, we are able to override : `cpu` , `memory` , `logDriver` , `logDriverOptions([[name: 'foo', value:'bar']])` , `portMappings([[containerPort: <port>, hostPort: <port>, protocol: 'tcp']])`.  
      
    Similarly, for `EC2 based ECS`, I couldn’t find any resource to follow along so it was most try around and find out :)
    
* Another one, please don’t forget to click on `Enable Command Execution` , as this would allow our Jenkinsfile to be executed on the agent node.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1753798848874/9879f0b4-119e-4437-9ad4-eb0ebf8e26c6.png align="center")
    

* So, we faced a lot of issues regarding reach-ability of Jenkins agents to Master node, making sure we’re listing and allowing traffic to the correct port for inbound agents under `Security` in Jenkins
    
* Using inbound-agent as the base image is a must, with ENTRYPOINT set to `["/usr/local/bin/jenkins-agent"]`, in the Dockerfile. If your tinkering around other base image, don’t forget to take `agent.jar` , `slave.jar` and `jenkins-agent` from the inbound-agent.
    
    ```dockerfile
    FROM jenkins/inbound-agent:alpine${ALPINE_VERSION}-${JDK_VERSION} AS jnlp
    
    FROM <your-final-base-image>
    
    COPY --from=jnlp /usr/local/bin/jenkins-agent /usr/local/bin/jenkins-agent
    COPY --from=jnlp /usr/share/jenkins/agent.jar /usr/share/jenkins/agent.jar
    COPY --from=jnlp /usr/share/jenkins/slave.jar /usr/share/jenkins/slave.jar
    
    ENTRYPOINT [ "/usr/local/bin/jenkins-agent" ]
    ```
    
* Making sure that the correct version of Docker Image for the Jenkins agent is present in your ECR, or wherever you’ve pushed your agent’s docker image. Otherwise, ECS tasks would first, return `de-provisioning` and then would return:  
      
    **Stopped reason:** CannotPullContainerError: pull image manifest has been retried 1 time(s): failed to resolve ref &lt;IMAGE\_PROVIDED&gt;: &lt;IMAGE\_PROVIDED&gt;: not found.
    

---

# Conclusion

Adopting a Jenkins Master-Slave (Controller-Agent) architecture with AWS ECS is more than just a technical tweak—it’s a transformation in how you approach CI/CD. By shifting away from rigid, monolithic servers to scalable, on-demand cloud agents, you can significantly reduce operational costs, eliminate infrastructure bottlenecks, and free your teams to move faster and more confidently.

This hybrid strategy—leveraging Fargate for standard tasks, EC2 for heavy workloads, and Spot for non-critical jobs—gives you the flexibility and efficiency that modern teams demand. With Infrastructure as Code, your setup is reproducible, manageable, and ready to scale with your needs.

The upfront investment in time pays off quickly: you gain improved reliability, faster build execution, and a system that adapts to your workload, not the other way around.

Begin small, experiment, and expand as you learn. Your Jenkins infrastructure can be agile, reliable, and cost-effective—unlocking both business value and peace of mind.

## Ready to Transform Jenkins?

* Clone the [Terraform modules & guid](https://github.com/deepanshu-rawat6/Jenkins-Master-Slave-infra-tf)[e.](https://github.com/deepanshu-rawat6/Jenkins-Master-Slave-infra-tf)
    
* Start small, experiment safely, then ramp up.
    
* Turn Jenkins into an engine for agility and savings!
    

Stay tuned for future posts, where we’ll dive into further optimizations and share tips for getting the most out of this powerful setup!

---