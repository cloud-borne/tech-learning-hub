---
title: AWS CLI
linktitle: ⌨️ CLI
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 10
---

Tips and Tricks

<!--more-->

## Overview

The AWS CLI is a **beast💪** of a command-line interface that provides commands for many and many different AWS services (**224** at the time of this writing).

The CLI is written in Python so you need to have a functioning Python environment to work with it. The CLI is very handy for scripting interactions or common tasks that you might do manually in the AWS Console. It also provides good capabilities to generate text-based reports to answer questions like: 

> How many RDS instances do we have and what version of are they running at?

You could look at the console to get this information or you can get a consolidated report by issuing the following command:

```bash
$ aws rds describe-db-instances \
      --query 'DBInstances[*].{InstanceName:DBInstanceIdentifier,Engine:Engine,Version:EngineVersion}' \
      --output table \
      --profile sandbox
--------------------------------------------------
|               DescribeDBInstances              |
+--------+---------------------------+-----------+
| Engine |       InstanceName        |  Version  |
+--------+---------------------------+-----------+
|  mysql |  aa1amiupj9ajs8h          |  5.6.21   |
|  mysql |  ad12vdn41gqz5or          |  5.6.21   |
|  mysql |  cortex-orchestrator-dev  |  5.6.19b  |
+--------+---------------------------+-----------+
```

## Installation

Installing the AWS CLI differs across operating systems, please follow the [official instructions](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) for your operating system to install version 2 of the AWS CLI on your machine.

## Configuration

After you install the CLI, you can configure your command line:

```bash
aws configure
AWS Access Key ID [****************Kweu]:
AWS Secret Access Key [****************CmqH]:
Default region name [us-west-1]:
Default output format [yaml]:
```
You could run the following commands to validate the installation.

```bash

# This command lists all the AWS regions in which we can make use of EC2 instances.
aws ec2 describe-regions
# List the AWS CLI configuration data.
aws configure list
# Retrieves information about the IAM user based on the AWS access key ID
# used to sign the request to this operation.
aws iam get-user
```

{{% callout note %}}

This stores the AWS credentials in the user's home directory. In Windows at C:\\Users\\[username]\\.aws

{{% /callout %}}

You can also use multiple AWS users from the same account or users from multiple accounts with the AWS CLI. You would have to configure the command-line first like so:

```bash
aws configure --profile <profile-name>
```
and then use it while running CLI commands like so:

```bash
aws s3 ls --profile <profile-name>
```

## Pagination

* You can control the number of items included in the output when you run a CLI command. By Default AWS CLI uses the page size of **1000**; i.e if you make this call:
  
  ```bash
  aws s3api list-objects --bucket <YOUR_BUCKET_NAME>
  ```
  and you have 2500 objects in your bucket, you will be making 3 API calls to S3 but displays all the items at one go. You can set the page-size option to return a smaller set for each API call.
  
  ```bash
  aws s3api list-objects --bucket <YOUR_BUCKET_NAME> --page-size 5
  ```
* Use max-items option to return fewer items in the CLI output.
  
  ```bash
  aws s3api list-objects --bucket <YOUR_BUCKET_NAME> --max-items 1
  ```
## Dry run

* Sometimes, we’d just like to make sure we have the **permissions**…But not actually run the commands!
Some AWS CLI commands (such as EC2) can become expensive if they succeed, say if we wanted to try to create an EC2 Instance. Some AWS CLI commands (not all) contain a ```--dry-run``` option to simulate API calls.

  Example:
  
  ```bash
  aws ec2 run-instances \
      --dry-run \
      --image-id ami-922914f7 \
      --count 1 \
      --instance-type t2.micro
  ```
  The above command would fail with the following error:
  
  ```bash
  An error occurred (DryRunOperation) when calling the RunInstances operation:
  Request would have succeeded, but DryRun flag is set.
  ```

## Decode Errors

* When you run API calls and they fail, you can get a long error message. This error message can be decoded using the STS command line: **sts decode-authorization-message**

  Example:

  ```bash
  aws sts decode-authorization-message --encoded-message <value>
  ```

## MFA with CLI

* To use MFA with the CLI, you must create a temporary session. To do so, you must run the STS GetSessionToken API call like so:
  
  ```bash
  aws sts get-session-token \
          --serial-number arn-of-the-mfa-device \
          --token-code code-from-token \
          --duration-seconds 3600
  ```
  This command would give an output with the access token like so:
  ```bash
  {
      "Credentials": {
          "AccessKeyId": "access-key-id",
          "SecretAccessKey": "secret-access-key",
          "SessionToken": "temporary-session-token",
          "Expiration": "expiration-date-time"
      }
  }
  ```
  We could then use these credentials to configure our CLI like so:
  
  ```bash
  aws configure --profile mfa
  ```

## Credentials Chain

