---
title: DevOps
summary: Project to demonstrate efficient use of DevOps Tool Chain
tags:
- Jenkins
- GitHub
- Artifactory
- Maven

date: "2021-04-27T00:00:00Z"
draft: true
toc: true

weight: 100

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption:
  focal_point: Smart

---

## Overview

### Install DevOps Toolchain

1. Setup Artifactory
   - Startup Artifactory
   - Setup Admin Account
   - Port 8082
2. Setup Tomcat
   - Startup Tomcat
   - Update tomcat-users.xml
   - Port 8080
3. Setup Jenkins
   - Startup Jenkins
   - Port 8090
4. Setup Maven
   - Startup MAVEN_HOME
   - Generate maven settings.xml and settings-security.xml

   mvn -emp <artifactory-password>
   mvn -ep <artifactory-password>

#### Create Artifactory repository

![](/images/uploads/CreateRepo-Step1.png)
![](/images/uploads/CreateRepo-Step2.png)
![](/images/uploads/CreateRepo-Step3.png)
![](/images/uploads/CreateRepo-Step4.png)

#### Development

1. Setup SSH for GitHub.
[Generating a new SSH key](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

2. Create a GitHub repository.
3. Create a Spring Boot Project.
4. Import Project to STS.
5. Configure for SCM and Artifactory.

{{% callout note %}}

We would be reusing this project to showcase the full [DevOps](/project/devops/) pipeline including Git, Jenkins and Artifactory and more.
To facilitate these activities we had made these additional changes:

- distributionManagement - Tells Maven that these are the repositories where I wish to push my code i.e springboot-junit-libs-release-local for releases and springboot-junit-libs-snapshot-local for the snapshot builds.

- scm - Tells Maven the Source Code Management repository we wish to connect to, [GitHub](https://github.com/avijitliberty/springboot-junit) in this case

- plugin - maven-release-plugin will help with releasing of the code to the releases repository.
{{% /callout %}}

6. Commit changes to the feature branch and push to GitHub.
7. Create a pull request to develop branch and merge.

#### Continuous Integration

1. SSH authentication between GitHub and Jenkins
2. Install and Configure Jenkins Plugins.
   - Conditional Buildstep Plugin
   - Deploy to container Plugin
   - Environment Injector Plugin
   - Git Parameter Plugin
   - GitHub Branch Source Plugin
3. Build SNAPSHOT version using Jenkins.
4. Configure Jenkins to Deploy to Tomcat.
