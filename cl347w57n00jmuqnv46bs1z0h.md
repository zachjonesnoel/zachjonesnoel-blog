---
title: "SQS Queue with a cross-account Lambda triggers"
datePublished: Sun Oct 10 2021 17:17:51 GMT+0000 (Coordinated Universal Time)
cuid: cl347w57n00jmuqnv46bs1z0h
slug: sqs-queue-with-a-cross-account-lambda-triggers
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432911614/e9y4FrNZp.jpeg

---

[AWS SQS](https://aws.amazon.com/sqs/) was limited to [AWS Lambda functions](https://aws.amazon.com/lambda/) triggers which belonged to the same account where the SQS Queue is created. Now with the recent announcement, we can [trigger Lambda functions from a different account](https://aws.amazon.com/about-aws/whats-new/2021/09/aws-lambda-lambda-function-amazon-sqs-queue/).
{% twitter 1443663581954773021 %}

#### Architecture
The architecture of cross account Lambda triggers would be as Account 1 which has the SQS Queue with existing Lambda function triggers in the same account, now the Lambda function in Account 2 can be added as the trigger with the Lambda function ARN.
![Architecute](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432903735/YdS583ITg.png)

#### Setting up Lambda function trigger
Navigate to the SQS console and select the Queue, under *Lambda triggers*, click on *Configure Lambda function trigger*
![Lambda trigger](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432905252/mU18lVJkF.png)
Select the the option *Enter AWS Lambda function ARN* and key in your Lambda function ARN from Account 2.
![Configure Lambda trigger](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432906787/boMVuNQHL.png)

#### Permission and access to the SQS Queue
Account 2 trigger Lambda function's execution role would need to be modified to give SQS access on Account 1 with the AWS managed policy `AWSLambdaSQSQueueExecutionRole` And also SQS Queue has to be provisioned with certain access policy so that Lambda function can process the messages

Lambda function execution role with `AWSLambdaSQSQueueExecutionRole` policy.
```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

SQS Queue access policy
```JSON
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__owner_statement",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::account1:root"
      },
      "Action": "SQS:*",
      "Resource": "arn:aws:sqs:us-east-1:account1:demo"
    },
    {
      "Sid": "demo_cross_account",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::account2:role/lambdaExecutionRole",
      },
      "Action": "SQS:*",
      "Resource": "arn:aws:sqs:us-east-1:account1:demo"
    }
  ]
}
```
Although, the policy states `"Action": "SQS:*"` with `"Effect": "Allow"`, the access rights for cross account doesn't grant you access to `AddPermission`, `CreateQueue`, `DeleteQueue`, `ListQueue`, `ListQueueTags`, `RemovePermission`, `SetQueueAttributes`, `TagQueue` and `UntagQueue` SQS actions. The only constraint for cross account Lambda triggers are both SQS and Lambda function have to be in the same AWS region.

#### Sending and receiving messages from SQS
From the console, you can click on *Send and receive messages* button and type your message and click on *Send message*.
![Send message](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432908407/njNPKQN3B.png)
Once the message is sent to the SQS Queue, it triggers the configured Lambda functions with a SQS message records. The message is logged with CloudWatch execution logs.
![Received message](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432910050/JfmJa0d-T.png)

This implementation could be tested with CLI also, you can follow the [tutorial](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs-cross-account-example.html) from AWS.

#### Conclusion
SQS being one of the oldest services has easied out integration with Lambda function triggers and now that been leveled up with cross-account Lambda function triggers it has eased the integration process for most developers who are working around to implement this. 