* The CLI will look👀 for credentials in this order:

  1. Command line options – --region, --output, and --profile
  2. Environment variables – **AWS_ACCESS_KEY_ID**, **AWS_SECRET_ACCESS_KEY**,
  and **AWS_SESSION_TOKEN**
  3. CLI credentials file – aws configure
  ~/.aws/credentials on Linux / Mac & C:\Users\user\.aws\credentials on Windows
  4. CLI configuration file – aws configure
  ~/.aws/config on Linux / macOS & C:\Users\USERNAME\.aws\config on Windows
  5. Container credentials – for ECS tasks
  6. Instance profile credentials – for EC2 Instance Profiles

* The Java SDK will look for credentials in this order:

  1. Environment variables –
  **AWS_ACCESS_KEY_ID** and **AWS_SECRET_ACCESS_KEY**
  2. Java system properties – aws.accessKeyId and aws.secretKey
  3. The default credential profiles file – ex at: ~/.aws/credentials, shared by
  many SDK
  4. Amazon ECS container credentials – for ECS containers
  5. Instance profile credentials– used on EC2 instances

## Examples

> WINDOWS users will need to use ```^``` (Shift + 6) instead of ```\``` for line continuation.

### EC2

```bash
aws ec2 describe-instances
aws ec2 describe-images
aws ec2 start-instances --instance-ids i-12345678c
aws ec2 terminate-instances --instance-ids i-12345678c
aws ec2 run-instances \
    --image-id ami-922914f7 \
    --count 1 \
    --instance-type t2.micro \
    --key-name puttykey \
    --security-group-ids sg-10af597a \
    --subnet-id subnet-07e36f7c \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=AWSCLI-EC2}]'
```
AWS **Instance Metadata** allows AWS EC2 instances to ”learn about themselves” without using  an IAM Role for that purpose.
* The URL for the metadata is: http://169.254.169.254/latest/meta-data
* You can retrieve the IAM Role name from the metadata, but you CANNOT retrieve the IAM Policy.
* Metadata = Info about the EC2 instance
* Userdata = launch script of the EC2 instance

### S3

```bash
# Make Bucket
aws s3 mb s3://cloudformation-sambucket --region us-west-1
# List Bucket contents
aws s3 ls s3://mybucket
# Remove bucket folder
aws s3 rm s3://mybucket/folder --recursive
# Copy myfolder to S3
aws s3 cp myfolder s3://mybucket/folder --recursive
aws s3 sync myfolder s3://mybucket/folder --exclude *.tmp
aws configure set default.s3.signature_version s3v4
aws s3 presign s3://<bucket-name>/<object-name> \
    --expires-in 300 \
    --region us-west-1
```

### Lambda

```bash
## List functions
aws lambda list-functions --region us-west-1

## Synchronous Invocations
aws lambda invoke \
    --function-name hello-world \
    --cli-binary-format raw-in-base64-out \
    --payload '{"key1": "value1", "key2": "value2", "key3": "value3" }' \
    --region us-west-1 response.json

## Asynchronous Invocations
aws lambda invoke \
    --function-name hello-world \
    --cli-binary-format raw-in-base64-out \
    --payload '{"key1": "value1", "key2": "value2", "key3": "value3" }' \
    --invocation-type Event \
    --region us-west-1 response.json

