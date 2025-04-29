---
title: Create AWS infrastructure via CloudFormation
summary: This post will take you through a step by step guide to creating a simple LAMP stack in AWS cloud.
tags:
- Spring Boot
- AWS
- RDS
- S3
- EC2
date: "2021-11-10T00:00:00Z"
toc: true
type: book
draft: true
weight: 30

---

### Overview

In this article, we are going to discuss the AWS CloudFormation service and how to provision the AWS services discussed here [Create AWS infrastructure via Console](/project/ec2-s3-rds/) using the CloudFormation template.

First we will briefly discuss what the AWS CloudFormation is. Then we will be discussing what we are going to build in this article using the CloudFormation template and finally, will be discussing each code segment in the CloudFormation template which is responsible for provisioning each AWS service.

### What is AWS CloudFormation?

AWS CloudFormation is an AWS service which allows us to declaratively describe and provision almost all of the AWS services using the JSON or YAML format. It is defined as an “Infrastructure as a code”. That allows us to spend less time on managing infrastructure and more time spending on our application logic.

In AWS CloudFormation we will work with the CloudFormation template and using the template we are creating a stack. The template is written either by using JSON or YAML and it describes what AWS services are been provisioned into the stack. There are 6 main sections will feature in the template and only the resources section is mandatory.

1. AWSTemplateFormatVersion: This is the template version. As of now, we do have only “2010–09–09” as a version.
2. Description: This section will describe the CloudFormation template. We can use String to do that.
3. Mappings: This section contains collections of key-value pairs. We can define constant values in here and later we can refer those values.
For an example, we can define our company public IPs.
4. Parameters: This section contains collections of key-value pairs and it allows us to pass parameter values to the template.
For an example, we can pass the instant type.
5. Resources: This section describes our AWS resources. This section is a mandatory section.
6. Outputs: In this section, we can define the values which need to take as an output.
For an example elastic IP or a load balancer DNS.

### What we are going to provision using the AWS CloudFormation template
In this article, I will be describing how to provision the AWS network infrastructure with consist of VPC, public and private subnet, and route tables, load balancer, and the EC2 instance using the sample CloudFormation template. Below diagram depicts the AWS services which will be created using our sample template.
A list of resources which we are going to create using our template and their resource types are listed in the below table as well.

![](/images/uploads/springboot-aws-starter.png)

