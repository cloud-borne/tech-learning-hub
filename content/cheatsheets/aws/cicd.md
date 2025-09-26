---
title: CICD
linktitle: ♾️ CICD
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 130
---

CICD in AWS

<!--more-->

## 🔎Overview

CICD♾️ in AWS is composed of the following services. Most enterprises use most or some of them together to give the ```Continuous Integration and Continuous Delivery``` experience. 

- ✏️ CodeCommit – version control
- 🚰 CodePipeline – automating releases from code to deployment
- 🏗️ CodeBuild – building and testing code
- 🚀 CodeDeploy – deploying the code to EC2 instances, Elastic Beanstalk, ECS...
- ✨ CodeStar – manage software development activities in one place
- 📦️ CodeArtifact – store, publish, and share software packages
- 🔎 CodeGuru – automated code reviews using Machine Learning

![AWS-CICD](/images/uploads/cicd-techstack.png)

## CodeCommit

{{< figure src="images/uploads/codecommit-iam-policy.png" class="alignright">}}

-  Interactions are done using Git (standard)
-  Authentication:
    - SSH Keys – AWS Users can configure SSH keys in their IAM Console
    - HTTPS – with AWS CLI Credential helper or Git Credentials for IAM user
-  Authorization
    - IAM policies to manage users/roles permissions to repositories
-  Encryption
    - Repositories are automatically encrypted at rest using AWS KMS
    - Encrypted in transit (can only use HTTPS or SSH – both secure)
-  Cross-account Access
    - Do NOT share your SSH keys or your AWS credentials
    - Use an IAM Role in your AWS account and use AWS STS (AssumeRole API)
-  By default, a user who has push permissions to a CodeCommit repository can contribute to any branch
-  Use IAM policies to restrict users to push or merge code to a specific branch -  Example: only senior developers can push to production branch.
>  Note: ```Resource Policy``` is not supported yet.

- You can monitor CodeCommit events in ```EventBridge``` (near real-time). So anytime a pullRequestCreated, pullRequestStatusChanged, referenceCreated, commentOnCommitCreated...and so on you can react to that via EventBridge like so:
![CodeCommit Eventbridge](/images/uploads/codecommit-eventbridge.png)

- You can migrate a project hosted on another Git repository (e.g., Github, GitLab...) to CodeCommit repository
![CodeCommit Migration](/images/uploads/codecommit-migration.png)

![CodeCommit Vs GitHub](/images/uploads/codecommit-vs-github.png)

## CodeBuild

* A fully managed continuous integration (CI) service
* Continuous scaling (no servers to manage or provision – no build queue)
* Compile source code, run tests, produce software packages...
* Alternative to other build tools (e.g., Jenkins)
* Charged per minute for compute resources (time it takes to complete the builds)
* Leverages ```Docker``` under the hood for reproducible builds
* Use prepackaged Docker images or create your own custom Docker image
* Security:
  * Integration with ```KMS``` for encryption of build artifacts
  * IAM for CodeBuild permissions, and VPC for network security
  * AWS CloudTrail for API calls logging

* **Source** – CodeCommit, S3, Bitbucket, GitHub
* **Build instructions**: Code file ```buildspec.yml``` at the project root.
* **Output** logs can be stored in Amazon S3 & CloudWatch Logs
* Use CloudWatch Metrics to monitor build statistics
* Use EventBridge to detect failed builds and trigger notifications
* Use CloudWatch Alarms to notify if you need **thresholds** for failures
* Build Projects can be defined within CodePipeline or CodeBuild

###### How it Works
![CodeBuild – Operation](/images/uploads/codebuild-operation.png)

{{< figure src="images/uploads/buildspec.png" width="300" height="1000" class="alignright">}}

* **buildspec.yml** file must be at the root of your code 
* **env** – define environment variables 
  * variables – plaintext variables 
  * parameter-store – variables stored in SSM Parameter Store 
  * secrets-manager – variables stored in AWS Secrets Manager 
