---
title: CloudFormation
linktitle: 📜 CloudFormation
type: book
tags:
  - AWS
date: "2023-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 115
---

AWS infrastructure via CloudFormation

<!--more-->

## 🔎Overview

![AWS-CloudFormation](/images/uploads/cloudformation.png)

AWS CloudFormation is an AWS service which allows us to **declaratively** describe and provision almost all of the AWS services using the ```JSON``` or ```YAML``` format. It is defined as an “Infrastructure as a code”. That allows us to spend less time on managing infrastructure and more time spending on our application logic.

In AWS CloudFormation we will work with the CloudFormation template and using the template we are creating a **stack**. The template describes what AWS services are been provisioned into the stack. 

## Template Anatomy

There are 7 main sections will feature in the template and only the ```Resources``` section is **mandatory**.

```markmap {height="200px"}
- CloudFormation
  - AWSTemplateFormatVersion
  - Description
  - Parameters
  - Conditions
  - Resources
  - Outputs
```

## Parameters

* ```Parameters``` enable us to input custom values to our template each time when we create or update stack.
* We can have maximum of **60** parameters in a cfn template.
* Each parameter must be given a logical name (logical id) which must be alphanumeric and unique among all logical names within the template.
* Each parameter must be assigned a **parameter type** that is supported by AWS CloudFormation. 
* Each parameter must be assigned a value at runtime for AWS CloudFormation to successfully provision the stack. We can optionally specify a default value for AWS CloudFormation to use unless another value is provided.
* Parameters must be declared and referenced within the same template. 
* We can reference parameters from the **Resources** and **Outputs** sections of the template.
* Syntax:
![CloudFormation-Parameters](/images/uploads/cloudformation-parameters.png)

* Here's a quick 🧠 for Parameter **properties** and **types**:

| **Parameter Properties ** | **Type \(Mandatory\) **                       |
|---------------------------|-----------------------------------------------|
| * AllowedPattern          | * String                                      |
| * AllowedValues           | * Number                                      |
| * ConstraintDescription   | * List\<Number\>                                |
| * Default                 | * CommaDelimitedList                          |
| * Description             | * AWS Specific:                                |
| * MaxLength               | &emsp;&emsp;* AWS::EC2::Instance::Id                    |
| * MaxValue                | &emsp;&emsp;* AWS::EC2::VPC::Id                         |
| * MinLength               | &emsp;&emsp;* List\<AWS::EC2::Subnet::Id\>                |
| * MinValue                | &emsp;&emsp;* AWS::SSM::Parameter::Name                 |
| * NoEcho                  | &emsp;&emsp;* AWS::SSM::Parameter::Value                |
|                           | &emsp;&emsp;* AWS::SSM::Parameter::Value\<List\<String\>\>  |

{{< figure src="images/uploads/cloudformation-mappings.png" class="alignright">}}

## Mappings

* ```Mappings``` section matches a key to a corresponding set of named values. 
* For example, if we want to set values based on a region, we can create a mapping that uses region name as a key and contains the values we want to specify for each region
* We can use Fn::FindInMap intrinsic function to retrieve values in map.
* You can't include parameters, pseudo parameters, or intrinsic functions in the Mappings section.

```yml 
Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-079db87dc4c10ac91
    us-east-2:
      HVM64: ami-0ee4f2271a4df2d7d
    us-west-1:
      HVM64: ami-0082110c417e4726e   
    us-west-2:
      HVM64: ami-01450e8988a4e7f44    
```

{{% callout note %}}

**Interesting read**: https://stackoverflow.com/questions/44027960/cloudformation-possible-to-have-nested-mappings

{{% /callout %}}

## Conditions

* ```Conditions``` section contains statements that define the circumstances under which entities are created or configured.
  * **Example: 1** - We can create a condition and then associate it with a resource or output so that AWS CloudFormation only creates the resource or output if the condition is true.
  * **Example: 2** - We can associate the condition with a property so that AWS CloudFormation only sets the property to a specific value if the condition is true, if the condition is false, AWS CloudFormation sets the property to a different value that we specify.
* We will use conditions, when we want to **re-use** the template in different contexts like ```dev``` and ```prod``` environments. 
* Conditions are evaluated based on predefined Psuedo parameters or input parameter values that we specify when we create or update stack. 
* Within each condition we can reference the other condition. 
* We can associate these conditions in three places:
  * Resources
  * Resource Properties
  * Outputs 
