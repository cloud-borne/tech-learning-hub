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

![AWS-IAM](/images/uploads/aws-iam-background.png)

When deploying applications to AWS, ensuring **robust** security is ``` Job No:1️⃣```, not just to protect user data, but to safeguard internal operations. AWS Identity and Access Management (```IAM```) provides the foundation for managing authentication and authorization across cloud resources.

Accessing AWS services requires two things:
- **Authentication**: Verifying the identity of the requester
- **Authorization**: Granting permission to perform specific actions on resources

## IAM Principals

This diagram gives an overview of how the main entities within IAM relate to each other:
![Users-Groups-Roles](/images/uploads/iam-intro-diagram.png)

In IAM, a **Principal**👥 is any entity that can be authenticated and make requests to AWS services. This includes:

- **IAM Users**
- **IAM Roles**
- **Applications** 
- **Federated Users**

> - User Example: Human users like say a ```sysadmin``` managing different departments with distinct permissions.
> - Role Example: An ```EC2``` instance assumes a role via the metadata service to access S3 securely.
> - Application Example: ```Containers``` using IAM roles for tasks (via ECS Task Roles or IRSA in EKS)
> - Federated Example: ```CI/CD``` pipelines assuming roles for deployment

- **IAM Groups** are permission containers, and **not Principals**. They:
  - Organize users
  - Allow shared policy management
  - **Cannot** authenticate or make API calls
  - **Cannot** be referenced in resource-based policies
  - **Cannot** associate with a Permission Boundary

### 👤 IAM Users

Individuals with long-term **credentials** (username/password or access keys). They represent human identities or service accounts created within your AWS account.

- 🔑 Use Credentials
  - **Username/password** for AWS ```Console``` access
  - **Access keys** for programmatic access (```CLI, SDK```)
- 🧠 Use Cases
  - Human users (e.g., developers, admins)
  - Service accounts for legacy systems

### 🧢 IAM Roles
IAM Roles are temporary identities assumed by trusted entities. IAM Roles operate under two distinct policies, each serving a different purpose:

  - 🔐 Trust Policy (*Who Can Assume the Role*)
    - A Trust Policy defines who is allowed to assume the role. It’s attached to the role and evaluated by AWS STS during role assumption.
    - Trust policies are written in JSON and use the ```Principal``` element to specify trusted entities.
   
      Example: **EC2** Trust Policy
      ```json
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
          }
        ]
      }
      ```

    - Assumption Mechanism:
      - Use AWS STS (Security Token Service) to retrieve credentials and impersonate the IAM Role you have access to (AssumeRole API)
      - Temporary credentials can be valid between 15 minutes to 12 hour

  - 📜IAM Policy – (*What the role can do once assumed*)
    - Defines the permissions granted once the role is assumed
    - Can be **inline** or **managed** policies
    - Controls access to AWS resources

      Example: IAM Policy attached to Role
      ```json
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::my-secure-bucket"
          }
        ]
      }
      ```
  - 🧠 Use Cases
    - EC2 accessing S3 without credentials
    - Lambda functions interacting with DynamoDB
    - Cross-account access
    - CI/CD pipelines assuming deployment roles

### 🧱 Applications

Applications often act as **Principals** by assuming IAM Roles to access AWS services securely.
- Assume roles via SDK or CLI  
- Use environment variables or metadata endpoints

🧠 Use Cases
- Backend services calling AWS APIs  
- Microservices accessing SQS/SNS  
- Containers using IAM roles for tasks (via ECS Task Roles or IRSA in EKS)

### 🌐 Federated Users

Federated users are **external identities** authenticated via identity providers (IdPs) and mapped to temporary AWS credentials.

- Federation via `sts:AssumeRoleWithSAML` or `sts:AssumeRoleWithWebIdentity`  
- Trust policy must reference the IdP ARN. 
- Example: GitHub OIDC Trust Policy
  ```json
  {
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::1234567890:oidc-provider/token.actions.githubusercontent.com"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:ref:refs/heads/main"
      }
    }
  }
  ```

Federation Types:
- **SAML 2.0**: Active Directory, ADFS  
- **OIDC**: GitHub Actions, Google Workspace  
- **Cognito**: Mobile/Web app users

🧠 Use Cases
- Employees accessing AWS Console via corporate SSO  
- CI/CD pipelines assuming roles for deployment  
- External partners accessing shared resources

## IAM Policies

- IAM **Policies** define what actions a **Principal**👥 can perform and on which resources. They are attached to IAM Users, Roles, or Groups.

- By default, all access is denied. Explicit `Allow` grants access; `Deny` overrides it.

- **JSON** Elements:

  | Element      | Description                                                                 |
  |--------------|-----------------------------------------------------------------------------|
  | `Version`    | Policy language version (`2012-10-17` is current)                           |
  | `Statement`  | One or more permission blocks                                               |
  | `Effect`     | Either `"Allow"` or `"Deny"`                                                |
  | `Action`     | List of API operations (e.g., `"s3:GetObject"`)                             |
  | `Resource`   | ARN(s) of the resources the action applies to                               |
  | `Condition`  | Optional block to refine when the policy applies                            |

  🔓 Example: **AmazonSQSFullAccess** (AWS Managed Policy)
  
  Grants full access to all SQS queues in the account.

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "sqs:*",
        "Resource": "*"
      }
    ]
  }
  ```

  🎯 Example: **CustomSQSQueuePolicy** (Customer Managed Policy)

  Scoped limited permissions to a single queue.

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "sqs:SendMessage",
          "sqs:ReceiveMessage",
          "sqs:DeleteMessage",
          "sqs:GetQueueAttributes"
        ],
        "Resource": "arn:aws:sqs:us-east-2:123456789012:customSqsQueue"
      }
    ]
  }
  ```

