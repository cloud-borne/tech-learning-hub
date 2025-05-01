---
title: CDK
linktitle: 🏗️ CDK
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 120
---

AWS Cloud Development Kit (CDK)

<!--more-->

## Overview

CDK acts as an abstraction on top of Cloud Formation Templates that allows developers to use actual declarative languages (Java, JavaScript, Python, TypeScript and .NET are currently supported) to define their cloud infrastructure. The CDK stack file is translated to Cloud Formation which is then used to create your stack. This way, we have a real ```Infrastructure As Code``` solution and don’t have to handle CloudFormation files in YAML or JSON anymore. 

If you consider yourself a **programmer**👨‍💻 and not a **ninja**🥷 of ```YAML``` and ```JSON``` files, you’ll :heart: to learn about the AWS Cloud Development Kit (CDK).

![AWS CDK Overview](/images/uploads/aws-cdk-overview.png)

This example diagram demonstrates the ability of AWS CDK to deploy cloud infrastructure into the AWS Cloud. A developer interacts with AWS CDK and its three main components (App, Stack, and Construct) to synthesize an AWS CloudFormation template, then deploy AWS resources for their organization's users.

Three basic components make up the core framework of an AWS CDK project. 

<details>
  <summary>Constructs</summary>

> * ```Constructs``` are the basic building blocks of AWS CDK apps. 
> * Constructs are classes which define a "piece of system state". Constructs can be composed together to form higher-level building blocks which represent more complex state.
> * Constructs are often used to represent the **desired state** of cloud applications. 

</details>
<details>
  <summary>Stack</summary>
  
> * The unit of deployment in the AWS CDK is called a ```Stack```. All AWS resources defined within the scope of a stack, either directly or indirectly, are provisioned as a single unit.
> * Because AWS CDK stacks are implemented through AWS CloudFormation stacks, they have the same limitations as in AWS CloudFormation.

</details>
<details>
  <summary>App</summary>

> * An ```App``` is a container for one or more stacks: it serves as each stack's scope. Stacks within a single App can easily refer to each others' resources (and attributes of those resources).

</details>

## Installation

