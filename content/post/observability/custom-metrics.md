---
title: Custom Metrics
subtitle:

# Summary for listings and search engines
summary:

# Link this post with a project
projects: []

# Date published
date: "2024-03-15T00:00:00Z"

toc: true

# Date updated
lastmod: "2024-03-15T00:00:00Z"

# Is this an unpublished draft?
draft: true

# Show this page in the Featured widget?
featured: false

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
- Unix
- AWS
- Obervability

categories:

---

<!--more-->

### Overview

You want to track system performance report on a daily basis. You decided to create a Linux script to accomplish this.

Create a config.sh file using the vi command: ```vi config.sh```. This creates the file and opens it in the vi text editor.

Copy the following content to a local text editor so that you can replace all instances of **<instance id>** with the instance ID of your EC2 instance. 

```bash
TCP_CONN=$(netstat -an | wc -l)

TCP_CONN_PORT_80=$(netstat -an | grep 80 | wc -l)

TCP_SOCKETS=$(ss -s |awk 'NR==8{print $2}')

USERS=$(uptime |awk '{ print $3 }')

IO_WAIT=$(iostat | awk 'NR==4{print $3}')

USEDMEMORY=$(free -m | awk 'NR==2{printf "%.2f\t", $3*100/$2 }')

aws cloudwatch put-metric-data \
    --metric-name memory-usage \
    --dimensions Instance=<instance id>  \
    --namespace "CustomMetric" \
    --value $USEDMEMORY

aws cloudwatch put-metric-data `
    --metric-name Tcp_connections \
    --dimensions Instance=<instance id>  \
    --namespace "CustomMetric" \
    --value $TCP_CONN

aws cloudwatch put-metric-data \
    --metric-name TCP_connection_on_port_80 \
    --dimensions Instance=<instance id>  \
    --namespace "CustomMetric" \
    --value $TCP_CONN_PORT_80

aws cloudwatch put-metric-data \
    --metric-name No_of_users \
    --dimensions Instance=<instance id> \
    --namespace "CustomMetric" \
    --value $USERS

aws cloudwatch put-metric-data \
    --metric-name IO_WAIT \
    --dimensions Instance=<instance id>  \
    --namespace "CustomMetric" \
    --value $IO_WAIT

aws cloudwatch put-metric-data \
    --metric-name TCP_SOCKETS \
    --dimensions Instance=<instance id>  \
    --namespace "CustomMetric" \
    --value $TCP_SOCKETS
```

>Note: This script defines the parameters which you want to publish to CloudWatch. Linux commands are used to get the real time values for these parameters and ```put-metric-data``` command is used to publish the values to CloudWatch.

In the AWS Console, press ESC + i to enter vi's Insert mode, then paste the code from your local editor into the vi text editor.

Close the file by typing Esc, then :wq and Enter to save the changes.

Make sure that the file is available by executing the command- ls -lrt

Change the permissions of the file using chmod +x config.sh.

Execute the script using the command sh -x config.sh 

---

Sending the daily performance report of the system in text form is a bit of an old process. Sam asked you to create something visual which displays the real time data so that he doesn't have to check the report manually by opening the email. CloudWatch in AWS is the best solution for this. You decided to create a cronjob which generates the realtime data and sends it to CloudWatch every minute.

In this challenge, you will first create a cronjob and then publish the custom metrics to CloudWatch. The cronjob will be created so that the script created in the previous challenge is executed every minute to publish the data to CloudWatch.

In the AWS CLI, create a cronjob by executing the command crontab -e

Press Esc + i to insert the following so that the script is executed every minute and data is published to CloudWatch:

```bash 
*/1 * * * * /home/ec2-user/config.sh
```

Press Esc and then  :wq and Enter to save the cronjob. Next you’ll observe the metrics in the CloudWatch console. 

Back in the AWS Management console tab, navigate to Services > CloudWatch

Click on Metrics in the left pane.

Under Custom Namespaces, you will see the custom metric with the name, CustomMetric, which you created in AWS CLI. Click on CustomMetric>Instance, you will see the custom metric with the instance id which you created in AWS CLI.

Upon clicking instance id, you will be able to see the various metrics which you will publish to CloudWatch. 

---

Now, since you have published the custom metrics data to CloudWatch, you thought of creating a dedicated dashboard which displays only your custom metrics data. This will help anyone monitoring the system to identify any performance issues.

If you're not already viewing the list of custom metrics from the previous challenge, navigate back to it via Cloudwatch > Metrics > CustomMetric > Instance.

Select all the metrics in the list by clicking their corresponding checkboxes.

Provide a name for the graph (eg SystemPerformance) by clicking the edit (pencil) icon in the top left corner and then click the Apply button. 

To the left of the Actions dropdown, select Number.

Select Add to dashboard from the Actions dropdown.

Create a new dashboard by clicking on Create new. 

Give the dashboard a name, then click the Create button below the name input field. 

Click Add to dashboard and then Save dashboard.

 Your custom metric has now been published in CloudWatch and you can monitor the same through the new dashboard. 

---

Sam is very happy with the custom metrics dashboard which you created. He has now asked  if anything can be done to monitor the system performance automatically. You proposed that an alarm can be created based on the metrics data which can send an email if a threshold limit is breached. He is happy with this approach and has asked you to implement it.

Navigate to Alarms from the CloudWatch Console.

Click Create alarm and then Select metric.

Select the CustomMetric namespace which you created using the Linux script in the first Challenge.

Click on Instance and then select the metric for which you want to configure the alarm, for example, Tcp_connections.

Click Select metric.

On the Specify metric and conditions page, define the threshold value under the Conditions tab: select Static and then select Greater and provide threshold value as 165. 

Click Next, then in the Notification section, select In alarm and then select Create new topic.

Enter your email in Email endpoints that will receive the notification field, then click on Create topic. 

Click Next, then provide a name for the alarm, for example, CustomMetricAlarm.

 Click Next.

On the Preview and create page, check the details and then click Create alarm. 

You will receive an email in your inbox; go read it and click the Confirm subscription link to confirm the subscription.

Note: You will not get any email alert once the lab is completed and shut down. 

You will be able to see the alarm created successfully in the CloudWatch Console home page. If any threshold limit is reached, a notification email will be sent to all listed parties.

