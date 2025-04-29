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

## Encryption

Encryption is a critical component of a defense-in-depth strategy, which is a security approach adopted by AWS with a series of defensive mechanisms designed so that if one security mechanism fails, there’s at least one more still operating.

There are 2 forms of encryption in practice:

### Encryption in transit:

* Data is encrypted before sending and decrypted after receiving
* SSL certificates help with encryption (HTTPS)
* Encryption in flight ensures no MITM (man in the middle attack) can happen

![HTTPS](/images/uploads/encryption-in-transit.PNG)

### Encryption at Rest

* Data is encrypted after being received by the server
* Data is decrypted before being sent
* It is stored in an encrypted form thanks to a key (usually a data key)
* The encryption / decryption keys must be managed somewhere and the server must have access to it.
*  There are two main methods to encrypt data at rest:

   * **Client-Side** Encryption: As the name implies this method encrypts your data at the client-side before it reaches backend servers or services. You have to supply encryption keys 🔑 to encrypt the data from the client-side. You can either manage these encryption keys by yourself or use AWS KMS(Key Management Service) to manage the encryption keys under your control.

     AWS provides multiple client-side SDKs to make this process easy for you. E.g. AWS Encryption SDK, S3 Encryption Client, DynamoDB Encryption Client etc…

     ![Client-Side](/images/uploads/encryption-at-rest-client.PNG)

   * **Server-Side** Encryption: In Server-Side encryption, AWS encrypts the data on your behalf as soon as it is received by an AWS Service. Most of the AWS services support server-side encryption. E.g. S3, EBS, RDS, DynamoDB, Kinesis, etc…

     All these services are integrated with AWS KMS in order to encrypt the data.

     ![Server-Side](/images/uploads/encryption-at-rest-server.PNG)

## KMS

AWS Key Management Store (KMS) is a managed service that enables you to easily encrypt your data.
AWS KMS provides a highly available key storage, management, and auditing solution for you to encrypt data within your own applications and control the encryption of stored data across AWS services.

AWS KMS (Key Management Service) is the service that manages encryption keys on AWS. These encryption keys are called “Customer Master Keys” or CMKs for short.

KMS is used to fully manage the keys & their policies:
* Create
* Rotation policies
* Disable
* Enable
* Able to audit key usage (using CloudTrail)
* Pay for API call to KMS ($0.03 / 10000 calls)
* Three types of Customer Master Keys (CMK):

  * AWS Managed Service Default CMK: free
  * User Keys created in KMS: $1 / month
  * User Keys imported (must be 256-bit symmetric key): $1 / month

### Customer Master Key (CMK) Types

* Symmetric (AES-256 keys)

  * First offering of KMS, single encryption key that is used to Encrypt and Decrypt
  * AWS services that are integrated with KMS use Symmetric CMKs
  * Necessary for envelope encryption
  * You never get access to the Key unencrypted (must call KMS API to use)
  * KMS can only help in encrypting up to 4KB of data per call. If data > 4 KB, then use Data Keys.
  * To give access to KMS to someone:

    * Make sure the Key Policy allows the user
    * Make sure the IAM Policy allows the API calls

* Asymmetric (RSA & ECC key pairs)

  * Public (Encrypt) and Private Key (Decrypt) pair
  * Used for Encrypt/Decrypt, or Sign/Verify operations
  * The public key is downloadable, but you access the Private Key unencrypted
  * Use case: encryption outside of AWS by users who can’t call the KMS API

### How does KMS work?
API – Encrypt and Decrypt

![KMS-API](/images/uploads/kms-encrypt-decrypt.PNG)

### Envelope Encryption

* KMS Encrypt API call has a limit of 4 KB
* If you want to encrypt >4 KB, we need to use Envelope Encryption using Data Keys
* Data Keys are generated from CMKs. There is a direct relationship between Data Key and a CMK. However, AWS does NOT store or manage Data Keys. Instead, you have to manage them.
* The main API that will help us is the GenerateDataKey API

You can use one Customer Master Key (CMK) to generate thousands of unique data keys. You can generate data keys from a CMK using two methods.
  * Generate both Plaintext Data Key and Encrypted Data Key (GenerateDataKey)
  * Generate only the Encrypted Data Key (GenerateDataKeyWithoutPlaintext)

Once you get the Plaintext data key and Encrypted data key from CMK, use the Plaintext data key to encrypt your data. After encryption, never keep the Plaintext data key together with encrypted data(Ciphertext) since anyone can decrypt the Ciphertext using the Plaintext key. So remove the Plaintext data key from the memory as soon as possible. You can keep the Encrypted data key with the Ciphertext.  

The method of encrypting the key using another key is called **Envelope** Encryption. By encrypting the key, that is used to encrypt data, you will protect both data and the key.

![KMS-Envelope-Encrypt](/images/uploads/kms-envelope-encrypt.PNG)

When you want to decrypt it, call the KMS API with the encrypted data key and KMS will send you the Plaintext key if you are authorized to receive it. Afterward, you can decrypt the Ciphertext using the Plaintext key.

![KMS-Envelope-Decrypt](/images/uploads/kms-envelope-decrypt.PNG)

### Encryption SDK

* The AWS Encryption SDK implemented Envelope Encryption for us
* The Encryption SDK also exists as a CLI tool we can install
* Implementations for Java, Python, C, JavaScript
* Feature - Data Key Caching:

  * re-use data keys instead of creating new ones for each encryption
  * Helps with reducing the number of calls to KMS with a security trade-off
  * Use LocalCryptoMaterialsCache (max age, max bytes, max number of messages)

### KMS Key Policies

