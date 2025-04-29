---
title: EventBridge
linktitle: 🛜 EventBridge
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params*toml`)
weight: 140
---

Eventing in AWS

<!--more-->

## Overview

Amazon EventBridge is a service that offers us a **serverless** event bus with ```real-time``` access to changes in our AWS environments, custom applications, or third-party SaaS vendors.

```EventBridge``` was the evolution essentially of the ```CloudWatch``` Events service offering which used a single **default** bus for all events and rules. One of the beautiful things about EventBridge is that it uses the same **CloudWatch Events API** to allow for a smoother transition with our applications and our system design.

## Key Components

###### Event Bus🚌:

```Serverless``` pipeline that receives and sends events from different sources. These can be **default** AWS services, **custom** sources, or they can be **third-party** sources.

* **Default** - Created at account creation, 1 per region. Used formerly by **CloudWatch** Events.
* **Custom** - Created by users. They can be **cross-account** or **cross-region** as well if you define appropriate ```Resource Policy``` like so:

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "allow_all_accounts_from_organization_to_put_events",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "events:PutEvents",
        "Resource": "arn:aws:events:us-east-1:<ACCOUNT_ID>:event-bus/custom-event-bus",
        "Condition": {
          "StringEquals": {
            "aws:PrincipalOrgID": "ou-12345"
          }
        }
      }
    ]
  }
  ```
* **SaaS vendors** - Third-Party event buses like SalesForce, Datadog and others*

  ![EventBridge-EventBus](/images/uploads/eventbridge-eventbus.png)


###### Events💬:

There are 3 main components to an event which we need to lookout for when creating rules to target events: 

* **detail** - A JSON object that contains information about the ```event```. The service generating the event determines the content of this field.
* **source** - The source field is meant to identify what ```source``` the event came from. All events that originate from other AWS services begin with **aws.**
* **detail_type** - The detail-type is related to the source AWS service or customer-generated event source. The ```detail-type``` identifies, in combination with the ```source``` field, what ```event``` occurred.

Here's an example of a EC2 Termination event: 

```json
{
	"version": "0",
	"id": "ec55f137-57e0-43e0-a9fa-df5d188c17d0",
	"detail-type": "EC2 Instance State-change Notification",
	"source": "aws*ec2",
	"account": "123456789101",
	"time": "2022-02-22T18:43:48Z",
	"region": "us-east-1",
	"resources": [
		"arn:aws:ec2:us-east-1:123456789101:instance/i-1234567890abcdef0"
	],
	"detail": {
		"instance-id": "i-1234567890abcdef0",
		"state": "terminated"
	}
}
```
###### Rules📝:

```Rules``` are created and associated with ```Event Buses``` and these rules are evaluated whenever an event is received. If an event matches a rule, it will trigger that action and send the events to their designated **targets**. There's ```fan-out``` pattern possible i.e single rule triggering multiple targets. 

Create rules the following ways:

* **Event pattern** 

  * EventBridge supports **declarative** content filtering using event patterns. Rules use ```event patterns``` to select events and send them to targets. An event pattern either matches an event or it doesn't. 

    The following event pattern processes all Amazon EC2 instance-termination events.
      ```json
      {
        "source": ["aws.ec2"],
        "detail-type": ["EC2 Instance State-change Notification"],
        "detail": {
          "state": ["terminated"]
        }
      }
      ```
  * You can write complex event patterns (with prefix matching, suffix matching, numeric matching...[^1]) that only match events under very specific circumstances.

    The following event pattern processes all S3 ```Object Created``` events in buckets with names prefixed with ```example-bucket``` and files with ```".png"``` suffix.
      ```json
      {
        "source": ["aws.s3"],
        "detail-type": ["Object Created"],
        "detail": {
          "bucket": {
            "name": [ { "prefix": "example-bucket" } ]
          },
          "object": {
            "key": [ { "suffix": ".png" } ]
          }
        }
      }
      ```
