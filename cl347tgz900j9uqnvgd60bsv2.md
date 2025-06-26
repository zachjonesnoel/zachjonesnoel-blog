---
title: "Standard v/s FIFO : SNS & SQS"
datePublished: Sun Dec 19 2021 15:59:13 GMT+0000 (Coordinated Universal Time)
cuid: cl347tgz900j9uqnvgd60bsv2
slug: standard-vs-fifo-sns-and-sqs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432786799/8hqdVzCSX.jpeg

---

[Amazon SNS](https://aws.amazon.com/sns/) and [Amazon SQS](https://aws.amazon.com/sqs/) both offer Standard and FIFO variants of topics and queues respectively which helps you build better micro-service focused, de-coupled applications. 
Some of the basic understanding of SNS and SQS with the technology and concepts behind is explained in - 
{%post aws-builders/getting-started-with-sns-and-sqs-3m4i %}
The previous blog post explains about when and how to choose SNS or SQS for your application need.
{%post aws-builders/when-to-sns-or-sqs-2aji %}
And this blog post, will give you a better understanding of Standard and FIFO variants of topics and queues.

#### What is Standard topic/queue?
Standard topics/queues are used often when the system can handle messages which can arrive more than once and without a proper order. 

![Standard topic/queue](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432779051/WlMJCMFw3.png)
 
#### What is FIFO topic/queue?
FIFO topics/queues follows the concept of *First-in,First-out* which guarantees the order of messages is very necessary. This strictly maintains the order of the message and also doesn't deliver the same message multiple times. 

![FIFO topic/queue](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432780407/N4tqzAmCF.png)

#### Standard v/s FIFO : SNS
[Amazon SNS](https://aws.amazon.com/sns/) offers *topics* of two variants - **Standard topics** and **FIFO topics**. 
![Standard v/s FIFO : SNS](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432781798/E53n0UOQU.png)
 
Feature| Standard topics | FIFO topics
--- | --- | ---
Throughput | Nearly unlimited no. of messages per second. | 300 messages per second or 10 MB per second (whichever is first).
Message order | Order of messages will not be same as published order. | Order is maintained by following *first-in-first-out (FIFO)*.
Message delivery | At-least once, can duplicate again. | Only once.
Subscription types | Amazon SQS, Amazon Kinesis Data Firehose, AWS Lambda Fns, HTTPS, SMS, Email, mobile push. | Only SQS FIFO queues.
Limits | 100,000 standard topics with 12.5M subscriptions per topic per account. | 1,000 FIFO topics with 100 subscriptions per topic per account.
Encryption (via [AWS KMS](https://aws.amazon.com/kms/)) | Both at rest and in transit. | Both at rest and in transit.
Pricing (more info in [SNS pricing](https://aws.amazon.com/sns/pricing/)) | First 1M requests free, $0.5 per 1M requests. | Publish and publish batch API requests are $0.30 per 1M and $0.017 per GB of payload data and subscription messages are $0.01 per 1M and $0.001 per GB of payload data.
Use-case | Whenever there is multi-subscriber scenario, Whenever the application can handle duplicate and unordered messages. | Whenever the need for ordering is a must.

#### Standard v/s FIFO : SQS
[Amazon SQS](https://aws.amazon.com/sqs/) offers *queues* of two variants - **Standard queues** and **FIFO queues**. 
![Standard v/s FIFO : SQS](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432783265/WSQzRyHDn.png)
 
Feature| Standard queues | FIFO queues
--- | --- | ---
Throughput | Nearly unlimited number of transactions per second per API action | 300 message per second with a batch of 10 messages per operation. 
Message order | Occasionally order of messages will not be same. | Order is maintained by following *first-in-first-out (FIFO)*.
Message delivery | At-least once but can get a copy of the same message | Only once.
Message persistence | Max upto 14 days. | Max upto 14 days.
Encryption (via [AWS KMS](https://aws.amazon.com/kms/)) | Both at rest and in transit. | Both at rest and in transit.
Resolvers / Triggers | Lambda Fns and SNS topics. | Lambda Fns and SNS topics.
Pricing (more info in [SNS pricing](https://aws.amazon.com/sqs/pricing/)) | First 1M requests free, $0.4 for 1M-100B requests, $0.24 for over 200B requests. | First 1M requests free, $0.5 for 1M-100B requests, $0.35 for over 200B requests.
Use-case | Whenever the application can handle unordered and multiple delivery of messages such as decoupling transaction ordering process, batch processes. | Whenever the message have to be ordered such as image/video decoding/encoding, using as subscriber to a FIFO topic.

#### Conclusion
Whenever choosing SNS or SQS choosing between Standard variant and FIFO variant is also necessary as the features of *message ordering*, *exactly once delivery* would help building efficient systems. 
![Variants](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432785260/oqiHUNvoH.png)
 