* One of the powerful features in KMS is the ability to define permission separately for those who use the keys and administrate the keys. This is achieved using Key Policies. You can control access to KMS keys, “similar” to S3 bucket policies.

* Default KMS Key Policy:
  * Created if you don’t provide a specific KMS Key Policy
  * Complete access to the key to the root user = entire AWS account
  * Gives access to the IAM policies to the KMS key
* Custom KMS Key Policy:
  * Define users, roles that can access the KMS key
  * Define who can administer the key
  * Useful for cross-account access of your KMS key

* The below **Key Policy** (Default) is applied to the root user of the account. It allows full access to the CMK for any user in the account.

```JSON
{
  "Sid": "Enable IAM User Permissions",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
  "Action": "kms:*",
  "Resource": "*"
}
```

* If you choose to distinguish users and roles who can manage key usage and key administration then we could achieve as follows:

  * The below **IAM policy** is applied to the IAM user ‘KeyUser’. Now he has permission to use the CMK for encryption and decryption. However, he is not allowed to administrate that CMK.

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
  * The below **IAM policy** allows administrators ('KeyManager') to administrate the CMK that it is applied to. However, the administrator cannot use the key to Encrypt or Decrypt data.

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

### KMS Request Quotas

* When you exceed a request quota, you get a ThrottlingException:

  * To respond, use exponential backoff (backoff and retry)
  * For cryptographic operations, they share a quota. This includes requests made by AWS on your behalf (ex: SSE-KMS)
  * For GenerateDataKey, consider using DEK caching from the Encryption SDK
  * You can request a Request Quotas increase through API or AWS support

| API operation                                                                                                                                                               | Request quotas (per second)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Decrypt<br>Encrypt<br>GenerateDataKey (symmetric)<br>GenerateDataKeyWithoutPlaintext (symmetric)<br>GenerateRandom<br>ReEncrypt<br>Sign (asymmetric)<br>Verify (asymmetric) | These shared quotas vary with the AWS Region and the type of CMK used in the request. Each quota is calculated separately.<br>Symmetric CMK quota:<br>* 5,500 (shared)<br>* 10,000 (shared) in the following Regions:<br>* us-east-2, ap-southeast-1, ap-southeast-2,<br>ap-northeast-1, eu-central-1, eu-west-2<br>* 30,000 (shared) in the following Regions:<br>* us-east-1, us-west-2, eu-west-1<br>Asymmetric CMK quota:<br>* 500 (shared) for RSA CMKs<br>* 300 (shared) for Elliptic curve (ECC) CMKs |

### S3 Encryption for Objects

* There are 4 methods of encrypting objects in S3 at rest

  * SSE-S3: encrypts S3 objects using keys handled & managed by AWS
  * SSE-KMS: leverage AWS Key Management Service to manage encryption keys
  * SSE-C: when you want to manage your own encryption keys
  * Client Side Encryption

  #### SSE-KMS

  * SSE-KMS: encryption using keys handled & managed by KMS
  * KMS Advantages: user control + audit trail via CloudTrail
  * Object is encrypted server side
  * Must set header: “x-amz-server-side-encryption": ”aws:kms"
  * SSE-KMS leverages the GenerateDataKey & Decrypt KMS API calls
  * These KMS API calls will show up in CloudTrail, helpful for logging
  * To perform SSE-KMS, you need:

    * A KMS Key Policy that authorizes the user / role
    * An IAM policy that authorizes access to KMS, otherwise you will get an access denied error
  * S3 calls to KMS for SSE-KMS count against your KMS limits
  * If throttling, try exponential backoff or you can request an increase in KMS limits
  * The service throttling is KMS, not Amazon S3
  * Security Policy for enforcing encryption via SSE-KMS:

    Here we need to deny all requests that use the wrong encryption type, i.e. x-amz-server-side-encryption = AWS256 or an incorrect KMS key, i.e. the value of x-amz-server-side-encryption-aws-kms-key-id. A rule for that looks like this (You need to replace $BucketName, $Region, $Accountid and $KeyId):  

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

## SSM ParameterStore

<img align="right" width="250" height="250" src="/images/uploads/ssm-parameter-store.PNG">

* Secure storage for configuration and secrets
* Optional Seamless Encryption using KMS
* Serverless, scalable, durable, easy SDK
* Version tracking of configurations / secrets
* Configuration management using path & IAM
* Notifications with CloudWatch Events
* Integration with CloudFormation
* SSM Parameter Store Hierarchy

  * /my-department/
    * my-app/
    <img align="right" width="300" height="300" src="/images/uploads/ssm-parameter-store-api.PNG">
      * dev/
        * db-url
        * db-password
      * prod/
        * db-url
        * db-password
    * other-app/
  * /other-department/
  * /aws/reference/secretsmanager/secret_ID_in_Secrets_Manager
  * /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

## Secrets Manager

* Newer service, meant for storing secrets
* Capability to force rotation of secrets every X days
* Automate generation of secrets on rotation (uses Lambda)
* Integration with Amazon RDS (MySQL, PostgreSQL, Aurora)
* Secrets are encrypted using KMS
* Mostly meant for RDS integration

## Secrets Manager vs. SSM ParameterStore

* **Secrets Manager ($$$)**

  * Automatic rotation of secrets with AWS Lambda
  * Integration with RDS, Redshift, DocumentDB
  * KMS encryption is mandatory
  * Can integration with CloudFormation

* **SSM ParameterStore ($)**

  * Simple API
  * No secret rotation
  * KMS encryption is optional
  * Can integration with CloudFormation
  * Can pull a Secrets Manager secret using the SSM Parameter Store API
