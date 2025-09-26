---
title: ECS
linktitle: 🐳 ECS
type: book
tags:
  - AWS
date: "2024-01-15T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 125
---

Container orchestration in AWS

<!--more-->

## 🔎Overview

Amazon Elastic Container Service (```ECS```) is a highly scalable and high-performance container orchestration service. It integrates seamlessly with the rest of the AWS ecosystem, allowing you to focus on your application without worrying about scheduling containers on virtual machines.

## ECS Cluster

An ```ECS``` cluster is a logical grouping of tasks or services that helps you manage containers on the Amazon Web Services (AWS) platform. It acts as a pool of resources where you can run your containerized applications. Within a cluster, you can deploy tasks and services using either ```EC2 instances``` or ```AWS Fargate```. You can also combine both launch types within a single cluster.

###### EC2 Launch Type 🖥️

The ```EC2``` launch type allows you to run your containers on a cluster of Amazon ```EC2``` instances. You have **full control** over the instances, including the underlying operating system and configuration.

![ecs-ec2](/images/uploads/ecs-ec2-type.png)

**Key Features**

- **Customization**: *You* can choose specific EC2 instance types, AMIs, and configure the instances as required.
- **Control**: *You* manage the lifecycle of the EC2 instances, including scaling, updating, and patching.
- **Networking**: Instances within an ECS cluster can be configured in multiple networking modes (```bridge```, ```host```, ```awsvpc``` etc.).

###### Fargate Launch Type ☁️

AWS ```Fargate``` is a **serverless** compute engine for containers that works with ECS. With ```Fargate```, you do not need to *provision*, *configure*, or *manage* EC2 instances.

![ecs-fargate](/images/uploads/ecs-fargate-type.png)

**Key Features**

- **Serverless**: No need to manage the underlying EC2 instances. AWS handles the infrastructure.
- **Scalability**: Automatically scales as needed without manual intervention.
- **Simplified** Management: Focus on deploying and managing your containerized applications rather than the infrastructure.

## Tasks

In ```ECS```, the basic unit of a deployment is a ```task```, a logical construct that models one or more ```containers```. This means that the ECS APIs operate on tasks rather than individual containers. In ECS, you *can’t* run a container: rather, you run a ```task```, which, in turns, run your container(s). A task contains *one or more* ```containers```.

- **Task Definition**: This is a ```JSON``` text file that describes one or more containers (up to ten) that form your application. Think of it as a <span style="color:blue">*blueprint*</span> for your application.
- **Parameters**: Specify containers, launch type, ports, and data volumes.

* Example Task Definition:

  ```json
  {
    "family": "nginx-task-def",
    "networkMode": "bridge",  // Change to "awsvpc" if using AWSVPC mode
    "containerDefinitions": [
      {
        "name": "nginx",
        "image": "nginx",
        "memory": 256,
        "cpu": 256,
        "essential": true,
        "portMappings": [
          {
            "containerPort": 80,
            "hostPort": 80
          }
        ],
        "logConfiguration": {
          "logDriver": "awslogs",
          "options": {
            "awslogs-group": "/ecs/nginx",
            "awslogs-region": "us-west-2",
            "awslogs-stream-prefix": "ecs"
          }
        }
      }
    ]
  }
  ```

## Service

Amazon ```ECS Service``` allows you to run and maintain a *desired state* by running a specified number of instances of ```tasks``` simultaneously within a cluster.

- **Desired State**: You specify the desired number of task instances. The ECS Service scheduler ensures that this number is maintained by replacing failed tasks.
- **Deployment Types**: ```ECS Service``` supports *rolling updates* and *blue/green* deployments using AWS CodeDeploy.
- **Load Balancing**: You can associate an ECS Service with an ```Elastic Load Balancer``` to distribute traffic across tasks.
- **Auto Scaling**: Automatically adjust the number of ```tasks``` in response to demand.
Health Checks. Monitor the health of the tasks and automatically replace unhealthy tasks.
- Example Service Definition:

  ```json
  {
    "serviceName": "nginx-service",
    "taskDefinition": "nginx-task-def",
    "desiredCount": 2,
    "launchType": "EC2",  // Or "FARGATE" if using Fargate
    "loadBalancers": [
      {
        "targetGroupArn": "arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-target-group",
        "containerName": "nginx",
        "containerPort": 80
      }
    ],
    "networkConfiguration": {
      "awsvpcConfiguration": {
        "subnets": [
          "subnet-abcde123",
          "subnet-bcde234f"
        ],
        "securityGroups": [
          "sg-0123456789abcdef0"
        ],
        "assignPublicIp": "ENABLED"
      }
    },
    "deploymentConfiguration": {
      "maximumPercent": 200,
      "minimumHealthyPercent": 100
    }
  }
  ```