* **phases** – specify commands to run: 
  * install – install dependencies you may need for your build 
  * pre_build – final commands to execute before build 
  * Build – actual build commands 
  * post_build – finishing touches (e.g., zip output) 
* **artifacts** – what to upload to S3 (encrypted with KMS) 
* **cache** – files to cache (usually dependencies) to S3 for future build speedup

###### Environment Variables

* Default Environment Variables
  * Defined and provided by AWS
  * AWS_DEFAULT_REGION, CODEBUILD_BUILD_ARN, CODEBUILD_BUILD_ID, CODEBUILD_BUILD_IMAGE...
* Custom Environment Variables
  * Static – defined at build time (override using start-build API call)
  * Dynamic – using SSM Parameter Store and Secrets Manager

###### Inside VPC

* By default, your CodeBuild containers are launched outside your VPC 
* It cannot access resources in a VPC
* You can specify a VPC configuration: 
  * VPC ID 
  * Subnet IDs 
  * Security Group IDs 
* Then your build can access resources in your VPC (e.g., RDS, ElastiCache, EC2, ALB...)
* Use cases: integration tests, data query, internal load balancers..

###### Validate Pull Requests

* Validate proposed code changes in PRs before they get merged🔀
* Ensure high level of code quality and avoid code conflicts

![CodeBuild – Validate Pull Requests](/images/uploads/codebuild-validate-pr.png)

###### Test Reports

{{< figure src="images/uploads/codebuild-test-reports.png" width="250" height="250" class="alignright">}}

* Contains details about tests that are run during builds
* Unit tests, configuration tests, functional tests
* Create your test cases with any test framework that can create report files in the following format:
  * JUnit XML, NUnit XML, NUnit3 XML
  * Cucumber JSON, TestNG XML, Visual Studio TRX
* Create a test report and add a Report Group name in ```buildspec.yml``` file with information about your tests

## CodeDeploy

AWS CodeDeploy is a **fully managed** service that automates your software deployments to a variety of compute services, including Amazon EC2, AWS Fargate, AWS Lambda, or on-premises servers. 

###### Deployment Types

When using CodeDeploy, there are two types of deployments available to you: **in-place** and <span style="color:blue">blue</span>/<span style="color:green">green</span>.

{{< tabs name="Deployment Types" >}}
{{% tab name="In-Place" %}}
* The application on each instance is **stopped**, the latest application revision is installed, and the **new** version of the application is started and validated. 
* Only deployments that use the Amazon ```EC2``` or ```on-premises``` compute platform can use **in-place** deployments.

![CodeDeploy In-Place](/images/uploads/codedeploy-inplace.png)

{{% /tab %}}
{{% tab name="Blue/Green" %}}
* A <span style="color:blue">blue</span>/<span style="color:green">green</span> deployment is used to update your applications while minimizing interruptions caused by the changes of a new application version. 
* CodeDeploy provisions your **new** application version alongside the **old** version before **rerouting** your production traffic. 
* This means during deployment, you’ll have **two** versions of your application running at the same time.
* When using a <span style="color:blue">blue</span>/<span style="color:green">green</span> deployment, you have several options for shifting traffic to the new green environment: ```Linear```, ```Canary```, ```AllAtOnce```.
* All ```Lambda``` and Amazon ```ECS``` deployments are <span style="color:blue">blue</span>/<span style="color:green">green</span>. An Amazon ```EC2``` or ```on-premises``` deployment can be in-place or blue/green. 

![CodeDeploy Blue/Green](/images/uploads/codedeploy-bluegreen.png)

{{% /tab %}}
{{< /tabs >}}

###### How it works

To automate the deployment to the appropriate compute resources, ```CodeDeploy``` needs to know:
- 👉which files to copy - ➡️ **appSpec.yml**
- 👉what scripts to run - ➡️ **deployment configuration**
- 👉where to deploy &emsp;- ➡️ **deployment group**

