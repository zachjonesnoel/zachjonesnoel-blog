---
title: "Cloud9 environments status monitoring with Step Functions"
datePublished: Sun Jun 19 2022 18:16:36 GMT+0000 (Coordinated Universal Time)
cuid: cl4rzr74605oe8unvdiyv09rx
slug: cloud9-environments-status-monitoring-with-step-functions
canonical: https://dev.to/aws-builders/cloud9-environments-status-monitoring-with-step-functions-26lm
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1656047214115/F53ab3712.jpeg

---

As developers using [AWS Cloud9](https://aws.amazon.com/cloud9/) as their choice of development environment, they often run the cost of the EC2 instance which is used under-the-hood. In this blog, you will understand how to monitor environments which are not in `stopped` state and trigger an SES email to a designated email address. This helps AWS administrators stay on track with Cloud9 environments.

### Understanding the workflow
The [Step Functions](https://aws.amazon.com/step-functions/)'s state machine uses [AWS Cloud9](https://aws.amazon.com/cloud9/) and [Amazon SES](https://aws.amazon.com/ses/) APIs which are integrated with [Step function's SDK integration](https://docs.aws.amazon.com/step-functions/latest/dg/supported-services-awssdk.html).
The different steps uses - [AWS Cloud9](https://aws.amazon.com/cloud9/) SDK API `listEnvironments` and `describeEnvironmentStatus` along with [Amazon SES](https://aws.amazon.com/ses/) SDK API `sendEmail`. So the state machine's IAM execution role would need access to perform the same actions.
![Complete workflow of the state machine](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047196963/ooFQya9Fq.png)
 
#### ListEnvironments state
![Snapshot of ListEnvironments state machine workflow and JSON definition](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047198630/8twaZuVvl.png)
Using [AWS Cloud9](https://aws.amazon.com/cloud9/) SDK API [`listEnvironments`](https://docs.aws.amazon.com/cloud9/latest/APIReference/API_ListEnvironments.html), we can list all the environment's IDs and the API restricts upto 25 max items then they would have to be paginated. This state machine is using the first page (first 25 records) only from the API response.

#### Map state
In state machines, we could use [map state](https://docs.aws.amazon.com/step-functions/latest/dg/amazon-states-language-map-state.html) for looping items. In this map state, each of the environment ID is used to fetch the status and do the needed processing. 
```json
"Map": {
      "Type": "Map",
      "Iterator": {
        "StartAt": "DescribeEnvironmentStatus",
        "States": {
          "DescribeEnvironmentStatus": {
            "Type": "Task",
            "Parameters": {
              "EnvironmentId.$": "$.value"
            },
            "Resource": "arn:aws:states:::aws-sdk:cloud9:describeEnvironmentStatus",
            "ResultPath": "$.envStatus",
            "Next": "Choice"
          },
          "Choice": {
            "Type": "Choice",
            "Choices": [
              {
                "Not": {
                  "Variable": "$.envStatus.Status",
                  "StringEquals": "stopped"
                },
                "Next": "SendEmail"
              }
            ],
            "Default": "Pass"
          },
          "SendEmail": {
            "Type": "Task",
            "End": true,
            "Parameters": {
              "Destination": {
                "ToAddresses": [
                  "zachjonesnoel@mailinator.com"
                ]
              },
              "Message": {
                "Body": {
                  "Html": {
                    "Charset": "UTF-8",
                    "Data.$": "States.Format('[{}] Environment ID {} is {}', $.envStatus.Status, $.value, $.envStatus.Message)"
                  }
                },
                "Subject": {
                  "Data": "Your Cloud9 Environment status!!!"
                }
              },
              "Source": "test@zachjonesnoel.com"
            },
            "Resource": "arn:aws:states:::aws-sdk:ses:sendEmail"
          },
          "Pass": {
            "Type": "Pass",
            "End": true
          }
        }
      },
      "End": true,
      "ItemsPath": "$.envs.EnvironmentIds",
      "Parameters": {
        "index.$": "$$.Map.Item.Index",
        "value.$": "$$.Map.Item.Value"
      }
    }
```

#### DescribeEnvironmentStatus state
![Snapshot of DescribeEnvironmentStatus state machine workflow and JSON definition](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047200286/TLqRkxZ97.png)
With each environment ID, we can get the status of that environment with [AWS Cloud9](https://aws.amazon.com/cloud9/) [`describeEnvironmentStatus` SDK API](https://docs.aws.amazon.com/cloud9/latest/APIReference/API_DescribeEnvironmentStatus.html). This responds with one of the status - 
+ `connecting` - Environment with connection in progress.
+ `creating` - For an environment which is creation in progress. 
+ `deleting` - Whenever the CloudFormation stack is deleting the environment and the other AWS resources used under-the-hood.
+ `error` - When the environment is in an error state.
+ `ready` - For a successfully connected environment which is ready for usage.
+ `stopped` - Based on the idle time stop, the environment would be stopped.
+ `stopping` - Whenever the stop is initiated.

#### Choice state
![Choice state based on the status response](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047201958/o0hgCOpxi.png)
 With response status from `describeEnvironmentStatus`, a choice condition is used to check if the status is anything other than `stopped` then trigger the next step else pass.

#### SendEmail state
![SendEmail state workflow and JSON definition](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047203893/Eam5Fis7d.png)
The [`sendEmail` SDK API](https://docs.aws.amazon.com/ses/latest/APIReference-V2/API_SendEmail.html) of [Amazon SES](https://aws.amazon.com/ses/) is used to send email from a verified identity on SES. This uses HTML based body. For ensuring the email body is well formatted, [`States.Format` intrinsic function](https://docs.aws.amazon.com/step-functions/latest/dg/amazon-states-language-intrinsic-functions.html) is used. 

### Workflow executions
![Complete execution walk-through](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047206726/BzWmM1P50.gif)
 
This state machine execution walks though the different states and also the iterations of each environment ID. There are cases, where the email has been triggered with *SendEmail state*.

When a new environment is created, an email with the status `creating` is sent along with the message.
![New environment in creation](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047209062/zRzz5hojX.png)

When the environment is successfully created, or the state is successfully connected it's status is changed to `ready`.
![Environment is successfully created and ready to use](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047210737/t5-xpvG_e.png)

When the environment is deleted, this invokes a CloudFormation stack delete and email with the following body is sent.
![Environment with deleting state triggered email](https://cdn.hashnode.com/res/hashnode/image/upload/v1656047212378/WuR2qI_8z.png)
 
### Conclusion
AWS Step Functions helps orchestration of various workflows. This involves some administrative monitoring workflows also automated helping to monitor different status changes in Cloud9 environment and notifying the needed users via email leveraging Amazon SES.
And as part of cost-optimizaton, the *pass state* could be avoided. 