* At stack creation or stack update, AWS CloudFormation evaluates all conditions in our template. During stack update, Resources that are now associated with a **false** condition are **deleted**. 
* We can use the below listed intrinsic functions to define conditions in cloud formation template. 
  * Fn::And
  * Fn::Equals
  * Fn::If
  * Fn::Not
  * Fn::Or

```yml
Conditions:
  CreateEIPForProd: !Equals [!Ref EnvironmentName, prod]
...
  ProdEIP:
    Type: AWS::EC2::EIP
    Condition: CreateEIPForProd
    Properties:
      InstanceId: !Ref VMInstance
```

{{% callout note %}}

**Important Note**: During stack update, we cannot update conditions by themselves. We can update conditions only when we include changes that add, modify or delete resources.

{{% /callout %}}
 
## Resources

* ```Resources``` are key components of a stack. 
* Resources section is a required section that need to be defined in cloud formation template.
* Syntax:
![CloudFormation-Resources](/images/uploads/cloudformation-resources.png)
* Resources Documentation:
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html

```yml
Resources:
  DevEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0082110c417e4726e
      InstanceType: t2.micro
      KeyName: cfn-key-1
      SecurityGroups:
        - !Ref SSHSecurityGroup
```

## EC2 User Data

* We can use UserData in CloudFormation template for ec2. 
* We need to use a intrinsic function ```Fn::Base64``` with UserData in CFN templates. This function returns the Base64 representation of input string. It passes encoded data to ec2 Instance. 
* YAML **Pipe** (|): Any indented text that follows should be interpreted as a multi-line scalar value which means value should be interpreted literally in such a way that preserves newlines.
* UserData **Cons**:
  * By default, user data scripts and cloud-init directives run only during the boot cycle when we first launch an instance.
  * We can update our configuration to ensure that our user data scripts and cloud-init directives run every time we restart our instance. (Reboot of server required)

```yml
  UserData:
    Fn::Base64:
     !Sub |
      #!/bin/bash -xe
      echo "Starting springboot-aws-starter"
      sudo su
      # install updates
      yum update -y
      # install java 8
      yum install java-1.8.0 -y
      # create the working directory
      mkdir /opt/springboot-aws-starter
      # download the maven artifact from S3
      aws s3 cp s3://cf-templates-us-west-1/springboot-aws-starter.jar /opt/springboot-aws-starter/ --region=us-west-1
      # change directories
      cd /opt/springboot-aws-starter
      # start application
      java -Ddb-instance-identifier=${DBInstanceIdentifier} -Ddatabase-name=${DBName} -DrdsUser=${DBUser} -DrdsPassword=${DBPassword} -Dspring.profiles.active=aws -jar springboot-aws-starter.jar
```

## Outputs

* ```Outputs``` section declares output values that we can 
  * Import in to other stacks (to create **cross-stack** references)
  * When using ```Nested``` stacks, outputs of a nested stack are used in ```Root``` Stack. 
* We can declare maximum of **60** outputs in a cfn template. 
* **Export** (Optional)
  * Exports contain resource output used for cross-stack reference. 
  * For each AWS account, **export name** must be **unique** with in the region. As it 
  should be unique we can use the export name as “AWS::StackName”-ExportName
  * We can’t create cross-stack references across regions.
  * We can use the intrinsic function ```Fn::ImportValue``` to import values that have been exported within the same region.  
  * For outputs, the value of the Name property of an Export can't use Ref or GetAtt
  functions that depend on a resource. 
  * We can’t delete a stack if another stack references one of its outputs. 
  * We can’t modify or remove an output value that is referenced by another stack. 
  * We can use Outputs in combination with Conditions. 

```yml
Outputs:
  JDBCConnectionString:
    Description: JDBC connection string for the database
    Value: !Join ['', ['jdbc:mysql://', !GetAtt [RDSInstance, Endpoint.Address], ':', !GetAtt [RDSInstance, Endpoint.Port], /, !Ref 'DBName']]
  ExternalUrl:
    Description: The url of the external load balancer
    Value: !Join ['', ['http://', !GetAtt 'LoadBalancer.DNSName','/context-root/']]
```