## Create the Lambda function as a zip file
aws lambda create-function \
    --zip-file fileb://function.zip \
    --function-name lambda-xray-with-dependencies \
    --runtime nodejs16.x \
    --handler index.handler 
    --role <ROLE_ARN>

  ```

### DynamoDB

  ```sh

  ## Create Table
  aws dynamodb create-table \
      --table-name Music \
      --attribute-definitions \
          AttributeName=Artist,AttributeType=S \
          AttributeName=SongTitle,AttributeType=S \
      --key-schema \
          AttributeName=Artist,KeyType=HASH \
          AttributeName=SongTitle,KeyType=RANGE \
      --provisioned-throughput \
          ReadCapacityUnits=10,WriteCapacityUnits=5

  ## Put Item
  aws dynamodb put-item \
     --table-name Music  \
     --item \
         '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat     Famous"}, "Awards": {"N": "1"}}'

  ## Batch-Write-Item
  aws dynamodb batch-write-item --request-items file://items.json

  ## where items.json has the items defined like so:
  {
   "SessionData": [
       {
           "PutRequest": {
               "Item": {
                   "UserID": {"N": "5346747"},
             "CreationTime": {"N": "1544016418"},
           "ExpirationTime": {"N": "1544140800"},
           "SessionId": {"N": "6734678235789"}
               }
           }
       },
       {
           "PutRequest": {
               "Item": {
                   "UserID": {"N": "6478533"},
             "CreationTime": {"N": "1544013196"},
           "ExpirationTime": {"N": "1544140800"},
           "SessionId": {"N": "6732672579220"}
               }
           }
       },
       {
           "PutRequest": {
               "Item": {
                   "UserID": {"N": "7579645"},
             "CreationTime": {"N": "1544030827"},
           "ExpirationTime": {"N": "1544140800"},
           "SessionId": {"N": "7657687845893"}
               }
           }
       }
   ]
  }

  ## Get Item
  aws dynamodb get-item --consistent-read \
      --table-name Music --region eu-west-2 \
      --key file://key.json

  ## where key.json has:
  {
      "Artist": {"S": "Acme Band"},
      "SongTitle": {"S": "Happy Day"}
  }

  ## Get specified attributes only:
  aws dynamodb get-item \
    --table-name ProductCatalog \
    --key file://key.json \
    --projection-expression "Description, RelatedItems[0], ProductReviews.FiveStar"

  ## where key.json has:
  {
      "Id": { "N": "102" }
  }

  ## Query Data:
  aws dynamodb query \
      --table-name Music \
      --key-condition-expression "Artist = :name" \
      --expression-attribute-values  file://key.json

  ## where key.json has:
  {
    ":name":{"S":"Acme Band"}
  }

  ## Update Item:
  aws dynamodb update-item \
        --table-name Music \
        --key '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}' \
        --update-expression "SET AlbumTitle = :newval" \
        --expression-attribute-values '{":newval":{"S":"Updated Album Title"}}' \
        --return-values ALL_NEW

  ## Delete table:
  aws dynamodb delete-table --table-name ProductCatalog

  ```

### KMS
  ```
  aws kms encrypt \
      --key-id YOURKEYIDHERE \
  	--plaintext fileb://secret.txt \
  	--output text \
  	--query CiphertextBlob | base64 --decode > encryptedsecret.txt

  aws kms decrypt \
      --ciphertext-blob fileb://encryptedsecret.txt \
  	--output text \
  	--query Plaintext | base64 --decode > decryptedsecret.txt

  aws kms re-encrypt \
      --destination-key-id YOURKEYIDHERE \
  	--ciphertext-blob fileb://encryptedsecret.txt | base64 > newencryption.txt

  aws kms enable-key-rotation --key-id YOURKEYIDHERE

  ```

### SSM

```bash
## Retrieve SSM managed EC2 instance details
aws ssm describe-instance-information
## Scan a managed node for patch compliance
aws ssm send-command \
    --document-name 'AWS-RunPatchBaseline' \
    --targets Key=InstanceIds,Values='i-<your-instance-id>' \
    --parameters 'Operation=Scan’

## Review the status of a Run command task
aws ssm list-commands  --command-id "<CommandId-value>"
## Securely connect to a managed node
aws ssm start-session --target i-<your-instance-id>
## Start an Automation runbook
aws ssm start-automation-execution \
    --document-name "AWS-RestartEC2Instance" \
    --parameters "InstanceId=i-<your-instance-id>"

## Status of particular Automation run
aws ssm get-automation-execution --automation-execution-id <your-execution-id>

## View applications installed on a managed node
aws ssm list-inventory-entries \
    --instance-id "i-<your-instance-id>" \
    --type-name "AWS:Application" \
    --max-results 1

## GET PARAMETERS
aws ssm get-parameters --names /my-app/dev/db-url /my-app/dev/db-password
## GET PARAMETERS WITH DECRYPTION
aws ssm get-parameters \
    --names /my-app/dev/db-url /my-app/dev/db-password \
    --with-decryption

## GET PARAMETERS BY PATH
aws ssm get-parameters-by-path --path /my-app/dev/
## GET PARAMETERS BY PATH RECURSIVE
aws ssm get-parameters-by-path --path /my-app/ --recursive
## GET PARAMETERS BY PATH WITH DECRYPTION
aws ssm get-parameters-by-path --path /my-app/ --recursive --with-decryption
## PUT PARAMETERS BY NAME
aws ssm put-parameter  --name "/dev/order-service/rds/username"  --type "String"  --value "orders_user"
aws ssm put-parameter  --name "/dev/order-service/s3/bucket"  --type "String"  --value "orders-service-dev-bucket"
aws ssm put-parameter  --name "/dev/order-service/feature/new-checkout"  --type "String"  --value "false"
```

{{% callout note %}}

If you want to use the AWS CLI to start and end sessions that connect you to your managed nodes, you must first install the Session Manager plugin on your local machine. See [Install the Session Manager plugin for the AWS CLI](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

{{% /callout %}}


### CloudWatch

```bash
## associate with existing log group
aws logs associate-kms-key \
    --log-group-name /aws/lambda/hello-world \
    --kms-key-id arn:aws:kms:eu-west-2:387124123361:key/0509dc31-00a4-4ef6-a739-3d77b2e011f5 \
    --region eu-west-2

