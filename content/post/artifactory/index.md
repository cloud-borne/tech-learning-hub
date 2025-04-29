---
title: Install Artifactory via Docker Compose
subtitle: Steps to install Artifactory OSS in an Ubuntu VM.

# Summary for listings and search engines
summary: Steps to install Artifactory OSS in an Ubuntu VM.

# Link this post with a project
projects: []

# Date published
date: "2021-10-26T00:00:00Z"

toc: true

# Date updated
lastmod: "2021-10-26T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: ''
  focal_point: ""
  placement: 2
  preview_only: false

authors:
- admin

tags:
- DevOps
- VirtualBox
- Vagrant

categories:
- Demo
---

### Overview

I would be following the steps as explained in the [JFrog Wiki](https://www.jfrog.com/confluence/display/JFROG/Installing+Artifactory#InstallingArtifactory-docker_volumes).
My goal was to get artifactory oss running on a Ubuntu VM in a quick start way and explore some of the inner workings under the hood. Obviously if you are looking to get going with the JFrog platform for an enterprise I would recommend reviewing [documentation](https://www.jfrog.com/confluence/display/JFROG/Installing+the+JFrog+Platform) and start [here](https://jfrog.com/artifactory/)

### Prerequisites

The things that you will need:

- 👉 [Docker](https://docs.docker.com/engine/install/ubuntu/)
- 👉 [DockerCompose](https://docs.docker.com/compose/install/)
- 👉 [Vagrant](https://www.vagrantup.com/downloads)
- 👉 [VirtualBox](https://www.virtualbox.org/wiki/Downloads/)

Below are the steps I followed. You could follow along if you have got the prerequisites ready.
If not jump to the [vagrant file](/post/artifactory/#vagrant) section where I script all these steps along with the prerequisites.

### Step 1: Docker volumes

Create named Docker volumes so that you could manage the artifactory data and do backup/restore/upgrade easily.

```
docker volume create --name=artifactory_data
docker volume create --name=postgres_data
```

### Step 2: Docker Compose archive

Go to the [download](https://jfrog.com/open-source/) page, click the green arrow to download Docker Compose. Extract the contents of the compressed archive (.tar.gz file) and then go to the extracted folder. At the time of writing this post the latest artifactory oss release was "7.27.6"

```
wget https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/docker/jfrog-artifactory-oss/7.27.6/jfrog-artifactory-oss-7.27.6-compose.tar.gz
tar -xvf  jfrog-artifactory-oss-7.27.6-compose.tar.gz
```

{{% callout note %}}
.env file included within the Docker-Compose archive. This .env file is used by docker-compose and is updated during installations and upgrades.
{{% /callout %}}

### Step 3: Setup docker-compose.yaml
Copy the docker-compose-volumes.yaml to the extracted folder.
```
cp templates/docker-compose-volumes.yaml docker-compose.yaml
```

### Step 4: Add the entries in the .env file.       

```
echo -e "JF_SHARED_NODE_IP=$(hostname -i)" >> .env
echo -e "JF_SHARED_NODE_ID=$(hostname -s)" >> .env
echo -e "JF_SHARED_NODE_NAME=$(hostname -s)" >> .env
```

### Step 5: Manage Artifactory

Using native Docker Compose commands from the extracted folder.

```
docker-compose -p rt up -d
docker-compose -p rt ps
docker-compose -p rt down
```

### Vagrant

Wouldn't it be nice to get all of this going at one go! I thought so too. So I created a Vagrant file just for that.
Check it out in [GitHub](https://github.com/avijitliberty/vagrant-virtualbox-docker-artifactory.git)

All you would need to do is clone the repo and do:
```
vagrant up
```

That's it! :dash: And you have a working standalone installation of Artifactory OSS.

Login at ```http://SERVER_IP:8082/ui/```

### Next steps

There were few other steps I had to take to get artifactory working in a meaningful way in my machine:

* Specify [File Handle Allocation Limit](https://www.jfrog.com/confluence/display/JFROG/System+Requirements#SystemRequirements-Xray-FileHandleAllocationLimit)
* Specify [JVM Memory Allocation](https://www.jfrog.com/confluence/display/JFROG/System+Requirements#SystemRequirements-Java)
* Specify [Log Rotation](https://www.jfrog.com/confluence/display/JFROG/Logging#Logging-ConsoleLog.1)

JFrog is the numero uno in the enterprise as a binary repository management and there's a lot to explore here.

Start with [JFROG Wiki](https://www.jfrog.com/confluence/display/JFROG/Get+Started)