## Intrinsic Functions

* AWS CloudFormation provides several built-in functions that help you manage your stacks. 
* Use intrinsic functions to assign values to properties that are not available until runtime.
* Functions could be nested if need be.

There's a Long and Short format for each function:

* **Long form**: Fn::```FunctionName```: param
* **Short form**: !```FunctionName``` param

{{% callout note %}}

You can't nest two instances of two functions in short form. For example, the following syntax isn't valid:

```yml
!Base64 !Sub string
!Base64 !Ref logical_ID
```
Instead, use the full function name for at least one of the functions, as shown in the following examples:

```yml
!Base64
  "Fn::Sub": string
Fn::Base64:
  !Sub string
```
{{% /callout %}}

Following are a 📃 of most used functions:

###### !Ref

* The intrinsic function ```Ref``` returns the value of the specified parameter or resource. 
* **Resource** Case: When we specify a resource logical name, it returns a value that we can typically use to refer to that resource.
* **Parameter** Case: When we specify a parameter logical name, it returns the value of that parameter.
{{< figure src="images/uploads/cloudformation-ref-function.png" class="alignright">}}

* Syntax: 
  * Long Form: Ref: logicalName
  * Short Form: !Ref logicalName

###### !FindInMap

* The intrinsic function ```FindInMap``` returns the value corresponding to keys in a **two-level** map that is declared in Mappings section.
* Parameters
  * Map Name
  * Top Level Key
  * Second Level Key

```yml
Mappings:     
  MyRegionMap:
    us-east-2:
      HVM64: ami-0cd3dfa4e37921605
    us-west-1:
      HVM64: ami-0ec6517f6edbf8044
    ...
Resources:
  MyVMInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap
        - MyRegionMap
        - !Ref 'AWS::Region'
        - HVM64
```

###### Conditions

* We can use the below listed intrinsic functions to define conditions in cloud formation template.

  * !Equals
  * !And
  * !Or
  * !If
  * !Not

```yml
Conditions:
  CreateEIPForProd: !Equals [!Ref EnvironmentName, prod]
  CreateProdSecurityGroup: !Equals [!Ref EnvironmentName, prod]
  CreateDevSecurityGroup: !Not [{Condition: CreateProdSecurityGroup}]
```

###### !GetAtt

* **Attributes** are attached to any resources you create.
* To know the attributes of your resources, the best place to look at is the documentation.
* For example: the ```AZ``` of an EC2 machine!

```yml
Resources:
  MyVMInstance:
    Type: AWS::EC2::Instance
    Properties:
    ...
Outputs:
  MyInstanceAvailabilityZone:
    Description: Instance Availability Zone
    Value: !GetAtt MyVMInstance.AvailabilityZone 
```

###### !GetAZs

* Returns an array that lists ```Availability Zones``` for a specified ```Region``` in alphabetical order. 

```yml
Subnet0: 
  Type: "AWS::EC2::Subnet"
  Properties: 
    VpcId: !Ref VPC
    AvailabilityZone: !Select 
      - 0
      - Fn::GetAZs: !Ref 'AWS::Region'
```

###### !Sub

* Fn::Sub, or ```!Sub``` as a shorthand, is used to substitute variables from a
text. 
* You can combine Fn::Sub with **References** or AWS **Pseudo Parameters**
* String must contain ```${VariableName}``` and will substitute them

```yml
Outputs:
  AppURL:
    Description: Tomcat App Access URL
    Value: !Sub 'http://${MyVMInstance.PublicDnsName}:8080/index.html'
```

###### !ImportValue

* Import values that are exported in other templates

```yml
# Stack-1
Outputs:
  MyDevGlobalSecurityGroup:
    Description: My Dev SG
    Value: !Ref MyDevGlobalSecurityGroup
    Export:
      Name: MyDevSSHGlobalSG  
    ...
# Stack-2
Resources:
  MyVMInstance:
    Type: AWS::EC2::Instance
    Properties:
      ...
      SecurityGroups: 
        - !ImportValue MyDevSSHGlobalSG
```

###### !Join

* Join values with a delimiter