We’ll be discussing a single fragment of YAML at a time. You can find the complete CloudFormation templates for the stack on GitHub at [springboot-aws-starter](https://github.com/avijitliberty/springboot-aws-starter.git). You may find it useful to pull the code locally so that you can experiment with it as you work through the tutorial.
As I explained earlier, our CloudFormation template will consist of six main sections.
Template version and the description are the first two section and we can define those section as below.

### AWSTemplateFormatVersion
```yml
"AWSTemplateFormatVersion":"2010-09-09"
```
### Description
```yml
Description: "Sample stack for demonstrating features of Spring Cloud AWS which
              discussed in the article: . **WARNING** This template creates an EC2
              instance and many more services. You will be billed for the AWS
              resources used if you create a stack from this template."
```
### Mappings

The 3rd section is for the Mappings section. I will be defining few constant values in this section and those are the AMI id in us-west-1 region and the VPC CIDR ranges.
> Please note that this CloudFormation template will run in the US-west-1 (California) region. AMI ids are different from region to region. You may need to update AMI id with respective one for your region.

```yml
Mappings:
  SubnetConfig:
    VPC:
      CIDR: "10.0.0.0/16"
    Public1:
      CIDR: "10.0.0.0/24"
    Public2:
      CIDR: "10.0.1.0/24"
    Private1:
      CIDR: "10.0.2.0/24"
    Private2:
      CIDR: "10.0.3.0/24"    
  RegionMap:
    us-west-1:
      HVM64: "ami-09a3e40793c7092f5"
```

### Parameters
The 4th section is for the Parameters section. I will be defining few parameter values in this section as below.

* KeyName: Name of an existing EC2 KeyPair to enable SSH access to the instance
* SSHLocation: The IP address range that can be used to SSH to the EC2 instances
* DB parameters like DBInstanceIdentifier, DBName, DBUser and DBPassword.
* AvailabilityZone1: The first availability zone. This will be a drop down box filled with the Availability Zones in the current region.
* AvailabilityZone2: The second availability zone. This will be a drop down box filled with the Availability Zones in the current region.

```yml
Parameters:
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: 9
    MaxLength: 18
    Default: 0.0.0.0/0
    AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.
  DBInstanceIdentifier:
    Default: starter-db
    Description: The database instance identifier
    Type: String
    MinLength: '1'
    MaxLength: '64'
    AllowedPattern: '[a-zA-Z-][a-zA-Z0-9.-]*'
    ConstraintDescription: must begin with a letter and contain only alphanumeric characters.
  DBName:
    Default: starter_db
    Description: The database name
    Type: String
    MinLength: '1'
    MaxLength: '64'
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9._]*'
    ConstraintDescription: must begin with a letter and contain only alphanumeric characters.
  DBUser:
    NoEcho: 'true'
    Description: The database admin account username
    Type: String
    MinLength: '1'
    MaxLength: '16'
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: must begin with a letter and contain only alphanumeric characters.
  DBPassword:
    NoEcho: 'true'
    Description: The database admin account password
    Type: String
    MinLength: '8'
    MaxLength: '41'
    AllowedPattern: '[a-zA-Z0-9!@#$&-]*'
    ConstraintDescription: must contain only alphanumeric characters.
  AvailabilityZone1:
    Description: The first availability zones where the resources will be initiated
    Type: AWS::EC2::AvailabilityZone::Name
    ConstraintDescription: 'Must be a valid availability zones. Ex: us-west-1a'
  AvailabilityZone2:
    Description: The second availability zones where the resources will be initiated
    Type: AWS::EC2::AvailabilityZone::Name
    ConstraintDescription: 'Must be a valid availability zones. Ex: us-west-1b'
```

### Resources

Section 5, the Resources section, is the most important section and the only section which is a mandatory section in the template.

#### 1. Defining the VPC
Here we specify the CIDR block to be as specified in the Mappings section.

Resource type: AWS::EC2::VPC

```YAML
SpringBootAwsStarterVpc:
    Type: AWS::EC2::VPC
    Properties:
      InstanceTenancy: default
      EnableDnsSupport: true
      EnableDnsHostnames: true
      CidrBlock:
        Fn::FindInMap:
          - "SubnetConfig"
          - "VPC"
          - "CIDR"
      Tags:
         -
           Key: "Name"
           Value: "SpringBootAwsStarterVpc"
```
#### 2. Defining public subnets

As per our infrastructure diagram, we do have two public subnets with two separate CIDR ranges.
These two subnets will be provisioned into two separate availability zones.

Resource type: AWS::EC2::Subnet

```YAML
SpringBootAwsStarterSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId:
        Ref: SpringBootAwsStarterVpc
      CidrBlock:
        Fn::FindInMap:
          - "SubnetConfig"
          - "Public2"
          - "CIDR"
      AvailabilityZone:
        Ref: AvailabilityZone2
      Tags:
         -
           Key: "Name"
           Value: "SpringBootAwsStarterSubnet2"
  SpringBootAwsStarterSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId:
        Ref: SpringBootAwsStarterVpc
      CidrBlock:
        Fn::FindInMap:
          - "SubnetConfig"
          - "Public1"
          - "CIDR"
      AvailabilityZone:
        Ref: AvailabilityZone1
      Tags:
         -
           Key: "Name"
           Value: "SpringBootAwsStarterSubnet1"
```

#### 3. Defining Private Subnets

Next we add two private subnets to the network stack for the RDS instance.
> Setting MapPublicIpOnLaunch to false makes the subnets private.

Resource type: AWS::EC2::Subnet

```YAML
SpringBootAwsStarterPrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId:
        Ref: SpringBootAwsStarterVpc
      CidrBlock:
        Fn::FindInMap:
          - "SubnetConfig"
          - "Private1"
          - "CIDR"
      AvailabilityZone:
        Ref: AvailabilityZone1
      MapPublicIpOnLaunch: false
      Tags:
         -
           Key: "Name"
           Value: "SpringBootAwsStarterPrivateSubnet1"
  SpringBootAwsStarterPrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId:
        Ref: SpringBootAwsStarterVpc
      CidrBlock:
        Fn::FindInMap:
          - "SubnetConfig"
          - "Private2"
          - "CIDR"
      AvailabilityZone:
        Ref: AvailabilityZone2
      MapPublicIpOnLaunch: false
      Tags:
         -
           Key: "Name"
           Value: "SpringBootAwsStarterPrivateSubnet2"
```
> Setting MapPublicIpOnLaunch to false makes the subnets private.

#### 4. Defining an Internet Gateway

In here we are defining an internet gateway and attaching it to the VPC.

Resource types: AWS::EC2::InternetGateway, AWS::EC2::VPCGatewayAttachment

```YAML
SpringBootAwsStarterInternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
      - Key: Name
        Value: "SpringBootAwsStarterInternetGateway"
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId:
        Ref: SpringBootAwsStarterVpc
      InternetGatewayId:
        Ref: SpringBootAwsStarterInternetGateway
```

#### 5. Defining the public route table and routes, associate those with public subnets.

In here we are defining a route table and route by assigning an above-created internet gateway. Then the subnet will be associated with this route table.

Resource types: AWS::EC2::RouteTable, AWS::EC2::Route, AWS::EC2::SubnetRouteTableAssociation

```YAML
SpringBootAwsStarterRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId:
        Ref: SpringBootAwsStarterVpc
      Tags:
      - Key: Name
        Value: "SpringBootAwsStarterRouteTable"
  SpringBootAwsStarterRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId:
        Ref: SpringBootAwsStarterRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId:
        Ref: SpringBootAwsStarterInternetGateway
  SubnetRouteTableAssociation1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId:
        Ref: SpringBootAwsStarterSubnet1
      RouteTableId:
        Ref: SpringBootAwsStarterRouteTable
  SubnetRouteTableAssociation2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId:
        Ref: SpringBootAwsStarterSubnet2
      RouteTableId:
        Ref: SpringBootAwsStarterRouteTable
```

#### 6. Defining the security groups for EC2, RDS, and the ELB

```YAML
SpringBootAwsStarterInstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for a instance which client application is running
      VpcId:
        Ref: SpringBootAwsStarterVpc
      Tags:
      - Key: Name
        Value: "SpringBootAwsStarterInstanceSecurityGroup"
  SpringBootAwsStarterRdsSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    DependsOn: SpringBootAwsStarterInstanceSecurityGroup
    Properties:
      GroupDescription: Security group for a DB which client application is running
      VpcId:
        Ref: SpringBootAwsStarterVpc
      Tags:
      - Key: Name
        Value: "SpringBootAwsStarterRdsSecurityGroup"
  SpringBootAwsStarterLBSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Security Group for Application LoadBalancer
      VpcId: !Ref SpringBootAwsStarterVpc
      Tags:
      - Key: Name
        Value: "SpringBootAwsStarterLBSecurityGroup"

  SpringBootAwsStarterInstanceSSHIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref SpringBootAwsStarterInstanceSecurityGroup
      IpProtocol: tcp
      FromPort: 22
      ToPort: 22
      CidrIp: !Ref 'SSHLocation'
  SpringBootAwsStarterInstanceWebIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref SpringBootAwsStarterInstanceSecurityGroup
      IpProtocol: tcp
      FromPort: 8080
      ToPort: 8080
      SourceSecurityGroupId: !GetAtt SpringBootAwsStarterLBSecurityGroup.GroupId
  SpringBootAwsStarterRdsClientIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref SpringBootAwsStarterRdsSecurityGroup
      IpProtocol: tcp
      FromPort: 3306
      ToPort: 3306
      CidrIp: !Ref 'SSHLocation'
  SpringBootAwsStarterRdsWebIngress:
    Type: AWS::EC2::SecurityGroupIngress
    Properties:
      GroupId: !Ref SpringBootAwsStarterRdsSecurityGroup
      IpProtocol: tcp
      FromPort: 3306
      ToPort: 3306
      SourceSecurityGroupId: !Ref SpringBootAwsStarterInstanceSecurityGroup
  SpringBootAwsStarterLBIngress:
    Type: 'AWS::EC2::SecurityGroupIngress'
    Properties:
      GroupId: !Ref SpringBootAwsStarterLBSecurityGroup
      IpProtocol: tcp
      FromPort: '80'
      ToPort: '80'
      CidrIp: 0.0.0.0/0  
```
#### 7. Define the S3 bucket

```YAML
SpringBootAwsStarterS3:
    Type: AWS::S3::Bucket
    DeletionPolicy: Delete
    Properties:
      BucketName: springboot-aws-starter-s3
      CorsConfiguration:
        CorsRules:
          - AllowedHeaders: ['Authorization']
            AllowedMethods: [GET]
            AllowedOrigins: ['*']
            Id: S3CORSRuleId
            MaxAge: '3000'
  SpringBootAwsStarterS3Policy:
    Type: 'AWS::S3::BucketPolicy'
    DependsOn: SpringBootAwsStarterS3
    Properties:
      Bucket: !Ref SpringBootAwsStarterS3
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Sid: 'AddPerm'
          Principal: '*'
          Action: 's3:GetObject'
          Effect: 'Allow'
          Resource:
            Fn::Join:
              - ""
              -
                - "arn:aws:s3:::"
                -
                  Ref: "SpringBootAwsStarterS3"
                - "/*"
```
#### 8. Define the instanceProfile for the EC2 SpringBootAwsStarterInstanceSSHIngress

```YAML
SpringBootAwsStarterInstanceRole:
   Type: 'AWS::IAM::Role'
   Properties:
     RoleName: SpringBootAwsStarterInstanceRole
     AssumeRolePolicyDocument:
       Version: 2012-10-17
       Statement:
         - Effect: Allow
           Principal:
             Service:
               - ec2.amazonaws.com
           Action:
             - 'sts:AssumeRole'
     Path: /
     Policies:
       - PolicyName: SpringBootAwsStarterInstanceRolePolicy
         PolicyDocument:
           Version: 2012-10-17
           Statement:
             - Effect: Allow
               Action:
                 - 'rds:DescribeDBInstances'
                 - 'rds:ListTagsForResource'
               Resource: '*'
             - Effect: Allow
               Action:
                 - 's3:PutObject'
                 - 's3:GetObject'
                 - 's3:DeleteObject'
               Resource: '*'
 SpringBootAwsStarterInstanceProfile:
   Type: 'AWS::IAM::InstanceProfile'
   DependsOn: SpringBootAwsStarterInstanceRole
   Properties:
     Path: /
     Roles:
       - Ref: SpringBootAwsStarterInstanceRole

```

#### 9. Define the RDS instance

```YAML
SpringBootAwsStarterDBSubnetGroup:
  Type: AWS::RDS::DBSubnetGroup
  Properties:
    DBSubnetGroupDescription: DB Subnet group for the VPC
    SubnetIds:
      - !Ref SpringBootAwsStarterPrivateSubnet1
      - !Ref SpringBootAwsStarterPrivateSubnet2
SpringBootAwsStarterRDS:
  Type: AWS::RDS::DBInstance
  DependsOn:
    - SpringBootAwsStarterInstanceRole
  Properties:
    AllocatedStorage: '20'
    AvailabilityZone: !GetAtt [SpringBootAwsStarterPrivateSubnet1, AvailabilityZone]
    EngineVersion: "5.7.23"
    AllowMajorVersionUpgrade: false
    AutoMinorVersionUpgrade: true
    DBInstanceIdentifier: !Ref 'DBInstanceIdentifier'
    DBInstanceClass: db.t2.micro
    Engine: MySQL
    DBName: !Ref 'DBName'
    MasterUsername: !Ref 'DBUser'
    MasterUserPassword: !Ref 'DBPassword'
    DBSubnetGroupName: !Ref SpringBootAwsStarterDBSubnetGroup
    Port: "3306"
    StorageType: "gp2"
    BackupRetentionPeriod: '7'
    PreferredBackupWindow: '07:47-08:17'
    PreferredMaintenanceWindow: 'mon:12:54-mon:13:24'
    LicenseModel: "general-public-license"
    MultiAZ: false
    PubliclyAccessible: true
    VPCSecurityGroups:
      - !Ref SpringBootAwsStarterRdsSecurityGroup
  DeletionPolicy: Delete
```
#### 10. Define the ELB

```YAML
SpringBootAwsStarterLoadBalancer:
    Type: AWS::ElasticLoadBalancing::LoadBalancer
    DependsOn: AttachGateway
    Properties:
      Subnets:
        - !Ref SpringBootAwsStarterPublicSubnet1
        - !Ref SpringBootAwsStarterPublicSubnet2
      HealthCheck:
        HealthyThreshold: '10'
        Interval: '30'
        Target: 'HTTP:8080/springboot-aws-starter/'
        Timeout: '5'
        UnhealthyThreshold: '2'
      ConnectionDrainingPolicy:
        Enabled: 'true'
        Timeout: '300'
      ConnectionSettings:
        IdleTimeout: '60'
      CrossZone: 'true'
      SecurityGroups:
        - !Ref SpringBootAwsStarterLBSecurityGroup
      Listeners:
        - InstancePort: '8080'
          LoadBalancerPort: '80'
          Protocol: HTTP
          InstanceProtocol: HTTP
          PolicyNames:
            - SpringBootAwsStarterLBCookieStickinessPolicy
      LBCookieStickinessPolicy:
        - PolicyName: SpringBootAwsStarterLBCookieStickinessPolicy
          CookieExpirationPeriod: '86400'
      Tags:
        - Key: name
          Value: SpringBootAwsStarterLoadBalancer
```

#### 11. Define Launch configuration

```YAML
SpringBootAwsStarterLauncher:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      AssociatePublicIpAddress: true
      ImageId: !FindInMap
        - RegionMap
        - !Ref 'AWS::Region'
        - HVM64
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      IamInstanceProfile: !Ref SpringBootAwsStarterInstanceProfile
      SecurityGroups:
        - Ref: SpringBootAwsStarterInstanceSecurityGroup
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeSize: 8
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash -xe
            echo "Starting springboot-aws-starter"
            sudo su
            # install updates
            yum update -y
            # remove java 1.7
            yum remove java-1.7.0-openjdk -y
            # install java 8
            yum install java-1.8.0 -y
            # create the working directory
            mkdir /opt/springboot-aws-starter
            # download the maven artifact from S3
            aws s3 cp s3://cf-templates-18bgz3ltbv0p6-us-west-1/springboot-aws-starter.jar /opt/springboot-aws-starter/ --region=us-west-1
            # change directories
            cd /opt/springboot-aws-starter
            # start application
            java -Ddb-instance-identifier=${DBInstanceIdentifier} -Ddatabase-name=${DBName} -DrdsUser=${DBUser} -DrdsPassword=${DBPassword} -Dspring.profiles.active=aws -jar springboot-aws-starter.jar
```

#### 12. Define AutoScaling Group and Scaling policies

```YAML
SpringBootAwsStarterScaler:
    Type: AWS::AutoScaling::AutoScalingGroup
    DependsOn: SpringBootAwsStarterRDS
    Properties:
      AvailabilityZones:
        - Ref: AvailabilityZone1
        - Ref: AvailabilityZone2
      Cooldown: '300'
      DesiredCapacity: '2'
      HealthCheckGracePeriod: '300'
      HealthCheckType: EC2
      MaxSize: '4'
      MinSize: '2'
      VPCZoneIdentifier:
        - !Ref SpringBootAwsStarterPublicSubnet1
        - !Ref SpringBootAwsStarterPublicSubnet2
      LaunchConfigurationName: !Ref SpringBootAwsStarterLauncher
      LoadBalancerNames:
        - !Ref SpringBootAwsStarterLoadBalancer
      Tags:
        - Key: name
          Value: SpringBootAwsStarterScaler
          PropagateAtLaunch: true
      TerminationPolicies:
        - Default

  SpringBootAwsStarterScaleDown:
    Type: 'AWS::AutoScaling::ScalingPolicy'
    Properties:
      AdjustmentType: ChangeInCapacity
      PolicyType: SimpleScaling
      ScalingAdjustment: -1
      AutoScalingGroupName: !Ref SpringBootAwsStarterScaler
  SpringBootAwsStarterScaleUp:
    Type: 'AWS::AutoScaling::ScalingPolicy'
    Properties:
      AdjustmentType: ChangeInCapacity
      PolicyType: SimpleScaling
      ScalingAdjustment: 1
      AutoScalingGroupName: !Ref SpringBootAwsStarterScaler

  SpringBootAwsStarterHighNetworkOutAlarm:
    Type: 'AWS::CloudWatch::Alarm'
    Properties:
      ActionsEnabled: 'true'
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      MetricName: NetworkOut
      Namespace: AWS/EC2
      Period: '300'
      Statistic: Average
      Threshold: '50000.0'
      AlarmActions:
        - !Ref SpringBootAwsStarterScaleUp
      Dimensions:
        - Name: AutoScalingGroupName
          Value: SpringBootAwsStarterScaler
  SpringBootAwsStarterLowNetworkOutAlarm:
    Type: 'AWS::CloudWatch::Alarm'
    Properties:
      ActionsEnabled: 'true'
      ComparisonOperator: LessThanThreshold
      EvaluationPeriods: '1'
      MetricName: NetworkOut
      Namespace: AWS/EC2
      Period: '300'
      Statistic: Average
      Threshold: '50000.0'
      AlarmActions:
        - !Ref SpringBootAwsStarterScaleDown
      Dimensions:
        - Name: AutoScalingGroupName
          Value: SpringBootAwsStarterScaler  
```

### Outputs

```YAML
JDBCConnectionString:
  Description: JDBC connection string for the database
  Value: !Join ['', ['jdbc:mysql://', !GetAtt [SpringBootAwsStarterRDS, Endpoint.Address], ':', !GetAtt [
        SpringBootAwsStarterRDS, Endpoint.Port], /, !Ref 'DBName']]
ExternalUrl:
  Description: The url of the external load balancer
  Value: !Join ['', ['http://', !GetAtt 'SpringBootAwsStarterLoadBalancer.DNSName','/springboot-aws-starter/']]
```
