---
title: Using CloudWatch for Resource Monitoring
subtitle:

# Summary for listings and search engines
summary:

# Link this post with a project
projects: []

# Date published
date: "2024-03-17T00:00:00Z"

toc: true

# Date updated
lastmod: "2024-03-17T00:00:00Z"

# Is this an unpublished draft?
draft: true

# Show this page in the Featured widget?
featured: false

weight: 10

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption:
  focal_point: "Center"
  placement: 1
  preview_only: false

authors:
- admin

tags:
- WebServer
- AWS
- Obervability

categories:

---

<!--more-->

### Overview

In this blog series we would dive deep into the basics of resource monitoring using Amazon ```CloudWatch```. Get ready to embrace the 💪 of CloudWatch and seize control of your AWS environment!

### Setup

For the purpose of our ⚗️ let's pick a Wordpress site. Use the template below ⬇️

{{% callout note %}}
This will provision a ```LAMP``` Stack which could cost you 💵. So delete the Stack once you are done.
{{% /callout %}}

<details>
  <summary>CloudFormation Template for Wordpress site</summary>

```yml
AWSTemplateFormatVersion: '2010-09-09'
Description: Using CloudWatch for Resource Monitoring
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.99.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: SysOpsVPC
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties: {}
  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  DMZ1public:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - '0'
        - !GetAZs ''
      CidrBlock: 10.99.1.0/24
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: DMZ1public
  DMZ2public:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - '1'
        - !GetAZs ''
      CidrBlock: 10.99.2.0/24
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: DMZ2public
  AppLayer1private:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - '0'
        - !GetAZs ''
      CidrBlock: 10.99.11.0/24
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: AppLayer1private
  AppLayer2private:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - '1'
        - !GetAZs ''
      CidrBlock: 10.99.12.0/24
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: AppLayer2private
  DBLayer1private:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - '0'
        - !GetAZs ''
      CidrBlock: 10.99.21.0/24
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: DBLayer1private
  DBLayer2private:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - '1'
        - !GetAZs ''
      CidrBlock: 10.99.22.0/24
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: DBLayer2Private
  PublicRT:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: PublicRT
  RouteTableAssociationA:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref DMZ1public
      RouteTableId: !Ref PublicRT
  RouteTableAssociationB:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref DMZ2public
      RouteTableId: !Ref PublicRT
  RoutePublicNATToInternet:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PublicRT
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
    DependsOn: VPCGatewayAttachment
  NATElasticIP:
    Type: AWS::EC2::EIP
    Properties:
      Domain: vpc
    DependsOn: VPCGatewayAttachment
  NATGateway:
    Type: AWS::EC2::NatGateway
    DependsOn: NATElasticIP
    Properties:
      AllocationId: !GetAtt NATElasticIP.AllocationId
      SubnetId: !Ref DMZ2public
  NATGatewayRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRT
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NATGateway
  PrivateRT:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: PrivateRT
  RouteTableAssociationC:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref AppLayer1private
      RouteTableId: !Ref PrivateRT
  RouteTableAssociationD:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref AppLayer2private
      RouteTableId: !Ref PrivateRT
  RouteTableAssociationE:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref DBLayer1private
      RouteTableId: !Ref PrivateRT
  RouteTableAssociationF:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref DBLayer2private
      RouteTableId: !Ref PrivateRT
  DMZNACL:
    Type: AWS::EC2::NetworkAcl
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: DMZNACL
  SubnetNetworkAclAssociationA:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref DMZ1public
      NetworkAclId: !Ref DMZNACL
  SubnetNetworkAclAssociationB:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref DMZ2public
      NetworkAclId: !Ref DMZNACL
  DMZNACLEntryIngress100:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '100'
      Protocol: '6'
      PortRange:
        From: '22'
        To: '22'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
  DMZNACLEntryIngress110:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '110'
      Protocol: '6'
      PortRange:
        From: '80'
        To: '80'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
  DMZNACLEntryIngress120:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '120'
      Protocol: '6'
      PortRange:
        From: '443'
        To: '443'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
  DMZNACLEntryIngress130:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '130'
      Protocol: '6'
      PortRange:
        From: '1024'
        To: '65535'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
  DMZNACLEntryEgress100:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '100'
      Protocol: '6'
      PortRange:
        From: '22'
        To: '22'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  DMZNACLEntryEgress110:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '110'
      Protocol: '6'
      PortRange:
        From: '80'
        To: '80'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  DMZNACLEntryEgress120:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '120'
      Protocol: '6'
      PortRange:
        From: '443'
        To: '443'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  DMZNACLEntryEgress130:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DMZNACL
    Properties:
      NetworkAclId: !Ref DMZNACL
      RuleNumber: '130'
      Protocol: '6'
      PortRange:
        From: '1024'
        To: '65535'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  AppNACL:
    Type: AWS::EC2::NetworkAcl
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: AppNACL
  SubnetNetworkAclAssociationC:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref AppLayer1private
      NetworkAclId: !Ref AppNACL
  SubnetNetworkAclAssociationD:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref AppLayer2private
      NetworkAclId: !Ref AppNACL
  AppNACLEntryIngress100:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: AppNACL
    Properties:
      NetworkAclId: !Ref AppNACL
      RuleNumber: '100'
      Protocol: '6'
      PortRange:
        From: '22'
        To: '22'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 10.99.0.0/16
  AppNACLEntryIngress110:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: AppNACL
    Properties:
      NetworkAclId: !Ref AppNACL
      RuleNumber: '110'
      Protocol: '6'
      PortRange:
        From: '80'
        To: '80'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 10.99.0.0/16
  AppNACLEntryIngress120:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: AppNACL
    Properties:
      NetworkAclId: !Ref AppNACL
      RuleNumber: '120'
      Protocol: '6'
      PortRange:
        From: '443'
        To: '443'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 10.99.0.0/16
  AppNACLEntryIngress130:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: AppNACL
    Properties:
      NetworkAclId: !Ref AppNACL
      RuleNumber: '130'
      Protocol: '6'
      PortRange:
        From: '1024'
        To: '65535'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
  AppNACLEntryEgress110:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: AppNACL
    Properties:
      NetworkAclId: !Ref AppNACL
      RuleNumber: '110'
      Protocol: '6'
      PortRange:
        From: '80'
        To: '80'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  AppNACLEntryEgress120:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: AppNACL
    Properties:
      NetworkAclId: !Ref AppNACL
      RuleNumber: '120'
      Protocol: '6'
      PortRange:
        From: '443'
        To: '443'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  AppNACLEntryEgress130:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: AppNACL
    Properties:
      NetworkAclId: !Ref AppNACL
      RuleNumber: '130'
      Protocol: '6'
      PortRange:
        From: '1024'
        To: '65535'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 10.99.0.0/16
  DBNACL:
    Type: AWS::EC2::NetworkAcl
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: DBNACL
  SubnetNetworkAclAssociationE:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref DBLayer1private
      NetworkAclId: !Ref DBNACL
  SubnetNetworkAclAssociationF:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref DBLayer2private
      NetworkAclId: !Ref DBNACL
  DBNACLEntryIngress100:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DBNACL
    Properties:
      NetworkAclId: !Ref DBNACL
      RuleNumber: '100'
      Protocol: '6'
      PortRange:
        From: '3306'
        To: '3306'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 10.99.0.0/16
  DBNACLEntryIngress105:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DBNACL
    Properties:
      NetworkAclId: !Ref DBNACL
      RuleNumber: '105'
      Protocol: '6'
      PortRange:
        From: '22'
        To: '22'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 10.99.0.0/16
  DBNACLEntryIngress110:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DBNACL
    Properties:
      NetworkAclId: !Ref DBNACL
      RuleNumber: '110'
      Protocol: '6'
      PortRange:
        From: '1024'
        To: '65535'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
  DBNACLEntryEgress100:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DBNACL
    Properties:
      NetworkAclId: !Ref DBNACL
      RuleNumber: '100'
      Protocol: '6'
      PortRange:
        From: '3306'
        To: '3306'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 10.99.0.0/16
  DBNACLEntryEgress105:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DBNACL
    Properties:
      NetworkAclId: !Ref DBNACL
      RuleNumber: '105'
      Protocol: '6'
      PortRange:
        From: '80'
        To: '80'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  DBNACLEntryEgress106:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DBNACL
    Properties:
      NetworkAclId: !Ref DBNACL
      RuleNumber: '106'
      Protocol: '6'
      PortRange:
        From: '443'
        To: '443'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  DBNACLEntryEgress110:
    Type: AWS::EC2::NetworkAclEntry
    DependsOn: DBNACL
    Properties:
      NetworkAclId: !Ref DBNACL
      RuleNumber: '110'
      Protocol: '6'
      PortRange:
        From: '1024'
        To: '65535'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Subnets:
        - !Ref DMZ1public
        - !Ref DMZ2public
      Name: load-balancer
      Type: application
      Scheme: internet-facing
      SecurityGroups:
        - !Ref LoadBalancerSecurityGroup
      IpAddressType: ipv4
  Listener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref TargetGroup
      LoadBalancerArn: !Ref LoadBalancer
      Port: '80'
      Protocol: HTTP
  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: '10'
      HealthCheckPath: /phpinfo.php
      HealthCheckPort: '80'
      HealthCheckProtocol: HTTP
      HealthyThresholdCount: '2'
      Name: TG1
      Port: '80'
      Protocol: HTTP
      UnhealthyThresholdCount: '2'
      VpcId: !Ref VPC
  WebInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Path: /
      Roles:
        - !Ref WebInstanceRole
  WebInstanceRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      Policies:
        - PolicyName: awscli
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - s3:*
                  - ec2:*
                  - rds:*
                  - elasticfilesystem:*
                  - logs:*
                  - cloudwatch:*
                Resource: '*'
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM
        - arn:aws:iam::aws:policy/AmazonSSMFullAccess
  DatabaseInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Path: /
      Roles:
        - !Ref DatabaseInstanceRole
  DatabaseInstanceRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      Policies:
        - PolicyName: awscli
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - ec2:*
                  - logs:*
                  - cloudwatch:*
                Resource: '*'
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM
        - arn:aws:iam::aws:policy/AmazonSSMFullAccess
  BastionSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: wordpress-bastion
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: BastionSG
      SecurityGroupIngress:
        - CidrIp: 0.0.0.0/0
          FromPort: 22
          IpProtocol: tcp
          ToPort: 22
  LoadBalancerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: wordpress-elb
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: LoadBalancerSG
      SecurityGroupIngress:
        - CidrIp: 0.0.0.0/0
          FromPort: 80
          IpProtocol: tcp
          ToPort: 80
        - CidrIp: 0.0.0.0/0
          FromPort: 443
          IpProtocol: tcp
          ToPort: 443
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: wordpress-ec2
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: WebServerSG
      SecurityGroupIngress:
        - FromPort: 22
          IpProtocol: tcp
          SourceSecurityGroupId: !Ref BastionSecurityGroup
          ToPort: 22
        - FromPort: 80
          IpProtocol: tcp
          SourceSecurityGroupId: !Ref LoadBalancerSecurityGroup
          ToPort: 80
  DatabaseSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: wordpress-rds
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: DatabaseSG
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          SourceSecurityGroupId: !Ref BastionSecurityGroup
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          SourceSecurityGroupId: !Ref WebServerSecurityGroup
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref WebServerSecurityGroup
  LaunchConfiguration:
    Type: AWS::AutoScaling::LaunchConfiguration
    Metadata:
      AWS::CloudFormation::Init:
        configSets:
          default:
            - install_packages
        install_packages:
          packages:
            yum:
              httpd24: []
              php70: []
              php70-mysqlnd: []
    Properties:
      ImageId: ami-0caeee8d52d79e604
      InstanceType: t3.micro
      IamInstanceProfile: !Ref WebInstanceProfile
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      AssociatePublicIpAddress: false
      UserData: !Base64
        Fn::Sub: |
          #!/bin/bash -xe
          /bin/echo '4Uf|7uMW' | /bin/passwd cloud_user --stdin
          yum update -y
          /opt/aws/bin/cfn-init -v --region ${AWS::Region} --stack ${AWS::StackName} --resource LaunchConfiguration
          yum list installed httpd24 php70 php70-mysqlnd
          cd /var/www/html
          curl -sL https://wordpress.org/latest.tar.gz | tar xfz -
          mv wordpress/* ./
          rm -rf wordpress
          chown -R apache /var/www
          chgrp -R apache /var/www
          chmod 2775 /var/www
          find /var/www -type d -exec sudo chmod 2775 {} \;
          find /var/www -type f -exec sudo chmod 0664 {} \;
          cp wp-config-sample.php wp-config.php
          sed -i "s/database_name_here/wordpress"/g wp-config.php
          sed -i "s/username_here/wpuser/g" wp-config.php
          sed -i "s/password_here/Password1/g" wp-config.php
          sed -i "s/localhost/${Database.PrivateIp}/g" wp-config.php
          echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
          service httpd start
          service httpd status
          chkconfig httpd on
          /opt/aws/bin/cfn-signal -e $? --region ${AWS::Region} --stack ${AWS::StackName} --resource LaunchConfiguration
      InstanceMonitoring: true
  Database:
    Type: AWS::EC2::Instance
    Metadata:
      AWS::CloudFormation::Init:
        configSets:
          default:
            - install_packages
        install_packages:
          packages:
            yum:
              mysql56-server: []
    Properties:
      ImageId: ami-0caeee8d52d79e604
      InstanceType: t3.micro
      IamInstanceProfile: !Ref DatabaseInstanceProfile
      SubnetId: !Ref DBLayer1private
      SecurityGroupIds:
        - !Ref DatabaseSecurityGroup
      UserData: !Base64
        Fn::Sub: |
          #!/bin/bash -xe
          /bin/echo '4Uf|7uMW' | /bin/passwd cloud_user --stdin
          yum update -y
          /opt/aws/bin/cfn-init -v --region ${AWS::Region} --stack ${AWS::StackName} --resource Database
          yum list installed mysql56-server
          service mysqld start
          chkconfig mysqld on
          mysql mysql << EOM
            CREATE DATABASE wordpress;
            GRANT ALL PRIVILEGES ON wordpress.* to wpuser IDENTIFIED BY 'Password1';
            DELETE FROM user WHERE user = '';
            FLUSH PRIVILEGES;
          EOM
          /opt/aws/bin/cfn-signal -e $? --region ${AWS::Region} --stack ${AWS::StackName} --resource Database
      Tags:
        - Key: Name
          Value: database
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      TargetGroupARNs:
        - !Ref TargetGroup
      LaunchConfigurationName: !Ref LaunchConfiguration
      MinSize: '1'
      MaxSize: '2'
      DesiredCapacity: '2'
      Cooldown: '300'
      HealthCheckGracePeriod: 300
      HealthCheckType: ELB
      VPCZoneIdentifier:
        - !Ref AppLayer1private
        - !Ref AppLayer2private
      Tags:
        - PropagateAtLaunch: true
          Value: instance-wordpress
          Key: Name
  BastionLaunchConfig:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      ImageId: ami-0caeee8d52d79e604
      InstanceType: t3.micro
      IamInstanceProfile: !Ref WebInstanceProfile
      SecurityGroups:
        - !Ref BastionSecurityGroup
      AssociatePublicIpAddress: true
      InstanceMonitoring: true
  BastionAutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      LaunchConfigurationName: !Ref BastionLaunchConfig
      MinSize: '1'
      MaxSize: '1'
      DesiredCapacity: '1'
      Cooldown: '300'
      HealthCheckGracePeriod: 300
      HealthCheckType: EC2
      VPCZoneIdentifier:
        - !Ref DMZ1public
        - !Ref DMZ2public
      Tags:
        - PropagateAtLaunch: true
          Value: bastion-host
          Key: Name
```