```yml
Outputs:
  JDBCConnectionString:
    Description: JDBC connection string for the database
    Value: !Join ['', ['jdbc:mysql://', !GetAtt [RDSInstance, Endpoint.Address], ':', !GetAtt [RDSInstance, Endpoint.Port], /, !Ref 'DBName']]
  ExternalUrl:
    Description: The url of the external load balancer
    Value: !Join ['', ['http://', !GetAtt 'LoadBalancer.DNSName','/context-root/']]
```

###### !Base64

* Returns the Base64 representation of the input string. 
* This function is typically used to pass encoded data to Amazon EC2 instances by way of the ```UserData``` property.

```YAML
SpringBootAwsStarterLauncher:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
    ...  
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash -xe
            echo "Starting springboot-aws-starter"
            ...
```

###### !Select 

The intrinsic function ```!Select``` returns a single object from a list of objects by index.

```yml

Subnet0: 
  Type: "AWS::EC2::Subnet"
  Properties: 
    VpcId: !Ref VPC
    AvailabilityZone: !Select 
      - 0
      - Fn::GetAZs: !Ref 'AWS::Region'

```
{{% callout note %}}

**!Select** doesn't check for null values or if the index is out of bounds of the array. Both conditions will result in a stack error, so you should be certain that the index you choose is valid, and that the list contains non-null values.

{{% /callout %}}

## Pseudo Parameters

* Pseudo parameters are parameters that are predefined by AWS CloudFormation.
* We don’t need to declare them in our template.
* We can use them the same way as we use parameters as an argument for ```!Ref``` function.
{{< figure src="images/uploads/cloudformation-pseudo-params.png" class="alignright">}}

* Following are the pseudo parameters available: 
  * AWS::AccountId
  * AWS::NotificationARNs
  * AWS::NoValue
  * AWS::Partition
  * AWS::Region
  * AWS::StackId
  * AWS::StackName
  * AWS::URLSuffix

## Metadata

We have three types of metadata keys which are listed below: 
* Metadata Keys
  * AWS::CloudFormation::Designer: Auto generated during resources drag and drop to canvas.
  * AWS::CloudFormation::Interface: Used for parameter grouping.
  * AWS::CloudFormation::Init: Used for application installation and configurations on our aws compute (EC2 instances).

## Helper Scripts

* By default, ```UserData``` scripts run only during the boot cycle when we first launch an instance. We cannot **update/evolve** the state of the EC2 instance without terminating it and creating a new one. Also there's no way of knowing that our EC2 user-data script completed successfully

* To solve this problem CloudFormation provides the following Python helper scripts that we can use to install software and start services on Amazon EC2 that we create as part of stack.

  {{< figure src="images/uploads/cloudformation-metadata-structure.png" width="300" height="1000" class="alignright">}}

  * cfn-init
  * cfn-signal
  * cfn-get-metadata
  * cfn-hup

* Type ```AWS::CloudFormation::Init``` will be used in the metadata section on an ec2 instance for ```cfn-init``` helper script.
* Configuration is separated into sections.
* Metadata is organized in to **config** keys, which we can even group into **configsets**. 
* The ```cfn-init``` helper script processes the configuration sections in the order specified in syntax section.

* We can use ```packages``` key to download and install pre-packaged software.
* We can use ```groups``` to create Linux/Unix groups and assign to group id’s.
* We can use the ```users``` key to create Linux/Unix users in EC2 Instance.
* We can use the ```sources``` key to download an archive file and unpack it in a target directory on EC2 Instance.

![CloudFormation-cfn-init](/images/uploads/cloudformation-cfn-init-1.png)

* We can use the ```files``` key to create files on EC2 Instance. The content can be either inline in the template or the content can be pulled from a URL.
* We can use ```commands``` key to execute commands on EC2 Instance.
* We can use ```services``` key to define which services should be enabled or disabled when the instance is launched. On Linux systems this key is supported by using sysvinit. On Windows systems, it is supported by using Windows Service Manager. Services key also allows us to specify dependencies on sources, packages and files so that if a restart is needed due to files being installed, cfn-init will take care of the service restart. 
* Supported Keys:
  * ensureRunning
  * enabled
  * files
  * sources
  * packages
  * commands 

![CloudFormation-cfn-init](/images/uploads/cloudformation-cfn-init-2.png)

