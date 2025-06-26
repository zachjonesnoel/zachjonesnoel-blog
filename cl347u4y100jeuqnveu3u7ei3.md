---
title: "Trigger Lambda Functions with event filtering"
datePublished: Sun Nov 28 2021 18:58:26 GMT+0000 (Coordinated Universal Time)
cuid: cl347u4y100jeuqnveu3u7ei3
slug: trigger-lambda-functions-with-event-filtering
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432817958/qsGmgLj05.jpeg

---

[AWS Lambda functions](https://aws.amazon.com/lambda/) recently announced an enhancement with event-triggers for [DynamoDB](https://aws.amazon.com/dynamodb/), [Amazon SQS](https://aws.amazon.com/sqs/), [Amazon Kinesis](https://aws.amazon.com/kinesis/) as event sources which makes it easier for event based Lambda function triggers to get invoked only based on the filter expression.
You can read about the official [announcement from AWS Blog post](https://aws.amazon.com/about-aws/whats-new/2021/11/aws-lambda-event-filtering-amazon-sqs-dynamodb-kinesis-sources/).
{%twitter 1464305840219852800%}

As it is known, AWS Lambda function supports for various event-sources for triggering the Lambda fn such as - DynamoDB Triggers, SQS Triggers, SNS Triggers and many more! For anyone who has worked with Lambda fn events, you would have to handle a lot of cases on your Lambda function code making it a *case* or *execution* handled by the developer. 
Eg. If you are working with DynamoDB Triggers/Streams, and your business logic needs the Lambda function to execute only when there is a new record *put* into the DynamoDB table then you would have to check for the *eventName* which has to be *INSERT* and then proceed further otherwise, end the Lambda function execution. 
Now with the new [announcement](https://aws.amazon.com/about-aws/whats-new/2021/11/aws-lambda-event-filtering-amazon-sqs-dynamodb-kinesis-sources/) it makes it easier to ensure the Lambda function gets invoked only based on the event source's filter which is configured. This not only enriches the developer's journey but also saves the milliseconds of Lambda function's execution making it a cost saving in the longer run.

#### Setting up event-filtering on Lambda functions
The Lambda Fn triggers with event filtering can be added or updated whenever the Lambda Fn is created or updated via console, CLI, CloudFormation.
When trying to add event-filtering from console, navigate to your Lambda function -> Configuration -> Triggers and add new. In the *additional* settings option, *Filter criteria* where you would have to enter a valid JSON object of your filter pattern/criteria.
![Setting up event-filtering on Lambda functions](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432815101/v5INb75U7.png)
 
In this demo, we will add a DynamoDB trigger for filtering only `"eventName":"INSERT"`. And to do so, we will have to add a valid JSON for the matching filter criteria. 
```JSON
{ "eventName" : ["INSERT"] }
```
Note, for adding DynamoDB Triggers, your Lambda fn should have the respective IAM policy which grants your Lambda fn access to DynamoDB. 
The same from CLI can be performed with the bash command 
```bash
aws lambda create-event-source-mapping \
--function-name DynamoDBEventFiltering \
--batch-size 100 \
--starting-position LATEST \
--event-source-arn arn:aws:dynamodb:us-east-1:XXXXXXXXXX:table/cars-demo \
--filter-criteria '{\"Filters\": [{\"Pattern\": \"{\"eventName\": [\"INSERT\"]}}]}'
```
The equivalent YAML template for CloudFormation or SAM is 
```YAML
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: An Amazon DynamoDB trigger that logs the updates made to a table.
Resources:
  DynamoDBEventFiltering:
    Type: 'AWS::Serverless::Function'
    Properties:
      Handler: app.handler
      Runtime: nodejs14.x
      CodeUri: src/
      MemorySize: 128
      Timeout: 3
      Events:
        cars-demo:
          Type: DynamoDB
          Properties:
            Stream: !GetAtt CarsDemoDynamoDB.StreamArn
            StartingPosition: TRIM_HORIZON
            BatchSize: 100
            Filters: 
              - Pattern: {\"Filters\": [{\"Pattern\": \"{\"eventName\": [\"INSERT\"]}}]}
  
```
The event filtering should follow a strict JSON format defined. You can find the rules [here](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventfiltering.html).
To test the Lambda function, you would need to do a DynamoDB operation such as - `put`, `delete` or `update` only for the DynamoDB Streams to trigger the Lambda function. If the same event JSON is used as Lambda Fn invoke JSON it would execute as the event-filtering is supported with the integration of DynamoDB, SQS, Kinesis. When performing actions such as creating an item on DynamoDB, updating it and deleting the same item, the DynamoDB Streams has triggered the Lambda fn only for the *INSERT*/`put` operation as defined by event-trigger filter criteria. 
![Event trigger](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432816559/wKrj5LW9k.png)
The other operations has not even invoked the Lambda function. 
 
#### Pricing
AWS Lambda functions confirms on the [announcement AWS Blog post](https://aws.amazon.com/about-aws/whats-new/2021/11/aws-lambda-event-filtering-amazon-sqs-dynamodb-kinesis-sources/) that there is no additional cost. The takeaway cookie here is with this event-filtering it saves the millsesonds of execution as the Lambda Fn is not even invoked which is a good save when the Lambda Fn is invoked millions of times on a production environment. 

#### Conclusion
The event-filtering for sources - DynamoDB, SQS and Kinesis helps Lambda Fn developers to integrate well with event-drivern architectures. This helps the developer experience in terms of integrating with DynamoDB, SQS and Kinesis but also it helps in saving cost as the Lambda Fn would not get invoked if the filter criteria is failed.
You can also refer to the AWS Blog about the same with [Kinesis event filtering implementation by Ben Smith](https://aws.amazon.com/blogs/compute/filtering-event-sources-for-aws-lambda-functions/).

For a deep-dive about how DynamoDB Streams based event-filtering works - 
{%post aws-builders/deep-dive-into-lambda-event-filters-for-dyanmodb-320%}