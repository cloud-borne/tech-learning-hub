---
title: KubeCtl
linktitle: CLI
type: book
tags:
  - Kubernetes
date: "2022-02-03T00:00:00+01:00"
draft: true
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 100
---

Working with Kubernetes Using kubectl

<!--more-->

### Overview
This hands-on lab will provide you with the opportunity to practice your basic kubectl skills in a real Kubernetes cluster. You will create and manipulate some simple Kubernetes objects using kubectl. This will allow you to exercise your kubectl skills.

Solution
Log in to the lab server using the credentials provided:

ssh cloud_user@<PUBLIC_IP_ADDRESS>
Note: When copying and pasting code into Vim from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes. To save and quit the file, press Escape followed by :wq. To exit the file without saving, press Escape followed by :q!.

Use kubectl to Check the Status of Worker Nodes in the Cluster
All commands are performed on the Control Plane Node.
Check the status of all the nodes in the cluster and verify all nodes are in READY status:
kubectl get nodes
Get a List of Pods Running in the Cluster
List the Pods in all namespaces:
kubectl get pods --all-namespaces
Delete an Existing Pod
Delete the Pod named web-frontend:
kubectl delete pod web-frontend
Create a New Pod
Create a new Pod called web-frontend using YAML file:
vi web-frontend.yml
Paste the following contents to the YAML file:
apiVersion: v1
kind: Pod
metadata:
  name: web-frontend
spec:
  containers:
  - name: nginx
    image: nginx
Save and exit the file:
:wq
Create a new pod:
kubectl create -f web-frontend.yml
Confirm everything is working as expected:
kubectl get pod web-frontend
Conclusion
Congratulations — you've completed this hands-on lab!

Tools
