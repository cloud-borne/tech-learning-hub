---
title: Security
linktitle: 🗝️ Security
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 90
---

Security Fundamentals in AWS

<!--more-->

## 🔎Overview

Security in AWS is a **layered** discipline - the [OSI Model](https://en.wikipedia.org/wiki/OSI_model) offers a clear way to think about where and how to apply security controls. Here's the 7 Layers and the respective AWS Services.

![OSI-Layers](/images/uploads/aws-osi-layers-security.png)

This cheatsheet maps the seven layers of the ```OSI model``` to AWS services that help implement **defense-in-depth**. 

## 📱Layer 7: Application

```Layer-7``` is where user-facing applications operate (e.g., **HTTP**, **HTTPS**, **DNS**, **SMTP**). Security at this layer is critical because it directly interacts with users and is often the most targeted by attackers.

⚠️ Common Threats at ```Layer 7```

| 🚨 **Threat**               | 📝 **Description**                                                                 |
|----------------------------|------------------------------------------------------------------------------------|
| [SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) | Malicious SQL queries to manipulate or access databases.                          |
| [Cross Site Scripting](https://owasp.org/www-community/attacks/xss/) | Injecting malicious scripts into web pages viewed by other users.              |
| [Cross-Site Request Forgery(CSRF)](https://owasp.org/www-community/attacks/csrf) | Tricks users into executing unwanted actions on a web app.         |
| [Code Injection](https://owasp.org/www-community/attacks/Code_Injection) | Exploiting vulnerabilities to run arbitrary code on the server.             |
| [API Abuse](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/) | Excessive or malformed API calls to disrupt or exploit services.                  |
| [DDOS](https://owasp.org/www-community/attacks/Denial_of_Service) | Flooding application endpoints with high-volume traffic.                          |
| [Broken Authentication](https://owasp.org/www-project-top-ten/2017/A2_2017-Broken_Authentication)  | Exploiting weak session or credential handling mechanisms.                        |
| [Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure) | Insecure transmission or storage of personal or confidential data.    

#### 🚨Distributed Denial of Service (DDOS Attack)

![DDOS](/images/uploads/aws-ddos-attack.png)

* Service becomes unavailable because it’s receiving too many requests (```Layer-4``` attacks)
  * ```SYN Flood```: send too many TCP connection requests
  * ```UDP Reflection```: get other servers to send many big UDP requests
  * ```DNS flood``` attack: overwhelm the DNS so legitimate users can’t find the site
  * ```Slow Loris``` attack: a lot of HTTP connections are opened and maintained
* Application level attacks (```Layer-7``` attacks):
  * Complex, Application Specific (spike in ```POST``` requests to ```/login```) 
  * **Cache Bursting**💥: Overload the backend database by invalidating cache

#### 🧱AWS WAF

* Protects your web applications from common web exploits (```Layer 7```)
  * Deploy on Application Load Balancer (```ALB```) (localized rules)
  * Deploy on ```API Gateway``` (rules running at the **regional** or **edge-level**)
  * Deploy on ```CloudFront``` (rules **globally** on **edge-locations**)
  * Used to front other solutions: ```CLB```, ```EC2``` instances, custom origins, ```S3``` websites
  * Deploy on ```AppSync``` (protect your **GraphQL** APIs)
* ```WAF``` is not **only** for ```DDoS``` protection
* Define **Web ACL** (Web Access Control List):
  * Rules can include IP addresses, HTTP headers, HTTP body, or URI strings
  * Size constraints, Geo match
  * Rate-based rules (to count occurrences of events)
  * Rule Actions: **Count** | **Allow** | **Block** | **CAPTCHA Challenge**
  * Library of over 190 ready-to-use **managed** rules
  * **Baseline Rule Groups** – general protection from common threats ```AWSManagedRulesCommonRuleSet```, ```AWSManagedRulesAdminProtectionRuleSet```, …
  * **Use-Case Specific Rule Groups** – protection for many AWS WAF use cases ```AWSManagedRulesSQLiRuleSet```, ```AWSManagedRulesWindowsRuleSet```,```AWSManagedRulesPHPRuleSet```, ```AWSManagedRulesWordPressRuleSet```, …
  * **IP Reputation Rule Groups** – block requests based on source (e.g. malicious IPs ```AWSManagedRulesAmazonIpReputationList```, ```AWSManagedRulesAnonymousIpList```
  * **Bot Control Managed Rule Group** – block and manage requests from bots ```AWSManagedRulesBotControlRuleSet```

#### Solution Architecture - Enhance CloudFront Origin Security

![WAF Security](/images/uploads/aws-waf-security-solution-architecture.png)

To ensure that only traffic routed through Amazon ```CloudFront``` reaches an Application Load Balancer (```ALB```) — and to block direct access from end-users — you can implement a layered security approach using AWS WAF and custom headers. 
1. First, apply a ```WAF``` Web ACL at the CloudFront level to filter client requests. 
2. Then, configure ```CloudFront``` to inject a custom HTTP header (e.g., ```X-Origin-Verify```) with a secret value into each request. 
3. On the ```ALB```, set up a WAF rule that only allows traffic containing this header, effectively blocking direct access. 
4. For enhanced security, automate header rotation using AWS ```Secrets Manager``` and a ```Lambda``` function to periodically update both the ```CloudFront``` **header** and the ```ALB``` WAF rule.

#### Solution Architecture - Bad Bot Protection

![WAF Bad Bot](/images/uploads/aws-badbot-log-parser-flow.png)

This solution uses AWS ```WAF```, Amazon ```Kinesis Data Firehose```, Amazon ```S3```, and AWS ```Lambda``` to detect and block malicious bots automatically. 

1. A hidden trap endpoint acts as a *honeypot* to catch bots scanning the site. 
2. Incoming requests are inspected by AWS ```WAF```, which applies Bad Bot Protection rules and labels suspicious traffic. 
3. Logs are streamed via ```Kinesis Data Firehose``` providing buffering, scaling, transformation(convert **JSON** to **Parquet**), and reliable delivery into Amazon S3. 
4. An AWS Lambda function parses WAF logs from S3, filters entries with **bad-bot** labels or trap endpoint hits, and extracts the **Source IP**s. 
5. These IPs are added then added back to a ```WAF``` IP set for automated blocking for a configurable duration. 

#### 🕵️AWS Inspector

{{< figure src="images/uploads/aws-inspector-security.png" width="300" height="500" class="alignright">}}

Amazon ```Inspector``` adds another layer of defense by performing vulnerability scans on ```EC2``` instances, ```Containers```, and ```Lambda``` functions against a database of known **CVE**s

- For ```EC2``` instances: 
  * Leveraging the AWS System Manager(```SSM```) agent
  * Analyze against unintended network accessibility
  * Analyze the running OS against known vulnerabilities
- For ```Container``` Images push to Amazon ```ECR```:
  * Assessment of Container Images as they are pushed
- For ```Lambda``` Functions:
  * Identifies software vulnerabilities in function code and package dependencies
  * Assessment of functions as they are deployed
- Reporting & integration with AWS ```Security Hub```
- Send findings to Amazon ```Event Bridge```

#### 💂AWS GuardDuty

Amazon ```GuardDuty``` serves as a threat detection engine, continuously analyzing AWS account activity, ```VPC Flow logs```, and ```DNS``` queries for signs of compromised resources or malicious behavior.

![WAF GuardDuty](/images/uploads/aws-guardduty-security.png)

* Intelligent Threat discovery to protect your AWS Account 
* Uses **Machine Learning** algorithms, anomaly detection, 3rd party data
* One click to enable (30 days trial), no need to install software
* Input data includes:
  * CloudTrail ```Events Logs``` – unusual API calls, unauthorized deployments
  * CloudTrail ```Management Events``` – create VPC subnet, create trail, …
  * CloudTrail S3 ```Data Events``` – get object, list objects, delete object, …
  * ```VPC Flow Logs``` – unusual internal traffic, unusual IP address
  * ```DNS Logs``` – compromised EC2 instances sending encoded data within DNS queries
  * Optional Features – ```EKS``` Audit Logs, ```RDS``` & ```Aurora```, ```EBS```, ```Lambda```, ```S3``` Data Events…

{{< figure src="images/uploads/aws-guardduty-organization-security.png" width="200" height="300" class="alignright">}}
* Can setup ```EventBridge``` rules to be notified in case of findings
* EventBridge rules can target AWS ```Lambda``` or ```SNS```
* Can protect against **CryptoCurrency** attacks (has a dedicated *finding* for it)
* AWS Organization member accounts can be designated to be a GuardDuty Delegated Administrator
* Have full permissions to enable and manage GuardDuty for all accounts in the Organization
* Can be done only using the Organization Management Account

#### 🔗SSM ParameterStore

* Secure storage for configuration and secrets
* Optional Seamless Encryption using KMS
* ```Serverless```, scalable, durable, easy SDK
* Version tracking of configurations / secrets
* Configuration management using path & ```IAM```
* Notifications with Amazon ```EventBridge```
* Integration with ```CloudFormation```
{{< figure src="images/uploads/ssm-parameter-store.PNG" width="250" height="300" class="alignright">}}
* SSM Parameter Store Hierarchy:
  * /my-department/
    * my-app/
      * dev/
        * db-url
        * db-password
      * prod/
        * db-url
        * db-password
    * other-app/
  * /other-department/

* Two parameter tiers available: **Standard** Vs **Advanced** 

  | **Characteristics**                                             | **Standard** | **Advanced**                              |
  |-----------------------------------------------------------------|--------------|-------------------------------------------|
  | Total number of parameters allowed (per AWS account and Region) | 10,000       | 100,000                                   |
  | Maximum size of a parameter value                               | 4 KB         | 8 KB                                      |
  | Parameter policies available                                    | No           | Yes                                       |
  | Cost                                                            | Free         | $0.05 per advanced parameter per month    |

* Here's some examples of SSM **Advanced** Parameter Polcies
  ![SSM-Policies](/images/uploads/aws-kms-ssm-policies.png)

###### SSM Public Parameters

SSM public parameters are **read-only**, globally available parameters published by AWS under the ```/aws/service/``` namespace. They’re designed to:
- ✅ Avoid hardcoding resource IDs (e.g., AMI IDs)
- 🔄 Auto-update when AWS releases new versions
- 🌍 Support multi-region deployments seamlessly
- 🔐 Enable secure referencing of secrets via SSM syntax
They’re especially useful in CloudFormation, CDK, Terraform, and CI/CD pipelines where you want modular, future-proof infrastructure.

- Here's a list of common examples:

| Parameter Category | Example Full Parameter Path                                                           | Usage / Description                                                                                                                |
|--------------------|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| Amazon Linux       | /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2                         | Retrieves the AMI ID for the latest Amazon Linux 2 with HVM virtualization for an x86_64 architecture using a GP2 root volume.     |
| Windows Server     | /aws/service/ami-windows-latest/Windows_Server-2022-English-Full-Base                 | Gets the AMI ID for the base image of Microsoft Windows Server 2022 (English, Full installation).                                  |
| ECS-Optimized      | /aws/service/ecs/optimized-ami/amazon-linux-2/recommended                             | Fetches the recommended AMI ID for running Amazon ECS container instances on an Amazon Linux 2 base.                               |
| EKS-Optimized      | /aws/service/eks/optimized-ami/1.30/amazon-linux-2/recommended                        | Provides the recommended AMI ID for an Amazon EKS worker node compatible with Kubernetes version 1.30 on Amazon Linux 2.           |
| Deep Learning      | /aws/service/deep-learning-amis/latest/ubuntu-22.04                                   | Finds the latest Deep Learning AMI based on Ubuntu 22.04, pre-packaged with popular machine learning frameworks.                   |
| RDS Custom         | /aws/service/rds/optimized-ami/oracle-ee/19.21.0.0.ru-2023-10.rur-2023-10.r1/x86_64/1 | Stores the AMI ID for a specific version of Amazon RDS Custom for Oracle Enterprise Edition, ensuring a compatible OS environment. |

#### 🗄️Secrets Manager

{{< figure src="images/uploads/aws-ssm-secrets-manager.png" width="200" height="300" class="alignright">}}

* Meant for storing secrets (e.g., passwords, API keys)
* Capability to **force rotation** of secrets every X days
  * Automate generation of secrets on rotation (uses ```Lambda```)
  * Natively supports Amazon ```RDS``` (all supported DB engines), Redshift, DocumentDB
  * Support other databases and services (custom Lambda function)
* Control access to secrets using ```Resource-based``` Policy
* Integration with other AWS services to natively, pull secrets from ```Secrets Manager```: CloudFormation, CodeBuild, ECS, EMR, Fargate, EKS, Parameter Store…

###### Secrets Manager – with CloudFormation
![SSM-Secrets-Manager](/images/uploads/aws-ssm-secrets-manager-cloudformation.png)

###### Secrets Manager – Cross Account

1. Account B must be allowed to ```decrypt``` the secret in Account A. 

    KMS ```Key Policy``` in Account A:

    ```json
    {
      "Sid": "AllowAccountBToDecrypt",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountB-ID>:role/<RoleName>"
      },
      "Action": [
        "kms:Decrypt",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    }
    ```
2. Attach a ```Resource-Based``` Policy to the Secret:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "AllowAccountBToAccessSecret",
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::<AccountB-ID>:root"
          },
          "Action": [
            "secretsmanager:GetSecretValue",
            "secretsmanager:DescribeSecret"
          ],
          "Resource": "*"
        }
      ]
    }
    ```
    ![SSM-Secrets-Manager-Cross-Account](/images/uploads/aws-ssm-secrets-manager-cross-account.png)

#### 🆚Secrets Manager vs. SSM ParameterStore

{{< tabs name="Secrets Manager vs SSM ParameterStore" >}}
{{% tab name="Secrets Manager" %}}
* Automatic rotation of secrets with AWS Lambda
* Integration with RDS, Redshift, DocumentDB
* KMS encryption is mandatory
* Can integration with CloudFormation
{{% /tab %}}
{{% tab name="SSM ParameterStore ($)" %}}
* Simple API
* No secret rotation
* KMS encryption is optional
* Can integration with CloudFormation
* Can pull a Secrets Manager secret using the SSM Parameter Store API
{{% /tab %}}
{{< /tabs >}}

## 🧬Layer 6: Presentation

The Presentation layer is responsible for translating, formatting, and securing data before it reaches the Application layer. In modern cloud environments, this often means ensuring data is **encrypted** both in transit🚇 and at rest💤 using trusted protocols and managed cryptographic services. Weaknesses at this layer can lead to data leakage, **man-in-the-middle** attacks, or violations of compliance requirements.

#### 🔐Encryption

Encryption is a critical component of a defense-in-depth strategy, which is a security approach adopted by AWS with a series of defensive mechanisms designed so that if one security mechanism fails, there’s at least one more still operating.

There are 2 forms of encryption in practice:

##### Encryption in transit🚇:

* Data is encrypted before sending and decrypted after receiving
* SSL certificates help with encryption (HTTPS)
* Encryption in flight ensures no **MITM** can happen

![HTTPS](/images/uploads/encryption-in-transit.PNG)

##### Encryption at Rest💤:

* Data is encrypted after being received by the server
* Data is decrypted before being sent
* It is stored in an encrypted form thanks to a key (usually a data key)
* The encryption/decryption keys must be managed somewhere and the server must have access to it.
* There are two main methods to encrypt data at rest:

  * **Client-Side** Encryption: As the name implies this method encrypts your data at the client-side before it reaches backend servers or services. You have to supply encryption keys 🔑 to encrypt the data from the client-side. You can either manage these encryption keys yourself or use AWS ```KMS```(Key Management Service) to manage the encryption keys under your control.
  * AWS provides multiple client-side SDKs to make this process easy for you. E.g. AWS Encryption SDK, S3 Encryption Client, DynamoDB Encryption Client etc…

  ![Client-Side](/images/uploads/encryption-at-rest-client.PNG)

  * **Server-Side** Encryption: In Server-Side encryption, AWS encrypts the data on your behalf as soon as it is received by an AWS Service. Most of the AWS services support server-side encryption. E.g. S3, EBS, RDS, DynamoDB, Kinesis, etc…

  * All these services are integrated with AWS ```KMS``` in order to encrypt the data.

  ![Server-Side](/images/uploads/encryption-at-rest-server.PNG)

#### AWS Certificate Manager(ACM)

{{< figure src="images/uploads/aws-acm-security.png" width="200" height="300" class="alignright">}}

* To host public SSL certificates in AWS, you can:
  * Buy your own and upload them using the CLI
  * Have ```ACM``` provision and renew public SSL certificates for you (free of cost)
* ```ACM``` loads SSL certificates on the following integrations:
  * ```Load Balancers``` (including the ones created by EB)
  * ```CloudFront``` distributions
  * APIs on ```API Gateways```
* ACM is a **regional** service
  * To use with a global application (multiple ```ALB``` for example), you need to issue an SSL certificate in each region where you application is deployed. 
  * You cannot copy certs across regions

#### 🗝️KMS

AWS Key Management Store (```KMS```) is a managed service that enables you to easily **encrypt** your data.
AWS KMS provides a highly available key storage, management, and auditing solution for you to encrypt data within your own applications and control the encryption of stored data across AWS services.

KMS is used to fully manage the keys & their policies:
* 🆕Create
* 🔄Rotation policies
* ⏸️Disable
* ▶️Enable
* 🔎Able to audit key usage (using CloudTrail)
* 💶Pay for API call to KMS ($0.03 / 10000 calls)

###### KMS Key Types

```markmap {height="300px"}
- 🔐**KMS Keys**
  - 🔧**Customer Managed Keys**
      Full lifecycle control  
      Custom key policies  
      $1/month + usage
    - 🔁**Symmetric Keys**
        AES-256  
        Used for ```Encrypt/Decrypt```  
        - 📨**Data Keys**
             Ephemeral symmetric keys  
             Used for local encryption of large data (Envelope Encryption)
        - 📥**Imported Key Material**
             Bring your own key (```BYOK```)
        - 🏦**External Key Store (```XKS```)**
             Key lives outside AWS  
             AWS KMS acts as proxy  
    - 🔀**Asymmetric Keys**
         RSA or ECC key pairs  
         Used for ```Encrypt/Decrypt``` or ```Sign/Verify```
    - 🌍**Multi-Region**
         Symmetric and Asymmetric keys  
         Enables **cross-region** encryption/decryption  
         Same key material, separate resource IDs
    - 🧰**Cloud HSM**
         Symmetric and Asymmetric keys
         AWS provsisioned **dedicated** hardware
  - 🛠️**AWS Managed Keys**
       Created and managed by AWS  
       Always 🔁**Symmetric Keys**  
       View Key Policy(ReadOnly) and audit in CloudTrail
       Free
  - 🏢**AWS Owned Keys**
       Internal to AWS  
       Used for **default** encryption  
       Always 🔁**Symmetric Keys**  
       No access to find/editds Key Policy
       Free
```

###### Customer Managed Key (CMK) Types

* 🔁**Symmetric** (```AES-256``` keys)

  * Single encryption key that is used to ```Encrypt``` and ```Decrypt```
  * AWS services that are integrated with KMS use ```Symmetric``` CMKs
  * Necessary for ```Envelope``` encryption
  * You never get access to the unencrypted Key (must call KMS APIs to use)
  * KMS can only help in encrypting up to **4KB** of data per call. If data > 4 KB, then use ```Data Keys```.
  * To give access to KMS to someone:
    * Make sure the Key Policy allows the user
    * Make sure the IAM Policy allows the API calls

![KMS-API](/images/uploads/kms-encrypt-decrypt.PNG)

* 🔀**Asymmetric** (```RSA``` & ```ECC``` key pairs)

  * Public (```Encrypt```) and Private Key (```Decrypt```) pair
  * Used for ```Encrypt/Decrypt```, or ```Sign/Verify``` operations
  * ```Public``` key is **downloadable**; ```Private``` key remains **protected**.
  * Anything encrypted with a ```public``` key can only be decrypted by the corresponding ```private``` key.
  * 🧠Use cases: 
    * Encryption outside of AWS by users who can’t call the KMS API
    * Public distribution of software packages (say ```Docker```) where end users needs to verify the authenticity.

{{% callout note %}}

Key Considerations:

1. A key drawback to ```asymmetric``` cryptography is the fact that you cannot encrypt large pieces of data. When you have a 2048-bit RSA key pair and encrypt something by using the cipher RSAES_OASEP_SHA_256, the largest amount of data that you can encrypt is **190** bytes.

2. In contrast, ```symmetric``` encryption ciphers that use a chained or counter-mode operation don’t have this limit, and they make it possible for you to encrypt data in the tens-of-gigabytes.

3. KMS keys (symmetric or asymmetric) encryption limit: **4KB** (4096 bytes) per Encrypt API call - this is a ```KMS``` service-imposed limit, not a ```cryptographic``` one. 

4. So typically the client would use a [hybrid cryptosystem](https://en.wikipedia.org/wiki/Hybrid_cryptosystem) i.e. the client encrypts its large payload by using a symmetric key, then encrypts that symmetric key by using the downloaded RSA public key. End clients then transmit only encrypted **data** and encrypted **key** across insecure channels, maintaining privacy of the payload data.

{{% /callout %}}

###### KMS Asymmetric Keys Encrypt-Decrypt (Offline) flow

![KMS-Encrypt-Decrypt](/images/uploads/aws-kms-asymmetric-encrypt-decrypt.png)

1. Create an ```RSA``` key pair in AWS KMS.
2. Download or pre-install the AWS KMS ```public key``` to an end-client device.
3. Generate an **AES 256-bit** (```symmetric```) key on an end client.
4. Encrypt a large payload of data on the end client by using the **AES 256-bit** key.
5. Encrypt the AES 256-bit key with the AWS KMS ```public key```.
6. Transfer the encrypted **payload** and **key**.
7. Decrypt the AES 256-bit key by using RSA ```private key``` in KMS.
8. Decrypt the payload data by using the now decrypted AES 256-bit key.

###### KMS Asymmetric Keys Sign-Verify flow

The asymmetric CMKs offer digital signature capability, which data consumers can use to verify that data is from a trusted producer and is unaltered in transit.

![KMS-Sign-Verify](/images/uploads/aws-kms-asymmetric-sign-verify.png)

1. During system setup, the ```Signer``` is provisioned with an ```asymmetric``` key pair. Signers such as AWS KMS support internal hardware security module (HSM)-backed, high-entropy asymmetric key generation and management.
2. The ```Verifiers``` are configured to trust the ```Signer``` through an offline import of the Signer’s ```public key```.
3. During runtime, AWS KMS is requested to hash and create a digital signature over some original data. AWS KMS hashes the provided data and uses the ```private key``` in the asymmetric key pair to compute the signature over the hash. The original data, along with its signature, is delivered to a client.
4. The client forwards the data and signature to one or more ```Verifiers```, and requests access to their protected resources.
5. The ```Verifiers``` verify the signature that is associated with the original data. The Verifiers use the Signer’s ```public key``` in this process. If the verification succeeds, meaning that the original data that was conveyed is unaltered and authentic, the ```Verifiers``` grant the client access to their protected resources.

* 🌍**Multi-Region Keys**

  * A set of identical KMS keys in different AWS Regions that can be used interchangeably (~ same KMS key in multiple Regions)
  * Encrypt in one Region and decrypt in other Regions (No need to re-encrypt or making cross-Region API calls)
  * ```Multi-Region``` keys have the same ```Key ID```, key material, automatic rotation, …
  * KMS Multi-Region are **NOT** global (**Primary** + **Replicas**)
  * Each Multi-Region key is managed **independently**
  * Only **one** primary key at a time, can promote replicas into their own primary
  * 🧠Use cases: Disaster Recovery, Global Data Management (e.g., DynamoDB Global Tables), Active-Active Applications that span multiple Regions, Distributed Signing applications

    ![KMS-Multi-Region](/images/uploads/aws-kms-multi-region-key.png)

* 📥KMS Key Source - External (**BYOK**)
  * Import your own key material into KMS key, Bring Your Own Key (```BYOK```)
  * You’re responsible for key material’s security, availability, and durability outside of AWS
  * Supports **only** ```Symmetric``` KMS keys
  * Manually rotate your KMS key (Automatic & On-demand Key Rotation are NOT supported)
{{% callout note %}} 
- AWS KMS enforces strict controls to ensure private keys remain secure and non-exportable.
- Allowing ```Asymmetric``` key import would break the trust model where KMS guarantees the private key never leaves its boundary.
{{% /callout %}}

  ![KMS-BYOK](/images/uploads/aws-kms-byok.png)

* 🧰**CloudHSM**
  
  {{< figure src="images/uploads/aws-kms-cloudhsm.png" width="300" height="500" class="alignright">}}
  * CloudHSM => **AWS** provisions encryption **hardware**
  * **Dedicated** Hardware (HSM = Hardware Security Module)
  * **You** manage your own encryption keys entirely (not AWS)
  * HSM device is tamper resistant, FIPS 140-2 Level 3 compliance
  * Supports both ```symmetric``` and ```asymmetric``` encryption (SSL/TLS keys)
  * **No free tier** available
  * Must use the CloudHSM Client Software
  * Redshift supports CloudHSM for database encryption and key management
  * Good option to use with SSE-C encryption

* 📨**Envelope Encryption**

  * KMS ```Encrypt API``` and ```Decrypt API``` calls have a limit of **4KB**
  * If you want to encrypt >4 KB, we need to use ```Envelope Encryption``` using ```Data Keys```
  * Data Keys are generated from CMKs. There is a direct relationship between Data Key and a CMK. However, AWS does NOT store or manage Data Keys. Instead, you have to manage them.
  * The main API that will help us is the ```GenerateDataKey``` API
  * You can use one Customer Managed Key (```CMK```) to generate thousands of unique data keys. You can generate data keys from a CMK using two methods:
    * Generate both Plaintext Data Key and Encrypted Data Key (```GenerateDataKey```)
    * Generate only the Encrypted Data Key (```GenerateDataKeyWithoutPlaintext```)
  * After encryption, never keep the Plaintext data key together with Encrypted data(Ciphertext) since anyone can decrypt the Ciphertext using the Plaintext key. So remove the Plaintext data key from the memory as soon as possible. You can keep the Encrypted data key with the Ciphertext.  
  * The method of encrypting the key using another key is called ```Envelope Encryption```. By encrypting the key, that is used to encrypt data, you will protect both data and the key.
  * The whole purpose of this ```Envelope encryption``` technique is to use KMS for what it's good at, which is to generate keys,
  and then the whole encryption and decryption happens at the **client side**.

  ```mermaid
  sequenceDiagram
      participant App
      participant KMS
      App->>KMS: GenerateDataKey
      KMS-->>App: Plaintext Key + Encrypted Key
      App->>App: Encrypt data with Plaintext Key
      App->>App: Discard Plaintext Key
      App->>Storage: Store Encrypted Data + Encrypted Key
  ```

  1. Client invokes the ```GenerateDataKey``` API into KMS.
  2. Client gets the **Plaintext** data key and **Encrypted** data key from KMS, use the Plaintext data key to encrypt your large data on the **client side**. 
  3. Discard the **Plaintext** data key and store the **Encrypted data key** together with the **Encrypted data** together as an "Envelope"

  ![KMS-Envelope-Encrypt](/images/uploads/kms-envelope-encrypt.PNG)

  4. When you want to decrypt it, call the KMS ```Decrypt``` API with the encrypted data key (🤔**4KB** limit).
  5. KMS will send you the Plaintext key if you are authorized to receive it. 
  6. Afterward, you can decrypt the Ciphertext using the Plaintext key on the **client side** again.

  ![KMS-Envelope-Decrypt](/images/uploads/kms-envelope-decrypt.PNG)

###### Encryption SDK

* The AWS Encryption SDK implemented Envelope Encryption for us
* The Encryption SDK also exists as a CLI tool we can install
* Implementations for Java, Python, C, JavaScript
* Feature - Data Key Caching:

  * Re-use data keys instead of creating new ones for each encryption
  * Helps with reducing the number of calls to KMS with a security trade-off
  * Use ```LocalCryptoMaterialsCache``` (max age, max bytes, max number of messages)

###### KMS Key Policies

* One of the powerful features in KMS is the ability to define permission separately for those who **use** the keys and **administrate** the keys. This is achieved using Key Policies (it's a ```Resource Based``` Policy). You can control access to KMS keys, *similar* to S3 bucket policies.

* **Default** KMS Key Policy:
  * Created if you don’t provide a specific KMS Key Policy
  * Complete access to the key to the ```root``` user = entire AWS account
  * Gives access to the IAM policies to the KMS key
  * The below Key Policy (**Default**) is applied to the ```root``` user of the account. It allows full access to the CMK for any user in the account.

    ```JSON
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
      "Action": "kms:*",
      "Resource": "*"
    }
    ```

* **Custom** KMS Key Policy:
  * Define users, roles that can access the KMS key
  * Define who can administer the key
  * Useful for **cross-account** access of your KMS key
  * If you choose to distinguish users and roles who can manage key usage and key administration then we could achieve as follows:
  * The below IAM policy is applied to the IAM user ```KeyUser```. Now he has permission to use the CMK for encryption and decryption. However, he is not allowed to administrate that CMK.

    ```JSON
      {
      "Sid": "Allow use of the key",
      "Effect": "Allow",  
      "Principal": {"AWS": "arn:aws:iam::111122223333:user/KeyUser"},
      "Action": [
        "kms:Decrypt",
        "kms:DescribeKey",
        "kms:Encrypt",
        "kms:GenerateDataKey*",
        "kms:ReEncrypt*"
      ],
      "Resource": "*"
    }
    ```
  * The below **IAM policy** allows administrators (```KeyManager```) to administrate the CMK that it is applied to. However, the administrator cannot use the key to Encrypt or Decrypt data.

    ```JSON
      {
      "Sid": "Allow access for Key Administrators",
      "Effect": "Allow",
      "Principal": {"AWS": [
        "arn:aws:iam::111122223333:user/KeyManager"
      ]},
      "Action": [
        "kms:Create*",
        "kms:Describe*",
        "kms:Enable*",
        "kms:List*",
        "kms:Put*",
        "kms:Update*",
        "kms:Revoke*",
        "kms:Disable*",
        "kms:Get*",
        "kms:Delete*",
        "kms:TagResource",
        "kms:UntagResource",
        "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion"
      ],
      "Resource": "*"
    }
    ```

###### Cross-Account Key Policy

KMS keys are generally scoped per **Region**. That means if you have to copy a KMS encrypted ```EBS``` Volume across region:
- 🚫 Default AWS Keys **can't** Be Used: Cross-region or cross-account snapshot copies require customer-managed CMKs — AWS-managed keys (```aws/ebs``` etc.) are not eligible because you cannot leverage a custom ```Key Policy```

- 🔄 Shared CMK (```Trusted Zone```): If both accounts are within a trusted boundary, you can share the source CMK via Key Policy. The target account can copy the snapshot using the same CMK ARN. For **Shared** CMK, the source ```Key Policy``` must explicitly allow cross-account access (```kms:Decrypt```, ```kms:ReEncrypt```, etc.)

- 📜 Key Policy & IAM Setup:  
```json
{
  "Version": "2012-10-17",
  "Id": "CrossAccountAccessKeyPolicy",
  "Statement": [
    {
      "Sid": "AllowRootAccountAFullAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_A_ID:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "AllowAccountBViaEC2",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_B_ID:root"
      },
      "Action": [
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:CreateGrant",
        "kms:DescribeKey"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "ec2.us-west-2.amazonaws.com",
          "kms:CallerAccount": "ACCOUNT_B_ID"
        },
        "Bool": {
          "kms:GrantIsForAWSResource": "true"
        }
      }
    }
  ]
}
```

- 🔐 Isolated CMK (```Untrusted Zone```): If CMK sharing isn't allowed, the target account must copy the snapshot using its own CMK. AWS securely decrypts and re-encrypts the data during transfer. For **Isolated** CMK, the target account ```IAM Role``` must allow ```ec2:CopySnapshot``` and ```kms:Encrypt``` using the target CMK.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CopySharedSnapshot",
      "Effect": "Allow",
      "Action": [
        "ec2:CopySnapshot",
        "ec2:DescribeSnapshots"
      ],
      "Resource": "*"
    },
    {
      "Sid": "UseTargetCMKForEncryption",
      "Effect": "Allow",
      "Action": [
        "kms:Encrypt",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "arn:aws:kms:us-west-2:TARGET_ACCOUNT_ID:key/TARGET_KEY_ID"
    }
  ]
}
```