![CodeDeploy Overview](/images/uploads/codedeploy-overview.png)

The concept of an ```application``` is used by CodeDeploy to ensure it knows what to deploy (**code**), where to deploy (**deployment group**), and how to deploy (**deployment configuration**).

###### Deployment group
  - A deployment group specifies the deployment targeted environment. The information it contains is specific to the target compute platform: AWS Lambda, Amazon ECS, Amazon EC2, or on-premises. 
  - A CodeDeploy ```application``` can have one or more deployment groups.
  - Security needs to be assigned so the environment can communicate with CodeDeploy.
  - The CodeDeploy ```agent``` is needed if you are deploying to Amazon EC2 or an on-premises compute platform. It is installed and configured on the target instances. It accepts and executes requests on behalf of CodeDeploy.

###### Deployment configuration
  - A deployment configuration is a set of deployment ```rules``` and deployment success and failure conditions used by AWS CodeDeploy during a deployment. 
  
  - CodeDeploy can deploy your application on **EC2** instances, **ECS** containers, **Lambda** functions, and even an **on-premises** environment. Each deployment platform requires a deployment configuration. CodeDeploy has **predefined** deployment configurations that are unique to each compute platform:

  {{< tabs name="Deployment Configurations" >}}
  {{% tab name="EC2 or on-premises" %}}
  * CodeDeployDefault.**AllAtOnce**
    * Attempts to deploy an application revision to as many instances as possible at once
  * CodeDeployDefault.**HalfAtATime**
    * Deploys to up to half of the instances at a time (with fractions rounded down)
  * CodeDeployDefault.**OneAtATime**
    * Deploys the application revision to only one instance at a time
  {{% /tab %}}
  {{% tab name="ECS containers" %}}
  * CodeDeployDefault.**ECSLinear10PercentEvery1Minutes**
    * Shifts 10 percent of traffic every minute until all traffic is shifted
  * CodeDeployDefault.**ECSLinear10PercentEvery3Minutes**
    * Shifts 10 percent of traffic every 3 minutes until all traffic is shifted
  * CodeDeployDefault.**ECSCanary10Percent5Minutes**
    * Shifts 10 percent of traffic in the first increment, and the remaining 90 
  percent is deployed 5 minutes later
  * CodeDeployDefault.**ECSCanary10Percent15Minutes** 
    * Shifts 10 percent of traffic in the first increment, and the remaining 90 
  percent is deployed 15 minutes later
  * CodeDeployDefault.**ECSAllAtOnce**
    * Shifts all traffic to the updated Amazon ECS container at once
  {{% /tab %}}
  {{% tab name="Lambda functions" %}}
  * CodeDeployDefault.**LambdaLinear10PercentEvery1Minute**
    * Shifts 10 percent of traffic every minute until all traffic is shifted
  * CodeDeployDefault.**LambdaLinear10PercentEvery2Minutes**
    * Shifts 10 percent of traffic every 2 minutes until all traffic is shifted
  * CodeDeployDefault.**LambdaLinear10PercentEvery3Minutes**
    * Shifts 10 percent of traffic every 3 minutes until all traffic is shifted
  * CodeDeployDefault.**LambdaLinear10PercentEvery10Minutes** 
    * Shifts 10 percent of traffic every 10 minutes until all traffic is shifted
  * CodeDeployDefault.**LambdaCanary10Percent5Minutes**
    * Shifts 10 percent of traffic in the first increment, and the remaining 90 
  percent is deployed 5 minutes later
  * CodeDeployDefault.**LambdaCanary10Percent10Minutes**
    * Shifts 10 percent of traffic in the first increment, and the remaining 90 
  percent is deployed 10 minutes later
  * CodeDeployDefault.**LambdaCanary10Percent15Minutes**
    * Shifts 10 percent of traffic in the first increment, and the remaining 90 
  percent is deployed 15 minutes later
  * CodeDeployDefault.**LambdaCanary10Percent30Minutes**
    * Shifts 10 percent of traffic in the first increment, and the remaining 90 
  percent is deployed 30 minutes later
  * CodeDeployDefault.**LambdaAllAtOnce** 
    * Shifts all traffic to the updated Lambda functions at once  
  {{% /tab %}}
  {{< /tabs >}}

