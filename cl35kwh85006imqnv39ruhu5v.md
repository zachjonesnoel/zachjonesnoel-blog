---
title: "Serverless apps : Why IaC should be the direction"
seoTitle: "Serverless apps : Why IaC should be the direction"
datePublished: Sat May 14 2022 08:00:31 GMT+0000 (Coordinated Universal Time)
cuid: cl35kwh85006imqnv39ruhu5v
slug: serverless-apps-why-iac-should-be-the-direction
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652431912588/ET5PxBK-F.png
tags: code, aws, infrastructure, serverless

---

Serverless applications is often inclusive of many services such as [EventBridge](https://aws.amazon.com/eventbridge/), [Simple Notification Service (SNS)](https://aws.amazon.com/sns/) and [Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) which play a vital role in decoupling and enhancing the performance and robustness of the Serverless application itself. And also services such as [Amazon API Gateway](https://aws.amazon.com/api-gateway/), [AWS AppSync](https://aws.amazon.com/appsync/) for API provisioning which are backed by [AWS Lambda functions](https://aws.amazon.com/lambda/), [Fargate](https://aws.amazon.com/fargate/), [AWS Step Functions](https://aws.amazon.com/step-functions/). These services have configurations and IAM permissions of their own. 

![Serverless developer looking at different compute options other than Lambda fns and Fargate](https://cdn.hashnode.com/res/hashnode/image/upload/v1652436926051/BvISp-8jC.png align="center")

Most of us who are familiar with server-based architectures (hosted on virtual machines on cloud) are also familiar with the "**lift and shift**" based migration or setting up with different development and staging environments. The application level configurations are more centered to the virtual machine itself. 

![This is not the case with Serverless](https://cdn.hashnode.com/res/hashnode/image/upload/v1652437069343/i_KygNxMH.png align="center")

But with serverless applications, the whole "**lift and shift**" based migration or sync between different development and staging environments is not maintained in one place rather the configuration is spread across different services and each of it would need an IAM permission of it's own (following the security recommendations of *least privileges*). 

![To simplify things](https://cdn.hashnode.com/res/hashnode/image/upload/v1652438022703/B0_SwMGR4.png align="left")
Serverless applications are best managed with Infrastructure as Code (IaC) which maintains a configuration file either as template file in YAML or in the programming language which specifies configuration as code (*literally*) with variables and method initialization. 

```TypeScript
   // Function that returns 201 with "Hello world!"
    const helloWorldFunction = new Function(this, 'helloWorldFunction', {
      code: new AssetCode('src'),
      handler: 'helloworld.handler',
      runtime: Runtime.NODEJS_12_X
    });

    // Rest API backed by the helloWorldFunction
    const helloWorldLambdaRestApi = new LambdaRestApi(this, 'helloWorldLambdaRestApi', {
      restApiName: 'Hello World API',
      handler: helloWorldFunction,
      proxy: false,
    });
```

The sample code is using AWS Cloud Development Kit (CDK) with TypeScript sample
+ Creation of *Lambda function* with `runtime`, `handler` and `code` properties defined to mention the Lambda function is powered with  NodeJS 12.x and the handler for the Lambda fn.
+ REST API via AWS API Gateway set with the properties `restApiName` for the name of the REST API, `handler` to mention the Lambda function name which the API Gateway is used to invoke and `proxy` based invocation type for the Lambda function.

The feasibility of defining and building your Serverless architecture with programming language of your choice on AWS CDK makes it a good choice of IaC for developers. However, if you are into defining the infrastructure with YAML based template file or a CLI which can walk you though is also available. 

### Options of IaC for AWS Serverless
+ [AWS CloudFormation](https://aws.amazon.com/cloudformation/)
+ [AWS Serverless Application Model (SAM) CLI](https://aws.amazon.com/serverless/sam/)
+ [AWS Cloud Development Kit (CDK)](https://aws.amazon.com/cdk/) 
+ [AWS Amplify CLI](https://docs.amplify.aws/cli)
+ [Terraform](https://www.terraform.io/)
+ [Serverless Framework](https://www.serverless.com/)

### Reasons to opt for IaC
+ **Single source of truth** helps in maintaining your complete infrastructure in one centralized Git or a template file shared across different developers so that everyone has access to same configuration which is ready to be deployed to their environment.
+ **Managing and provisioning resources on the cloud with code** has become very easy for developers to get started with the feasibility of code of their choice.
+ **Easy to replicate to multiple instances** as the common template which defines the infrastructure could be easily **deployed / re-deployed / rollbacked**.
+ **Best known method for CI/CD pipelines** as the CI/CD pipelines are automated and runs based on actions or triggers in the cloud. 
+ **Test in multiple instances / environments** has become a standard of testing process and with CI/CD pipelines which is powered with IaC makes is easier for load testing on a specific replica of production environment.
+ **Improvised development practice** for developers to collaborate between different developers without having to disturb other development or staging environments.  
+ **Consistent configuration across environments** as different environments are used, configurations should also have to be in sync with all environments.

To know more about how to build Serverless apps using IaC, the session explains about IaC with SAM CLI.

%[https://youtu.be/4mAaPF4L-ec]