* UserData
* Helper Scripts are updated periodically.
* We need to ensure that the below listed command is included in UserData of our template before we call the helper scripts to ensure that our launched instances get the latest helper scripts.
* The cfn-init helper script reads template metadata from the AWS::CloudFormation::Init key and acts accordingly to:
  * Fetch and parse metadata from AWS CloudFormation
  * Install packages
  * Write files to disk
  * Enable/disable and start/stop services
* The ```cfn-signal``` helper script signals AWS CloudFormation to indicate whether Amazon EC2 instances have been successfully created or updated. If we install and configure software applications on instances, we can signal AWS CloudFormation when those software applications are ready. We can use the cfn-signal script in conjunction with a ```CreationPolicy```.
* The **CreationPolicy** attribute is a CloudFormation resource attribute that you can define on your EC2 instance resources. It pauses the resource creation for a specific time you define in your template and waits for a **success** signal to continue. If this success signal does not come within this period or a failure signal is received, it fails the resource creation, and the stack creation **rollsback**.
* ```cfn-hup``` helper is a **daemon** that detects changes in resource metadata and runs user-specified actions when a change is detected.
* cfn-hup.conf file stores the name of the stack and the AWS credentials that the cfn-hup daemon targets.
* User actions that cfn-hup daemon calls periodically are defined in hooks.conf.

![CloudFormation-cfn-init](/images/uploads/cloudformation-cfn-init-3.png)

## Nested Stacks

* The ```AWS::CloudFormation::Stack``` type nests a stack as a resource in a top-level template. 
* The nested stacks have to be stored in a ```versioned``` S3 bucket which the root stack can access.
* We can add **output** values from a nested stack within the root stack.
* We use ```Fn::GetAtt``` function with nested stacks logical name and the name of the output value in nested stack
* With nested stacks, you deploy and manage all resources from a single stack i.e the **root** stack. **Never** ```update/delete``` the nested stack directly.
  * If you have to ```delete```, then delete the ```root``` stack.
  * If you have to ```update```, then update the ```nested``` stack, upload to versioned S3 bucket. Then update the ```root``` stack to propogate the changes. 
* You can use outputs from one stack in the nested stack group as inputs to another stack in the group. This differs from ```exporting``` values.
* If you want to isolate information sharing to within a stack group, you use **nested** stacks. To share information with other stacks (not just within the group of nested stacks), you would **export** values.

![CloudFormation-Nested](/images/uploads/cloudformation-nested-stack.png)

```yml
# Root Stack
Parameters:
  ...
  VpcBlock:
    Type: String
    Default: 10.0.0.0/16
    Description: VPC CIDR Tange
      
  Subnet01Block:
    Type: String
    Default: 10.0.1.0/24
    Description: CidrBlock for Subnet 01 within the VPC.    

Resources:

  VPCStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.us-east-2.amazonaws.com/nestedbucket/nestedstacks/NestedStack-VPC.yml 
      Parameters:  
        VpcBlock: !Ref VpcBlock
        Subnet01Block: !Ref Subnet01Block 
      TimeoutInMinutes: 5            

  SecurityGroupStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.us-east-2.amazonaws.com/nestedbucket/nestedstacks/NestedStack-SG.yml
      Parameters:
        VPCId: !GetAtt VPCStack.Outputs.VpcId
      TimeoutInMinutes: 5        

  MyVMInstance:
    Type: AWS::EC2::Instance
    Properties:
      ...
      NetworkInterfaces:
        - AssociatePublicIpAddress: "true"
          DeviceIndex: "0"
          SubnetId: !GetAtt VPCStack.Outputs.Subnet01Id
          GroupSet:
            - !GetAtt SecurityGroupStack.Outputs.DevSGGroupId  
          ...
# Nested VPC Stack

Parameters:
  VpcBlock:
    Type: String
    Default: 10.0.0.0/16
    Description: VPC CIDR Tange
      
  Subnet01Block:
    Type: String
    Default: 10.0.1.0/24
    Description: CidrBlock for Subnet 01 within the VPC.        
              
Resources:
  myVPC:
    Type: AWS::EC2::VPC
    Properties:
      ...  

Outputs:
  Subnet01Id:
    Description: Subnet 01 Id
    Value: !Ref Subnet01

  VpcId:
    Description: Vpc Id
    Value: !Ref myVPC      

# Nested SG Stack

Parameters:
  VPCId: 
    Description: Create security group in this respective VPC
    Type: AWS::EC2::VPC::Id

Resources:
  DevSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      ...
      VpcId: !Ref VPCId

Outputs:
  DevSGGroupId:
    Description: Dev Security Group
    Value: !Ref DevSecurityGroup
  
```

