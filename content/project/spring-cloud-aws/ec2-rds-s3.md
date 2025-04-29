---
title: Create AWS infrastructure via Console
summary: This post will take you through a step by step guide to creating a simple LAMP stack in AWS cloud.
tags:
- Spring Boot
- AWS
- RDS
- S3
- EC2
date: "2021-11-10T00:00:00Z"
type: book
draft: true
toc: true

weight: 20

---

### Overview

This article is a continuation for setting up AWS infrastructure for the springboot-aws-starter application we developed here: [Spring Cloud AWS](/project/spring-cloud-aws/).
The infrastructure we would build along looks like above.

For this section you’ll need access to the AWS console. If you haven’t already done so you should register for a [free account](https://aws.amazon.com/free/).

### VPC

A VPC (Virtual Private Cloud) allows users to configure a logically isolated network infrastructure for their applications to run on. For the purposes of this blog we will be creating a simple VPC with 2 public subnets. We would then launch all instances into one or both.

Below diagram depicts the architecture which will be created using Console:

#### Step 1 - Create VPC:

![](/images/uploads/vpc-wizard-1.png)

First we will pick the CIDR block for our VPC. For this demo we picked 10.0.0.0/16.
What that means is we have got: $2 ^ (32-16) - 5 = 65,531$ IP addresses to work with in our VPC.

Next we will add a public subnet. We have to pick the CIDR block for the subnet too. Here we pick 10.0.0.0/24 in us-west-1a zone. Again that means $2 ^ (32 - 24) - 5 = 251$ IP addresses.

{{% callout note %}}
Whenever you create a subnet the first four IP addresses and the last IP address in each subnet CIDR block are not available for you to use, and cannot be assigned to an instance.
{{% /callout %}}


For HA we will be deploying the application in 2 different AZs. So we will be manually adding one more public subnet after the VPC wizard is finished.

> The CIDR block of the subnet is a subset of the the VPC CIDR block.

![](/images/uploads/vpc-wizard-2.png)

#### Step 2: Add Routing Table entry:

If you have to make a subnet public you will have to add a route table and associate it with the public subnet.
The Routing Table the VPC wizard created currently does not have any way for instances to connect to the internet. So we do need to add an entry to the Routing Table to allow all traffic to go through an Internet Gateway into the public internet. The Internet Gateway is basically just a connection to the internet. So select the Route Table associated with the VPC:

![](/images/uploads/vpc-wizard-3.png)

Here's there's a single entry that says "Any IP address referencing our local VPC CIDR block should resolve locally"

![](/images/uploads/vpc-wizard-4.png)

Add an entry to allow all outgoing traffic to reach the outside world.

![](/images/uploads/vpc-wizard-5.png)

> Note: This Internet Gateway was created by the VPC wizard in Step 1.

#### Step 3: Create another public Subnet:

Because we would like to deploy our app with Auto-scaling/Load Balancing model we would need to define a different Subnet in another AZ in the same region.

> A Subnet can exist in a single AZ. Make sure to pick a different CIDR block than the other Subnet.

![](/images/uploads/vpc-wizard-6.png)

#### Step 4: Creating a Security Group to access RDS

Security groups provide a means of granting granular access to AWS services. Before creating a database instance on RDS we need to create a security group that will make the database accessible from the internet. This is required because as we are developing the application locally we still would need to be able to connect to the database instance on RDS.

> Note: in a production environment your database should never be publicly accessible and should only be accessible to EC2 instances within your Virtual Private Cloud.

Pick the EC2 Console and click on Security Groups.

We need to define a single inbound rule that will allow TCP traffic on port 3306 (port used by MySQL). In the rule config below I’ve set the inbound Source to Anywhere, meaning that the database instance will accept connections from any source IP. This is handy if you’re connecting to a development database instance from public WIFI where your IP will vary. In most cases we’d obviously narrow this to a specified IP range. The default outbound rule allows all traffic to all IP addresses.

![](/images/uploads/vpc-wizard-7.png)

### RDS

#### Step 1: Create DB Subnet Group:

To work with RDS services in a VPC we must have a DB subnet group created. You create a DB subnet group by specifying the subnets you created. Amazon RDS uses that DB subnet group and your preferred Availability Zone to choose a subnet and an IP address within that subnet to assign to your DB instance.

From the main AWS dashboard click RDS. On the main RDS dashboard click "Subnet Groups".

![](/images/uploads/rds-wizard-1.png)

#### Step 2: Create DB Instance:

On the main RDS dashboard click Launch a DB Instance. Select MySQL as the DB engine.

![](/images/uploads/rds-wizard-2.png)

In the next section we define the main database instance settings. We’ll retain most of the default settings so I’ll describe only the most relevant settings below.

* DB Instance Class – the size of the DB instance to launch. Choose t2-micro as this is currently the smallest available and is free as part of free tier usage.
* Multi AZ Deployment – indicates whether or not we want the DB deployed across multiple available zones for high availability. We don’t need this for a simple demo.
* Storage Type – the underlying persistence storage type used by the instance. General purpose Solid State Drives are now available by default so we’ll use those.
* Allocated Storage – The amount of physical storage available to the database. 20 GB is suffice for this demo.
* DB Instance Identifier – the name that will uniquely identify this database instance. This value is used by the _AWSResourceConfig_ class we looked at earlier.
* Master Username – the username we’ll use to connect to the database.
* Master Password – the password we’ll use to authenticate with.

  ![](/images/uploads/rds-wizard-3.png)

Next we’ll configure some of the advanced settings. Again we’ll be able to use many of the default values here so I’ll only describe the settings that are most relevant.

* VPC – Select the VPC from Step 1.
* Subnet Group – Select the subnet from Step 5.
* Publicly Accessible – Set to true so that we can connect to the DB from our local dev environment.
* Availability Zone – Select No Preference and allow AWS to decide which AZ the DB instance will reside.
* VPC Security Groups – Select the Security Group we defined earlier, in Step 4. This will apply the defined inbound and outbound TCP rules to the database instance.
* Database Name – select a name for the database. This will be used along with the _database identifier_ we defined in the last section to connect to the database.
* Database Port – Use default MySQL port 3306.
* The remaining settings in the Database Options section should use the defaults as shown below.
* Backup – Use default retention period of 7 days and No Preference for backup window. Carefully considered backup settings are obviously very important for a production database but for this demo we’ll stick with the defaults.
* Monitoring & Maintenance – Again these values aren’t important for our demo app so we’ll use the defaults shown below.

  ![](/images/uploads/rds-wizard-4.png)

  ![](/images/uploads/rds-wizard-5.png)

Click Launch DB Instance and wait for a few moments while instance is brought up. Click View Your DB Instance to see the configured instance in the RDS instance screen. In the RDS instances view, the newly create instance should be displayed with status "Available". If you expand the instance view you’ll see a summary of the configuration details we defined above.

![](/images/uploads/rds-wizard6.png)

#### Step 3: Connecting to the database:

Now that the database instance is up and running we can connect via any SQL client. We will be using MySQL Workbench as a SQL client tool. You could install the same from [here](https://www.mysql.com/products/workbench/).

Pick the Hostname from the generated RDS endpoint from Step 6.

![](/images/uploads/mysql-client.PNG)

### S3

On the main AWS management console select S3 under the storage and content delivery section. When the S3 management console loads, click Create Bucket.

Before we create our first bucket, a few important learning in S3:

* S3 uses buckets to store your objects
* S3 creates a URL namespace for those objects.

  e.g if you created your bucket in the N. California (US West 1) region and named your bucket: springboot-aws-starter-s3, S3 will assign this URL: [https://springboot-aws-starter-s3.s3-us-west-1.amazonaws.com](https://springboot-aws-starter-s3.s3-us-west-1.amazonaws.com "https://springboot-aws-starter-s3.s3-us-west-1.amazonaws.com") as the root for any objects.

  > Because of the above the bucket name has to be unique across S3 **globally**, not just in your account.
* Objects are identified via their key which is the path and file name, i.e. images/image.png (excluding the bucket name)

Enter a bucket name and ensure its matches the name specified in SpringCloudS3Service.java we defined earlier and hit create.

![](/images/uploads/s3-wizard-1.PNG)

### EC2

#### Step 1: Create a role for EC2:

Before we create the EC2 instance we’ll create a Role through Identity Access Management. The role will be granted to the EC2 instance as part of the set up and will allow access to the database instance on RDS and S3 storage.

Log into the AWS console and navigate to Identity Access Management. On the left hand side select Roles and click Create New Role.                                       

* Select Role Type Amazon EC2

![](/images/uploads/iam-role-1.PNG)

* Attach _AmazonS3FullAccess_ and _AmazonRDSFullAccess_ policies to the role to allow read/write access to RDS and S3.

![](/images/uploads/iam-role-2.PNG)

![](/images/uploads/iam-role-3.PNG)

* Add a name tag:

![](/images/uploads/iam-role-4.PNG)

* Add a role name and save:

![](/images/uploads/iam-role-5.PNG)

#### Step 2: Creating an EC2 Instance:

Now that we’ve created a role that will provide read/write access to RDS and S3, we’re ready to create the EC2 instance.

* Navigate to the EC2 console and click launch instance.
* Choose the AWS Linux AMI. This is the base server image we’ll to create the EC2 instance.

  ![](/images/uploads/ec2-wizard-1.png)
* To keep costs down select t2 micro as the instance type. This is a pretty light weight instance with limited resources but is sufficient for running our demo app.

  ![](/images/uploads/ec2-wizard-2.png)
* We only need one instance for the demo and can deploy it to the VPC we created earlier. Ensure that auto assign public IP is enabled, either via Use Subnet Setting or explicitly. This is required so that the instance can be accessed from the internet. Select the IAM role we created earlier so that RDS and S3 services can be accessed from the instance. The remaining settings can be left defaulted as shown below. When all values have been selected click Next:Add Storage.

  ![](/images/uploads/ec2-wizard-3.png)
* Use the default storage settings for this instance and click Next:Tag Instance

  ![](/images/uploads/ec2-wizard-4.png)
* Add a single tag to store the instance name and click Next:Configure Security Group

  ![](/images/uploads/ec2-wizard-5.png)
* The security group settings define what type of traffic is allowed to access your instance. We need to configure SSH access to the instance so that we can SSH onto the box to set it up and run the application. We also need HTTP access so that we can access the application once its up and running. The Source value specifies what IPs the instance will accept traffic from. I spend quite a bit of time on the train (public WI-FI) where the IP address changes regularly, so for handiness I’m leaving the Source open. Ordinarily we’d want to limit this value so that the instance is not open to the world.

  ![](/images/uploads/ec2-wizard-6.png)
* The final step is to review the configuration settings and click Launch.
* You’ll be prompted to select a key pair that will be used to SSH onto the EC2 instance. If you don’t already have a key pair you can create one now.

![](/images/uploads/ec2-wizard-7.png)

* Click Launch Instance to display the launch status screen shown below. At this point the instance is being created so you can navigate back to the EC2 instance landing screen.

![](/images/uploads/ec2-wizard-8.png)

* When the instance state changes to _running_ the instance is ready to use. Open the description tab and get the public IP that will be used to SSH onto the instance.

#### Step 3: SSH onto the instance:

We are using a windows machine, so we don't have native SSH client. We will be using putty and winscp to SSH onto the machine. Here's how:

* Install Putty, PuttyGen and WinSCP from here: [https://winscp.net/eng/downloads.php#putty_additional](https://winscp.net/eng/downloads.php#putty_additional "https://winscp.net/eng/downloads.php#putty_additional")
* From the **Start** menu, choose **All Programs**, **PuTTY**, **PuTTYgen**.
* Under **Type of key to generate**, choose **RSA**. If you're using an older version of PuTTYgen, choose **SSH-2 RSA**.

  ![](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/images/puttygen-key-type.png)
* Choose **Load**. By default, PuTTYgen displays only files with the extension `.ppk`. To locate your `.pem` file, choose the option to display files of all types.

  ![](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/images/puttygen-load-key.png)
* Select your `.pem` file for the key pair that you specified when you launched your instance and choose **Open**. PuTTYgen displays a notice that the `.pem` file was successfully imported. Choose **OK**.
* To save the key in the format that PuTTY can use, choose **Save private key**. PuTTYgen displays a warning about saving the key without a passphrase. Choose **Yes**.
* Specify the same name for the key that you used for the key pair (for example, `my-key-pair`) and choose **Save**. PuTTY automatically adds the `.ppk` file extension.
* Start PuTTY (from the **Start** menu, choose **All Programs, PuTTY, PuTTY**).
* In the **Category** pane, choose **Session** and complete the following fields:
  * In the **Host Name** box, do one of the following:
    * (Public DNS) To connect using your instance's public DNS name, enter _`my-instance-user-name`_@_`my-instance-public-dns-name`_.

    For information about how to get the user name for your instance, and the public DNS name of your instance, see [Get information about your instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connection-prereqs.html#connection-prereqs-get-info-about-instance).
  * Ensure that the **Port** value is 22.
  * Under **Connection type**, select **SSH**.

    ![](/images/uploads/ssh-putty-1.PNG)
* (Optional) You can configure PuTTY to automatically send 'keepalive' data at regular intervals to keep the session active. This is useful to avoid disconnecting from your instance due to session inactivity. In the **Category** pane, choose **Connection**, and then enter the required interval in the **Seconds between keepalives** field. For example, if your session disconnects after 10 minutes of inactivity, enter 180 to configure PuTTY to send keepalive data every 3 minutes.

  ![](/images/uploads/ssh-putty-3.PNG)
* In the **Category** pane, expand **Connection**, expand **SSH**, and then choose **Auth**. Complete the following:
  * Choose **Browse**.
  * Select the `.ppk` file that you generated for your key pair and choose **Open**.
  * (Optional) If you plan to start this session again later, you can save the session information for future use. Under **Category**, choose **Session**, enter a name for the session in **Saved Sessions**, and then choose **Save**.
  * Choose **Open**.

    ![](/images/uploads/ssh-putty-4.PNG)
  * If this is the first time you have connected to this instance, PuTTY displays a security alert dialog box that asks whether you trust the host to which you are connecting. Choose **Yes**. A window opens and you are connected to your instance.

#### Step 4: Installing JDK 1.8

Once we’re connected to the instance we need to do some basic setup. Switch to the root user, remove the default Java 7 JDK that comes bundled with the Amazon Machine Image and install JDK 1.8.

      sudo su
      yum remove java-1.7.0-openjdk -y
      yum install java-1.8.0 -y

#### Step 5: Run the application in AWS      

The EC2 image should now be ready to use, so all that remains is to copy up our application JAR and run it. On the command line use SCP to copy the application JAR to the EC2 instance..
When you SSH onto the EC2 instance the springboot-aws-starter.jar should be in /home/ec2-user/. Launch the application by running the same command you ran locally, not forgetting to supply the application config JSON.

      java -Dspring.application.json='{"db-instance-identifier":"starter-db","database-name": "starter_db","rdsUser":"XXXX","rdsPassword": "XXXXXXXXX"}' -Dspring.profiles.active=aws -jar springboot-aws-starter.jar

> Note: We won't need to specify the accesskey and secret accesskey here since the EC2 instance had the instanceProfile role: springboot-aws-starter-ec2-role.

When the application starts you should be able to access it on port 8080 using the public DNS displayed in description tab of the EC2 instances page.

  ![](/images/uploads/springboot-aws-starter-home.png)

In a production environment we wouldn’t access the application directly on the EC2 instance. Instead we’d configure an Elastic Load Balancer to route and distribute incoming traffic across multiple EC2 instances.

We will describe setting up Auto Scaling and Load Balancing in the next article here: [AutoScaling via Console](/aws/projects/autoscaling-loadbalancing/)