###### Code

  - Identify the correct version🔖 (revision) of the code.
  - With the code, you provide an application specification file ```appSpec.yml``` file which is used to manage each deployment. During deployment, CodeDeploy looks for your AppSpec file in the **root** directory of the application's source. 
  - The AppSpec file specifies where to copy the code and how to get it running. 
  - The AppSpec file is used to manage each deployment as a series of lifecycle event ```hooks```, which are defined in the file. 
  - Lifecycle ```hooks``` are areas where you can specify Lambda functions or local scripts to run **tests**, **healthchecks** and verify the deployment of your application was successful. 
  - Some tests might be as simple as checking a dependency before an application is installed using the **BeforeInstall** hook. Some might be as complex as checking your application’s output before allowing production traffic to flow through using the **BeforeAllowTraffic** hook. 
  - The structure of the ```AppSpec``` file can differ depending on the compute platform you choose:
  
  {{< tabs name="Deployment Lifecycle Hooks" >}}
  {{% tab name="EC2 In-Place" %}}
  ![codedeploy-ec2-inplace-hooks](/images/uploads/codedeploy-ec2-inplace-hooks.png)
  {{% /tab %}}
  {{% tab name="EC2 Blue/Green" %}}
  ![codedeploy-ec2-blue/green-hooks](/images/uploads/codedeploy-ec2-bluegreen-hooks.png)
  {{% /tab %}}
  {{% tab name="ECS Blue/Green" %}}
  * **AfterAllowTestTraffic** – run AWS Lambda function after the test ELB
Listener serves traffic to the Replacement ECS Task Set like perform health checks on the application and trigger a rollback if the health checks are not successful
  ![codedeploy-ecs-blue/green-hooks](/images/uploads/codedeploy-ecs-bluegreen-hooks.png)
  {{% /tab %}}
  {{% tab name="Lambda Blue/Green" %}}
  * **BeforeAllowTraffic** and **AfterAllowTraffic** hooks can be used to check
the health of the Lambda function
  ![codedeploy-lambda-blue/green-hooks](/images/uploads/codedeploy-lambda-bluegreen-hooks.png)
  {{% /tab %}}
  {{< /tabs >}}

  
  Here is an example of an AppSpec file for an ```in-place``` deployment to an EC2 instance.

  ```yml
  version: 0.0
  os: linux
  files:
    - source: Config/config.txt
      destination: /webapps/Config
    - source: source
      destination: /webapps/myApp
  hooks:
    BeforeInstall:
      - location: Scripts/UnzipResourceBundle.sh
      - location: Scripts/UnzipDataBundle.sh
    AfterInstall:
      - location: Scripts/RunResourceTests.sh
        timeout: 180
    ApplicationStart:
      - location: Scripts/RunFunctionalTests.sh
        timeout: 3600
    ValidateService:
      - location: Scripts/MonitorService.sh
        timeout: 3600
        runas: codedeployuser
  ```

  Here is an example of an AppSpec file written in YAML for deploying an Amazon ```ECS``` service.

  ```yml
  version: 0.0
  Resources:
    - TargetService:
        Type: AWS::ECS::Service
        Properties:
          TaskDefinition: "arn:aws:ecs:us-east-1:<account-id>:task-definition/task-definition-name:1"
          LoadBalancerInfo:
            ContainerName: "SampleApplicationName"
            ContainerPort: 80
  # Optional properties
          PlatformVersion: "LATEST"
          NetworkConfiguration:
            AwsvpcConfiguration:
              Subnets: ["subnet-1234abcd","subnet-5678abcd"]
              SecurityGroups: ["sg-12345678"]
              AssignPublicIp: "ENABLED"
          CapacityProviderStrategy:
            - Base: 1
              CapacityProvider: "FARGATE_SPOT"
              Weight: 2
            - Base: 0
              CapacityProvider: "FARGATE"
              Weight: 1
  Hooks:
    - BeforeInstall: "LambdaFunctionToValidateBeforeInstall"
    - AfterInstall: "LambdaFunctionToValidateAfterInstall"
    - AfterAllowTestTraffic: "LambdaFunctionToValidateAfterTestTrafficStarts"
    - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeAllowingProductionTraffic"
    - AfterAllowTraffic: "LambdaFunctionToValidateAfterAllowingProductionTraffic"

  ```

  Here is an example of an AppSpec file written in YAML for deploying a ```Lambda``` function version.

  ```yml
  version: 0.0
  Resources:
    - myLambdaFunction:
        Type: AWS::Lambda::Function
        Properties:
          Name: "myLambdaFunction"
          Alias: "myLambdaFunctionAlias"
          CurrentVersion: "1"
          TargetVersion: "2"
  Hooks:
    - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeTrafficShift"
    - AfterAllowTraffic: "LambdaFunctionToValidateAfterTrafficShift"

  ```