###### KMS Request Quotas

* When you exceed a request quota, you get a ThrottlingException:

  * To respond, use **exponential**🌀 backoff (backoff and retry)
  * For cryptographic operations, they share a quota. This includes requests made by AWS on your behalf (ex: SSE-KMS)
  * For ```GenerateDataKey```, consider using DEK caching from the Encryption SDK
  * You can request a Request Quotas increase through API or AWS support

| API operation                                                                                                                                                               | Request quotas (per second)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Decrypt<br>Encrypt<br>GenerateDataKey (symmetric)<br>GenerateDataKeyWithoutPlaintext (symmetric)<br>GenerateRandom<br>ReEncrypt<br>Sign (asymmetric)<br>Verify (asymmetric) | These shared quotas vary with the AWS Region and the type of CMK used in the request. Each quota is calculated separately.<br>Symmetric CMK quota:<br>* 5,500 (shared)<br>* 10,000 (shared) in the following Regions:<br>* us-east-2, ap-southeast-1, ap-southeast-2,<br>ap-northeast-1, eu-central-1, eu-west-2<br>* 30,000 (shared) in the following Regions:<br>* us-east-1, us-west-2, eu-west-1<br>Asymmetric CMK quota:<br>* 500 (shared) for RSA CMKs<br>* 300 (shared) for Elliptic curve (ECC) CMKs |