</details>

Now let's generate some traffic to monitor...Once the Stack is created, hit the **DNS** name of the load balancer into a browser. The WordPress setup should appear. On the WordPress setup popup window pick your language and click ```Continue```.

Next type the following information:

- Site Title: CloudWatch-Demo
- Username: <<Your UserName>>
- Password: <<Your Password>>
- Your Email: <<Your Email>>
- Click ```Install``` WordPress.

Enter the credentials just created above and log in and ```Voila!``` your new WordPress site should be ready.

### CloudWatch Metrics

```Metrics``` are the fundamental concept in CloudWatch. A metric represents a **time-ordered** set of data points that are published to CloudWatch. So let's start there.

#### Viewing Metrics

In the AWS Management Console on the Services menu, click ```CloudWatch```.

1. In the left navigation menu, under Metrics, click on ```All Metrics```.
2. Click on EC2 namespace, then **Per-Instance Metrics**. 
3. Lets zoom in on our instances by clicking on the drop-down next to one of the instances in the **InstanceId** column and select ```Add to search```. This will filter metrics on a that Instance.
4. Select **CPUUtilization** and **EBSWriteBytes**.
  
  ![CloudWatch-Metrics](/images/uploads/cloudwatch-metrics-graph.png)

  You might notice the below graph isn't very helpful as it shows both a unit of Bytes and CpuUtilization which is a percentage in a single graph. Let's make our graph more useful!

