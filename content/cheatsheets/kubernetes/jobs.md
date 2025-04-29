---
title: Jobs and CronJobs
linktitle: Jobs and CronJobs
type: book
tags:
  - Kubernetes
date: "2022-03-05T00:00:00+01:00"
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 50
---

Running Jobs and CronJobs in Kubernetes

<!--more-->

## Overview

Kubernetes **Jobs** are objects that are designed to run a containerized task successfully to completion.
**CronJob** basically takes that same idea and extends it and allows us to run Jobs periodically, according to a schedule.

## Jobs

Create a simple Job that runs the command echo This is a test! .

```yml
vi my-job.yml
## Job Definition
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: print
        image: busybox:stable
        command: ["echo", "This is a test!"]
      restartPolicy: Never
  backoffLimit: 4
  activeDeadlineSeconds: 10
```

Our container is just going to run its code and then it's going to stop. It's not going to continue to run forever.
That's why we have ```restartPolicy``` set to **Never**, because this container never needs to restart. It just needs to fire and finish.

{{% callout note %}}

* **activeDeadlineSeconds** : This is the maximum number of seconds that Kubernetes will allow the Job to run.
So if our command gives back an error, and our Job doesn't actually complete successfully, Kubernetes will try to spin that container up again
and retry the Job,

* **backofflimit**: It just limits
essentially how many times Kubernetes is going to retry
that Job execution.

{{% /callout %}}

Run the Job.
```
kubectl apply -f my-job.yml
```
Check the status of the Job.
```
kubectl get jobs
```

View the Job output. First, you will need to find the name of the Job's Pod.
```
kubectl get pods
kubectl logs $JOB_POD_NAME
```

## CronJobs

CronJob basically takes that same idea and extends it and allows us to run Jobs periodically, according to a schedule.

Creating a CronJob that will run the previous Job task every minute is like so: Eerything under the Job template is pretty much everything we used
when we just created the Job. In addition we have the ```schedule``` field, the value of which uses the Cron expression syntax.

```yml
vi my-cronjob.yml
## Job Definition
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: print
           image: busybox:stable
           command: ["echo", "This is a test!"]
          restartPolicy: Never
      backoffLimit: 4
      activeDeadlineSeconds: 10
```

Run the Job.

```
kubectl apply -f my-cronjob.yml
```

Check the CronJob status.
```
kubectl get cronjob
```
View the Job output. First, you will need to find the name of the Job's Pod.
```
kubectl get pods
kubectl logs $JOB_POD_NAME
```

Delete pods
```
$ kubectl delete -f multi-pod.yml
```