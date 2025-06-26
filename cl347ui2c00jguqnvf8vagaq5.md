---
title: "Getting started with SNS and SQS"
datePublished: Sun Nov 14 2021 18:58:52 GMT+0000 (Coordinated Universal Time)
cuid: cl347ui2c00jguqnvf8vagaq5
slug: getting-started-with-sns-and-sqs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432834629/DBD98BCK1.jpeg

---

[Amazon Simple Notification Service (SNS)](#sns) and [Amazon Simple Queue Service (SQS)](#sqs) are the famous services which is leveraged to architect applications which are event driven and decoupled. 

#### Event-driven architecture
**Event-driven architecture (EDA)** is one where the components of the architecture are triggered whenever an event has occurred. My first post on Dev.To explains more about EDA.
{%post zachjonesnoel/what-why-when-event-driven-architecture-4fpa %}

#### Decoupled architecture
Whenever a system is architected, it is designed with the key concept of **decoupling** where the components of the architecture are not dependent on each other rather it functions as a stand-alone component and communicates whenever required. This exponential removes high dependence between components. And if something fails, it does not disturb the rest of the system. This is also the idea behind **event-driven architecture** which is extensively adapted in the serverless world.

#### FIFO
**FIFO** which is abbreviated as **First In First Out** which preservers the order of messages such that the first message sent is the first message which is delivered. 

### What is SNS? <a name="sns"></a>
[Amazon Simple Notification Service (SNS)](https://aws.amazon.com/sns/) as the name goes it is a notification service which works on the *publisher-subscriber (Pub/Sub)* messaging model. This is a fully managed messaging service for application integrations. 
![Pub/Sub](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432831776/iSiL_q6unJ.png)
#### The concepts of Pub/Sub - 
+ **Topic** : A common medium/channel for the message delivery between a *publisher* and *subscriber*.
+ **Message** : The serialized message which the publisher sends to the *topic* without the information as to who are the *subscribers* and what is the action done after receiving the message.
+ **Publisher** : The component which *sends/publishes* the *message* to a specific *topic*.
+ **Subscriber** : The component which has *subscribed/registered* to listen for any incoming message from a specific *topic*.

SNS facilitates with *publishers* from external or triggered from AWS Services such as [EventBridge](https://aws.amazon.com/eventbridge/) or triggers from [S3](https://aws.amazon.com/s3/) object events, [Lambda fns](https://aws.amazon.com/lambda/) and many more or it could be involved programmatically with `sns:publish` which publish to a designated *topic* and all the *subscribers* to that specific *topic* receives the message. 

#### Key features of SNS
+ **Pub/Sub model** makes it really easy to integrate with an existing application. 
+ The types of *topics* offered by SNS are - **Standard Topics** and **FIFO Topics**.
+ **Multiple publisher** supported with AWS services natively such as [S3](https://aws.amazon.com/s3/), [EventBridge](https://aws.amazon.com/eventbridge/), [SQS](https://aws.amazon.com/sqs/)and also for application integration and programmatic integration for publishing message to a topic.
+ **Multiple subscribers** are supported for a topic which could be [Lambda fns](https://aws.amazon.com/lambda/), [SQS](https://aws.amazon.com/sqs/), [Kinesis](https://aws.amazon.com/kinesis/), HTTP endpoint. The subscribers are not only limited to applications but also *email notifications*, *push notifications* on the mobile apps and *SMS messages*. 
+ [**Attribute based filtering**](https://aws.amazon.com/blogs/compute/simplify-pubsub-messaging-with-amazon-sns-message-filtering/) is supported on a topic where you can filter the message based on the message's attribute where the subscribers uses a *subscription filter policy* based on which the message is delivered to the satisfying subscriber. 
+ **Message encryption** is considered an important aspect for any messaging service, this is achieved with [AWS Key Management System](https://aws.amazon.com/kms/) which uses the *customer master key (CMK)* and *256-bit AES-GCM algorithm* to encrypt the message in rest and the message is decrypted when the subscribers receive the message.

#### Pricing
SNS pricing is categorized with *type of topic* and the *sms* cost. 
SMS messages are modeled as *pay-as-you-go* for both *transactional* and *promotional* messages. Due to geographic restrictions and also international SMS cost would apply. 
Based on topic cost is - 
*Standard topic* has first 1 million requests per month free and $0.50 per 1 million requests thereafter
*FIFO topic* has `Publish` API requests $0.30 per 1 million and $0.017 per GB of payload data. `Subscription` messages are $0.01 per 1 million and $0.001 per GB of payload data.
More details available in [SNS Pricing](https://aws.amazon.com/sns/pricing/).

### What is SQS? <a name="sqs"></a>
[Amazon Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) is the message queuing service. This fully-managed service helps applications which leverages *serverless architectures*, *micro-services architectures* and *distributed services* to design and architect a perfect decoupled application. 
![SQS](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432833370/X-KbW4BZB.png)
#### Key features of SNS
+ Types of *queues* offered by SQS - **Standard Queue** and **FIFO Queue**. 
+ **Dead Letter Queue (DLQ)** supported for the messages which are not processed. 
+ **Message encryption** with [AWS Key Management System](https://aws.amazon.com/kms/) is supported where the message is encrypted as soon as it is pushed into the queue and encrypted until in the queue whereas it is decrypted when sent to the consumer.
+ **Long polling** helps reducing the cost of explicit polling. The long polling request can wait upto 20 seconds until the next message is received when the queue is empty.
+ Recently, AWS announced SQS Queues can be trigger Lambda functions which belong to another AWS Account.
{%post aws-builders/sqs-queue-with-across-account-lambda-triggers-4job%}

#### Pricing
SQS has the free tier with 1 million SQS requests free and after which the cost is categorized with the *type of queue per million requests*.
More details available in [SQS Pricing](https://aws.amazon.com/sqs/pricing/)



