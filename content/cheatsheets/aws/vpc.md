---
title: VPC
linktitle: 🛡️ VPC
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 15
---

AWS VPC & Networking

<!--more-->

## Overview

A foundational networking service in AWS is the Amazon Virtual Private Cloud (Amazon VPC). It is a ```software-defined``` virtual private network. It is a service that you use to create secure private networks in AWS to host your applications and data. Your private services will run from this Amazon VPC. You can also connect your Amazon VPCs from your AWS private network to your on premises, creating a hybrid environment. 

Within a Amazon Virtual Private Cloud (Amazon VPC), you can launch AWS resources in a logically **isolated** virtual network that you've defined. This virtual network closely resembles a traditional network with the benefits of the scalable infrastructure of AWS.

## Regions

  ![AWS-Infrastructure](/images/uploads/aws-infrastructure.png)
 
 * AWS has Regions all around the world 
 * Names can be us-east-1, eu-west-3… 
 * A region is a cluster of data centers 
 * Most AWS services are region-scope
 * 👉 [AWS Global Infrastructure](https://infrastructure.aws/)

## Availability Zones

 * Each region has many availability zones. 
   Example:
    - ap-southeast-2a
    - ap-southeast-2b
    - ap-southeast-2c
* Each availability zone (AZ) is one or more discrete data centers with redundant power, networking, and connectivity
* They’re separate from each other, so that they’re isolated from disasters
* They’re connected with high bandwidth, ultra-low latency networking

{{% callout note %}}

> Availability Zones (```AZs```) Aren’t **Globally** Consistent Across Accounts

AZ Names Are Mapped Per Account:
- AWS uses logical names like ```us-west-2a```, ```us-west-2b```, etc. for Availability Zones.
- These names are not globally fixed — they’re mapped differently for each AWS account.
- So, us-west-2a in Account A might physically correspond to a different data center than us-west-2a in Account B

🧠 Why AWS Does This?
- Load balancing & fault isolation: AWS spreads customers across ```AZs``` to avoid overloading any single data center.
- Security & abstraction: It prevents users from inferring physical infrastructure layout.
- Flexibility: AWS can reassign logical AZ names without disrupting customer configurations.

Even though both accounts use us-west-2a, they’re actually pointing to different physical zones.

{{< figure src="/images/uploads/vpc-az-id.png" width="300" height="300">}}

{{% /callout %}}

## Edge Locations

 * Amazon has 400+ Points of Presence (400+ Edge Locations & 10+ Regional Caches) in 90+ cities across 40+ countries
 * Content is delivered to end users with lower latency

## VPC

Private network to deploy your resources (regional resource). 
 
Here's how to create a VPC that you can use for a two-tier architecture in a production environment. To improve resiliency, you deploy the servers in two Availability Zones.

![AWS-VPC](/images/uploads/vpc-network.png)

## IPv4 and IPv6

```IPv4``` and ```IPv6``` are the two primary versions of the Internet Protocol used to identify devices on a network. AWS supports both IPv4 and IPv6, but **IPv4 cannot be disabled for a VPC**, ensuring compatibility with existing infrastructure.

###### IPv4

- **IPv4** is widely used to identify devices on a network using numerical addresses.
- **Address Space**: IPv4 provides about `4.3 billion` unique addresses (`2^32`), which are rapidly being exhausted.
- **Format**: IPv4 addresses are written in dotted decimal notation — four octets separated by dots (e.g., `192.168.0.1`).
- **Address Types**: Includes **public**, **private**, **loopback**, and **broadcast** addresses. Private ranges include `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`.
- **Subnetting**: CIDR notation (e.g., `/24`) is used to define network boundaries and calculate usable host addresses.

###### IPv6

- **IPv6** is the successor to **IPv4**, designed to overcome address exhaustion and support the growing number of internet-connected devices.
- **Address Space**: IPv6 provides approximately `3.4 × 10^38` unique IP addresses — enough to assign trillions per person on Earth.
- **Format**: An IPv6 address consists of 8 groups of 4 hexadecimal digits, separated by colons (e.g., `2001:db8:3333:4444:5555:6666:7777:8888`).
- **Zero Compression**: Consecutive groups of zeros can be compressed using `::` (e.g., `2001:db8::1234:5678`).
- **AWS Note**: All IPv6 addresses in AWS are **public and Internet-routable** — there are no private IPv6 ranges like in IPv4.

## CIDR Blocks

- <span style="color:red">C</span>lassless <span style="color:red">I</span>nter-<span style="color:red">D</span>omain <span style="color:red">R</span>outing – a method for allocating IP addresses used in Security Groups rules and AWS networking in general
- **Formula**: The number of IP addresses in a CIDR block is `2^(32 - prefix length)` for IPv4.
- **Prefix Length**: CIDR notation (e.g., `/26`) specifies how many bits are reserved for the network. The rest are for host addresses.
- **Usable Hosts**: Subtract 2 from the total to account for the **network address** and **broadcast address**, which are not assignable.
- **Example**: 
  - For `10.0.0.0/26`, `2^6 = 64` total IPs → **62 usable** host addresses.
  - WW.XX.YY.ZZ/32 => one IP
  - 0.0.0.0/0 => all IPs
 
## Subnets

Subnets allow you to partition your network inside your VPC (Availability Zone resource)
* A public subnet is a subnet that is accessible from the internet
* A private subnet is a subnet that is not accessible from the internet
* To define access to the internet and between subnets, we use Route Tables

## Route Tables

 * VPC has an implicit router, and you use route tables to control where network traffic is directed. 
 * Each subnet in your VPC must be associated with a route table, which controls the routing for the subnet.
 * A private subnet has a route defined to the NAT Gateway  for outbound traffic.
 * A public subnet has a route defined to the Internet Gateway for outbound traffic.
 * A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same subnet route table. 

## Internet Gateway

 * Internet Gateways helps our VPC instances connect with the internet
 * Public Subnets have a route to the internet gateway

## NAT Instances 

## NAT Gateway

 * NAT Gateways (AWS-managed) & NAT Instances (self-managed) allow your instances in your Private Subnets 
to access the internet while remaining private

## Network Access Control List (NACL)

* A firewall which controls traffic from and to a subnet
* Can have ALLOW and DENY rules
* Are attached at the Subnet level
* Rules only include IP addresses
* Rules in the NACL are evaluated in order, starting with the lowest numbered rule. If traffic matches a rule, the rule is applied and no further rules are evaluated.
* NACLs are stateless, ingress does not equal egress. Traffic that matches a rule for one direction will not be automatically allowed in the opposite direction. You would have to add an outbound rule.

 ![NACL](/images/uploads/vpc-nacl.png)

 Here's an example of a NACL Outbound rule:

![NACL](/images/uploads/nacl-outbound.png)

{{% callout note %}}

 - Rule number 100 is denying the HTTP(80) traffic from all IPs.
 - Rule number 110 is allowing the HTTP(80) traffic from all IPs.

Rule evaluation starts from the lowest number, when there is matching rule found no futher evaluation happens. 
For this scenario, rule 100 is matched and denies all HTTP(80) traffic in the subnet.

{{% /callout %}}

## Security Groups

* A firewall that controls traffic to and from an ENI / an EC2 Instance
* You can specify the source, port range, and protocol for each inbound rule. 
* You can also specify the destination, port range, and protocol for each outbound rule.
* Can have only ALLOW rules. **Deny** is by default i.e anything not permitted is denied.
* Rules include IP addresses and other security groups
* All rules are evaluated to decide to allow or deny traffic. The most permissive rule is applied.
* Security Groups are stateful, ingress equals egress. Traffic that matches a rule for one direction will also be allowed automatically in the opposite direction.

 ![Security-Groups](/images/uploads/vpc-security-groups.png)

There are several key areas to compare between Security Groups and NACLs:

 | Area                       	| AWS Security Group                                                                                     	| AWS Network ACL                                                                                                                                                      	|
|----------------------------	|--------------------------------------------------------------------------------------------------------	|----------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| Scope                      	| Operates at the instance level                                                                         	| Operates at the subnet level                                                                                                                                         	|
| Association                	| Applies to an instance only if it is associated with the instance                                      	| Applies to all instances deployed in the associated subnet (providing an additional layer of defense if Security Group rules are too permissive)                     	|
| Quantity Limits            	| Instance can have multiple Security groups. The initial quota is 2500 per VPC with 60 rules per group. 	| Subnet can have only one NACL. The initial quota is 200 per VPC.                                                                                                     	|
| Destination                	| Destination can be a CIDR subnet, specific IP address, or another Security group                       	| Destination can only be a CIDR subnet,                                                                                                                               	|
| Implicit Deny              	| Implicit Deny. You can only add “allow” rules.                                                         	| Implicit Allow. You can add “allow” rules and “deny” rules                                                                                                           	|
| Evaluation Order and Logic 	| All rules are evaluated to decide to allow or deny traffic. The most permissive rule is applied.       	| Rules in the NACL are evaluated in order, starting with the lowest numbered rule. If traffic matches a rule, the rule is applied and no further rules are evaluated. 	|
| Stateful vs Stateless      	| Stateful: Ingress == Egress. Response traffic is allowed by default.                                   	| Stateless: Ingress != Egress. Return traffic must be explicitly allowed by rules                                                                                     	|

To visualize the stateful nature of AWS Security Groups vs the stateless nature of network ACLs here's a diagram:

![SecurityGroups-vs-NACL](/images/uploads/securitygroups-vs-nacl.png)

{{% callout note %}}

Without any outbound rules on the network ACL, traffic is blocked outbound, but with the outbound rule added, traffic flows correctly.

Whereas with the Security Group, because it is stateful, traffic is permitted outbound automatically once the inbound connection is established.

{{% /callout %}}


## Elastic Network Interface (ENI)

An elastic network interface is a logical networking component in a VPC that represents a virtual network card (NIC). It can include the following attributes:

{{< figure src="images/uploads/vpc-ec2-default-eni.png" width="300" height="300" class="alignright">}}

 - A primary private IPv4 address from the IPv4 address range of your VPC
 - A primary IPv6 address from the IPv6 address range of your VPC
 - One or more secondary private IPv4 addresses from the IPv4 address range of your VPC
 - One Elastic IP address (IPv4) per private IPv4 address
 - One public IPv4 address
 - One or more IPv6 addresses
 - One or more security groups
 - A MAC address
 - A source/destination check flag
 - A description

 Some of the key features of an ENI to remember:

 1. Each instance in your VPC has a default network interface (the primary network interface — eth0) that assigns a private IPv4 address from the IPv4 address range of your VPC.
 2. You cannot detach the default (primary) network interface from an instance.
 3. You can create an attach an additional/ secondary ENIs to an instance in your VPC. However, these ENIs should be created within the same availability zone of the EC2 instance that you are trying to attach your secondary ENI. The number of network instances you can attach varies by instance type.
 4. A **Security Group** is attached to an **ENI** not an **EC2** instance. With this approach you can have multiple routes to the same EC2 instance with different security configurations.
 5. You can create a network interface, attach it to an instance, detach it from an instance and attach it to another instance within the same Availability Zone. 
 6. A network interface’s attributes follow it as it is attached or detached from an instance and reattached to another instance. You could use this approach to change the publicIP of your instance if need be.

 ![Multiple-Eni](/images/uploads/vpc-ec2-multiple-eni.png)

## VPC Endpoint

If we attempt to access a S3 bucket ( or any other **global** AWS service) from within a private subnet in a VPC, we could not do that directly since there's no internet connectivity. We could solve this in a naive way by going via:

```EC2``` ➡️ ```NAT Gateway``` ➡️ ```Internet Gateway``` ➡️ 🌎 ➡️ ```S3``` and then the same way back. 
![S3 Access via Nat](/images/uploads/private-ec2-s3-access.png)

{{< figure src="images/uploads/a-b-path.jpg" class="alignright">}}

It almost seems like going from A ➡️ B via a detour.

Issues with this approach:
- **Expensive**: Nat Gateways are 💰 and charged per hour.
- **Security**: Traffic is going out of the VPC and AWS Cloud and coming back.

This is the problem the ```VPC Endpoint``` would tend to solve.

A VPC endpoint is a **virtual** scalable networking component you create in a VPC and use as a private entry point to supported AWS services and third-party applications. Currently, two types of VPC endpoints can be used to connect to Amazon S3: ```Interface``` VPC endpoint and ```Gateway``` VPC endpoint. In both scenarios, the communications between the service consumer and the service provider never gets out of the AWS network.

{{< tabs name="VPC Endpoint" >}}
{{% tab name="Gateway Endpoint" %}}
![VPC Gateway Endpoint](/images/uploads/vpc-gateway-endpoint.png)
* The gateway endpoint is created at the ```VPC``` level. 
* We need to attach an endpoint policy to the Gateway endpoint that allow access to the S3 service and also specify a ```route``` in the route table in subnet 10.18.0.0 so the EC2 instance can find a path to the S3 bucket.
* VPC Endpoint policy and Resource-based policies can be used for fine-grained access control i.e which bucket, read/write access...
* Only supports ```S3``` and ```DynamoDB```.
* The biggest benefit for me is that it is 🆓. Your S3 or DynamoDB Traffic are not billed if ```Gateway Endpoint``` is used. 

{{% /tab %}}
{{% tab name="Interface Endpoint" %}}
![VPC Interface Endpoint](/images/uploads/vpc-interface-endpoint.png)
* When you create a Interface Endpoint, it creates an ```ENI``` in your ```private subnet``` and assign it a **private IP** address from the subnet address range.
* VPC Interface endpoints enable connectivity to services powered by AWS ```PrivateLink```. Services include AWS services like CloudTrail, CloudWatch, etc., services hosted by other AWS customers and partners in their own VPCs (referred to as ```Endpoint Services```), and supported AWS Marketplace partner services.
* Interface Endpoints can be used to create custom applications in VPC and configure them as an AWS PrivateLink-powered service (referred to as an endpoint service) exposed through a ```Network Load Balancer```.
* You would need to associate a ```Security Group``` around a Interface Endpoint. Also the EC2 Instance would have a Security Group associated. Then all we got to do is:
  - On the VPC Endpoint Security Group Allow **INBOUND** from EC2 Security Group.
  - On the EC2 Security Group Allow **OUTBOUND** to the VPC Endpoint Security Group
* Interface Endpoints only allow traffic from VPC resources to the endpoints and not vice versa.
* PrivateLink endpoints can be accessed across both intra- and inter-region VPC peering connections, Direct Connect, and VPN connections.
* VPC Interface Endpoints, by default, have an address like **vpce-svc-01234567890abcdef.us-east-1.vpce.amazonaws.com** which needs application changes to point to the service.
* Majority of AWS services can be privately connected through ```Interface Endpoint```.
* You are billed🧾 for **hourly** usage and data processing charges. 

{{% /tab %}}
{{< /tabs >}}

###### S3 VPC Endpoints Strategy

S3 is accessible with both ```Gateway Endpoints``` and ```Interface Endpoints```. Here's a mind map to help you decide based on your architecture needs:

| **S3 Gateway Endpoints**                      | **S3 Interface Endpoints**                                 |
|-----------------------------------------------|------------------------------------------------------------|
| Network traffic remains on the AWS network\.  | Network traffic remains on the AWS network\.               |
| Uses Amazon S3 public IP addresses            | Uses private IP addresses from the VPC to access Amazon S3 |
| Does not allow access from on\-premises       | Allows access from on\-premises                            |
| Does not allow access from another AWS Region | Allow access from a VPC in another AWS Region using VPC peering or AWS Transit Gateway         |
| Free                                          | Billed per hour                                            |


{{% callout note %}}
With VPC Endpoints you do need to allow VPC ```DNS HostName``` resolution, otherwise this solution won't work.
{{% /callout %}}


## VPC Peering

{{< figure src="/images/uploads/vpc-peering.png" width="500" height="500" class="alignright">}}

- Privately connect two VPCs using AWS’ network
- Make them behave as if they were in the same network
- Must not have overlapping CIDRs
- VPC Peering connection is NOT transitive (must be established for each VPC that need to communicate with one another)
- You must update route tables in VPC’s subnets each to ensure EC2 instances can communicate with each other
- You can create VPC Peering connection between VPCs in different AWS accounts/regions
- You can reference a security group in a peered VPC (works cross accounts – same region)

![VPC Peering SGs](/images/uploads/vpc-peering-sg.png)

{{% callout note %}}

🛠️ Cross-Account VPC Peering
{{< figure src="/images/uploads/vpc-peering-az-id.png" width="300" height="300">}}

If you need to ensure consistent AZ usage across accounts (e.g., for multi-account VPC peering or DR setups):
- Use AZ IDs instead of names
- Run: 
  ```
  aws ec2 describe-availability-zones --region us-west-2 --all-availability-zones
  ```
- Look for the ZoneId field (e.g., use2-az1, use2-az2)
- Map AZ names to AZ IDs manually
- Create a shared mapping table across accounts.
- Use AZ IDs to align resources (e.g., ensure both accounts use use2-az1).

{{% /callout %}}


## VPC Flow Logs

## Site to Site VPN

## Direct Connect 

## Direct Connect Gateway

## Transit Gateway

## Futher Read

[AWS Whitepaper - Connectivity models](https://docs.aws.amazon.com/whitepapers/latest/hybrid-connectivity/connectivity-models.html)






