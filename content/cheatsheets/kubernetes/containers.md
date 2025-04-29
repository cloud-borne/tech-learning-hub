---
title: Containers
linktitle: Containers
type: book
draft: true
tags:
  - Kubernetes
date: "2022-02-03T00:00:00+01:00"
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 20
---

Running Containers with Kubernetes

Introduction
The primary focus of Kubernetes is running containers. This lab will allow you get hands-on with the core functionality of Kubernetes. You will use Kubernetes Pods to run containers in an existing Kubernetes cluster. This will allow you to practice your basic Kubernetes skills.

Solution
Log in to the lab server using the credentials provided:

ssh cloud_user@<PUBLIC_IP_ADDRESS>
Note: When copying and pasting code into Vim from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes. To save and quit the file, press Escape followed by :wq. To exit the file without saving, press Escape followed by :q!.

Create an Nginx Pod to Function as a Simple Web Server
All commands are performed on the Control Plane Node.
Create a YAML definition file:
vi nginx-pod.yml
Paste the following contents to the YAML file:
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
Save and exit the file:
:wq
Create a new Pod in the cluster:
kubectl create -f nginx-pod.yml
Run a Redis Instance Using a Kubernetes Pod
Create a YAML definition file:
vi redis-pod.yml
Paste the following contents to the YAML file:
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
Save and exit the file:
:wq
Create a new Pod in the cluster:
kubectl create -f redis-pod.yml
Verify Pods are up and running:
kubectl get pods -o wide
Confirm communication with Nginx Pod using it's IP address:
curl <nginx IP address>
Check Nginx Container Logs
View the logs from the Nginx container:
kubectl logs nginx
Conclusion
Congratulations — you've completed this hands-on lab!