- To enable ```auto scaling```, you can define a scaling policy using AWS Application Auto Scaling. Below is an example to scale based on **CPU utilization**:

  ```json
  {
    "scalableTarget": {
      "maxCapacity": 5,
      "minCapacity": 2,
      "resourceId": "service/nginx-cluster/nginx-service",
      "roleARN": "arn:aws:iam::account-id:role/aws-service-role/ecs.application-autoscaling.amazonaws.com/AWSServiceRoleForApplicationAutoScaling_ECSService",
      "scalableDimension": "ecs:service:DesiredCount",
      "serviceNamespace": "ecs"
    },
    "scalingPolicy": {
      "policyName": "nginx-cpu-scaling-policy",
      "policyType": "TargetTrackingScaling",
      "targetTrackingScalingPolicyConfiguration": {
        "targetValue": 50.0,
        "predefinedMetricSpecification": {
          "predefinedMetricType": "ECSServiceAverageCPUUtilization"
        },
        "scaleInCooldown": 60,
        "scaleOutCooldown": 60
      }
    }
  }
  ```

## ECS Agent

The ```ECS Agent``` is a daemon component that **has** to run on each Amazon ```EC2``` instance within an ```ECS cluster```. It is responsible for managing the state of containers on the EC2 instance. The agent communicates with the ECS service to *start*, *stop*, and *monitor* the containers.

**Key Responsibilities**

- **Communication with ECS**: The ```agent``` continuously communicates with the ```ECS service``` to receive instructions and report the **status** of running tasks.

- **Task Lifecycle Management**: It handles the lifecycle of Docker containers, including pulling images, starting containers, and stopping them when required.

- **Monitoring and Reporting**: The agent collects and reports metrics, logs, and the current state of the containers back to the ECS service.

- **Resource Management**: It manages the allocation of CPU, memory, and other resources to the running containers on the EC2 instance.

Here's a simplified diagram showing the interaction between the ```ECS Agent```, ```ECS Service```, and the ```EC2``` instances:

```
+---------------------+              +-------------------+
|     ECS Service     | <--------->  |      ECS Agent    |
+---------------------+              +-------------------+
          ^                                    ^
          |                                    |
          v                                    v
    +------------+                   +-------------------+
    |  ECS Task  |                   |     EC2 Instance  |
    +------------+                   +-------------------+
```

## IAM Roles

###### ecsInstanceRole 🔐

The ```ecsInstanceRole``` is an AWS ```IAM role``` that provides the necessary permissions for the ```ECS Agent``` to interact with other AWS services on behalf of the ```EC2``` instance.

**Key Permissions**

