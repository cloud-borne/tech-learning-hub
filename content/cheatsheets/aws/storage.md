---
title: Storage
linktitle: 🗄️ Storage
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 80
---

Storage Options in AWS

<!--more-->

## EBS
An EBS (Elastic Block Store) Volume is a network drive (i.e. not a physical drive) you can attach to your instances while they run. Well suited for use as the primary storage for file systems, databases or for any applications that require fine granular updates and access to raw, unformatted block level storage. It allows your instances to persist data.

* An EC2 machine loses its root volume (main drive) when it is manually terminated.
* Unexpected terminations might happen from time to time (AWS would email you)
* Sometimes, you need a way to store your instance data somewhere
* An EBS (Elastic Block Store) Volume is a network drive you can attach to your instances while they run
* It allows your instances to persist data

### Features

* It’s a network drive (i.e. not a physical drive)

  * It uses the network to communicate the instance, which means there might be a bit of latency
  * It can be detached from an EC2 instance and attached to another one quickly
* It’s locked to an Availability Zone (AZ)

  * An EBS Volume in us-east-1a cannot be attached to us-east-1b
  * To move a volume across, you first need to snapshot it
* Have a provisioned capacity (size in GBs, and IOPS)

  * You get billed for all the provisioned capacity
  * You can increase the capacity of the drive over time

### Types of EBS volumes

* **GP2**: General Purpose Volumes (cheap)

  * Recommended for most workloads like System boot volumes, Virtual desktops, Low-latency interactive apps,  Development and test environments
  * 3 IOPS / GiB, minimum 100 IOPS, burst to 3000 IOPS, max 16000 IOPS
  * 1 GiB – 16 TiB , +1 TB = +3000 IOPS
* **IO1**: Provisioned IOPS (expensive)

  * Critical business applications that require sustained IOPS performance, or more than 16,000 IOPS per volume (gp2 limit)
  * Large database workloads, such as: MongoDB, Cassandra, Microsoft SQL Server, MySQL, PostgreSQL, Oracle
  * Min 100 IOPS, Max 64000 IOPS (Nitro) or 32000 (other)
  * 4 GiB - 16 TiB.
  * The maximum ratio of provisioned IOPS to requested volume size (in GiB) is 50:1
* **ST1**: Throughput Optimized HDD

  * Streaming workloads requiring consistent, fast throughput at a low price like Big data, Data warehouses, Log processing, Apache Kafka
  * Cannot be a boot volume
  * 500 GiB – 16 TiB , 500 MiB /s throughput
* **SC1**: Cold HDD, Infrequently accessed data

  * Throughput-oriented storage for large volumes of data that is infrequently accessed. Scenarios where the lowest storage cost is important.
  * Cannot be a boot volume
  * 500 GiB – 16 TiB , 250 MiB /s throughput

## InstanceStore

Some instance do not come with Root EBS volumes, instead, they come with “Instance Store” (= ephemeral storage)
Instance store is physically attached to the machine (EBS is a network drive)

  * Pros:

    * Very High IOPS (in millions because it's physical)
    * Disks up to 7.5 TiB (can change over time), stripped to reach 30 TiB (can change over time…)
    * Better I/O performance (EBS gp2 has an max IOPS of 16000, io1 of 64000)
    * Good for buffer / cache / scratch data / temporary content
    * Data survives reboots
  * Cons:

    * On stop or termination, the instance store is lost
    * You can’t resize the instance store
    * Backups must be operated by the user

## EFS

Amazon Elastic File System (EFS) is a scalable cloud file storage solution for use with EC2 instances. It’s elastic because it will automatically grow and shrink as you add/remove files and is priced with pay per use model. It’s similar to EBS, but with EBS you can only mount your virtual disk to one EC2 instance. You can have multiple instances sharing an EFS volume in multiple AZs. Uses security group to control access to EFS.

![EFS](/images/uploads/efs.png)

### Features

* Use cases: content management, web serving, data sharing, Wordpress
* Uses NFSv4.1 protocol
* Uses security group to control access to EFS
* Compatible with Linux based AMI (not Windows)
* Encryption at rest using KMS
* POSIX file system (~Linux) that has a standard file API
* File system scales automatically, pay-per-use, no capacity planning!
* EFS Scale

  * 1000s of concurrent NFS clients, 10 GB+ /s throughput
  * Grow to Petabyte-scale network file system, automatically
* Performance mode (set at EFS creation time)

  * General purpose (default): latency-sensitive use cases (web server, CMS, etc…)
  * Max I/O – higher latency, throughput, highly parallel (big data, media processing)
* Storage Tiers (lifecycle management feature – move file after N days)

  * Standard: for frequently accessed files
  * Infrequent access (EFS-IA): cost to retrieve files, lower price to store

## Comparison

* **EBS**

  <img align="right" width="350" height="350" src="/images/uploads/ebs-migrate.png">

  * Can be attached to only one instance at a time. They are locked at the Availability Zone (AZ) level
  * EBS is a network drive.
  * EBS gp2 has an max IOPS of 16000, io1 of 64000
  * To migrate an EBS volume across AZ
    * Take a snapshot
    * Restore the snapshot to another AZ
    * EBS backups use IO and you shouldn’t run them while your application is handling a lot of traffic
  * Root EBS Volumes of instances get terminated by default if the EC2 instance gets terminated.(you can disable that)

* **EFS**

  <img align="right" width="200" height="200" src="/images/uploads/efs-mount.png">

  * EFS is a network drive
  * Mounting 100s of instances across AZ
  * EFS share website files (WordPress)
  * Only for Linux Instances (POSIX)
  * EFS has a higher price point than EBS
  * Can leverage EFS-IA for cost savings

* **InstanceStore**

  * Instance store is physically attached to the machine
  * Better I/O performance (in millions)
  * On stop or termination, the data in instance store is lost
  * The InstanceStore size is determined by the Instance type. You can’t resize the instance store.
  * Backups must be operated by the user

## 📖Further Read

👉 [Making an Amazon EBS volume available for use on Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html)

👉 [Walkthrough: Create an Amazon EFS File System and Mount It on an Amazon EC2 Instance Using the AWS CLI](https://docs.aws.amazon.com/efs/latest/ug/wt1-getting-started.html)