5. Click on the **Graphed metrics** tab.
6. Next let's move EBSWriteBytes to the Right ```Y-Axis```. Notice that the Graph looks much better now. We have Percent on the **left Axis** and Bytes on the **right Axis**
  ![CloudWatch-Metrics-Graph](/images/uploads/cloudwatch-metrics-graph-3.png)

  However, the CPU utilization looks off. We'd expect a Graph to go from **0%** to **100%**. Let's make one final adjustment!

7. Click ```Options```. For the left Axis which should hold our CPU Utilization (Percent), set a minimum of 0 and a maximum of 100. Set the Left Y Axis Label to Utilization (%). Great, our Graph looks much better than before!
  
  ![CloudWatch-Metrics-Graph](/images/uploads/cloudwatch-metrics-graph-4.png)

8. In the previous section we manually added metrics to a graph. But what happens if the resources **change**? What happens if EC2 instances are replaced via ```Auto-Scaling```? That's when Metrics ```Explorer``` view becomes handy. They enable you to visualize Metrics for **Tagged** resources. Expand the side panel menu and under Metrics, select ```Explorer```. Fill in the appropriate fields:
    - Metric: EC2 Instance: CPUUtilization
    - From: Name = instance-wordpress
    - columns: 1
    - rows: 1

  ![CloudWatch-Metrics-Explorer](/images/uploads/cloudwatch-metrics-explorer.png)