- 👉 First install [Node.js](https://nodejs.org/en/download/)
- Then use npm to install the AWS CDK Toolkit.

```bash
npm install -g aws-cdk             # install latest version
npm install -g aws-cdk@X.YY.Z      # install specific version
```      


## Creating the CDK App

```
cdk init app --language=typescript
```
If we look👀 in the generated code we find these 2 classes:
- **cdk-workshop.ts**
```js
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { CdkWorkshopStack } from '../lib/cdk-workshop-stack';

const app = new cdk.App();
new CdkWorkshopStack(app, 'CdkWorkshopStack', {
});
```
- **cdk-workshop-stack.ts**
```js
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
// import * as sqs from 'aws-cdk-lib/aws-sqs';

export class CdkWorkshopStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    // The code that defines your stack goes here
    const vpc = new ec2.Vpc(this, "MyVPC", {maxAzs: 2});
  }
}
```

That’s all the code we need for a working CDK app! That **one line of code** generates the Whole Shebang! i.e VPC, Subnets, Route Tables, Routes, Internet Gateway...🤯

```cdk-workshop.ts``` is the main class of the app. It creates an App
instance and a ```CdkWorkshopStack``` instance where we define our CloudFormation resources.

###### Synthesize the CDK App

Synthesize the code into an AWS CloudFormation template using the the cdk synth command.It would generate the template and write to the **cdk.out** folder.

```bash
cdk synth
```

###### Bootstrapping the AWS Environment

Deploying AWS CDK applications into your AWS account might require that you provision resources the AWS CDK needs to perform the deployment. 

If we don’t bootstrap our AWS environment and, for example, try to deploy a 100 kilobyte AWS CloudFormation template, the deployment will fail. To avoid such scenarios and to complete all possible setup steps at least once, we’re going to bootstrap our AWS environment.

These resources include an **S3** bucket for storing large Lambda functions or other necessary assets and **IAM** roles that grant permissions necessary to perform deployments.

The process of provisioning these initial resources is called ```bootstrapping```. 

```bash
cdk bootstrap aws://YOUR-ACCOUNT-NUMBER/YOUR-REGION --profile YOUR-PROFILE
```

###### Deploying the CDK App

Let’s try to deploy the generated CDK app. This is as easy as executing the ```cdk deploy``` command in the folder of the app.

```
cdk deploy
```
This should have successfully deployed an ```empty``` stack. If we log in
to the AWS web console and navigate to the CloudFormation service, we should
see a stack deployed there:

![AWS CDK Empty Stack](/images/uploads/aws-cdk-empty-stack.png)

The stack contains a single resource called CDKMetadata, which the CDK needs
to work with that stack. 

###### Destroying the CDK App

Before moving on, let’s destroy the stack again with:

```bash
cdk destroy
```

## CDK Commands

Following are a 🧠 list of most used cdk commands:

| **Command**            | **Function **                                                                                |
|------------------------|----------------------------------------------------------------------------------------------|
| cdk list (ls)          | Lists the stacks in the application                                                          |
| cdk synthesize (synth) | Synthesizes and prints the AWS CloudFormation template for the specified stack or stacks     |
| cdk bootstrap          | Deploys the AWS CDK Toolkit stack, required to deploy stacks containing assets               |
| cdk deploy             | Deploys the specified stacks                                                                 |
| cdk destroy            | Destroys the specified stacks                                                                |
| cdk diff               | Compares the specified stack with the deployed stack or a local AWS CloudFormation template  |
| cdk metadata           | Displays metadata about the specified stack                                                  |
| cdk init               | Creates a new AWS CDK project in the current directory from a specified template             |
| cdk context            | Manages cached context values                                                                |
| cdk docs (doc)         | Opens the AWS CDK API reference in your browser                                              |
| cdk doctor             | Checks your AWS CDK project for potential problems                                           |

## Constructs

AWS CDK uses compositions to define complex constructs. A composition establishes a parent-child build hierarchy. The parent composition is composed of child constructs or compositions. This **nesting** pattern can continue with deeply nested compositions in which you define constructs inside other constructs. 

{{< figure src="images/uploads/aws-cdk-construct-tree.png" class="alignright">}}

To enable this pattern, you can define constructs inside the scope of another construct. This scoping pattern results in a hierarchy of constructs known as a construct tree. In the AWS CDK, the root of the tree represents your entire AWS CDK ```App```. Within the app, you typically define one or more stacks. ```Stacks``` are the actual unit of deployment, and are similar to AWS CloudFormation stacks. Stacks are made up of constructs and compositions.

Constructs are implemented in classes that extend the ```Construct``` base class. You define a construct by instantiating the class. All constructs take three parameters when they are initialized: **scope**, **id**, and **props**.

![CDK Construct Definition](/images/uploads/aws-cdk-construct-typescript.png)

<details>
  <summary>Scope</summary>

> The first argument, **scope** is the construct in which this construct is created. In most cases, you define a construct in the scope of itself, which means you usually pass ```this``` for the first argument.

</details>
<details>
  <summary>Id</summary>

> The second argument, **id**, is the local identifier of the construct that must be unique within this scope. AWS CDK uses this identifier to calculate the AWS CloudFormation logical ID for each resource defined within this scope. In this example, ```MyVpc``` is is passed as the ID.

</details>
<details>
  <summary>Props</summary>

> The third argument, **props**, is a set of initialization properties that are specific to each construct and define its initial configuration. For example, the Vpc construct accepts properties such as maxAzs, cidr, and subnetConfiguration etc.

</details>

The AWS CDK includes the [Construct Library](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) which includes constructs that represent all the resources available on AWS.

The library contains three levels of constructs: 

![CDK Construct Levels](/images/uploads/aws-cdk-construct-levels.png)

## Stacks

The unit of deployment in the AWS CDK is called a stack. All AWS resources defined within the scope of a stack, either directly or indirectly, are provisioned as a single unit.

```js
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class CdkWorkshopStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
    const vpc = new ec2.Vpc(this, "MyVPC", {maxAzs: 2});
  }
}
```
AWS CDK gains its deployment power from AWS CloudFormation. However, it is also bound by AWS CloudFormation resource limit of **200** resources. A way around the resource limit is to create a ```Nested``` stack. 
Each nested stack can contain up to 200 resources while only counted as 1 resource in the parent stack. During app synthesis, each nested stack is synthesized into its own AWS CloudFormation template. Nested stacks are not treated as independent deployments and cannot be individually deployed or listed.

![CDK Nested Stacks](/images/uploads/aws-cdk-nested-stacks.png)

This diagram represents an AWS CDK app. Within the app is a stack containing a construct and a nested stack with its own construct. At synthesis, AWS CDK produce two AWS CloudFormation templates, one for each stack. AWS CDK then uses both templates to create the singular deployment.

## App

Every AWS CDK application is represented by the AWS CDK class ```App```. Made up of one or more stacks, Apps can contain one or more constructs. This construct is normally the root of the construct tree. 

```js
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { CdkWorkshopStack } from '../lib/cdk-workshop-stack';

const app = new cdk.App();
new CdkWorkshopStack(app, 'CdkWorkshopStack', {
});
```

You typically define an App instance in your program's entry point, and then define constructs where the app is used as the parent scope. You are able to use the App instance for defining a single instance of your stack.

## Identifiers

AWS CDK uses various types of ```identifiers```. Identifiers must be **unique** within the scope in which they were created. Familiarize yourself with the types of identifiers used:

{{< figure src="images/uploads/aws-cdk-construct-path.png" class="alignright">}}

- **Construct IDs**: 
```id``` is the most common identifier. It is passed as the second argument when instantiating a construct. This identifier must be **unique** only in the ```scope``` in which it is created.
- **Paths**: 
The constructs in an AWS CDK application form a hierarchy rooted in the App class. This hierarchy is called a ```path```. Constructs must have unique IDs at their own ```scope```. This requirement creates a pattern where a path for two different constructs is unique. This uniqueness ensures that the hash generated to form a ```Logical ID``` is also unique.
- **Logical IDs**: 
Unique IDs serve as the logical identifiers of resources in the generated AWS CloudFormation templates for those constructs that represent AWS resources. Logical identifiers are sometimes called logical names.

## Environments

Environment (```env```) represents the AWS **Account** and AWS **Region** in which a stack is deployed. AWS CDK selects the **default** Region and account in your current AWS CLI profile. However, you can manually override the environment [using the ```--profile``` option] by specifying a different set of values than the default. 

```js
  /* If you don't specify 'env', this stack will be environment-agnostic.
   * Account/Region-dependent features and context lookups will not work,
   * but a single synthesized template can be deployed anywhere. */

  /* Uncomment the next line to specialize this stack for the AWS Account
   * and Region that are implied by the current CLI configuration. */
  // env: { account: process.env.CDK_DEFAULT_ACCOUNT, region: process.env.CDK_DEFAULT_REGION },

  /* Uncomment the next line if you know exactly what Account and Region you
   * want to deploy the stack to. */
  // env: { account: '123456789012', region: 'us-east-1' },
```
Constructs deploying with environments using AWS CDK CLI environment variables are considered environment-agnostic and any constructs defined in such a stack cannot use any information about their environment. 

For example, you cannot write code to validate Region because you will not be able to check for a specific Region in the construct itself. These features do not work at all without an explicit environment specified. To use them, you must specify env.

## Context

```Context``` values are **key-value** pairs that can be associated with a stack or construct. AWS CDK uses context to **cache** information from your AWS account, such as the **Availability Zones** in your account or the Amazon Machine Image (**AMI**) IDs used to start your instances. Because these values are provided by your AWS account, they can change between runs of your CDK application. This makes them a potential source of unintended change. The CDK Toolkit's caching behavior "freezes" these values for your CDK app until you decide to accept the new values. You can also create your own context values that your apps or constructs can use. 

Context values that you create are **scoped** to the construct that created them, meaning they are visible to child constructs but not to siblings. Context values set by the AWS CDK Toolkit are set on the **App** construct, and so they are visible to every construct in the app.

To retrieve the cached context values from your AWS account, use the following AWS CDK command: 

```bash
cdk context
```
The resulting information is also visible in various locations, including the ```cdk.json``` project file.
![CDK Context](/images/uploads/aws-cdk-context.png)

Imagine the following scenario without context caching. Let's say you specified "latest Amazon Linux" as the AMI for your Amazon EC2 instances, and a new version of this AMI was released. Then, the next time you deployed your CDK stack, your already-deployed instances would be using the outdated ("wrong") AMI and would need to be upgraded. Upgrading would result in replacing all your existing instances with new ones, which would probably be unexpected and undesired.

{{% callout warning %}}

Because they're part of your application's state, **cdk.json** must be committed to source control along with the rest of your app's source code. Otherwise, deployments in other environments (for example, a CI pipeline) might produce inconsistent results.

{{% /callout %}}

## Assets

```Assets``` are local files, directories, or Docker images that can be bundled into AWS CDK libraries and apps. With AWS CDK, developers can reference assets from **Amazon S3** and **Amazon ECR**.

AWS CDK generates a source hash for assets. You can use the hash at construction time to determine whether the contents of an asset have changed. By default, AWS CDK creates a copy of the asset in the cdk.out directory of your project tree.

Use the ```aws-s3-assets``` module to package and upload these assets. Modules that support assets, such as aws-lambda, have convenient methods that make it easier to use assets. For Lambda functions, you can use the ```lambda.Code.asset``` property to specify directories or zip files as Amazon S3 assets.

```js
import { Asset } from '@aws-cdk/aws-s3-assets';
import * as path from 'path';

const imageAsset = new Asset(this, "SampleAsset", {
  path: path.join(__dirname, "images/my-image.png")
});

new lambda.Function(this, "myLambdaFunction", {
  code: lambda.Code.asset(path.join(__dirname, "handler")),
  runtime: lambda.Runtime.PYTHON_3_6,
  handler: "index.lambda_handler",
  environment: {
    'S3_BUCKET_NAME': imageAsset.s3BucketName,
    'S3_OBJECT_KEY': imageAsset.s3ObjectKey,
    'S3_URL': imageAsset.s3Url
  }
});
```
AWS CDK also supports bundling local Docker images as assets. Images are built from a local Docker context directory using a Dockerfile. Images are then uploaded to **Amazon ECR** by using the AWS CDK CLI or the CI/CD pipeline of your app. You can reference the images in your AWS CDK app.

```js
import * as ecs from '@aws-cdk/aws-ecs';
import * as path from 'path';
import { DockerImageAsset } from '@aws-cdk/aws-ecr-assets';

const asset = new DockerImageAsset(this, 'MyBuildImage', {
  directory: path.join(__dirname, 'my-image')
});
const taskDefinition = new ecs.FargateTaskDefinition(this, "TaskDef", {
  memoryLimitMiB: 1024,
  cpu: 512
});
```
## Testing CDK App

With the AWS CDK, you can test your infrastructure in the same way as any other code you write. 

AWS CDK ships with an ```assert``` library (@aws-cdk/assert) that simplifies making assertions on your infrastructure. All constructs in the AWS Constructs library use the assert library to ensure that the constructs perform as expected.   

## Further Read

* You can learn more of CDK from the 🐎 mouth here: https://docs.aws.amazon.com/cdk/v2/guide/home.html
* “API Reference” in the AWS CDK Reference Guide: https://docs.aws.amazon.com/cdk/api/v2/