- The `Condition` block allows fine-grained control over when a policy applies. It supports multiple operators and keys.

  **Syntax**
  ```json
  "Condition": {
    "{ConditionOperator}": {
      "{ConditionKey}": "{ConditionValue}"
    }
  }
  ```
  
  Examples: 
  - String Conditions:
    ```json
    "Condition": {
      "StringEquals": {
        "aws:PrincipalTag/job-category": "iamuser-admin"
      }
    }
    ```
    
    ```json
    "Condition": {
      "StringLike": {
        "s3:prefix": [ "", "home/", "home/${aws:username}/" ]
      }
    }
    ```
  - Numeric Conditions
    ```json
    "Condition": {
      "NumericLessThan": {
        "s3:max-keys": "1000"
      }
    }
    ```
  - Date Conditions
    ```json
    "Condition": {
      "DateLessThan": {
        "aws:TokenIssueTime": "2025-09-30T00:00:00Z"
      }
    }
    ```

  - Boolean Conditions
    ```json
    "Condition": {
      "Bool": {
        "aws:SecureTransport": "true"
      }
    }
    ```

    ```json
    "Condition": {
      "Bool": {
        "aws:MultiFactorAuthPresent": "true"
      }
    }
    ```

  - IP Address Conditions
    ```json
    "Condition": {
      "IpAddress": {
        "aws:SourceIp": "203.0.113.0/24"
      }
    }
    ```

  - ARN Conditions
    ```json
    "Condition": {
      "ArnEquals": {
        "aws:SourceArn": "arn:aws:s3:::my-bucket"
      }
    }
    ```

{{% callout note %}}

**PowerUserAccess**
- Provide full access to AWS services and resources, but does not allow management of Users and groups. 
- Note how ```NotAction``` is used instead of ```Deny```

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "NotAction": [
          "iam:*",
          "organizations:*",
          "account:*"
        ],
        "Resource": "*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "account:GetAccountInformation",
          "account:GetPrimaryEmail",
          "account:ListRegions",
          "iam:CreateServiceLinkedRole",
          "iam:DeleteServiceLinkedRole",
          "iam:ListRoles",
          "organizations:DescribeEffectivePolicy",
          "organizations:DescribeOrganization"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

{{% /callout %}}

## Permission Boundaries

Permission boundaries are **IAM controls** that define the maximum permissions a **Principal** (user or role) can have—even if their attached policies allow more.

- Boundaries are **managed policies** attached to IAM users or roles  
  ![IAM-Permission-Boundaries](/images/uploads/aws-iam-permissions-boundary.png)

- Can be used in combinations of AWS Organizations SCP
{{< figure src="images/uploads/aws-iam-effective-permissions.png" class="alignright">}}
- AWS evaluates:
  - The **permissions policy** attached to the principal  
  - The **permission boundary**  
  - Organizational **SCP** policies
- The **intersection** of the 3 determines the effective permissions

  🧠 Use Cases
  - Delegate IAM access safely  
  - Enforce guardrails for automation  
  - Limit scope of temporary elevated access

## STS

AWS Security Token Service (```STS```) enables you to request temporary, limited-privilege credentials for users.

- ```STS``` Important APIs:

   {{< figure src="images/uploads/aws-iam-assume-role.png" class="alignright">}}
  - **AssumeRole**: access a role within your account or cross-account
    - Provide access for an IAM user in your **own** AWS account or to another AWS account in the same AWS Organization (```Zone of Trust```) to access resources.
    - Provide access to IAM users in AWS accounts owned by **third parties**
    - Your users must actively **switch** to the role using the AWS Management Console or assume the role using the AWS CLI or  AWS API
    - You can add **multi-factor** authentication (MFA) protection to the role.
    - Ability to revoke active sessions and credentials for a role (by adding a policy using a time statement – ```AWSRevokeOlderSessions```)
    - When you **assume** a role (user, application or service), you **give up** your original permissions and take the permissions assigned to the Role


  - **AssumeRoleWithSAML**: return credentials for users logged with SAML
  - **AssumeRoleWithWebIdentity**: return creds for users logged with an IdP
    - Example providers include Amazon Cognito, Login with Amazon, Facebook, Google, or any OpenID Connect-compatible identity provider
    - AWS recommends using Cognito instead
  - **GetSessionToken**: for MFA, from a user or AWS account root user
  - **GetFederationToken**: obtain temporary creds for a federated user, usually a proxy app that will give the creds to a distributed app inside a corporate network


## Resource Policies

## IAM Roles vs Resource Based Policies

## IAM Access Analyzer

## IAM Access Advisor

## CredentialsReport

## Further Read

[Connecting to an RDS or Aurora database with IAM authentication](https://www.capside.com/labs/rds-aurora-database-with-iam-authentication/)

[Switching to a role (console)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-console.html)

[AWS - Cross Account access using #IAM #role](https://www.youtube.com/watch?v=n1r9Fp7GKvk)

[Lambda function to assume a role from another AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-function-assume-iam-role/)
