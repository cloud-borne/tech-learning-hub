---
title: Custom Logging
type: book
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


You were tasked with setting up a Web Server with basic logging enabled. The Web Server will run on an EC2 instance and its logs must be made available in CloudWatch. This will help other members of the team monitor requests to the Web Server without having to directly access the logs files in the instance.

Create an IAM Role for CloudWatch Agent

Under the Services tab, find and go to the IAM services pages.
In the navigation pane on the left, click Roles.
On the Roles page, click Create role.
Make sure that AWS ```ec2``` service is selected under the Select type of ```trusted entity``` section.

For Choose a use case, choose EC2 under Common use cases.

Choose Next: Permissions. This will take you to the Attach permissions policies page. 

In the new page, use the Search input field to search for CloudWatchAgentServerPolicy. This should filter down to a single policy of the same name

In the list of policies,  select the checkbox next to CloudWatchAgentServerPolicy.  This policy includes all necessary permissions for an EC2 instance to send log data to CloudWatch.

Click Next: Tags and then Next: Review.

On the Review page, type CloudWatchAgent in the Role name input field.

Click Create Role. This will take you back to the list of roles and the newly created role should be visible.

﻿

If the CloudWatchAgent role is displayed on the list of roles, congratulations! You’ve successfully created a Service Role for a CloudWatch Agent to run on an EC2 instance. Move on to the next challenge.