## Rollbacks

* Stack **Creation** Fails:
  * Default: Everything rollsback (gets **deleted**). We can look at the log. There is an option to disable rollback and troubleshoot what happened.
* Stack **Update** Fails:
  * The stack automatically rolls back to the previous known working state. Ability to see in the log what happened and error messages

## Drift

* CloudFormation doesn’t protect you against **manual** configuration changes after the Stack is created.
* To detect if our resources have changed, we can use CloudFormation **drift** feature.
* Not all resources are supported yet: 
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift-resource-list.html

## Stack Policy

* By default, anyone with stack update permissions can update all of the stack resources, and during the update process some resources might require **downtime** or even they get **replaced**😬 (that could be 💥in **Prod**)
* Stack policies prevents **accidental/unintentional** updates to Stack Resources.
* Stack Policy applies only during stack **updates**. Its doesn't provide access controls like AWS ```IAM``` Policies.
* We need to use Stack Policy as a **fail-safe** mechanism to prevent accidental updates.
* Stack policies are written in JSON.
* Stack Policy have the following elements:
  * **Effect**: Allow / Deny - Determines the action we specify should be allowed are denied.
  * **Action**: Specifies the update actions that are denied or allowed.
    * Update: Modify
    * Update: Replace
    * Update: Delete
    * Update :*
  * **Principal**: Principal element specifies the entity that the policy applies to. 
  * **Resource**: Specifies the logical id of the resource.
  * **ResouceTypes**: Generic type for that respective type of resources like ["AWS::EC2::SecurityGroup"]. We can even use wild card with resource types like ["AWS::EC2::*"]
  * **Condition**: Specifies the conditional where the policy applies to. 

{{% callout note %}}

If a stack policy includes **overlapping** statements (both allowing and denying on a resource) a ```Deny``` statement always overrides an ```Allow``` statement. 

{{% /callout %}}

```json
{
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "Update:*",
			"Principal": "*",
			"Resource": "*"
		},
		{
			"Effect": "Deny",
			"Action": "Update:*",
			"Principal": "*",
			"Resource": "LogicalResourceId/MyRDSInstance"
		},
		{
			"Effect": "Deny",
			"Action": "Update:*",
			"Principal": "*",
			"Resource": "*",
			"Condition": {
				"StringEquals": {
					"ResourceType": [
						"AWS::EC2::SecurityGroup"
					]
				}
			}
		}
	]
}

```
* Finally if you **have** to update a resource protected by an ```Update``` policy, you could use an updated ```Update``` policy for that **particular** change.
That policy would be temporary and would apply to that one change only.

## DeletionPolicy 

* You can put a **DeletionPolicy** on any resource to control what happens when the CloudFormation template is **deleted**
* DeletionPolicy=**Retain**:
  * Specify on resources to preserve / backup in case of CloudFormation deletes
* DeletionPolicy=**Snapshot**:
  * EBS Volume, ElastiCache Cluster, ElastiCache ReplicationGroup, RDS DBInstance, RDS DBCluster, Redshift Cluster
* DeletePolicy=**Delete**: **default** behavior for most resouces.

```yml

Resources:
  MySG:
    Type: AWS::EC2::SecurityGroup
    DeletionPolicy: Retain
    Properties:
      ...

  MyEBS:
    Type: AWS::EC2::Volume
    DeletionPolicy: Snapshot
    Properties:
      ...
```

{{% callout note %}}

* For ```AWS::RDS::DBCluster``` resources, the default policy is **Snapshot**
* To delete an ```S3``` bucket, you need to first **empty** the bucket of its contents

{{% /callout %}}

## Custom Resources

**Custom resources** enable you to write custom provisioning logic in templates that AWS CloudFormation runs anytime you ```create```, ```update``` or ```delete``` stacks. For example, you might want to include resources that aren't available as AWS CloudFormation resource types. All such use-cases could be served by a ```Custom Resource``` implemented using ```Lambda``` function or ```SNS```. Here's how it works:  

