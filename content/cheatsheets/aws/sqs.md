---
title: SQS
linktitle: 📨 SQS
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 110
---

Managed Message Queues

<!--more-->

## Overview

Amazon SQS is a web service that gives you access to a message queue that can be used to store messages while waiting for a computer to process them.
SQS is a distributed queue system that enables webservice applications to quickly and reliably queue messages that one component in the application generates to be consumed by another component.

## SQS Key Facts

* SQS is **pull** based
* Messages can be at most **256** KB in size
* Messages can be kept in the queue from 1 min to upto 14 days
* Default retention period is 4 days
* SQS guarantees that your messages would be processed at least once.
* Visibility Timeout -
  * The amount of time that the message is invisible in the SQS queue after a reader picks up that message. If the consumer processed the message before the **Visibility Timeout** expires, the message will be deleted from the queue. If the consumer is not finished by the Visibility timeout, then the message will become visible again and could be picked up by another reader. This could result in the same message being delivered twice.
  *   Default - 30 seconds
  *   Max - 12 hours

* Standard Queues - (Default) - best effort ordering; message delivered at least once. A standard queue let's you have nearly unlimited transactions per second.
* FIFO Queues - Ordering strictly preserved, message delivered once, no duplicates. FIFO queues are limited to 300 transactions per second.
* Short Polling - returns immediately even if there are no messages.
* Long Polling - polls the queue periodically and only returns a response when there's a message in the queue or the timeout expires.

## Security

* Encryption:

  * In-flight encryption using HTTPS API
  * At-rest encryption using KMS keys
  * Client-side encryption if the client wants to perform encryption/decryption itself

* Access Controls: IAM policies to regulate access to the SQS API

* SQS Resource Policies (similar to S3 bucket policies)

  * Useful for cross-account access to SQS queues
  * Useful for allowing other services (SNS, S3…) to write to an SQS queue

## Visibility Timeout

* After a message is polled by a consumer, it becomes invisible to other consumers
* By default, the “message visibility timeout” is 30 seconds
* That means the message has 30 seconds to be processed
* After the message visibility timeout is over, the message is again “visible” in SQS
* A consumer could call the **ChangeMessageVisibility** API to get more time
* If visibility timeout is high (hours), and consumer crashes, re-processing will take time
* If visibility timeout is too low (seconds), we may get duplicates

![SQS – Message Visibility Timeout](/images/uploads/sqs-visibility-timeout.PNG)

## Dead Letter Queue

<img align="right" width="200" height="200" src="/images/uploads/sqs-dlq.PNG">

* If a consumer fails to process a message within the Visibility Timeout…the message goes back to the queue!
* We can set a threshold of how many times a message can go back to the queue
* After the MaximumReceives threshold is exceeded, the message goes into a dead letter queue (DLQ)
* Useful for debugging!
* Make sure to process the messages in the DLQ before they expire.
* Good to set a retention of 14 days in the DLQ

## Delay Queue

* Delay a message (consumers don’t see it immediately) up to 15 minutes
* Default is 0 seconds (message is available right away)
* Can set a default at queue level
* Can override the default on send using the DelaySeconds parameter

## Long Polling

<img align="right" width="150" height="150" src="/images/uploads/sqs-long-polling.PNG">

* When a consumer requests messages from the queue, it can optionally “wait” for messages to arrive if there are none in the queue
* This is called Long Polling
* LongPolling decreases the number of API calls made to SQS while increasing the efficiency and latency of your application.
* The wait time can be between 1 sec to 20 sec (20 sec preferable)
* Long Polling is preferable to Short Polling
* Long polling can be enabled at the queue level or at the API level using **WaitTimeSeconds**

## Extended Client

* Message size limit is 256 KB, how to send large messages, e.g. 1GB?
* Using the SQS Extended Client (Java Library)
![SQS Extended Client](/images/uploads/sqs-extended-client.PNG)

## API

* CreateQueue (MessageRetentionPeriod),
* DeleteQueue
* PurgeQueue: delete all the messages in queue
* SendMessage (DelaySeconds),
* ReceiveMessage,
* DeleteMessage,
* ReceiveMessageWaitTimeSeconds: Long Polling
* ChangeMessageVisibility: change the message timeout
* Batch APIs for SendMessage, DeleteMessage, ChangeMessageVisibility helps decrease your costs

## FIFO Queue

* FIFO = First In First Out (ordering of messages in the queue)
* Limited throughput: 300 msg/s without batching, 3000 msg/s with
* Exactly-once send capability (by removing duplicates)
* Messages are processed in order by the consumer

  ### FIFO – Deduplication

  * De-duplication interval is 5 minutes
  * Two de-duplication methods:

    * Content-based deduplication: will do a SHA-256 hash of the message body
    * Explicitly provide a Message Deduplication ID

  ### FIFO – Message Grouping

  * If you specify the same value of MessageGroupID in an SQS FIFO queue, you can only have one consumer, and all the messages are in order
  * To get ordering at the level of a subset of messages, specify different values for MessageGroupID
  * Messages that share a common Message Group ID will be in order within the group
  * Each Group ID can have a different consumer (parallel processing!)
  * Ordering across groups is not guaranteed

  ![SQS FIFO – Message Grouping](/images/uploads/sqs-fifo-message-grouping.PNG)