#### Search expressions

```Search Expressions``` enable you to create **dynamic** graphs that automatically add appropriate metrics to the display, even if those metrics don't exist when you first create the graph.

For example, you can create a search expression that displays the AWS/EC2 ```CPUUtilization``` metric for all instances in the **Region**. If you later launch a new instance, the CPUUtilization of the new instance is automatically added to the graph.

1. Select the ```Per-Instance Metrics``` in the **EC2** namespace as before.
2. Click the drop-down next to ```CPUUtilization``` and select Add to search
3. Finally click ```Graph Search``` button.

You'll end up with a Graph that looks like this:

![CloudWatch-Search-Expressions](/images/uploads/cloudwatch-search-expressions.png)

Let's look at the details and see the syntax for the search expression:

``` 
SEARCH('{AWS/EC2,InstanceId} MetricName="CPUUtilization"', 'Average', 300) 
```
We can see that the ```SEARCH``` function takes 3 arguments:

* {**Namespace**, **DimensionName1**, **DimensionName2**, ...} **SearchTerm**
* **Statistic**: the name of any valid CloudWatch statistic. (e.g. Minimum, Maximum, Sum or Average).
* **Period**: the aggregation time period in seconds.

Changing ```InstanceId``` to ```InstanceType``` changes the graph to show one line for each instance type used in the **Region**. Data from all instances of each type is aggregated into one line for that instance type.
```
SEARCH(' {AWS/EC2,InstanceType} MetricName="CPUUtilization" ', 'Average')
```