###### HealthChecks

Health checks are tests performed on resources. These resources might be your application, compute resources like Amazon Elastic Cloud Compute (Amazon EC2) instances, and even your Elastic Load Balancers.

Health checks can be implemented in the deployment of your application in several different ways. One is with CodeDeploy and the help of your application specification (**AppSpec**) file. 

{{< accordion "Liveness checks" >}}

Liveness checks test the basic connectivity to a service and the presence of a server process. They are often performed by a load balancer or external monitoring agent, and they are unaware of the details about how an application works. 

Some examples of **liveness** checks include:

* Tests that confirm a server is listening on its expected ```port``` and accepting new TCP connections
* Tests that perform basic HTTP requests and make sure the server responds with a ```HTTP 200``` status code
* Status checks for Amazon EC2 that test for network ```reachability``` necessary for any system to operate.

{{< /accordion >}}

{{< accordion "Local health checks" >}}

Local health checks go further than liveness checks to verify that the application is likely to be able to function. 

Some examples of situations local health checks can identify are: 

* Ability to ```write``` to or ```read``` from disk. 
* Missing support processes: Hosts missing their monitoring ```daemons``` might leave operators unaware of the health of their services. 

{{< /accordion >}}

{{< accordion "Dependency health checks" >}}

Dependency health checks thoroughly inspect the ability of an application to interact with its adjacent systems. These checks ideally catch problems local to the server, such as expired credentials, that are preventing it from interacting with a dependency. They can also have ```false positives``` when there are problems with the dependency itself.

* A process might asynchronously look for updates to metadata or configuration but the update mechanism might be broken on a server. The server can become significantly out of sync with its peers.
* Inability to communicate with peer servers or dependencies. Software issues, such as deadlocks or bugs in ``connection pools``, can also hinder network communication.
* Other unusual software bugs that require a process bounce: ``Deadlocks``, ``memory leaks``, or state corruption bugs can make a server spew errors.
{{< /accordion >}}

{{< accordion "Anomaly detection" >}}

Anomaly detection checks all servers in a fleet to determine if any server is behaving oddly compared to its peers. By aggregating monitoring data per server, you can continuously compare error rates, latency data, or other attributes to find anomalous servers and automatically remove them from service. Anomaly detection can find divergence in the fleet that a server cannot detect about itself, such as the following:

* ```Clock skew```: When servers are under high load, their clocks have been known to skew abruptly and drastically. Security measures, such as those used to evaluate signed requests, require that the time on a client's clock is within five minutes of the actual time. If it is not, requests fail.
* ```Old code```: A server might disconnect from the network or power off for a long time. When it comes back on line, it could be running dangerously outdated code that is incompatible with the rest of the fleet.
* Any unanticipated failure mode: Sometimes, servers fail in such a way that they return errors they identify as the client’s instead of theirs (HTTP 400 instead of 500). Servers might slow down instead of failing, or they might respond faster than their peers, which is a sign they’re returning false responses to their callers. 