{{< figure src="images/uploads/CustomResource-Lambda.png" class="alignright">}}

1. CloudFormation retrieves your package source from S3.
2. CloudFormation Deploys Lambda Function.
3. Lambda runs and returns data to CF.
4. CloudFormation deploys other resources.

CloudFormation Templates require three elements to utilize Lambda Functions:

* Lambda Function (Either inline or a zip file in S3)
  - Handler
  - Runtime
  - Role
  - Timeout
* Lambda Execution Role
* Custom Resource (```ServiceToken``` property → Lambda Function ```ARN```)

Here's some scenarios

- ❓If we hardcode AMI Id's in CloudFormation mapping, and when AWS rolls out new AMI's? Our template would become **stale**:sneezing_face: 
  - 💡Lambda Function which retrieves AMI ID for instance type/region in real time. Use returned AMI to provision EC2:
    [Custom-Resources-Lambda-Lookup-Amiids](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/walkthrough-custom-resources-lambda-lookup-amiids.html)

- ❓How would we delete a stack with a **non-empty** s3 bucket created via cloudformation, the bucket needs to be emptied󠀠󠀠 first!
  - 💡Create a lambda function to clean up your bucket first from your CloudFormation stack 

- ❓If we create an IAM user with a custom password via CloudFormation, we need to make sure the password is correct!
  - 💡A second parameter (confirm password) can be created, and a Lambda Function checks and confirms if they match. Stack creation only proceeds if they match.

## SSM Parameters

* Reference parameters in ```Systems Manager``` Parameter Store
* Specify SSM parameter **key** as the value in CloudFormation template.
* CloudFormation always fetches the latest value (you can’t specify parameter version)
* Validation done on SSM parameter keys, but not values
* Supported SSM Parameter Types:
  * AWS::SSM::Parameter::Name
  * AWS::SSM::Parameter::Value<String>
  * AWS::SSM::Parameter::Value<List<String>> or 
  * AWS::SSM::Parameter::Value<CommaDelimitedList>
  * AWS::SSM::Parameter::Value<AWS-Specific Parameter>
  * AWS::SSM::Parameter::Value<List<AWS-Specific Parameter>>

```yml
Parameters:
  InstanceType:
    Description: WebServer EC2 instance type
    Type: AWS::SSM::Parameter::Value<String>
    Default: /dev/ec2/instanceType
    
  ImageId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !Ref ImageId
```

## Dynamic References

* Reference values stored in ```SSM``` Parameter Store of type String and StringList
* If no version specified, CloudFormation uses the latest version
* Unlike SSM Parameters, ```Dynamic``` references are used directly on the Resources.
* Doesn’t support public SSM parameters (e.g., Amazon Linux 2 AMI)

```yml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref ImageId
      KeyName: !Ref KeyName
      # ssm dynamic reference
      InstanceType: '{{resolve:ssm:/ec2/instanceType:1}}'

  MyIAMUser:
    Type: AWS::IAM::User
    Properties:
      UserName: 'sample-user'
      LoginProfile:
        # ssm-secure dynamic reference (latest version)
        Password: '{{resolve:ssm-secure:/iam/userPassword}}'

  MyDBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t2.micro
      Engine: mysql
      AllocatedStorage: "20"
      VPCSecurityGroups:
      - !GetAtt [DBEC2SecurityGroup, GroupId]
      # secretsmanager dynamic reference
      MasterUsername: '{{resolve:secretsmanager:MyRDSSecret:SecretString:username}}'
      MasterUserPassword: '{{resolve:secretsmanager:MyRDSSecret:SecretString:password}'
```

## StackSets

AWS CloudFormation ```StackSet``` extends the functionality of stacks by enabling you to create, update, or delete stacks across multiple accounts and regions with a single operation. A stack set lets you create stacks in AWS accounts across regions by using a single AWS CloudFormation template. Using an administrator account of an ```AWS Organization```, you define and manage an AWS CloudFormation template, and use the template as the basis for provisioning stacks into selected target accounts across specified regions.

A StackSet is a ```regional``` resource. If you create a StackSet in one AWS Region, you can only see or change it when viewing that Region.

![CloudFormation-StackSet](/images/uploads/cloudformation-stackset.png)