#### Math expressions

CloudWatch ```Math``` Expressions are used to perform mathematical calculations and transformations on metric data in Amazon CloudWatch. They allow you to combine, aggregate, and manipulate metrics to gain insights and create custom metrics for monitoring and analysis.

Below expression searches for the CPUUtilization metric of all instances in the specified Auto Scaling group and calculates the average CPU utilization over the last 5 minutes (300 seconds):
```
SEARCH('AutoScalingGroupName="my-auto-scaling-group" MetricName="CPUUtilization"', 'Average', 300)
```

Below expression calculates the average latency of the specified Elastic Load Balancer:
```
AVG(Latency, LoadBalancerName="my-load-balancer")
```

#### Dynamic Labels

### CloudWatch Logs

##### CloudWatch Logs Insights
##### CloudWatch Metric Filter

### CloudWatch Alarms

### CloudWatch Dashboard

1. In the CloudWatch Management Console, select Dashboards from the left navigation menu. Click Create dashboard on the right.
2. In the Create new dashboard popup window, type DMZLayer in the Dashboard name field. Click Create dashboard.
3. In the Add widget popup window, select the Line option. For the data source, select Metrics. Click Next.
4. Within the Metrics pane, select EC2. Then, select Per-Instance Metrics.
In the search bar at the top right of the Metrics pane, type CPUUtilization and press Enter.
Click on the gear icon on the right side of the screen.
Under Select visible columns, toggle on Instance name. Then click Confirm.
In the list of instances below, select the checkbox next to bastion-host.
Click Custom at the top right-hand corner of the screen and select 15 Minutes.
Click Create widget in the bottom right-hand corner of the screen.
In your DMZLayer dashboard, toggle the Autosave On option on top right-hand corner of the screen.