* **Schedule**
  
  * Event Bridge rule can run periodically on a schedule (cron and rate).
  * With EventBridge ```Scheduler```, you can schedule one-time or recurrently tens of millions of tasks across many AWS services without provisioning or managing underlying infrastructure.
  * While you can use rules to schedule tasks, Amazon EventBridge ```Scheduler``` is better suited for scheduling events **at scale**. If you need to schedule tasks that exceed the EventBridge rules **quotas** on the number of rules or invocation throughput, EventBridge Scheduler might fit your needs.

{{% callout note %}}
* There is a limit of **300** rules per event bus.
* You can only create scheduled rules using the **default** event bus.
{{% /callout %}}

###### Targets🎯

* ```Targets``` are resources (like Lambda) and endpoints(like REST API) that EventBridge can send events to after matching a rule.
* For endpoints any ```HTTP``` method is supported besides **CONNECT** and **TRACE**.
* We can **transform**[^2] event inputs, if need be, before events get sent to their destinations.
* Requests cannot extend past **5** seconds or a **timeout** will occur.
* We can configure upto **5** targets for a single EventBridge rule

## Archive and Replay 

* Amazon EventBridge can **capture** past events and **archive**📮them for testing later on. We no longer need to wait for new events to come in.
* We specify event **patterns** to designate which type of events we want to archive
* AWS **encrypts** event data by default using an AWS-owned customer managed key (```CMK```)
* We dictate the **retention** period of an archived event. This can be indefinite!
* We can **replay**📨 the archived events when need be. Obviosly this is huge for testing and troubleshooting. We can specify the following options when replaying:
  * Source archive 
  * Start and end time for the event replay
  * Target event bus (currently limited to source event bus)
  * Replays processed events based on the event time and repeats in 1-minute intervals
  * Maximum of **10** concurrent replays

## Schema Registry

```Schema```🔗 registries are a container that defines the structure of events that are sent to EventBridge.
* We can create and upload our own schemas, or we can **infer/discover** them based on events within EventBridge.
* Both **OpenAPI 3** and **JSONSchema Draft4** formats are supported.
* Schemas could be versioned.
* **Code Bindings** are downloadable libraries and packages that can be used directly in your code to help develop applications that use events in EventBridge. And we can download them for the language of our choice like Python, Java, Go, Typescript.

## Use Cases

There's a plethora of use cases for ```EventBridge``` integration, here's some examples:

* Reduce Costs by Deleting Orphaned EBS Volumes

* Remediating EC2 Auto Scaling Group Modifications with EventBridge

* Implementing Amazon GuardDuty and Amazon EventBridge

* Triggering Events with CloudTrail Logs

## Event Driven Architectures

Here's a few important motivations for a Event Driven Architecture and EventBridge is at the ```cornerstone``` of this pattern:

**PAY FOR USE**

Event-driven architectures are **push-based**, so everything happens **on-demand** as the event presents itself in the router. This way, you’re not paying💰 for continuous polling to check for an event. This means less network bandwidth consumption, less CPU utilization, less idle fleet capacity, and less SSL/TLS handshakes.

**SCALE AUTOMATICALLY**

Let the cloud provider handle the scaling for you. No more estimating workloads and scaling policies. And if you've been a part of that process, you know it can be a nightmare💤💀

**SCALE AND FAIL** INDEPENDENTLY

By decoupling:chains: your services, they are only aware of the **event router**, not each other. This means that your services are interoperable, but if one service has a failure, the rest will keep running. The event router acts as an elastic buffer that will accommodate surges in workloads.

**AUDITING**

An event router acts as a centralized location to audit🕵 your application and define policies. These policies can restrict who can publish and subscribe to a router and control which users and resources have permission to access your data.

## Further Read

https://aws.amazon.com/blogs/compute/introducing-amazon-eventbridge-scheduler/
https://dev.to/deeheber/eventbridge-emoji-event-patterns-53h4

[^1]:https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns-content-based-filtering.html
[^2]:https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-transform-target-input.html

