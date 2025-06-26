---
title: "Serverless workflow design and development using Application Composer and Step Functions"
seoTitle: "Serverless workflow design and development using Application Composer"
seoDescription: "Learn about how AWS Application Composer integrates with VS Code IDE and Workflow Studio to enhance local development of Serverless workflows"
datePublished: Sat Feb 10 2024 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clsuly9tb00030bl77pey9p8k
slug: serverless-workflow-design-and-development-using-application-composer-and-step-functions
canonical: https://blog.theserverlessterminal.com/serverless-workflow-design-and-development-using-application-composer-and-step-functions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1708448084414/885465b9-41f2-4db1-a4a1-e70976654a34.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1708448129050/da18ec23-a533-48b0-b8db-d5028945d6b3.png
tags: aws, serverless, aws-step-functions, amazon-bedrock, application-composer

---

AWS Step Functions has become one of the crucial architectural components for prompting Serverless orchestrations on AWS. Since Step Functions's launch, defining the state machine using JSON has been a pain point.

In this blog, we will look at the importance of IaC and how tools like Application Composer and Workflow Studio enable it with the low-code approach of drag-and-drop of components with the comfort of VS Code.

# Infrastructure as Code (IaC)

When building applications on the cloud, irrespective of Serverless, Containers, or Virtual Machines; it is recommended to use the Infrastructure as Code (IaC) approach so that provisioning and deploying resources to the cloud would be automated via workflows or with IaC tools that refers to a single source of truth where all the configurations of all the needed resources are defined.

