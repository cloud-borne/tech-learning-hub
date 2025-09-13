---
title:  S3 Cross Account Access
subtitle: Guide to Share S3 Bucket🪣 Cross Account

# Summary for listings and search engines
summary: Guide to Share S3 Bucket🪣 Cross Account

# Link this post with a project
projects: []

# Date published
date: "2025-09-11T00:00:00Z"

toc: true

# Date updated
lastmod: "2025-09-11T00:00:00Z"

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
- IAM
- S3
- AWS

categories:

---

<!--more-->

### Overview

Demo a secure and common pattern for managing AWS resources across accounts. 

- ```Development``` Account: This account houses IAM groups for **Developers** and **Testers**:
  - **Developers** can assume a role that allows them to upload and download objects.
  - **Testers** can assume a more restrictive role that only allows them to view and list objects.

- ```Production``` Account: 
  - This account hosts the primary **S3** bucket🪣 with the core application data. 
  - It also hosts a **Lambda** function that acts as an automated processor. This function is triggered by S3 events (like s3:ObjectCreated:*), ensuring that actions are taken in real-time as data changes.

- ```Reporting``` Account: This account is used for storing metadata and logs. 
  - It hosts a reporting **S3** bucket🪣 storing metadata for each object on the Production bucket.
  - The Lambda function from the Production account assumes a role to write data into an S3 bucket here. 

```mermaid
graph TD
    subgraph Development Account
        dev_users[IAM Group: Developers]
        test_users[IAM Group: Testers]
    end

    subgraph Production Account
        prod_s3[S3 Bucket: Production Data]
        lambda[Lambda Function: Metadata Processor]
        prod_s3 -- S3 Event Trigger --> lambda
    end

    subgraph Reporting Account
        report_s3[S3 Bucket: Reporting Data]
    end

    dev_users -- "AssumeRole: Upload & Download (s3:PutObject, s3:GetObject)" --> prod_s3
    test_users -- "AssumeRole: View Only (s3:GetObject, s3:ListBucket)" --> prod_s3
    lambda -- "AssumeRole: Write Metadata (s3:PutObject)" --> report_s3

    style dev_users fill:#4a86e8,stroke:#fafafa,stroke-width:2px,color:#ffffff
    style test_users fill:#f1c232,stroke:#fafafa,stroke-width:2px,color:#000000
    style prod_s3 fill:#6aa84f,stroke:#fafafa,stroke-width:2px,color:#ffffff
    style lambda fill:#e06666,stroke:#fafafa,stroke-width:2px,color:#ffffff
    style report_s3 fill:#674ea7,stroke:#fafafa,stroke-width:2px,color:#ffffff

```

{{% callout soon %}}
Coming soon...
{{% /callout %}}