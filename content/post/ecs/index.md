---
title: Transform Your Deployment Strategy
subtitle: Crafting a DevOps <span style="color:blue">Blue</span>/<span style="color:green">Green</span> Pipeline with Amazon ECS!

# Summary for listings and search engines
summary: Crafting a DevOps <span style="color:blue">Blue</span>/<span style="color:green">Green</span> Pipeline with Amazon ECS!

# Link this post with a project
projects: []

# Date published
date: "2024-01-16T00:00:00Z"

toc: true

# Date updated
lastmod: "2024-01-16T00:00:00Z"

# Is this an unpublished draft?
draft: true

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption:
  focal_point: "Center"
  placement: 1
  preview_only: false

authors:
- admin

tags:
- ECS
- CICD
- AWS

categories:

---

<!--more-->

### Overview

Let's kick things off with a **monolithic** application and experience the ⚡ speed and 🚀 agility that ```AWS``` offers. You'll also achieve **zero-downtime** deployments 🔄, even with distributed development teams collaborating on a shared codebase!

We'll take a ```AWS Console``` walk through approach here to understand the concepts and look under the hood🔧, the process of:

- **Auto-generating a Monolithic project**: We'll kick off by creating a ```Todo Management``` application using the [JHipster](https://start.jhipster.tech/) platform.
  {{% callout note %}}
  [Jhipster](https://start.jhipster.tech/)  is an [Apache 2.0-licensed](https://github.com/jhipster/generator-jhipster/blob/master/LICENSE.txt) development platform to quickly generate, develop, and deploy modern web applications & microservice architectures. It supports many frontend technologies, including ```Angular```, ```React```, and ```Vue``` and mobile app support for ```Ionic``` and ```React Native```! On the backend, it support ```Spring Boot``` (with Java or Kotlin), ```Micronaut```, ```Quarkus```, ```Node.js```, and ```.NET```.
  {{% /callout %}}
- **Containerizing the Application**: Next, we'll containerize the ```Todo Management``` *monolith* and host its database on Amazon *Relational Database Service* (```RDS```).
- **Implementing a CI/CD Pipeline**: We'll set up a DevOps CI/CD pipeline using a <span style="color:blue">blue</span>/<span style="color:green">green</span> architecture on AWS, leveraging Amazon *Elastic Container Service* (```ECS```).
- **Deployment Options**: We'll explore two key deployment options for the *containerized* monolith on Amazon ECS:
  1. Amazon ```EC2```: Manage your own fleet of EC2 servers to host containers.
  2. AWS ```Fargate```: Experience serverless compute for containers—no need to manage any EC2 fleet!

### Prerequisites

The things that you will need:

- 👉 [Install Java](https://aws.amazon.com/corretto) (```21```)
- 👉 [Install Node](https://nodejs.org/en/download/package-manager)(```20```)
- 👉 [Install Docker](https://docs.docker.com/engine/install/)
- 👉 [GitHub Account](https://github.com/)
- 👉 [AWS Account](https://aws.amazon.com/)

*Optional* installations:
- Install a Java build tool: Whether you choose to use ```Maven``` or ```Gradle```, you normally don't have to install anything, as ```JHipster``` will automatically install the ```Maven Wrapper``` or the ```Gradle Wrapper``` for you. If you don't want to use those wrappers, go to the official Maven website or Gradle website to do your own installation.
- Install IDE: Install [VSCode](https://code.visualstudio.com/Download) or your :heart: IDE

{{% callout note %}}
  [Jhipster](https://start.jhipster.tech/) has an active community and they keep the platform updated with latest versions of ```SpringBoot```, ```Java```, ```Node```... So if you are following along please verify compatibility and install JDK and Node versions accordingly.
{{% /callout %}}


{{% stepper %}}
<div class="step">

  ## Step 1: Create Monolith Application

  * Open the [Jhipster Online](https://start.jhipster.tech/)
  * In the main pane, choose ```Create Application```.
  * Override below given default values:
    * Under *Application Name* & *Repository Name*, mention ```todomgmt```
    * Under *What is your default Java package name?*, mention ```com.example.todomgmt```
    * Under *Which type of database would you like to use?*, pick ```SQL```
    * Under *Which production database would you like to use?*, pick ```PostgreSQL```
    * Under *Which development database would you like to use?* pick ```H2 with disk-based persistence```
    * Under *Do you want to use the Spring cache abstraction?*, select ```No```
    * Under *Would you like to use Maven or Gradle for building the backend?*, choose ```Gradle```
    * Under *Please choose additional languages to install*, choose your choice of additional language.
    * Select *Download as Zip* file, ```todomgmt.zip``` file will be download on your local machine.

</div>
<div class="step">

  ## Step 2: Containerize Monolith Application

  * In order to containerize our ```Todo Management``` monolith application, we will need the ```Dockerfile``` and ```entrypoint.sh``` files:

    <details>
    <summary>Dockerfile</summary>

    ```yaml
    FROM amazoncorretto:21-alpine

    ENV SPRING_OUTPUT_ANSI_ENABLED=ALWAYS \
        JHIPSTER_SLEEP=0 \
        JAVA_OPTS=""

    # Add a jhipster user to run our application so that it doesn't need to run as root
    RUN adduser -D -s /bin/sh jhipster
    WORKDIR /home/jhipster

    COPY build/libs/todomgmt-0.0.1-SNAPSHOT.war app.war

    ADD entrypoint.sh entrypoint.sh
    RUN chmod 755 entrypoint.sh && chown jhipster:jhipster entrypoint.sh
    USER jhipster

    ENTRYPOINT ["./entrypoint.sh"]

    EXPOSE 8080
    ```

    </details>

    <details>
    <summary>entrypoint.sh</summary>

    ```sh
    #!/bin/sh

    echo "The application will start in ${JHIPSTER_SLEEP}s..." && sleep ${JHIPSTER_SLEEP}
    exec java ${JAVA_OPTS} -Djava.security.egd=file:/dev/./urandom -jar "${HOME}/app.war" "$@"
    ```
    </details>
  

  * Build the application with the ```production``` flag. This command may take ~ 3 - 5 mins.
    ```
    ./gradlew bootWar -Pprod -Pwar
    ```
  
  * Create a ```docker``` image.
    ```
    docker build -t todomgmt .
    ```

  * Test your application locally using ```docker-compose```.
    ```
    docker-compose -f src/main/docker/app.yml up 
    ```

  * Test the Application by signing in.
  ![todomgmt-v0.0.1](/images/uploads/todomgmt-v0.0.1.png)

  * Great, Congratulations !!!🎉 You have successfully created and containerized a Java and Angular based Monolith Application. Let's learn and explore the next section for deploying a containerized Monolith Application through AWS DevOps services.

</div>
<div class="step">

  ## Step 3: Create required AWS IAM Roles

  * **Task Execution Role**: The task execution role grants the ```Amazon ECS``` container and ```Fargate``` agents permission to make AWS API calls on your behalf. The task execution ```IAM role``` is required depending on the requirements of your task. You can have multiple task execution roles for different purposes and services associated with your account.

    - To create the ```ecsTaskExecutionRole``` IAM role:

    - Open the ```IAM``` console 
    - In the navigation pane, choose Roles, *Create Role*.
    - In the *Trusted Entity* type, select ```AWS Service```
    - Under *Use case*, select ```Elastic Container Service``` and ```Elastic Container Service Task```, then choose Next: *Permissions*.
    - In the *Attach permissions* policy section, search for ```AmazonECSTaskExecutionRolePolicy```, select the policy, and then choose Next: *Review*.
    - For Role Name, type ```ecsTaskExecutionRole``` and choose *Create role*.
    - Please save value of the ```Role ARN``` to use in the later step.
    
  * **ECS Container Instance Role**: This IAM role is required for the EC2 launch type. The Amazon ECS container agent makes calls to the Amazon ECS API on your behalf. Container instances that run the agent require an IAM policy and role for the service to know that the agent belongs to you.

    - To create the ```ecsInstanceRole``` IAM role:

    - Open the IAM console 
    - In the navigation pane, choose Roles, *Create role*.
    - In the *Trusted Entity* type, select ```AWS Service```
    - Under *Use case*, select ```EC2```, then choose Next: *Permissions*.
    - In the *Attach permissions* policy section, search for ```AmazonEC2ContainerServiceforEC2Role```, select the policy, and then choose Next: *Review*.
    - For Role Name, type ```ecsInstanceRole``` and choose *Create role*.
    
  * **ECS CodeDeploy Role**: Before you can use the ```CodeDeploy``` blue/green deployment type with Amazon ECS, the CodeDeploy service needs permissions to update your Amazon ECS service on your behalf. These permissions are provided by the CodeDeploy ```IAM role```.

    - To create the ```ecsCodeDeployRole``` IAM role:

    - Open the IAM console 
    - In the navigation pane, choose Roles, *Create role*.
    - In the *Trusted Entity* type, select ```AWS Service```
    - Under *Use case*, select ```CodeDeploy``` and ```CodeDeploy - ECS```, then choose Next: *Permissions*.
    - In the *Attach permissions* policy section, ```AWSCodeDeployRoleForECS``` policy should come as selected, choose Next: *Review*.
    - For Role Name, type ```ecsCodeDeployRole``` and choose *Create role*.
    
  * **Service Linked Role for Amazon ECS**: Amazon ECS uses the **service-linked** role named ```AWSServiceRoleForECS``` to enable Amazon ECS to call AWS APIs on your behalf. The AWSServiceRoleForECS service-linked role trusts the ```ecs.amazonaws.com``` service principal to assume the role. Under most circumstances, this role should already exist, if not

    - To create a service-linked role (CLI): 

      ```
      aws iam create-service-linked-role --aws-service-name ecs.amazonaws.com
      ```
  * **Service Linked Role for Amazon EC2 Auto Scaling**: Amazon EC2 Auto Scaling uses **service-linked** roles for the permissions that it requires to call other AWS services on your behalf. Amazon EC2 Auto Scaling uses the ```AWSServiceRoleForAutoScaling``` service-linked role. Under most circumstances, this role should already exist, if not

    - To create a service-linked role (CLI):

      ```
      aws iam create-service-linked-role --aws-service-name autoscaling.amazonaws.com
      ```

</div>
<div class="step">

  ## Step 4: Create GitHub repository

  * We will need a code repository for storing the ```Todo Management``` Monolith Application source code. You can use any ```CodePipeline``` source action supported [source control](https://docs.aws.amazon.com/codepipeline/latest/userguide/integrations-action-type.html#integrations-source) system. For this post, we will use ```GitHub```.

  Here's my [GitHub](https://github.com/cloud-borne/todomgmt.git) repo.

</div>
<div class="step">

  ## Step 5: Create Container repository

  * Open the Amazon ```ECR console```.
  * In the navigation pane, choose ```Repositories```.
  * On the Repositories page, choose *Create repository*.
  * For *Visibility* settings, choose ```Private```.
  * For *Repository* name, enter a unique name for your repository ```devops/todomgmtdemo```
  * For *Tag immutability*, keep it ```Mutable```. Repositories configured with immutable tags will prevent image tags from being overwritten. 
  * For *Scan on push*, keep it ```Disabled```. Repositories configured to scan on push will start an image scan whenever an image is pushed, otherwise image scans need to be started manually. 
  * For *Encryption* settings, keep the default ```AES-256```. You can also use AWS Key Management Service (```KMS```) to encrypt images stored in this repository, instead of using the default encryption settings.
  * Choose *Create repository*.
  * Please save value of the ```Repository URL``` to use in the later section.

</div>
<div class="step">

  ## Step 6: Create Aurora PostgreSQL DB

  * In this section, we will setup ```Aurora PostgreSQL``` RDS with ```Multi-AZ``` configuration.

    - Sign in to the AWS Management Console and open the Amazon ```RDS console```.
    - In the navigation pane, choose *Databases*. Choose *Create database*.
    - In Choose a Database *Creation Method*, choose ```Standard Create```.
    - In *Engine options*, choose ```Aurora``` (PostgreSQL Compatible).
    - In *Templates*, choose ```Dev/Test``` template.
    - In the *DB cluster* identifier field, give Aurora DB cluster name, ```todomgmtdb-cluster```
    - To enter your master password, do the following in *Credential Settings*:
      - Pick *Self Managed* and clear *Auto generate a password* check box.
      - Enter *Master password* value of your choice and enter the same password in *Confirm password*.
    - For *Cluster storage configuration* pick ```Aurora Standard```
    - For *Instance configuration*, choose ```db.t4g.medium``` under *Burstable classes* (includes t classes)
    - For *Availability & Durability*, choose ```Create an Aurora Replica/Reader``` node in a different AZ (recommended for scaled availability)
    - Keep defaults for *Connectivity*, *Babelfish* settings, *Database authentication*, *Monitoring* sections.
    - For *Additional Configuration*, enter ```todomgmtdb``` under *Initial Database Name*.
    - Choose *Create database*.
    - For Databases, choose the name of the new Aurora DB cluster.
    - On the *Connectivity & Security* tab, note the port and the endpoint of the writer DB instance. Please save value of the ```endpoint``` and ```port``` of the cluster. Endpoint URL should be of format ```todomgmtdb-cluster.cluster-UNIQUEID.AWSREGION.rds.amazonaws.com```
    - Click on *Writer Node* and on the *Connectivity & Security* tab, please save value of the *Security Group Id* under Security section (i.e. ```sg-xxxxxxxx```).

</div>
<div class="step">

  ## Step 7: Create Application Load Balancer

  * In this section, we will create an Amazon EC2 ```Application Load Balancer```. This will be the **public** endpoint to access ```Todo Management``` Monolith Application.
  * The load balancer must use a ```VPC``` with two **public** subnets in different ```Availability Zones```. In these steps, we create a load balancer, and then create two target groups for our load balancer.
  * First create *Target Groups* for the ```Load Balancer```
    - Sign in to the AWS Management Console and open the Amazon *EC2 console* 
    - In the navigation pane, choose *Target Groups* and select *Create Target Group*.
    - Under *Basic configuration*, select ```IP addresses``` target type.
    - In *Name*, enter a target group name ```alb-tg-todomgmtdemo-1```
    - In *Protocol* choose ```HTTP```. In *Port*, enter ```80```.
    - Keep rest of the settings as defaults.
    - Select Next: *Register Targets*.
    - Select *Create Target Group*.
    - Repeat above steps to create another target group for Protocol ```HTTP``` and Port ```8080```, name it ```alb-tg-tripmgmtdemo-2```
  
  {{% callout note %}}
  We must create **two** target groups for our load balancer, in order for your deployment to run. You only need to save ```ARN``` value of your first target group. This ```ARN``` is used in the ```create-service``` JSON file in the next section.
  {{% /callout %}}
  
  * To create an Amazon EC2 ```Application Load Balancer```

    - Sign in to the AWS Management Console and open the Amazon ```EC2 console``` 
    - In the navigation pane, choose *Load Balancers*, choose *Create Load Balancer*.
    - Choose *Application Load Balancer*, and then choose *Create*.
    - In *Name*, enter the name of your load balancer ```todomgmtdemo-alb```
    - In *Scheme*, choose ```Internet-Facing```.
    - In *IP address* type, choose ```IPv4```.
    - Under Network mapping, in VPC, choose the default VPC, and then choose any two availability zones under Mappings. Please save value of the Availability Zone / Subnet ids in the open text file to use in the later section of the workshop.
    - Under Security groups, choose Create new security group
    - Give ALBSecurityGroup under Security group name
    - Add Inbound Rule, Allow 80 port (HTTP) inbound traffic from My IP
    - Add Inbound Rule, Allow 8080 port (CustomTCPPort) inbound traffic from My IP
    - Select Create security group
    - Please save value of the newly created Security Group Id (i.e. sg-xxxxxxxx..) in the open text file to use in the later section of the workshop.
    - Come back to Create Application Load Balancer tab, refresh Security groups and select newly created Security group, removing any existing pre-selected Security groups.
    - Under Listeners and routing, configure two listener ports for your load balancer:
    - Under Protocol, choose HTTP, and Under Port, enter 80.
    - Under Default action, select alb-tg-tripmgmtdemo-1 Target Group.
    - Choose Add listener.
    - Under Protocol, choose HTTP, and Under Port, enter 8080.
    - Under Default action, select alb-tg-tripmgmtdemo-2 Target Group.
    - Choose Create load balancer.
    - Go to Load Balancers and click on the newly created load balancer. From the Description tab, please save value of the DNS name of the Load Balancer in the open text file to use in the later section of the workshop.

</div>
<div class="step">

  ## Step 8: Create ECS Cluster

</div>
<div class="step">

  ## Step 9: Create ECS Task

</div>
<div class="step">

  ## Step 10: DevOps Pipeline with Blue/Green Deployment

</div>
<div class="step">

  ## Step 11: Setup CodeBuild Project

</div>
<div class="step">

  ## Step 12: Deploy on Amazon ECS with EC2

</div>
<div class="step">

  ## Step 13: Deploy on Amazon ECS with Fargate

</div>
<div class="step">

  ## Step 14: Cleanup 🧹

</div>

{{% /stepper %}}

## Wrapping It Up 🎁
