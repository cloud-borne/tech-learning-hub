---
title: StatefulSets
linktitle: StatefulSets
type: book
date: "2023-07-27T00:00:00+01:00"
tags:
  - Docker
  - Kubernetes
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 50
---

Kubernetes StatefulSets

<!--more-->

{{% callout soon %}}
Coming soon...
{{% /callout %}}

Run a Collection of Stateful Pods That Need a Sticky Identity

Create a YAML file called legacy.yml to hold the manifest for a headless service as well as a YAML file for the StatefulSet deployment.

```
vi legacy.yml
```

```yml

apiVersion: v1
kind: Service
metadata:
  name: legacy-svc
  labels:
spec:
  ports:
  - name: web
    port: 80
  clusterIP: None
  selector:
    app: legacy-svc
    
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: legacy
spec:
  selector:
    matchLabels:
      app: legacy
  serviceName: legacy-svc
  replicas: 5
  template:
    metadata:
      labels:
        app: legacy
    spec:
      containers:
      - name: nginx
        image: nginx:stable

```
