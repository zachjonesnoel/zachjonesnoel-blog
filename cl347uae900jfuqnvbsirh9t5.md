---
title: "When to : SNS or SQS"
datePublished: Sun Nov 21 2021 17:51:23 GMT+0000 (Coordinated Universal Time)
cuid: cl347uae900jfuqnvbsirh9t5
slug: when-to-sns-or-sqs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432825097/TiZt8ZznG.jpeg

---

[Simple Notification Service](https://dev.to/aws-builders/getting-started-with-sns-and-sqs-3m4i#sns) and [Simple Queue Service](https://dev.to/aws-builders/getting-started-with-sns-and-sqs-3m4i#sqs) are the managed services which are offered by AWS which helps in architecting your application with the concept of *decoupling* and promotes adoption of *event-driven architectures*.
The getting started post, explains the technical concepts behind both SNS and SQS.
{%post aws-builders/getting-started-with-sns-and-sqs-3m4i%}

This blog post will explain what to choose and how to choose between *SNS* or *SQS* for your architecture. 

### Key things to keep-in-mind when choosing SNS
[Simple Notification Service (SNS)](https://aws.amazon.com/sns/) is a **distributed pub-sub model** where the messages are pushed to the subscriber. SNS provides the **feasibility of having multiple subscribers** (Standard topics : max 12,500,000 subscriptions and FIFO topics : max 100 subscriptions) and all the subscribers of the topic receive the message, and the multiple subscribers can be across various other types - **email message, Lambda fn invocation, posting to a SQS queue, invoke an HTTP end-point, sending a SMS message, a mobile push notification** with all the possible combinations. With SNS, we can integration with [Amazon Device Messaging](https://developer.amazon.com/docs/adm/overview.html), [Apple Push Notification Service](https://developer.apple.com/notifications/), [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) and other popular providers to send push notifications to the mobile app. There can be scenarios where you can decide to send a SMS message to the subscriber or send an email message to the subscriber based on the message itself with the feature of **message filtering with attributes**. 
The down-side of SNS is the message once sent to the subscriber, it is not available but if undelivered, based on the retry configuration message is available until it is successfully delivered. That been said, if a topic exists with 0 subscribers, if the message is pushed to the topic, that message is lost.

### Key things to keep-in-mind when choosing SQS
[Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) is a **distributed queue model** where the consumer should poll the queue to check if a new message has been pushed into the queue. SQS messages are stored and **available upto 14 days**. The SQS can be configured with different fan-out periods - **Visibility timeout (0s - 12 hours), Message retention period(1 min - 14 days, Delivery delay(0s - 15 mins), Maximum message size (1KB - 256KB), Receive message wait time (0s - 20s)**. A **Dead-letter Queue (DLQ)** can also be configured for capturing undelivered messages. SQS supports good batch processing of upto 10 messages per batch.
DLQ can get over-filled if there is some error in delivering huge amount of messages. As SQS uses the polling mechanism, there is a **latency** with message delivery. SQS **does not delivery to multiple consumers at the same time** even if there are multiple Lambda fns or even SNS topics as triggers for the queue but if anyone of the consumer deletes the message when polled, it is **lost for all the consumers**.

---

|Use-case|SNS|SQS|
|---|:---:|:---:|
|On a certain action sending email / mobile push / SMS message | ✅ | 🚫 |
|Notify multiple systems via HTTP post | ✅ | 🚫 |
|Immediate message processing | ✅ | 🚫 |
|Process an image and get the meta-data via other AWS services | ✅ Topic can send a message to multiple other subscribers which be SQS queues  | ✅ These SQS queues can process each type of meta-data generation |
|Single subscriber processing each message | ✅ | 🚫 |
|Batch processing | 🚫 | ✅ |
|Cross account Lambda triggers for queue | 🚫 | ✅ |
|Decoupling your Lambda fn | ✅ Multiple sub-modules of the Lambda fn can be individual subscribers to the topic| ✅ Load control your Lambda fn with various fan-out options|
|Handling undelivered messages | 🚫 | ✅ |

### Know your limits
Whenever taking a choice of service for your event-driven architecture which is also loosely coupled, understand the limits of the service itself. 
SNS Limits : [Service end-points](https://docs.aws.amazon.com/general/latest/gr/sns.html#sns_region) [Service quotas](https://docs.aws.amazon.com/general/latest/gr/sns.html#limits_sns) 
SQS Limits : [Service end-points](https://docs.aws.amazon.com/general/latest/gr/sqs-service.html#sqs_region) [Service quotas](https://docs.aws.amazon.com/general/latest/gr/sqs-service.html#limits_sqs) 

### Conclusion
SNS and SQS have their own advantages and limitations making it an evaluation-based choice for the Cloud Architects/Developers to choose either SNS or SQS or the combination of two. And to understand the working of the combination of SNS and SQS, Jeff Barr's post [**SQS Queues and SNS Notifications – Now Best Friends**](https://aws.amazon.com/blogs/aws/queues-and-notifications-now-best-friends/) (also the 4th use-case listed above) explains how both the services together makes a best pair for decoupling.

To know more about SNS and SQS offering Standard and FIFO variants of topics and queues respectively - 
{%post https://dev.to/aws-builders/standard-vs-fifo-sns-sqs-38ld %}