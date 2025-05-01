---
title: IAM
linktitle: 🔐 IAM
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 20
draft: false
---

Manage access to AWS resources

<!--more-->

## Overview

When deploying applications to a cloud service such as AWS, reliable security
concepts are key. After all, we not only want to protect our users’ data but also
make sure that security within our organization isn’t compromised.

With AWS Identity and Access Management (```IAM```), we’re able to address these
concerns for our cloud applications and in the larger context of security and
access management within an organization.

Using AWS services requires both **authentication** and appropriate **authorization** (or: privileges) for accessing a particular resource

## Users, Groups, and Roles

In computer security, an abstract entity that can be authenticated is often
referred to as a ```principal```.

This diagram gives an overview of how the main entities within IAM relate to
each other:
![Users-Groups-Roles](/images/uploads/aws-iam-users-groups-roles.png)

In the context of AWS and IAM, a ```principal``` can be a **person** or an **application**
identified by either user credentials or an associated role. Users are individuals
with an identity and ```credentials``` (usually username and password) whereas
```roles``` don’t have credentials but are usually assumed temporarily by either
applications or already authenticated users. 

Roles, therefore, allow us to assign permissions without storing AWS user credentials with an application. Roles also allow already authenticated AWS users to switch between different sets of
permissions based on their current task or context (think of a system administrator working for several departments within an organization, for example).

Allowing or restricting access to a resource is a two-part process consisting of
authentication and subsequent authorization. First, a principal needs to identify through authentication. The most common way of doing so is via providing both username and password. 

Keep in mind, though, that with IAM both users and roles can be principals, so a valid username/password combination is just one method of authentication. 

Once successfully authenticated, a principal can access the resources they are authorized for. For each request, AWS will check if the principal is permitted to perform the request action on the requested
resource, as given by the resource’s Amazon Resource Name (```ARN```)

## Policies

Once we’ve created our IAM user(s) for our daily work with AWS we can proceed to grant them privileges to AWS resources. Following the principle of **least privilege**, we should prefer adding permissions as needed instead of granting blanket permissions.

We can grant privileges by attaching so-called ```policies``` to a user, role, or group. We can revoke privileges by detaching the policies again. For many common access scenarios, AWS provides ready-to-use policies managed by AWS itself.
Some example policies are:
• ```AmazonEC2FullAccess```: All permissions required for creating and managing EC2 resources.
• ```AmazonSQSReadOnlyAccess```: Read-only permissions to SQS resources.
• ```SystemAdministrator```: All permissions required for common operations tasks.

If, for instance, we want our developers to have unrestricted access to EC2 we can grant them the ```AmazonEC2FullAccess``` policy. It’s worth noting, however, that IAM has a limit of **10 policies** per user, role, or group.

This example shows the JSON representation of the managed ```AmazonEC2FullAccess``` policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "ec2:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "elasticloadbalancing:*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "cloudwatch:*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "autoscaling:*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:CreateServiceLinkedRole",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "iam:AWSServiceName": [
            "autoscaling.amazonaws.com",
            "ec2scheduled.amazonaws.com",
            "elasticloadbalancing.amazonaws.com",
            "spot.amazonaws.com",
            "spotfleet.amazonaws.com",
            "transitgateway.amazonaws.com"
          ]
        }
      }
    }
  ]
}
```

```Policies``` consist of a collection of Statements, each of which has ```Action```, ```Effect```,
and ```Resource``` properties plus an optional ```Condition``` property.

The ```Effect``` property can have a value of **Allow** or **Deny**. By default, access to resources is denied. When setting Effect to Allow we’re granting access to the resource(s) specified by the Resource property of the statement. 

Assigning Effect a value of Deny overrides a previous Allow again.

The ```Resource``` property can either take a full ARN or a partial ARN with wildcard characters for matching multiple resources (“*” for any number of characters or “?” for any single character).
In the ```AmazonEC2FullAccess``` policy above, the Resource is set to * in all statements, giving access to all resources. If we want to restrict the resources that a statement is valid for, we could instead use arn:aws:sqs:us-east2:<ACCOUNT_ID>:mySqsQueue to only grant access to a specific SQS queue, for
example.

The ```Action``` property allows us to specify the exact action(s) to be allowed on a resource. An action consists of a namespace belonging to an AWS service and an action name under that namespace.
The first statement in the AmazonEC2FullAccess policy above, for example, grants permission to the actions ```ec2:*```, i.e. all actions in the “ec2” namespace, while the last statement grants permission to the specific action ```iam:CreateServiceLinkedRole``` only.
A statement can also specify a list of actions:

```
"Action": [
"sqs:SendMessage",
"sqs:ReceiveMessage",
"ec2:StartInstances",
"iam:ChangePassword",
"s3:GetObject"
]
```

Finally, the optional ```Condition``` property allows us to apply further constraints
to when a policy takes effect. In the AmazonEC2FullAccess policy above, the action ```iam:CreateServiceLinkedRole``` will only be allowed from one of the AWS services listed under the
StringEquals attribute of the Condition. Once we have created a policy, we can attach it to users, groups, and roles just as we would do with predefined policies.

[Connecting to an RDS or Aurora database with IAM authentication](https://www.capside.com/labs/rds-aurora-database-with-iam-authentication/)

[Switching to a role (console)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-console.html)

[AWS - Cross Account access using #IAM #role](https://www.youtube.com/watch?v=n1r9Fp7GKvk)

[Lambda function to assume a role from another AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-function-assume-iam-role/)
