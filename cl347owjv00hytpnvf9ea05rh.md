---
title: "Step Functions to check if you have public S3 buckets"
datePublished: Fri Mar 11 2022 06:08:43 GMT+0000 (Coordinated Universal Time)
cuid: cl347owjv00hytpnvf9ea05rh
slug: step-functions-to-check-if-you-have-public-s3-buckets
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432572511/gelVX5g62w.jpeg

---

[Amazon S3](https://aws.amazon.com/s3/) provisions buckets which can be public or private, having public buckets has certain concerns as anybody off the internet can access your *S3 bucket* and the *objects* it holds. According to [security best practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html) mentioned by AWS, one of the best practice is to **ensure your S3 buckets are not publicly accessible**. And this can be achieved by a *Account*/*Organization* level setting and also during creation of buckets to *block public access by default*.

If by any chance, any *S3 bucket* is set with *public* access you could use different ways to identify and make it private. 
+ AWS Console
+ AWS CLI
+ AWS S3 SDK
+ AWS S3 APIs

### Understanding the workflow
In this blog-post, we will be using [Step Functions](https://aws.amazon.com/step-functions/) with [Amazon S3](https://aws.amazon.com/s3/) SDK integration of `ListBuckets`, `GetPublicAccessBlock` and `PutPublicAccessBlock`. And also [Simple Notification Service](https://aws.amazon.com/sns/) SDK `Publish` to notify which bucket policy had *public access*.
![State machine diagram](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432553398/P28hzSpUm.png)
#### ListBuckets state
We will first use `ListBuckets` SDK API to list all the buckets in that account. 
![Using ListBuckets state](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432554836/uuWMmHviB.png)
#### GetPublicAccessBlock state
And with the `Map` flow of Step Functions, we will loop through each of the bucket to `GetPublicAccessBlock`, this returns the set of parameters `BlockPublicAcls`, `BlockPublicPolicy`, `IgnorePublicAcls`, `RestrictPublicBuckets` which would be set to `false` if the bucket has *public access*.
![Using GetPublicAccessBlock state](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432556448/0-F2LnNz-.png)
If there is an error which occurs, this step has a catch block enabled to pass though and continue with the next items.
#### Choice state
For checking this condition we will use `Choice` with the multiple *and* statements.
![Using Choice for the conditional branching](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432558089/No17CiF4c.png)
If the condition does not satisfies, we will proceed to the *pass* state.
```JSON
{
     "And": [
         {
              "Variable": "$.res.PublicAccessBlockConfiguration.BlockPublicAcls",
                    "BooleanEquals": false
         },
         {
              "Variable": "$.res.PublicAccessBlockConfiguration.BlockPublicPolicy",
                    "BooleanEquals": false
         },
         {
              "Variable": "$.res.PublicAccessBlockConfiguration.IgnorePublicAcls",
                    "BooleanEquals": false
         },
         {
              "Variable": "$.res.PublicAccessBlockConfiguration.RestrictPublicBuckets",
                    "BooleanEquals": false
         }
     ],
     "Next": "SNS Publish"
}
```
#### SNS Publish state
If the conditions are satisfied, the *bucket* would be *publicly accessible*. The JSON payload would be published to a SNS topic to capture and process it accordingly.
![Using SNS Publish state](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432560125/ShBccnxed.png)
#### PutPublicAccessBlock state
After notifying the SNS Topic, we would also be changing the bucket to *block all public access* with the `PutPublicAccessBlock ` SDK API.
![Using PutPublicAccessBlock state](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432561740/tXD6likI0.png)

The complete state machine is as follows -
```JSON
{
  "Comment": "State machine to identify public buckets and make them private",
  "StartAt": "ListBuckets",
  "States": {
    "ListBuckets": {
      "Type": "Task",
      "Parameters": {},
      "Resource": "arn:aws:states:::aws-sdk:s3:listBuckets",
      "Next": "Map",
      "ResultPath": "$.s3ListBuckets"
    },
    "Map": {
      "Type": "Map",
      "End": true,
      "Iterator": {
        "StartAt": "GetPublicAccessBlock",
        "States": {
          "GetPublicAccessBlock": {
            "Type": "Task",
            "Parameters": {
              "Bucket.$": "$.s3JSON.Name"
            },
            "Resource": "arn:aws:states:::aws-sdk:s3:getPublicAccessBlock",
            "ResultPath": "$.res",
            "Catch": [
              {
                "ErrorEquals": [
                  "States.ALL"
                ],
                "Next": "Pass"
              }
            ],
            "Next": "Choice"
          },
          "Choice": {
            "Type": "Choice",
            "Choices": [
              {
                "And": [
                  {
                    "Variable": "$.res.PublicAccessBlockConfiguration.BlockPublicAcls",
                    "BooleanEquals": false
                  },
                  {
                    "Variable": "$.res.PublicAccessBlockConfiguration.BlockPublicPolicy",
                    "BooleanEquals": false
                  },
                  {
                    "Variable": "$.res.PublicAccessBlockConfiguration.IgnorePublicAcls",
                    "BooleanEquals": false
                  },
                  {
                    "Variable": "$.res.PublicAccessBlockConfiguration.RestrictPublicBuckets",
                    "BooleanEquals": false
                  }
                ],
                "Next": "SNS Publish"
              }
            ],
            "Default": "Pass"
          },
          "SNS Publish": {
            "Type": "Task",
            "Resource": "arn:aws:states:::aws-sdk:sns:publish",
            "Parameters": {
              "Message.$": "$",
              "TopicArn": "arn:aws:sns:us-east-1:228628157461:ErrorNotification"
            },
            "Next": "PutPublicAccessBlock",
            "ResultPath": "$.sns"
          },
          "PutPublicAccessBlock": {
            "Type": "Task",
            "End": true,
            "Parameters": {
              "Bucket.$": "$.s3JSON.Name",
              "PublicAccessBlockConfiguration": {
                "BlockPublicAcls": true,
                "BlockPublicPolicy": true,
                "IgnorePublicAcls": true,
                "RestrictPublicBuckets": true
              }
            },
            "Resource": "arn:aws:states:::aws-sdk:s3:putPublicAccessBlock"
          },
          "Pass": {
            "Type": "Pass",
            "End": true
          }
        }
      },
      "ItemsPath": "$.s3ListBuckets.Buckets",
      "Parameters": {
        "s3Index.$": "$$.Map.Item.Index",
        "s3JSON.$": "$$.Map.Item.Value"
      },
      "ResultPath": "$.s3Buckets"
    }
  }
}
```
### Workflow execution
The workflow execution works without any *specific input* to the state machine. For the execution, the state machine is running in *us-east-1* and the SDK works with buckets with the region *us-east-1*. 
As the state machine kicks off, all the buckets of the account is listed and each bucket is looped through to check for public access or not. If the bucket has public access, as per the flow SNS topic gets published and also the bucket is made *private* by *blocking all public access*.
![Looking at the execution of state machine](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432564611/xJytaWWki.gif)
As part of the SNS topic, there is an email subscriber who gets notified as email JSON.
![Email subscriber to SNS Topic with email JSON](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432566933/gen01zqs_.png) 
Before the execution started, I had identified the *public bucket*.
![Publicly available bucket](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432568593/R2kkDEPd-.png)
After the execution of state machine, the same *publicly accessible* bucket is now *private*.
![Bucked changed to private](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432570398/tU9w2X6ec.png)
 
### Conclusion
Often times, managing and ensuring S3 buckets are *private* is a tedious task and with Step Functions and SDK integration with S3, we can ease the process. 

As we are nearing to 3-14 (March 14th), popularly know as **PI Day** it also marks S3's 16th birthday and there is a celebration happening [**AWS Pi Day 2022**](https://pages.awscloud.com/NAMER-field-OE-Pi-Day-2022-reg-event.html).