###### S3 Encryption for Objects

* There are several methods of encrypting objects in S3 at **rest**

  * ```SSE-S3```: encrypts S3 objects using keys handled & managed by AWS - **AWS Owned Keys**
  * ```SSE-KMS```: leverage AWS Key Management Service to manage encryption keys - **AWS Managed Keys**
  * ```SSE-C```: when you want to manage your own encryption keys - key not stored in ```KMS```. **CloudHSM** could be used
  * ```Client Side``` Encryption - key not stored in ```KMS```
  * ```Glacier```: all data is **AES-256** encrypted, key under AWS control

* Encryption in **transit** (```SSL/TLS```)

  - Every S3 bucket and object is accessible via:
    - **HTTP**: http://bucket-name.s3.amazonaws.com
    - **HTTPS**: https://bucket-name.s3.amazonaws.com
  - These endpoints are globally available and automatically provisioned by AWS.
  - You’re free to use the endpoint you want, but HTTPS is recommended
  - HTTPS is mandatory for ```SSE-C```
  - To enforce HTTPS, use a Bucket Policy with a ```ws:SecureTransport```

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "AllowSSLRequestsOnly",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:*",
        "Resource": [
          "arn:aws:s3:::your-bucket-name",
          "arn:aws:s3:::your-bucket-name/*"
        ],
        "Condition": {
          "Bool": {
            "aws:SecureTransport": "false"
          }
        }
      }
    ]
  }
  ```

  ###### SSE-KMS

  * ```SSE-KMS``` encryption is using keys handled & managed by KMS
  * Object is encrypted server side
  * Must set header: ```x-amz-server-side-encryption```: ”aws:kms"
  * SSE-KMS leverages the ```GenerateDataKey``` & ```Decrypt``` KMS API calls
  * Even if S3 bucket/objects were made **public**, if it's encrypted with ```SSE-KMS```, they can never be read. Because a public **anonymous** user will **never** have access to ```KMS key```.
  * These KMS API calls will show up in CloudTrail, helpful for logging
  * To perform SSE-KMS, you need:
    * A KMS ```Key Policy``` that authorizes the user / role
    * An ```IAM policy``` that authorizes access to KMS, otherwise you will get an ```Access Denied``` error
    * Security Policy for enforcing encryption via SSE-KMS:
      ```JSON
      {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Deny",
                "Principal": "*",
                "Action": "s3:PutObject",
                "Resource": "arn:aws:s3:::$BucketName/*",
                "Condition": {
                    "StringNotEqualsIfExists": {
                        "s3:x-amz-server-side-encryption": "aws:kms"
                    },
                    "Null": {
                        "s3:x-amz-server-side-encryption": "false"
                    }
                }
            },
            {
                "Effect": "Deny",
                "Principal": "*",
                "Action": "s3:PutObject",
                "Resource": "arn:aws:s3:::$BucketName/*",
                "Condition": {
                    "StringNotEqualsIfExists": {
                        "s3:x-amz-server-side-encryption-aws-kms-key-id": "arn:aws:kms:$Region:$AccountId:key/$KeyId"
                    }
                }
            }
        ]
      }
      ```
  
  * KMS **Pros**: User control + Audit trail via ```CloudTrail```
  * KMS **Cons**: Each object upload typically triggers a ```GenerateDataKey``` API call to KMS, which incurs cost, latency and is rate-limited.
    * If throttling, try exponential backoff or you can request an increase in KMS limits
    * The service throttling is KMS, not Amazon S3
  {{< figure src="images/uploads/aws-kms-s3-bucket-key.png" width="300" height="500" class="alignright">}}
  * ```S3 Bucket Key``` for SSE-KMS: To circumvent these limitations AWS has introduced a ```S3 Bucket Key``` which is a **bucket-level** data key that Amazon S3 uses to reduce the number of direct calls to AWS KMS.
  * **Auditability** — fewer KMS calls means fewer CloudTrail entries, but you still get visibility into the initial key generation.
  * How it works:
    1. When enabled, KMS issues a **bucket-level** key (encrypted under your CMK) for the same ```GenerateDataKey``` API call
    2. S3 **caches** this bucket key internally for a limited time.
    3. For subsequent object uploads, S3 uses this cached bucket key to generate per-object data keys, without calling KMS again.
    4. The object is encrypted using the derived data key, and the metadata includes the encrypted bucket key.

## 🗪 Layer 5: Session

The session layer manages the creation, maintenance, and termination of communication sessions between systems. In modern applications, these sessions often involve authenticated users, federated identities, or temporary roles. At this layer, threats like session hijacking, token replay, or unauthorized impersonation can compromise the integrity of secure interactions.

#### AWS Cognito

#### AWS STS

#### Amazon API Gateway

#### IAM Identity Center

## 🚦Layer 4: Transport

The transport layer governs how data is transmitted between systems over protocols such as TCP and UDP. At this level, attackers often attempt to exploit open ports, overwhelm systems with malformed packets or connection floods, or entirely disrupt communication channels. Protecting this layer means controlling access to specific ports, managing protocol behavior, and filtering out malicious traffic early in the request flow.

#### 🛡️AWS Shield

AWS ```Shield``` provides automated, scalable, and **cost-protected** DDoS defense across network and application layers.

* AWS ```Shield``` Standard:
  * **Free** service that is activated for every AWS customer
  * Provides protection from attacks such as SYN/UDP Floods, Reflection attacks and other ```Layer-3```/```Layer-4``` attacks
* AWS ```Shield Advanced```: 
  * Optional DDoS mitigation service (**$3,000**💰 per month per organization) 
  * Protect against more sophisticated attack on Amazon EC2, Elastic Load Balancing (ELB), Amazon CloudFront, AWS Global Accelerator, Route 53
  * 24/7 access to AWS DDoS response team (DRP)
  * Protect against higher fees during usage spikes due to **DDoS**

#### 🧑‍💻AWS Firewall Manager

* Manage rules in all accounts of an AWS Organization
* Security policy: common set of security rules
* WAF rules (Application Load Balancer, API Gateways, CloudFront)
* AWS Shield Advanced (ALB, CLB, NLB, Elastic IP, CloudFront)
* Security Groups for EC2, Application Load Balancer and ENI resources in VPC
* AWS Network Firewall (VPC Level)
* Amazon Route 53 Resolver DNS Firewall
* Policies are created at the region level
* Rules are applied to new resources as they are created (good for 
compliance) across all and future accounts in your Organization

* ```WAF```, ```Shield``` and ```Firewall Manager``` are used together for comprehensive protection
* Define your Web ACL rules in ```WAF```
* For granular protection of your resources, ```WAF``` alone is the correct choice
* If you want to use AWS WAF across accounts, accelerate WAF configuration, automate the 
protection of new resources, use ```Firewall Manager``` with AWS ```WAF```
* ```Shield Advanced``` adds additional features on top of AWS WAF, such as dedicated support 
from the Shield Response Team (SRT) and advanced reporting.
* If you’re prone to frequent DDoS attacks, consider purchasing Shield Advanced

## 🌐Layer 3: Network

The network layer controls how data is routed between systems using IP addressing. It plays a central role in cloud security architecture, as it’s often the first line of defense against external threats. At this layer, attackers may attempt [IP spoofing](https://owasp.org/www-community/pages/attacks/ip_spoofing_via_http_headers) and [Distributed denial-of-service (DDoS)](https://owasp.org/www-community/attacks/Denial_of_Service) attacks to overwhelm resources or probe for vulnerable endpoints.

#### Amazon Virtual Private Cloud (VPC)

```VPC``` is your private network space in AWS. It gives you full control over IP addressing, subnetting, and routing. From a security standpoint, it’s the foundational, you isolate workloads, define access boundaries, and segment environments (e.g., ```prod``` vs ```dev```). Think of it as your digital fortress🏰

#### Network ACLs (NACL)

```NACLs``` act as **stateless** firewalls at the ```subnet``` level. They inspect traffic entering and leaving subnets, applying rules to **ALLOW** or **DENY** based on IP, port, and protocol. Since they’re stateless, both **inbound** and **outbound** rules must be defined. They’re great for broad perimeter filtering and protecting against unwanted traffic. 

#### Route Tables

```Route Tables``` determine how traffic flows within your VPC. While not a security tool per se, they’re critical for controlling exposure, for example, ensuring **private** subnets don’t route traffic to the internet. Misconfigured routes can lead to data leaks or unintended access paths.

#### NAT Gateways

```NAT Gateways``` allow instances in **private** subnets to initiate outbound connections to the internet (e.g., for updates) without exposing them to inbound traffic. This is a key security control, you get internet access without compromising internal resource visibility.

#### Route 53

```Route 53``` is AWS’s DNS service. It supports private ```hosted zones```, which are essential for internal name resolution without exposing DNS records publicly. It also offers DNS failover and health checks, helping maintain availability and resilience against attacks like DNS spoofing. 

## 🔢Layer 2: Data Link

The ```Data Link``` Layer handles node-to-node communication using Ethernet frames, MAC addresses, and VLANs. In AWS, traditional ```Layer-2``` risks like **MAC spoofing**, **ARP poisoning**, and **VLAN hopping** are mitigated because ```Layer-2``` is fully abstracted from customers. Amazon ```VPC``` is designed to isolate network traffic at the virtualization layer, eliminating direct exposure to MAC addresses or switching infrastructure.

**__Key AWS Controls:__**

- No raw ```Layer 2``` exposure: Customers cannot send Ethernet frames directly.
- VPC Isolation: Logical isolation prevents cross-tenant traffic.
- Managed ARP & No Broadcast Domains: Eliminates ARP spoofing and floods.
- ```Security Groups``` & ```NACLs```: Enforce traffic filtering at higher layers.

**__Direct Connect & Layer 2:__**

- Provides **dedicated physical connectivity** using Ethernet (```Layer-2```).
- Supports **802.1Q VLAN** tagging and **Link Aggregation** (LAG).
- AWS enforces **port-level security** and **MAC filtering** at Direct Connect locations.
vant Resources

## 🏢Layer 1: Physical

The physical layer secures the most basic infrastructure — cabling, power, servers, and data center access. In AWS, this responsibility shifts from customers to AWS, which owns and operates global facilities.

Threats include hardware tampering, rogue devices, signal interception, theft of media, and environmental risks like power loss or overheating. These bypass software defenses, making physical security the first line of protection.
AWS prevents customer access and enforces strict controls validated by compliance reports via ```AWS Artifact``` (**SOC 2**, **ISO 27001**, **PCI DSS**). 
![AWS-Artifact](/images/uploads/aws-artifact-security.png)

While customers don’t manage hardware, monitoring tools like AWS ```CloudTrail``` and ```AWS Config``` help detect anomalies in workloads.

```mermaid
graph TD
    subgraph Synchronous Vendor Request Flow (3 Hops)
        V[Vendor] -- 1. Request Claim Data (GET /claims/{id}) --> AGG[Aggregator API]
        AGG -- 2. Orchestrates & Shapes (GET /canonical/claims/{id}) --> CAG[Canonical Gateway API]
        CAG -- 3. Transforms & Invokes (GET /claims/{id}, /policy) --> GW[Guidewire Insurance Suite (Monolith)]
        GW -- 4. Returns Raw Data --> CAG
        CAG -- 5. Returns Transformed Data --> AGG
        AGG -- 6. Returns Shaped Data --> V
    end

    subgraph Asynchronous Data Flow (for AI/ML)
        CAG -- A. Publishes Canonical Model (Non-blocking) --> MQ[(Message Queue)]
        MQ -- B. Consumes Canonical Events --> CDS[Canonical DB Sync Service]
        CDS -- C. Persists Canonical Model --> CDB[Canonical Database (Read-Optimized)]
        CDB -- D. Reads Canonical Data for Training/Inference --> AIML[AI/ML Services]
    end

    style V fill:#f9f,stroke:#333,stroke-width:2px
    style AGG fill:#bbf,stroke:#333,stroke-width:2px
    style CAG fill:#bfb,stroke:#333,stroke-width:2px
    style GW fill:#fbb,stroke:#333,stroke-width:2px
    style MQ fill:#fcb,stroke:#333,stroke-width:2px
    style CDS fill:#ccf,stroke:#333,stroke-width:2px
    style CDB fill:#ffc,stroke:#333,stroke-width:2px
    style AIML fill:#fdb,stroke:#333,stroke-width:2px

    linkStyle 0 stroke:#0066cc,stroke-width:2px,fill:none;
    linkStyle 1 stroke:#0066cc,stroke-width:2px,fill:none;
    linkStyle 2 stroke:#0066cc,stroke-width:2px,fill:none;
    linkStyle 3 stroke:#0066cc,stroke-width:2px,fill:none;
    linkStyle 4 stroke:#0066cc,stroke-width:2px,fill:none;
    linkStyle 5 stroke:#0066cc,stroke-width:2px,fill:none;

    linkStyle 6 stroke:#663399,stroke-width:2px,fill:none,stroke-dasharray: 5 5;
    linkStyle 7 stroke:#663399,stroke-width:2px,fill:none,stroke-dasharray: 5 5;
    linkStyle 8 stroke:#663399,stroke-width:2px,fill:none,stroke-dasharray: 5 5;
    linkStyle 9 stroke:#663399,stroke-width:2px,fill:none,stroke-dasharray: 5 5;  
```

## 📖Further Read

- [How to use AWS KMS RSA keys for offline encryption](https://aws.amazon.com/blogs/security/how-to-use-aws-kms-rsa-keys-for-offline-encryption/)
- [How to verify AWS KMS signatures in decoupled architectures at scale](https://aws.amazon.com/blogs/security/how-to-verify-aws-kms-signatures-in-decoupled-architectures-at-scale/)
- [OSI Model Security with AWS Services](https://newmathdata.com/blog/osi-model-aws-security-defense-in-depth-guide/)
- [AWS WAF to block DDOS Events](https://aws.amazon.com/blogs/architecture/how-scale-to-win-uses-aws-waf-to-block-ddos-events/)
- [Security Automations for AWS WAF](https://docs.aws.amazon.com/solutions/latest/security-automations-for-aws-waf/solution-overview.html)