### Create a CloudWatch Dashboard for the Application Layer

Create the Dashboard
In the left navigation menu, select Metrics > All metrics.

Note: If your left navigation menu is collapsed, click the hamburger menu icon in the top left-hand corner of your screen to display it.

Within the Metrics pane, select EC2.

Then, select Per-Instance Metrics.

Scroll down to find CPUUtilization under Metric Name and click the down arrow next to the name. Select Search for this only.

Select the database instance and all displayed instance-wordpress instances by clicking the checkboxes next to their names.

Click Custom at the top of the screen, and select 15 Minutes.

Click the dropdown at the top of the screen with the word Line displayed. Select Stacked area from the list of options.

Click Actions in the top right-hand corner of the screen, and select Add to dashboard.

In the Add to dashboard popup window, click Create new under Select a dashboard. Enter AppLayer in the box that appears and click Create.

Add the RequestCount Metric
Click Add to dashboard.

Click the + (plus) sign in the top right-hand corner of the screen to add a widget.

In the Add widget popup window, select Number.

Click Next.

Within the Metrics pane, select ApplicationELB.

Then, select Per AppELB Metrics.

Find RequestCount under Metric Name and click the down arrow next to the name. Select Search for this only from the menu.

You will see one, or multiple, load balancer(s) displayed in the list below. Follow the steps below to confirm which load balancer to select.

Search for EC2 in the top search bar in the CloudWatch Management Console.

Right-click EC2 from the dropdown menu, and choose Open Link in New Tab.

Go to the new tab, and from the EC2 Management Console, select Target Groups under Load Balancing, in the left navigation menu.

Select the TG1 target group.

On the Details page, click load-balancer under Load balancer.

This will open up your load balancer in a new tab.

In the new tab, under Basic Configuration in the bottom half of the screen, scroll down to locate the ARN.

Verify the identity of the correct load balancer by taking note of the last few digits of the ARN.

Navigate back to the first CloudWatch Management Console tab, and select the corresponding load balancer (the one that ends in the same last few digits) by clicking the checkbox next to it.

Create the NetworkIn Metric
Click Create widget.
Click the + (plus) sign in the top right-hand corner of the screen to add a widget.
In the Add widget popup window, select Line.
Then, select Metrics.
Click Next.
Within the Metrics pane, select EC2.
Then, select Per-Instance Metrics.
Scroll down to find NetworkIn under Metric Name, and click the down arrow next to the name. Select Search for this only from the menu.
Select the database instance and all displayed instance-wordpress instances by clicking the checkboxes next to their names.
Click Create widget.
In the top right-hand corner of the screen, select the down arrow next to the Refresh icon. Select 10s, so the dashboard will refresh every 10 seconds.
Test the Widgets
Navigate back to your EC2 Management Console tab.


Navigate back to the CloudWatch Management Console in the first tab.

Within the console, optionally, click the refresh icon in the top-right corner.

You should be able to see that the RequestCount and CPUUtilization have gone up slightly, indicating that your CloudWatch dashboard and widgets are working as intended.