## create new log group
aws logs create-log-group \
    --log-group-name /example-encrypted \
    --kms-key-id arn:aws:kms:eu-west-2:387124123361:key/0509dc31-00a4-4ef6-a739-3d77b2e011f5 \
    --region eu-west-2

```

### Kinesis

  ```
  ## Create Stream
  aws kinesis create-stream --stream-name <stream-name> --shard-count 1

  ## List streams in an account
  aws kinesis list-streams

  ## Get stream details
  aws kinesis describe-stream --stream-name <stream-name>

  ## Put record into the stream
  aws kinesis put-record \
              --stream-name <stream-name> \
              --partition-key 1 \
              --data 1

  ## Get Shard-Iterator
  aws kinesis get-shard-iterator \
              --shard-id shardId-000000000000 \
              --shard-iterator-type TRIM_HORIZON \
              --stream-name <stream-name>

  ## Get Records using Shard-Iterator
  > Shard iterators have a valid lifetime of 300 seconds

  aws kinesis get-records --shard-iterator AAAAAAAAAAHSywljv0zEgPX4NyKdZ5wryMzP9yALs8NeKbUjp1IxtZs1Sp+KEd9I6AJ9ZG4lNR1EMi+9Md/nHvtLyxpfhEzYvkTZ4D9DQVz/mBYWRO6OTZRKnW9gd+efGN2aHFdkH1rJl4BL9Wyrk+ghYG22D2T1Da2EyNSH1+LAbK33gQweTJADBdyMwlo5r6PqcP2dzhg=

  ## If you are running this from a system that supports PowerShell,\
  you can automate acquisition of the shard iterator using a command such as this

  aws kinesis get-records \
      --shard-iterator
      (
        (aws kinesis get-shard-iterator \
             --shard-id shardId-000000000000 \
             --shard-iterator-type TRIM_HORIZON \
             --stream-name <stream-name>).split('"')[4]
      )

  ## Split shards
  aws kinesis split-shard \
              --stream-name test-stream \
              --shard-to-split shardId-000000000000 \
              --new-starting-hash-key <hash-key>
  ```

### Code Deploy

```bash
# Create your application.zip and load it into CodeDeploy:
  
aws deploy create-application --application-name mywebapp
aws deploy push \
    --application-name mywebapp \
    --s3-location s3://<bucket-name>/webapp.zip \
    --ignore-hidden-files
  
# To deploy with this revision, run:
  
aws deploy create-deployment \
    --application-name mywebapp \
    --s3-location bucket=<bucket-name>,key=webapp.zip,bundleType=zip,eTag=d1de362aaf3254ee2d6cc5aab0e3f7d4 \
    --deployment-group-name <deployment-group-name> \
    --deployment-config-name <deployment-config-name> \
    --description <description>
```

### CloudFormation

```bash
# To create your EC2 instance using CloudFormation:
aws cloudformation create-stack --stack-name CodeDeployDemoStack \
    --template-url http://s3-eu-west-1.amazonaws.com/cftemplates/CF_Template.json \
    --parameters  ParameterKey=InstanceCount,ParameterValue=1 \
                  ParameterKey=InstanceType,ParameterValue=t2.micro \
                  ParameterKey=KeyPairName,ParameterValue=cfn-key \
                  ParameterKey=OperatingSystem,ParameterValue=Linux \
                  ParameterKey=SSHLocation,ParameterValue=0.0.0.0/0 \
                  ParameterKey=TagKey,ParameterValue=Name \
                  ParameterKey=TagValue,ParameterValue=CodeDeployDemo \
                  --capabilities CAPABILITY_IAM

# Verify that the Cloud Formation stack has completed using:
aws cloudformation describe-stacks \
    --stack-name CodeDeployDemoStack \
    --query "Stacks[0].StackStatus" \
    --output text

# The AWS CLI has a CloudFormation validation tool
aws cloudformation validate-template --template-body file:///path/to/file.yaml
```

### ECS

    aws ecs create-cluster
      --cluster-name=NAME
      --generate-cli-skeleton

    aws ecs create-service

## Elastic Beanstalk

### Configuration

* .elasticbeanstalk/config.yml - application config
* .elasticbeanstalk/dev-env.env.yml - environment config

    eb config

See: [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html)

### ebextensions

* [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers.html)
* [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html)

## SAM

Package your deployment:
  ```YAML
  sam package \
  --template-file ./lambda.yml \
  --output-template-file sam-template.yml \
  --s3-bucket cfsambucket
  ```
Deploy your package:

  ```YAML
  sam deploy \
  --template-file sam-template.yml \
  --stack-name mystack \
  --capabilities CAPABILITY_IAM
  ```

## Also see

* [AWS CLI](https://aws.amazon.com/cli/)
* [Documentation](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
* [All commands](http://docs.aws.amazon.com/cli/latest/reference/#available-services)
