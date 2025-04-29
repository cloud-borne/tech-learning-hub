---
title: Deployments
linktitle: Deployments
type: book
date: "2022-03-26T00:00:00+01:00"
tags:
  - Docker
  - Kubernetes
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 40
---

Kubernetes Deployments

<!--more-->

{{% callout soon %}}
Coming soon...
{{% /callout %}}


Run a Collection of Pods That Can Be Updated with Zero Downtime

Create a YAML file called web.yml to hold the manifest for the deployment:
```
vi web.yml
```

We will start with the basic skeleton of which is available from the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:stable

```
