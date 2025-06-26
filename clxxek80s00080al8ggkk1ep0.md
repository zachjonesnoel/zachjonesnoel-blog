---
title: "Orchestration of HTTP invocation made possible with Step Functions"
seoTitle: "Orchestration of HTTP invocation made possible with Step Functions"
seoDescription: "Learn about Step Functions' new capability of HTTP invocation in the world of HTTP endpoints are invoked in multiple stages of workloads"
datePublished: Sat Jun 22 2024 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clxxek80s00080al8ggkk1ep0
slug: orchestration-of-http-invocation-made-possible-with-step-functions
canonical: https://blog.theserverlessterminal.com/orchestration-of-http-invocation-made-possible-with-step-functions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719500986211/dbb0ec20-faf3-471b-a3b2-ca17b1a0e88d.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1719501043295/a45057a9-89d6-432a-a418-20ec5769c9b7.png
tags: aws, apis, events, aws-step-functions

---

When building applications, we often interact with various systems, such as in-house microservices or third-party systems with HTTP endpoints as applications would need data and depend on third-party systems for business-critical tasks or data.

![Application service integrating with third party systems via HTTP invocations](https://cdn.hashnode.com/res/hashnode/image/upload/v1715266608652/ef3cccdf-bbde-4b76-8bfd-4097f48ba512.png align="center")

## HTTP API invocation from Lambda functions

This process may require using AWS Lambda functions that utilize HTTP libraries (such as `request` or `axios` for NodeJS) to construct the request with the appropriate `headers` and `body`, ensuring that the request is sent and the response is processed successfully.

![State machine with Lambda states to invoke HTTP endpoint](https://cdn.hashnode.com/res/hashnode/image/upload/v1719493021893/e997e0c8-910e-4e1e-a62d-bb9fc76d15eb.png align="center")

* **Increased complexity**: Introducing the Lambda function to invoke HTTP endpoints adds components that must be managed and maintained.
    
* **Increased latency**: Since the request response has to hop off another architectural component, it adds to the latency of the response.
    
* **Increased cost**: Running a separate Lambda function to handle the HTTP invocation will incur additional costs, as you'll be charged for the Lambda function execution in addition to the Step Functions execution.
    

Also, this opens up the flexibility to invoke the HTTP endpoints with whitelisted IP addresses and more security aspects of the Lambda function with Secrets Manager to maintain HTTP credentials.

## HTTP API invocation with EventBridge API destinations

In an Event-Driven world, Amazon EventBridge's API destination feature allows you to configure the HTTP endpoint and credentials from Secrets Manager, collectively known as EventBridge Connection.

![API Destinations to invoke HTTP endpoint](https://cdn.hashnode.com/res/hashnode/image/upload/v1719496333854/e3570b20-4d66-4fad-92b6-2f855d9d755a.png align="center")

In a blog about [How Freshworks Developer Platform integrates with AWS via EventBridge](https://medium.com/freshworks-developer-blog/a-guide-to-understanding-freshworks-integrations-with-amazon-eventbridge-9782e54f1076), I've explained in detail the integration but in this section of the blog, we will focus on the API destination part of the architecture.

```yaml
########################   API Connection   ############################
MyConnection:
    Type: AWS::Events::Connection
    Properties:
      AuthorizationType: BASIC
      Description: 'My connection with username/password'
      AuthParameters:
        BasicAuthParameters :
          Username : !Ref APIKeyValue
          Password : !Ref Password
########################   API Destination   ############################
MyApiDestination:
    Type: AWS::Events::ApiDestination
    Properties:
      Name: 'FreshdeskAPI'
      ConnectionArn: !GetAtt MyConnection.Arn
      InvocationEndpoint: !Ref FreshdeskURL
      HttpMethod: PUT
      InvocationRateLimitPerSecond: 10
########################   Event Rule to invoke API destination   ############################
EventRule: 
    Type: AWS::Events::Rule
    Properties: 
      Description: "EventRule"
      State: "ENABLED"
      EventBusName: !Ref MyEventBus
      EventPattern: 
        source:
          - "fw-sentiment"       
      Targets: 
        - Arn: !GetAtt MyApiDestination.Arn
          RoleArn: !GetAtt EventBridgeTargetRole.Arn
          Id: "MyAPIdestination"
          InputTransformer:
            InputPathsMap:
              id: $.detail.id
              priority: $.detail.priority
            InputTemplate: >
                {
                  "priority": <priority>
                }
          HttpParameters:
            PathParameterValues:
              - $.detail.id
          DeadLetterConfig:
            Arn: !GetAtt MyDLQueue.Arn
```

### API Destination brings to the table

* **Improved Security**: API connections use AWS Secrets Manager to securely store and manage authentication credentials, reducing the risk of exposing sensitive information, which is more secure than hardcoding credentials in Lambda functions.
    
* **Reduced complexity**: Using API connections with HTTP Tasks in Step Functions eliminates the need to write and maintain Lambda functions solely for making HTTP requests, simplifying the architecture and reducing the amount of code to manage.
    
* **Dependency of EventBridge Bus**: Introduces dependency on Event Buses to be able to invoke API destinations. Although it's managed, adding in additional components would mean that you have to define the Event Rules.
    

## HTTP invoke on Step Functions

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719496941010/aee9cefc-7776-4f91-9424-07674ba3f0ff.png align="center")

The above image from Application Composer indicates that when creating a State Machine with `HTTPInvoke` State, it would also require EventBridge Connection. You can also refer to the SAM template of the State Machine.

```yaml
  StateMachine:
    Type: AWS::Serverless::StateMachine
    Properties:
      Definition:
        StartAt: Call third-party API
        States:
          Call third-party API:
            Type: Task
            Resource: arn:aws:states:::http:invoke
            Parameters:
              Method: GET
              ApiEndpoint: ${ApiEndpoint}
              Authentication:
                ConnectionArn: ${EBConnectionARN}
            Retry:
              - ErrorEquals:
                  - States.ALL
                BackoffRate: 2
                IntervalSeconds: 1
                MaxAttempts: 3
                JitterStrategy: FULL
            ResultPath: $.ApiResponse
            End: true
      Logging:
        Level: ALL
        IncludeExecutionData: true
        Destinations:
          - CloudWatchLogsLogGroup:
              LogGroupArn: !GetAtt StateMachineLogGroup.Arn
      Policies:
        - AWSXrayWriteOnlyAccess
        - Statement:
            - Effect: Allow
              Action:
                - logs:CreateLogDelivery
                - logs:GetLogDelivery
                - logs:UpdateLogDelivery
                - logs:DeleteLogDelivery
                - logs:ListLogDeliveries
                - logs:PutResourcePolicy
                - logs:DescribeResourcePolicies
                - logs:DescribeLogGroups
              Resource: '*'
      Tracing:
        Enabled: true
      Type: STANDARD
      DefinitionSubstitutions:
        ApiEndpoint: !Ref APIEndpoint
        EBConnectionARN: !GetAtt EventBridgeConnection.Arn
```

Now that the State Machine is defined, let's also define EventBridge Connection along with the authorization credentials.

```yaml
EventBridgeConnection:
    Type: AWS::Events::Connection
    Properties:
      AuthorizationType: BASIC
      AuthParameters:
        BasicAuthParameters:
          Password: !Ref EventBridgeConnectionPassword
          Username: !Ref EventBridgeConnectionUsername
```

The resources - `StateMachine` and `EventBridgeConnection` are defined but for the StateMachine to successfully invoke the HTTP point, you would need to authorize the State Machine IAM execution policy as below -

```yaml
- Statement:
    - Effect: Allow
      Action:
          - events:RetrieveConnectionCredentials
      Resource:
          - !GetAtt EventBridgeConnection.Arn
    - Effect: Allow
      Action:
          - secretsmanager:GetSecretValue
          - secretsmanager:DescribeSecret
      Resource:
          - !Sub arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:events!connection/*
    - Effect: Allow
      Action:
          - states:InvokeHTTPEndpoint
      Resource:
          - !Sub arn:aws:states:${AWS::Region}:${AWS::AccountId}:stateMachine:*
```

Note that, EventBridge Connection under the hood uses AWS Secrets Manager so you would also need to allow `GetSecretValue` and `DescribeSecret`.

### HTTP Invoke state with it's advantages

Since HTTP invocation on Step Functions uses EventBridge Connection, it brings in the added advantage over invoking HTTP endpoints via Lambda Functions.

* **Enhanced execution insights**: Step Functions provides you with built-in monitoring and logging capability of the State Machine execution which makes it easier to trace and debug the HTTP requests within the context of the workflow.
    
* **Streamlined workflow management**: Directly integrating HTTP requests within Step Functions allows for more streamlined and cohesive workflow definitions. This makes it easier to manage and update workflows without needing to modify and redeploy Lambda functions
    
* **Error retry and handling**: Step Functions' error handling techniques give you the flexibility to define the errors and their retry by handling the errors elegantly. Additionally, EventBridge Connections also retries failed requests by default.
    

### HTTP Invoke state with it's disadvantages

* **Dependency on EventBridge Connections**: The HTTP invoke state depends on EventBridge connections for managing authentication credentials. This adds extra setup steps and dependencies, which can make the process more complicated. Users need to create and manage EventBridge connections and related secrets in AWS Secrets Manager, which can be confusing for some.
    
* **No Support for Private Endpoints**: The HTTP invoke state does not support private endpoints within a VPC. This limitation means it can't be used for secure, internal communication, forcing developers to use Lambda functions or other methods to interact with private APIs.
    
* **IAM Permissions Complexity**: To use the HTTP invoke state, you need to set up specific IAM permissions for the state machine's role. This includes permissions to make HTTP requests, use the EventBridge connection, and access the connection's secret. Managing these permissions can add complexity and potential security risks if not configured correctly.
    
* **Error debugging**: While error retry and handling are supported, error debugging becomes a challenge as HTTP requests could fail for various reasons - timeouts, authentication fail, missing header or parameter that can be hard to debug the error as it's an EventBridge Connection used under the hood.
    

## Wrap up

Yes, orchestrating the HTTP invocations is made possible with Step Functions but it does have its advantages and disadvantages. While building Serverless applications or workflows, the trade-offs are important to consider.

At times, you would need the flexibility to make dynamic HTTP invocations which is possible on the Lambda function rather than via EventBridge Connection. Additionally, it would require you to be careful with the IAM execution policy to allow the needed Secrets.