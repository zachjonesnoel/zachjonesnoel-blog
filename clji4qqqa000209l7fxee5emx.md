---
title: "Buses and queues: Head-on"
seoTitle: "Buses and queues: Head-on"
seoDescription: "Learn about Amazon EventBridge features and Amazon SQS features which sets them apart from each other and also comparison between the service."
datePublished: Fri Jun 30 2023 05:24:27 GMT+0000 (Coordinated Universal Time)
cuid: clji4qqqa000209l7fxee5emx
slug: buses-and-queues-head-on
canonical: https://blog.theserverlessterminal.com/buses-and-queues-head-on
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688102530708/953af92f-8fce-4d99-9ab7-0593b9ecb7cb.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1688102622284/33d27793-2d1b-4784-ba49-797c9866b258.png
tags: aws, serverless

---

Originally published on [The Serverless Terminal blog](https://blog.theserverlessterminal.com/buses-and-queues-head-on).

[Amazon EventBridge](https://aws.amazon.com/eventbridge/) is a serverless managed service for event-driven applications to build loosely coupled applications and route events smartly across different services.  
You can read more about how [Amazon EventBridge is the missing piece to your app](https://blog.theserverlessterminal.com/amazon-eventbridge-the-missing-piece-to-your-app).

Also, if you are wondering about [Amazon SQS](https://aws.amazon.com/sqs/), learn about [getting started with SQS and SNS](https://blog.theserverlessterminal.com/getting-started-with-sns-and-sqs) and [understand Standard and FIFO queues (in SQS) and topics (in SNS)](https://blog.theserverlessterminal.com/standard-vs-fifo-sns-and-sqs) with the understanding of [when to use SQS and SNS](https://blog.theserverlessterminal.com/when-to-sns-or-sqs) in your Serverless architectures.

In this blog, we will look into Amazon EventBridge buses and Amazon SQS queues and how and where they fit right into your Serverless architectures.

## Catch the bus!

![Take a bus meme](https://cdn.hashnode.com/res/hashnode/image/upload/v1688051996038/d156b0f4-50d9-416c-9858-438630d55d2b.gif align="center")

### Building event-driven architectures

When building EDA applications, Amazon EventBridge is best suited, as EventBridge provides you with different capabilities for service-to-service integrations across various sources in the Serverless space. Also ensuring the architecture is loosely coupled.

### Event routing and filtering

EventBridge is known for its smart routing feature with event rules and also the support of filtering based on different filters and filter patterns. When building for service-to-service integrations, the routing helps with designated events routed to destined destinations.

### Integrations

EventBridge supports different service integrations such as AWS Lambda, AWS Step Functions, SQS, SNS, and many more. While these are natively supported service integrations, EventBridge also supports SaaS Partner integrations and third-party HTTP end-points with API destinations.

## Get in the queue!

![In a queue](https://cdn.hashnode.com/res/hashnode/image/upload/v1688053575975/63ed572b-367a-4da7-aac6-3c591eacf23a.gif align="center")

### Building message queues

When building applications, you may have to build messaging queues that need reliable and ordered delivery. SQS ensures this with FIFO queues and also enables load balancing with horizontal scaling.

### Fan out patterns

Messages which have to be distributed across multiple consumers would efficiently process each message by each consumer. Fan-out patterns are best suited with Amazon SQS as it enables a balancing queue for the consumer.

### Time-bound delivery

Amazon SQS has capabilities where messages could be configured to deliver the message to the consumer using the `DelaySeconds` parameter. Also, the messages can hold a time-to-live (TTL) attribute beyond which the message is removed from the queue. SQS supports visibility timeouts where the message is invisible and can be reversed for consumption later.

## EventBridge v/s Queue

| Feature | Amazon EventBridge | Amazon SQS |
| --- | --- | --- |
| Complex processing | ✅ EventBridge supports transformation and filtering which makes it easy to process complex events. | 🚫 SQS can have targets as Lambda to process but cannot process them on its own. |
| Multiple destinations | ✅ EventBridge supports multiple AWS services as the destination and also API destinations for external HTTP endpoints | 🚫 Would need a Lambda function or EventBridge pipe to consume the message and programmatically route it to the destination. |
| Scheduled messages/events | ✅ EventBridge scheduler helps to build event scheduling with CRON patterns. | 🚫 SQS cannot trigger or push messages at scheduled times. |
| Ordering | 🚫 EventBridge doesn't ensure strict ordering. | ✅ FIFO queues could be used to ensure FIFO ordering. |
| Delay messages | 🚫 EventBridge immediately delivers the event as it occurs. | ✅ SQS supports delays in messages with `DelaySeconds` |
| Message retention | ✅ Only when archival is enabled, messages are retained for future use. | ✅ SQS Supports TTL and DLQs for handling failed messages. |
| Throughput | 10,000,000 events per second in a bus. | 10,000 messages per second in a queue. |
| Payload limits | 256 KB max supported. | 256 KB max supported. |

## Co-existence of EventBridge and SQS

While individually EventBridge and SQS can add a lot of value to the Serverless architecture, the combination of the two in certain patterns is possible.

**Messaging queues with event-driven patterns** where both SQS and EventBridge can be used in combination. EventBridge in this pattern would broadcast events to multiple subscribers and SQS would be enforcing correct message ordering.

Messages in SQS could be consumed by EventBridge pipes for better event enrichment and transformation of events to the needed destination without having Lambda functions to be performing a transformational workload on messages.

Ultimately, it is about what the workload is and how each of the services - EventBridge or SQS either individually or in combination can elevate your Serverless architecture.