- **Register/De-register Instances**: Permissions to *register* and *deregister* instances with the ```ECS cluster```.
- **Send Container Logs**: Permissions to send logs to Amazon ```CloudWatch```.
- **Retrieve Secrets and Parameters**: Permissions to retrieve **secrets** from AWS ```Secrets Manager``` and parameters from AWS Systems Manager Parameter Store.
- **Access to ECS API**: Permissions to make API calls to the ```ECS service``` to report the status of containers and instances.
- Example ```IAM Policy``` that might be attached to the ```ecsInstanceRole```:

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ecs:CreateCluster",
          "ecs:DeregisterContainerInstance",
          "ecs:DiscoverPollEndpoint",
          "ecs:Poll",
          "ecs:RegisterContainerInstance",
          "ecs:StartTelemetrySession",
          "ecs:UpdateContainerInstancesState",
          "ecs:Submit*"
        ],
        "Resource": "*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "ecr:GetAuthorizationToken",
          "ecr:BatchCheckLayerAvailability",
          "ecr:GetDownloadUrlForLayer",
          "ecr:BatchGetImage"
        ],
        "Resource": "*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": [
          "arn:aws:logs:*:*:log-group:/ecs/*"
        ]
      },
      {
        "Effect": "Allow",
        "Action": [
          "ssm:GetParameters",
          "secretsmanager:GetSecretValue",
          "kms:Decrypt"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

- Visual Representation of ```ecsInstanceRole```

  ```
      +-----------------+
      |   ECS Agent     |
      +--------^--------+
                |
                v
  +----------------------------+
  |    ecsInstanceRole (IAM)   |
  +-------------^--------------+
                |
                v
  +----------------+   +-------------------+   +-------------+
  |     ECS        |   | Amazon CloudWatch |   | AWS Secrets |
  |     Service    |   |  (Logs, Metrics)  |   |  Manager    |
  +----------------+   +-------------------+   +-------------+

  ```

###### ecsTaskExecutionRole 🔐

The ```ecsTaskExecutionRole``` is another IAM role, but it is assigned to the ```ECS tasks``` rather than the EC2 instances. This role provides the necessary permissions for the ECS service to pull container images and manage secrets for running containers.

**Key Permissions** 🔑
- **ECR Access**: Permissions to pull container images from Amazon ECR.
- **CloudWatch Logs**: Permissions to create log streams and put log events.
- **Secrets Manager**: Permissions to retrieve secrets for use within containers.
- **Systems Manager**: Permissions to retrieve parameters from the Systems Manager Parameter Store.

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ecr:GetDownloadUrlForLayer",
          "ecr:BatchGetImage",
          "ecr:GetAuthorizationToken"
        ],
        "Resource": "*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": [
          "arn:aws:logs:*:*:log-group:/ecs/*"
        ]
      },
      {
        "Effect": "Allow",
        "Action": [
          "secretsmanager:GetSecretValue",
          "ssm:GetParameters",
          "kms:Decrypt"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

- Visual Representation of ```ecsTaskExecutionRole```

  ```
          +------------------------------+
          |         ECS Task              |
          +---------------^--------------+
                          |
                          v
          +------------------------------+
          |    ecsTaskExecutionRole (IAM) |
          +---------------^--------------+
                          |
                          v
  +----------------+   +-------------------+   +-------------+
  |  Amazon ECR    |   | Amazon CloudWatch |   | AWS Secrets |
  |   (Images)     |   |  (Logs)           |   |  Manager    |
  +----------------+   +-------------------+   +-------------+
  ```

## Networking

Amazon ECS supports multiple networking modes for tasks, providing flexibility in how your containers interact with each other and the internet.

###### Bridge Mode 🌉

- **Default for EC2**: If the network mode is ```bridge```, the task utilizes ```Docker```'s built-in virtual network a.k.a ```bridge``` network which runs inside each EC2  instance.
- There's **one** ```ENI``` for each EC2 instance and ```Security Groups``` are attached at the ```ENI``` level i.e at each EC2 instance in this case.
- Tasks are attached to this bridge network.
- No network **isolation** between containers

- **Port Mapping**: 

  - **Static**: Explicitly map container ports to host ports. Cannot run more than **one** instance of the task on the same ```host```.
    ![ecs-bridge-mode-static](/images/uploads/ecs-bridge-mode-static.png)

  - **Dynamic**: Specify the container port but leave the host port as ```0```. This allows ECS to automatically assign an available host port at *runtime*.
    ![ecs-bridge-mode-dynamic](/images/uploads/ecs-bridge-mode-dynamic.png)

###### Host Mode 🖧

- **High Performance**: Containers share the host’s network stack, eliminating the need for port mapping.
- **Use Case**: Suitable for applications that need high network throughput.

![ecs-host-mode](/images/uploads/ecs-host-mode.png)

###### AWSVPC Mode 🔌

- **VPC Integration**: Each task gets its own Elastic Network Interface (```ENI```) and a unique private IP address within the VPC.
- In addition ```Security Groups``` are attached at the ```ENI``` level i.e. each task gets it's own Security Group. 
- **Security and Isolation**: Achieves network solation, best for secure, isolated environments.

![ecs-awsvpc-mode](/images/uploads/ecs-awsvpc-mode.png)

  ```json
  {
    "networkMode": "awsvpc",
    "containerDefinitions": [
      {
        "name": "my-container",
        "image": "my-image",
        "portMappings": [
          {
            "containerPort": 80,
            "hostPort": 80
          }
        ],
        "networkInterfaces": [
          {
            "eniId": "eni-0123456789abcdef0"
          }
        ]
      }
    ]
  }
  ```

###### No Networking 🚫

- **Standalone**: Tasks are isolated and do not have any networking capabilities.
- **Use Case**: Useful for batch processing where networking is unnecessary.

![ecs-none-mode](/images/uploads/ecs-none-mode.png)

###### Network Compatibility

| Networking Mode | Linux Compatibility | Windows Compatibility | Fargate Compatibility |
|-----------------|---------------------|-----------------------|-----------------------|
| Bridge Mode     | Yes                 | No                    | No                    |
| Host Mode       | Yes                 | No                    | No                    |
| AWSVPC Mode     | Yes                 | Yes                   | Yes                   |
| No Networking   | Yes                 | Yes                   | Yes                   |


By utilizing these features and modes effectively, you can optimize your AWS ECS environments for performance, security, and scalability.  