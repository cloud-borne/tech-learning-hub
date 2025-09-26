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

## 🔎Overview

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

## Documentation
[AWS-CDK](https://docs.aws.amazon.com/cdk/api/v2/) Reference is a wealth of knowledge and something you should refer to often. Open it like so:

```bash
cdk docs
```

## Creating the CDK App

Create an empty directory on your system:
```
mkdir todo-app-cdk-ts && cd todo-app-cdk-ts
```
We will use ```cdk init``` to create a new TypeScript CDK project:
```
cdk init sample-app --language=typescript
```
If we look👀 in the generated code we find these 2 classes:
- **bin/todo-app-cdk-ts.ts**

```js
#!/usr/bin/env node
import * as cdk from 'aws-cdk-lib';
import { TodoAppCdkTsStack } from '../lib/todo-app-cdk-ts-stack';

const app = new cdk.App();
new TodoAppCdkTsStack(app, 'TodoAppCdkTsStack');
```
- **lib/todo-app-cdk-ts-stack.ts**

```js
import { Duration, Stack, StackProps } from 'aws-cdk-lib';
import * as sns from 'aws-cdk-lib/aws-sns';
import * as subs from 'aws-cdk-lib/aws-sns-subscriptions';
import { Construct } from 'constructs';

export class TodoAppCdkTsStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);
  // The code that defines your stack goes here
  const vpc = new ec2.Vpc(this, "MyVPC", {maxAzs: 2}); 
}
```

That’s all the code we need for a working CDK app! That **one line of code** generates the Whole Shebang! i.e VPC, Subnets, Route Tables, Routes, Internet Gateway...🤯

```todo-app-cdk-ts.ts``` is the main class of the app. It creates an App
instance and a ```todo-app-cdk-ts-stack.ts``` instance where we define our CloudFormation resources.

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

## Stack Communication

There are several strategies we can use based on the use cases. 

- **Property Parameters**:
  - <u>Use Case</u>

    When you have a ```parent``` stack and one or more ```child``` stacks that need to share specific configuration values, property parameters are ideal. A NetworkStack defining a VPC and passing the ```VPC ID``` and ```subnets``` to an ApplicationStack that needs to launch resources in that VPC.
  
  - <u>Steps</u>
    1. Define parameter in stack code
    ![CDK Params](/images/uploads/property-params-1.png)
    2. Define parameter in dependent stack code as part of constructor
    ![CDK Params](/images/uploads/property-params-2.png)
    3. Reference parameter in app definition
    ![CDK Params](/images/uploads/property-params-3.png)

{{% callout note %}}
You could also pull in entire stacks this way, if the sheer number of params that needs to be passed makes the code messy.
```ts
const orderUpLambdaStack = new LambdaStack(app, 'OrderUpLambdaStack', orderUpDBStack, { }) ;
```
{{% /callout %}}

- **Property Interface**:   
  - <u>Use Case</u>

    Property interfaces are suitable when you want to define a consistent interface for passing parameters between stacks. This approach is useful when you have multiple stacks with similar configurations or when you want to enforce a **contract** for what properties must be provided. 
    
    When building a ```microservices``` architecture, if each service stack requires similar properties like service names, database configurations, etc., you can create an interface to standardize these inputs.
  
  - <u>Steps</u>
    1. Define parameter in stack code
    ![CDK Params](/images/uploads/property-params-1.png)
    2. Define parameter in properties interface in the dependent code, reference interface in constructor
    ![CDK Params](/images/uploads/property-params-4.png)
    3. When we call the dependent stack we feed in the parameter information like so:
    ![CDK Params](/images/uploads/property-params-5.png)

- **Context Variables**:
  - <u>Use Case</u>

    Context variables are useful when you want to pass configuration values at synthesis that might differ between ```environments``` (e.g., dev, staging, production). These values can be defined in the ```cdk.json``` file or passed through CLI, making it easy to customize the deployment without changing the code.
  
  - <u>Steps</u>
    1. Define context variable in app definition
    ![CDK Params](/images/uploads/context-params-1.png)
    2. Call variable in dependent stack
    ![CDK Params](/images/uploads/context-params-2.png)

- **SSM Paramters**:
  - <u>Use Case</u>

    When you need to manage **secrets** or **configuration** values centrally, AWS Systems Manager (```SSM```) Parameter Store is an ideal choice. This is particularly useful for sensitive information like API keys or database passwords that should not be hardcoded in the code or templates. Only caveat is they have to be *Strings*
  
  - <u>Steps</u>
    1. Define parameter in stack code and save to SSM 
    ![CDK Params](/images/uploads/ssm-params-1.png)
    2. Call parameter in dependent stack
    ![CDK Params](/images/uploads/ssm-params-2.png)

## Identifiers

AWS CDK uses various types of ```identifiers```. Identifiers must be **unique** within the scope in which they were created. Familiarize yourself with the types of identifiers used:

{{< figure src="images/uploads/aws-cdk-construct-path.png" class="alignright">}}

- **Construct IDs**: 
```id``` is the most common identifier. It is passed as the second argument when instantiating a construct. This identifier must be **unique** only in the ```scope``` in which it is created.
- **Paths**: 
The constructs in an AWS CDK application form a hierarchy rooted in the App class. This hierarchy is called a ```path```. Constructs must have unique IDs at their own ```scope```. This requirement creates a pattern where a path for two different constructs is unique. This uniqueness ensures that the hash generated to form a ```Logical ID``` is also unique.
- **Logical IDs**: 
Unique IDs serve as the logical identifiers of resources in the generated AWS CloudFormation templates for those constructs that represent AWS resources. Logical identifiers are sometimes called logical names.

## Context

```Context``` values are **key-value** pairs that can be associated with an ```App```, ```Stack``` or ```Construct```. These values are retrieved during **synthesis** and could be used in the CDK code. CDK looks for the ```Context``` information in the following locations:

![CDK Context](/images/uploads/aws-cdk-context-locations.png)

AWS CDK uses ```Context``` to **cache** information from your AWS account, such as the **Availability Zones** in your account or the Amazon Machine Image (**AMI**) IDs used to start your instances. Because these values are provided by your AWS account, they can change between runs of your CDK application. This makes them a potential source of unintended change. The CDK Toolkit's caching behavior "freezes" these values for your CDK app until you decide to accept the new values. You can also create your own context values that your apps or constructs can use. 

Context values that you create are **scoped** to the construct that created them, meaning they are visible to child constructs but not to siblings. Context values set by the AWS CDK Toolkit are set on the **App** construct, and so they are visible to every construct in the app.

To retrieve the cached context values from your AWS account, use the following AWS CDK command: 

```bash
cdk context
```
The resulting information is also visible in various locations, including the ```cdk.json``` project file.
![CDK Context](/images/uploads/aws-cdk-context.png)

Imagine the following scenario without context caching. Let's say you specified "latest Amazon Linux" as the AMI for your Amazon EC2 instances, and a new version of this AMI was released. Then, the next time you deployed your CDK stack, your already-deployed instances would be using the outdated ("wrong") AMI and would need to be upgraded. Upgrading would result in replacing all your existing instances with new ones, which would probably be unexpected and undesired.

{{% callout note %}}

Because they're part of your application's state, **cdk.json** must be committed to source control along with the rest of your app's source code. Otherwise, deployments in other environments (for example, a CI pipeline) might produce inconsistent results.

{{% /callout %}}

## Context Methods

Some CDK context values can be dynamically determined from the contextual information in any AWS environment. This is done using Context methods.

* **HostedZone.fromLookup()**:

  * ```HostedZone.fromLookup()``` is a ```static``` method that imports an existing Route 53 Hosted Zone by querying your AWS account.
  * It requires at least the domainName property to identify the hosted zone.
  * The CDK performs a context lookup during **synthesis** — it fetches the hosted zone ID and other details for you.
  * This allows you to reference and add A records to an existing hosted zone without hardcoding IDs.

  ```ts
    // Lookup existing Hosted Zone by domain name
    const zone = HostedZone.fromLookup(this, 'MyHostedZone', {
      domainName: 'example.com',
    });

    // Create or import a VPC (here we create a default one)
    const vpc = new Vpc(this, 'MyVpc');

    // Create an EC2 instance
    const instance = new Instance(this, 'MyInstance', {
      vpc,
      instanceType: new InstanceType('t3.micro'),
      machineImage: MachineImage.latestAmazonLinux(),
    });

    // Create an A record pointing to the EC2 instance
    new ARecord(this, 'InstanceARecord', {
      zone,
      recordName: 'app.example.com', // Subdomain you want to use
      target: RecordTarget.fromAlias(new InstanceTarget(instance)),
    });

  ```

* **stack.availabilityZones**

  * ```stack.availabilityZones``` returns a string array of AZ names (e.g. ['us-east-1a', 'us-east-1b', 'us-east-1c']) for the region the stack is targeting.
  * You can use it to distribute resources (like subnets, instances) across different AZs for high availability.

  ```ts
    const azs = stack.availabilityZones;

    const vpc = new ec2. Vpc(this, 'VPC', {
    maxAzs: stack.availabilityZones.length,
    });
  ```

* **StringParameter.valueFromLookup()**

  * ```StringParameter.valueFromLookup()``` allows you to import the value of an existing SSM Parameter Store string parameter at **synthesis** time.
  * This means CDK fetches the value when you run ```cdk synth``` or ```cdk deploy```, and embeds it as a literal value in the CloudFormation template.
  * The method takes two arguments:
    * The CDK scope (```this```)
    * The parameter name (e.g., ```/my/parameter/name```)

  ```ts
  // Lookup the DB password stored in SSM Parameter Store at synthesis time
  // Assuming password is stored under '/myapp/prod/db-password'
  const dbPassword = StringParameter.valueFromLookup(this, '/myapp/prod/db-password');

  // Create the PostgreSQL RDS instance
    new DatabaseInstance(this, 'PostgresInstance', {
      engine: DatabaseInstanceEngine.postgres({ version: PostgresEngineVersion.VER_14 }),
      vpc,
      vpcSubnets: {
        subnetType: SubnetType.PRIVATE_WITH_EGRESS,
      },
      credentials: {
        username: 'postgres',                                  // fixed username or from config
        password: cdk.SecretValue.unsafePlainText(dbPassword), // use the looked-up password
      },
      ...
    });

  ```

{{% callout note %}}
For this lookup to succeed, your AWS credentials must have **permission** to read the SSM parameter, and the parameter must exist in the AWS account and region of your stack's environment.
{{% /callout %}}

* **StringParameter.valueForStringParameter()**

  * ```StringParameter.valueForStringParameter()``` imports the parameter dynamically at **deployment** time by referencing it in the CloudFormation template.
  * Unlike ```valueFromLookup()```, which fetches the parameter value at **synthesis** time, this method generates a CloudFormation **dynamic** reference (like {{```resolve:ssm:/my/parameter/name```}}).
  * This is more **secure** for secrets or frequently changing parameters because the actual value is fetched only when the stack deploys or updates, not embedded in the synthesized template.

  ```ts
    // Import SSM Parameter value dynamically at deploy time
    const dbPassword = StringParameter.valueForStringParameter(this, '/myapp/prod/db-password');

    // Create a Lambda function that receives the password as environment variable
    const lambdaFn = new Function(this, 'MyFunction', {
      runtime: Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: Code.fromInline(`
        exports.handler = async function(event) {
          console.log("DB Password:", process.env.DB_PASSWORD);
          return "Done";
        };
      `),
      environment: {
        DB_PASSWORD: dbPassword, // dynamically resolved at deploy time
      },
    });
  ```

* **Vpc.fromLookup()**

  * ```Vpc.fromLookup()``` lets you import an existing VPC into your CDK stack by:
    * Providing the VPC ID directly (vpcId), or
    * Searching by ```tags```, or other filters (e.g., ```isDefault```).
  * CDK performs a context lookup during **synthesis** to fetch the VPC configuration.
  * This imported VPC is a reference — you cannot modify it, but you can use it to launch resources inside it.


  ```ts
  // Look up an existing VPC by its ID or tags
    const vpc = Vpc.fromLookup(this, 'ExistingVPC', {
      // Option 1: Specify the VPC ID directly
      vpcId: 'vpc-0abc123def456ghij',

      // Option 2: Or find by tags (uncomment this and comment vpcId if you prefer)
      // tags: {
      //   'Environment': 'production',
      // },
    });

    // Now you can use the imported VPC in your constructs, e.g., create an ECS cluster
    // new ecs.Cluster(this, 'EcsCluster', { vpc });
  ```

{{% callout note %}}  
* Make sure your AWS credentials have permission to describe VPCs, and that the VPC exists in the stack’s AWS account and region.
* Running ```cdk synth``` or ```cdk deploy``` will cache the lookup result in ```cdk.context.json``` for faster subsequent syntheses. If the VPC changes or you want to refresh, use ```cdk context --clear``` or manually edit the context file.
{{% /callout %}}

* **MachineImage.lookup()**

  * ```MachineImage.lookup()``` performs a context-aware lookup to find the most recent AMI that matches given criteria.
  * You specify filters such as name patterns, owners, and additional filters to narrow down the AMI.
  * CDK queries AWS during synthesis and caches the result in ```cdk.context.json```.
  * This approach ensures you get the latest compatible AMI without hardcoding AMI IDs that vary per region and over time.

  ```ts
  // Lookup the latest Amazon Linux 2 AMI with specific filters
    const ami = MachineImage.lookup({
      name: 'amzn2-ami-hvm-*-x86_64-gp2',  // AMI name pattern
      owners: ['amazon'],                   // Owner ID (Amazon official)
      filters: {
        'architecture': ['x86_64'],
        'root-device-type': ['ebs'],
        'virtualization-type': ['hvm'],
      },
    });

    // Create an EC2 instance using the looked-up AMI
    new Instance(this, 'MyInstance', {
      vpc,
      instanceType: new InstanceType('t3.micro'),
      machineImage: ami,
    });
  ```

## Environments

When working with AWS CDK, you’ll often hear about *environments* — but what does that really mean? It can get a bit confusing because **"environment"** can refer to two distinct concepts:

1. **AWS Environment (Account & Region)**
2. **Deployment Environment (Dev, QA, Prod, etc.)**

Let’s break it down!

### AWS Environment

Environment parameter (```env```) represents the AWS **Account** and AWS **Region** in which a stack is deployed. AWS CDK selects the **default** Region and account in your current AWS CLI profile. However, you can manually override the environment [using the ```--profile``` option] by specifying a different set of values than the default. 

```typescript
const app = new cdk.App();

new MyStack(app, 'MyStack-Dev', {
  env: { account: '111111111111', region: 'us-west-2' },
});

new MyStack(app, 'MyStack-Prod', {
  env: { account: '222222222222', region: 'us-east-1' },
});
```

{{% callout note %}}

> Constructs deploying with environments using AWS CDK CLI environment variables are considered environment-agnostic and any constructs defined in such a stack cannot use any information about their environment.
 
When you *don’t* specify the `env` property explicitly in your CDK stack (i.e., you rely on the CDK CLI’s default environment variables like `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION`), your stack is **environment-agnostic**.  

**Environment-agnostic** stacks are flexible and portable — they can theoretically deploy to *any* AWS account or region without being tied down. But because of this flexibility, the CDK **cannot make assumptions or access concrete information** about the deployment environment **at synthesis time**.

Many AWS resources or configurations depend on knowing where (which account/region) they will be deployed. For example:  

- Conditional logic based on the AWS Region  
- Using region-specific AMIs  
- Validating that deployment only occurs in allowed regions/accounts

Without an explicit `env` defined, your CDK code **cannot perform these environment-aware checks or customizations**, because CDK synthesizes your stack without concrete account or region info.

---
```typescript
class MyStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    if (this.region === 'us-east-1') {
      // This won't work if env is not explicitly defined
      // because `this.region` is undefined or tokens unresolved at synth time.
      console.log('Deploying to US East 1');
    }
  }
}
```
---

{{% /callout %}}

### Deployment Environments

These are logical environments representing stages of your software lifecycle, often with different configurations, feature flags, or scale.
*i.e* say you want your ```Prod``` DB to have multi-AZ replication vs your ```Dev``` DB which deploys to a single AZ. 

![CDK Deployments](/images/uploads/deployment-envs.png)

* AWS CDK ```context``` is a way to pass environment-specific configuration values to your CDK app, which can be accessed during **synthesis**. This allows you to dynamically configure your stacks based on the environment (e.g., ```dev```, ```test```, ```prod```) without hardcoding values.
* Use Parameter Store / Secrets Manager + Lookup in CDK
* Leverage CDK Pipelines or CI/CD tooling (e.g., ```CodePipeline```, ```GitHub Actions```) to inject environment variables, config files, or context into CDK synth/deploy.

**Steps:**

1. Define environment-specific values in ```cdk.context.json``` or any of the other places where you could defined ```context``` as mentioned [earlier](#context)
      ```json
        {
          "context": {
            "envs": {
              "dev": {
                "stageName": "dev",
                "instanceType": "t3.micro",
                "dbName": "devdb"
              },
              "test": {
                "stageName": "test",
                "instanceType": "t3.small",
                "dbName": "testdb"
              },
              "prod": {
                "stageName": "prod",
                "instanceType": "t3.large",
                "dbName": "proddb"
              }
            }
          }
        }
      ```

  2. ```bin/app.ts``` — read AWS account and region from environment or CLI params and only use ```context``` for config

      ```ts
        ...
        const app = new cdk.App();

        const env = {
          account: process.env.CDK_DEFAULT_ACCOUNT || '891377100138',
          region: process.env.CDK_DEFAULT_REGION || 'us-east-1',
        };

        const stageName = app.node.tryGetContext('stageName') || 'dev';
        const envConfig = app.node.tryGetContext('environments')?.[stageName];

        const taskVpc = new VpcStack(app, `TaskVPC-${envConfig.stageName}`, {
          env,
          stageName: envConfig.stageName,
          maxAzs: envConfig.maxAzs,
        });

        const taskDb = new DatabaseStack(app, `TaskDB-${envConfig.stageName}`, {
          env,
          stageName: envConfig.stageName,
        });

        new ApiStack(app, `TaskAPI-${envConfig.stageName}`, {
          env,
          stageName: envConfig.stageName,
          table: taskDb.table,
        });
      ```

3. Deploy commands - You can specify the AWS profile, account, and region by setting environment variables or using CLI parameters

    ```ts
    # Using AWS profile
    export AWS_PROFILE=dev-profile
    cdk deploy -c env=dev

    # Or specify region inline
    AWS_PROFILE=test-profile AWS_REGION=us-west-2 cdk deploy -c env=test

    # Or export both account & region explicitly (used by CDK)
    export CDK_DEFAULT_ACCOUNT=111111111111
    export CDK_DEFAULT_REGION=us-west-2
    cdk deploy -c env=dev

    ```


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

## 📖Further Read

* You can learn more of CDK from the 🐎 mouth here: https://docs.aws.amazon.com/cdk/v2/guide/home.html
* “API Reference” in the AWS CDK Reference Guide: https://docs.aws.amazon.com/cdk/api/v2/