![Using IaC as part of developer workflows](https://cdn.hashnode.com/res/hashnode/image/upload/v1707844629719/410bae99-f882-4bcd-aba7-ab91ff9d6aff.png align="center")

You can read more about [why IaC should be the direction for building applications](https://blog.theserverlessterminal.com/serverless-apps-why-iac-should-be-the-direction).

# Workflow Studio

[AWS Step Functions launched Workflow Studio in 2021](https://aws.amazon.com/about-aws/whats-new/2021/announcing-workflow-studio-a-new-low-code-visual-workflow-designer-foraws-step-functions/), where the new low code tool has been revolutionary for drag and drop for over 1000+ API Actions across multiple AWS Services.

![Workflow studio demonstration of creating a state machine with drag and drop](https://cdn.hashnode.com/res/hashnode/image/upload/v1707829499925/6177060b-50e9-480e-b1fc-5dd171cec475.gif align="center")

Workflow Studio enables visual building and understanding of the workflow execution with different AWS Services API actions invocation via optimized or SDK integration. As part of the integration, you can define the API parameters and share data across different States.

![Using the Workflow Studio to generate ASL](https://cdn.hashnode.com/res/hashnode/image/upload/v1707829998108/3e2e2fbb-92e1-4b15-b7da-e367ae736815.gif align="center")

Workflow Studio generates the Amazon States Language (ASL) in real-time based on the States defined visually with all the parameters. Workflow Studio also facilitates different configurations such as error handling, input, and output for different states and also defines the flow with parallel, map for loops, and choice for branching.

# Application Composer

![Application Composer adds to improved developer productivity](https://cdn.hashnode.com/res/hashnode/image/upload/v1707842329475/f7ecc5ae-7343-4855-9f01-9d6d0ab8b952.png align="center")

[AWS Application Composer](https://aws.amazon.com/application-composer) is a visual IaC builder for making drag-and-drop designs of different AWS Services on the canvas to generate an equivalent Infrastructure as Code (IaC) template with SAM (Serverless Application Model) Template using YAML. The experience of designing applications with AWS Services that generate SAM templates to synchronize when using the Google Chrome browser locally was the first step to making IaC generation and a real-time sync to the local file system.

Application Composer initially supported a few AWS Serverless services and now it [supports over 1000 CloudFormation supported resources](https://aws.amazon.com/about-aws/whats-new/2023/09/aws-application-composer-1000-cloudformation-resources/). During AWS re:Invent 2023, [Application Composer announced the IDE (VS Code) experience of building IaC as part of AWS Toolkit](https://aws.amazon.com/about-aws/whats-new/2023/11/ide-extension-aws-application-composer/) where you can use Application Composer to design your architectures while the SAM template would be generated in your VS Code directory.

Now you are also able to use [AWS Step Functions' Workflow Studio in the Application Composer VS Code experience](https://aws.amazon.com/about-aws/whats-new/2023/11/aws-application-composer-step-functions-workflow-studio/) as the integration enables using Workflow Studio that generates Amazon States Language (ASL) in real-time which synchronizes either with SAM template or as a `asl.json` or `asl.yaml` files in the local project directory.

# Application Composer + Workflow Studio on VS Code

Install the latest [VS Code extension of AWS Toolkit](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-toolkit-vscode) that includes Application Composer with Workflow Studio.

In an empty project directory, create a `template.yaml` file and click on "Application Composer" icon on the top right that launches Application Composer in VS Code.

## Creating a State Machine with an external ASL file

![Application Composer with Step Function resource creation with boiler plate State Machine](https://cdn.hashnode.com/res/hashnode/image/upload/v1707845515709/a3657a54-5bc9-42f3-8f27-aacdf24d922b.gif align="center")

When you drag-and-drop `StateMachine` resource, it generates the boilerplate State Machine with Lambda task, and the equivalent changes are made to `template.yaml` with the State Machine resource. Since the option of an external file for State Machine ASL was selected, `statemachine.asl.json` file is generated with the ASL definition of the Lambda task.

## Defining State Machine using Workflow Studio

In the Application Composer, the `StateMachine` resource which has an option to launch Workflow Studio locally on VS Code.

![Launching Workflow Studio from Application Composer](https://cdn.hashnode.com/res/hashnode/image/upload/v1707846308891/0aa4b6c5-cb0e-4a23-99cc-8281d0b33dc4.png align="center")

### Bedrock's `InvokeModel` task

At AWS re:Invent 2023, [Step Functions launched the support for optimized integration for Amazon Bedrock](https://aws.amazon.com/about-aws/whats-new/2023/11/aws-step-functions-optimized-integration-bedrock/) that enables the state machines to invoke Bedrock APIs.

From the list of supported resources on Workflow Studio, choose Bedrock's `InvokeModel` API Action that requires you to define the LLM model used on Bedrock along with the input.

![Adding Bedrock InvokeModel API via Workflow Studio](https://cdn.hashnode.com/res/hashnode/image/upload/v1707846209713/9be56c02-2b90-4995-981a-4a502518e894.gif align="center")

Below is the generated ASL from Workflow Studio.

```json
"Bedrock InvokeModel": {
    "Type": "Task",
	"Resource": "arn:aws:states:::bedrock:invokeModel",
	"Parameters": {
	    "ModelId": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-v2:1",
			"Body": {
				"prompt": "Human:Tell me a fun story about MARVEL characters using Infrastructure as Code to build Serverless Applications\nAssistant:",
				"max_tokens_to_sample": 2000
			}
		},
		"ResultPath": "$.response"
}
```

### Adding a DynamoDB `PutItem` task

Now that Bedrock would generate a story, it has to be stored on DynamoDB. To do so, on Workflow Studio add `PutItem` API action to the state machine definition where the table name is passed with CloudFormation substitution and story from the previous task's response. Also, using `STATES.UUID()` intrinsic function to auto-generate UUIDs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707847003215/e3c64f5b-1cb9-48b3-8338-04b2abfb4937.gif align="center")

```json
"DynamoDB PutItem": {
	"Type": "Task",
	"Resource": "arn:aws:states:::aws-sdk:dynamodb:putItem",
	"Parameters": {
		"TableName": "${MyDynamoDBTable}",
			"Item": {
				"id": {
					"S": "STATES.UUID()"
				},
				"story": {
					"S.$": "$.response.Body.completion"
				}
			}
		}
}
```

## Defining DynamoDB resource

In the Workflow Studio, the DynamoDB's `PutItem` action would need the table to be created, and to do so, on Application Composer drag-and-drop DynamoDB and define the table structure.

![Defining DynamoDB using Application Composer](https://cdn.hashnode.com/res/hashnode/image/upload/v1707847423829/581f0f32-18e6-4976-a469-81e9c82b2166.gif align="center")

Once the DynamoDB table is defined, update the CloudFormation references for the State Machine to use the table that is defined in the SAM template.

![Using CloudFormation reference for DDB table name on State Machine](https://cdn.hashnode.com/res/hashnode/image/upload/v1707847660337/4a2d6a15-a8d8-48a9-9d42-411abf76bfe9.gif align="center")

Application Composer can detect the different resources used in the SAM project and provides all the resource options when using `!Ref`.

## Updating IAM execution role for State Machine

When `StateMachine` is created, Application Composer adds the policies `AWSXrayWriteOnlyAccess` and CloudWatch logs permissions by default but even though State Machine is defined; Step Function doesn't auto-generate the needed IAM execution role and we would have to update the IAM policy with access to DynamoDB `PutItem` and Bedrock `InvokeModel` APIs.

```yaml
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
        - Statement:
            - Effect: Allow
              Action:
                - dynamodb:PutItem
              Resource: !GetAtt Table.Arn
        - Statement:
            - Effect: Allow
              Action:
                - bedrock:InvokeModel
              Resource: arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-v2:1
```

# Deploy using SAM sync

Now that the SAM template is complete with Application Composer and Workflow Studio, use `sam sync --stack-name "<your-stack-name>"` to deploy the resources to your AWS Account.

![Deploying the stack to cloud via sam sync](https://cdn.hashnode.com/res/hashnode/image/upload/v1707848254096/01ec3fa9-6ea2-488f-85a3-58f9969a4ffe.gif align="center")

# It's deployed!

Now that the State Machine is deployed, time to start the execution to see what story about "Marvel heroes using IaC for Serverless" is generated.

![Fun story generated with Gen AI](https://cdn.hashnode.com/res/hashnode/image/upload/v1707848519520/4250418c-3fe1-4a19-91fc-e31b39a30550.png align="center")

Check out Arshad Zackeriya and me talking about Application Composer on [The Zacs' Show Talking AWS](https://www.youtube.com/channel/UCr8ggoShNiCvMvyMEKlpxnA). 