{{< /accordion >}}

While health checks can identify problems, the key to a successful continuous delivery strategy is to also implement **remediations** when these tests fail. You can build logic into your tests that indicate to CodeDeploy that the deployment was unsuccessful and start the **rollback** process.

  
* Rolling deployment

  - With a rolling deployment, your production fleet is divided into groups so the entire fleet isn’t upgraded all at once. Your fleet will run both the new and existing software versions during the deployment process. 

  - This method enables a zero-downtime update. If the deployment fails, only the upgraded portion of the fleet will be affected.

  - With rolling deployments, you are updating your live production environment.

You can use a variety of rolling deployment options through CodeDeploy:

{{< tabs name="Rolling deployment" >}}
{{% tab name="One-At-A-Time" %}}
Deploys the application revision to only one instance at a time.
{{% /tab %}}
{{% tab name="Half-At-A-Time" %}}
Deploys to as many as half of the instances at a time (with fractions rounded down). The overall deployment succeeds if the application revision is deployed to at least half of the instances (with fractions rounded up). Otherwise, the deployment fails.
{{% /tab %}}
{{% tab name="Custom" %}}
Deploys a set number or percentage of resources selected by you, at time intervals you specify.
{{% /tab %}}
{{< /tabs >}}

There are some obvios **Pros** and **Cons** to Rolling deployment: 

| **Pros**                                                          | **Cons**                                                                                                                                                                                                                                          |
|-------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Zero downtime                                                           | Speed: Because resources are deployed in small increments, it could take a long time to deploy all the necessary hosts in a large environment\.                                                                                                            |
| Lower overall risk of bringing down your entire production application  | Complexity: There are two different application versions during the deployment operating at once in the same environment\. You will need to make sure your application can handle interoperability between these versions\.                                |
| No additional resources required, which minimizes deployment costs      | Rollback: If a resource in a rolling deployment fails to deploy correctly, reverting to a previous version can be complicated\. It might require you to redeploy the previous application version in a new resource or fix the failed resource manually\.  |


With a blue/green deployment, you provision a new set of containers on which CodeDeploy  installs the latest version of your application. CodeDeploy then reroutes load balancer traffic from an existing set of containers running the previous version of your application to the new set of containers running the latest version. After traffic is rerouted to the new containers, the existing containers can be terminated. Blue/green deployments allow you to test the new application version🔖 before sending production traffic to it.

![Codepipeline – BlueGreen](/images/uploads/codepipeline-blue-green.png)

If there is an issue with the newly deployed application version, you can roll back to the previous version faster than with in-place deployments.

## CodePipeline

AWS CodePipeline is a **fully managed** continuous delivery service that enables you to model, visualize, and automate the steps required to release your software. 

-  Visual Workflow to orchestrate your CICD
-  📝**Source** – CodeCommit, ECR, S3, Bitbucket, GitHub
-  🏗️**Build** – CodeBuild, Jenkins, CloudBees, TeamCity
-  🧪**Test** – CodeBuild, AWS Device Farm, 3rd party tools...
-  🚀**Deploy** – CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3...
-  📞**Invoke** – Lambda, Step Functions
-  Consists of stages:
    -  Each stage can have sequential actions and/or parallel actions
    -  Example: Build :arrow_right: Test :arrow_right: Deploy :arrow_right: Load Testing :arrow_right: …
    -  Manual approval can be defined at any stage

![Codepipeline – Artifacts](/images/uploads/codepipeline-artifacts.png)

 

## CodeArtifact

## CodeGuru

Amazon CodeGuru is a service that scans pull requests, gives a description of the issue, and recommends how to remediate it.
CodeGuru uses program analysis and ```machine learning``` to scan code and detect potential defects that are difficult for developers to find on their own. 