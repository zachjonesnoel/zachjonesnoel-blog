---
title: "AWS Lambda functions in VPC"
datePublished: Mon Feb 14 2022 11:58:59 GMT+0000 (Coordinated Universal Time)
cuid: cl347q7w100iouqnv4sh30y0h
slug: aws-lambda-functions-in-vpc-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432635155/uohXDyU1b.jpeg

---

[AWS Lambda functions](https://aws.amazon.com/lambda/) is one of the popular *Compute* service for *Serverless* which provides powerful executions for your business logic. Lambda functions have seamless integration to every other AWS Service, but some of the services cannot be available over public network to resolve this Lambda functions can also be configured with a [Virtual Private Cloud (VPC)](https://aws.amazon.com/vpc/). 

### Why in VPC?
![Lambda function in VPC](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432631855/j75GNm60-.png)
Whenever your Lambda function is invoking or communicating with services such as - 
+ Amazon RDS
+ Aurora
+ Elasticache
+ Internal APIs with containers/EC2
+ Elastic File System

With this you can not only get a network separation but also you will be able to securely connect to the dedicated services for all `read` or `write` as all the services in a VPC can communicate effectively.

Additionally, if the use-case involves external systems whitelisting your IP Address for invocation of external systems then you can add an Elastic IP and add the Lambda function to that VPC. 

If your Lambda function is not in the VPC and if you are trying to invoke RDS or Aurora, the query results in a timed-out request.

### Down-side to having Lambda in VPC
When your Lambda function is in a VPC, there are some down-side also which can affect the execution/computation for your application. 
+ Lambda fn will not have access to DynamoDB
+ Lambda fn will not have access to SNS, SQS, EventBridge.
+ Restricted access to only the services which are there in VPC.
+ Lambda fn will not have access to external internet.
+ Elongated cold starts with Lambda fns in VPC.
+ Exhausting ENI Limits for the IP address which are associated to a subnet.

### When to add VPC to your Lambda and when not to?
[AWS Serverless Application Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/wellarchitected-serverless-applications-lens.pdf) gives a perfect overview of how to choose and decide if you want the Lambda function in a VPC or not. 

![Decision tree for choosing VPC for Lambda functions](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432633371/GBKF8j9bQ.png)

This makes it clear that unless, the Lambda function is using only AWS Services which are in VPC and works only in VPC such as **RDS** or **Elasticache**, don't go with VPC and if at all a VPC is setup and your Lambda function needs internet access, setup a NAT Gateway.

### Conclusion
Lambda functions in VPC can affect connection to other AWS Services in a good and bad way, to help us decide better AWS Serverless Application Lens whitepaper provides a decision tree where it makes it clear that *unless needed, not to setup Lambda function in VPC*.