---
title: "AWS Lambda powered up by AWS Graviton2"
datePublished: Sun Oct 03 2021 14:57:44 GMT+0000 (Coordinated Universal Time)
cuid: cl347x95h00j2tpnv7dwm57q2
slug: aws-lambda-powered-up-by-aws-graviton2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432963310/X3MNCFgYO.jpeg

---

[AWS Lambda](https://aws.amazon.com/lambda/) recently announced the launch of [AWS Graviton2 processor](https://aws.amazon.com/ec2/graviton/) (arm64 architecture) for Lambda functions which would not only make your Lambda function to execute faster but also optimizes the cost for execution. Graviton2 is an AWS built and owned processor.

You can read more about it from the [announcement blog](https://aws.amazon.com/blogs/aws/aws-lambda-functions-powered-by-aws-graviton2-processor-run-your-functions-on-arm-and-get-up-to-34-better-price-performance/).
{% twitter 1443337084786331652 %}

Lambda functions which are executing with Amazon Linux 2 runtime, now supports two architectures -
+ arm64 - 64-bit ARM architecture, for the AWS Graviton2 processor.
+ x86_64 – 64-bit x86 architecture, for x86-based processors.

All the existing Lambda functions would be running on **x86_64 architecture** and this architecture is the default architecture for any new Lambda function. 

#### Key takeaways from the blog
+ [Graviton2 (arm64) for Lambda functions.](#graviton2)
+ [Setting up Graviton2 (arm64) powered Lambda functions.](#setting-up-arm64)
+ [Performance with arm64 vs x86_64 architecture.](#arm64-vs-x86_64)

#### Graviton2 (arm64) for Lambda functions <a name="graviton2"></a>
The Gaviton2 based arm64 architecture makes the computation faster which results in faster execution of the Lambda function and in turn the cost for Lambda is also optimized. 

> AWS Lambda functions running on Graviton2, using an Arm-based processor architecture designed by AWS, deliver up to 34% better price performance compared to functions running on x86 processors. 

The **arm64 architecture** for Lambda functions is currently available in selected AWS Regions - 
+ US East (N. Virginia) us-east-1
+ US East (Ohio) us-east-2
+ US West (Oregon) us-west-2
+ Asia Pacific (Mumbai) ap-south-1
+ Asia Pacific (Singapore) ap-southeast-1
+ Asia Pacific (Sydney) ap-southeast-2
+ Asia Pacific (Tokyo) ap-northeast-1
+ EU (Frankfurt) eu-central-1
+ EU (Ireland) eu-west-1
+ EU (London) eu-west-2

The supported runtimes based on Amazon Linux 2 are -
+ NodeJS 12.x and 14.x
+ Python 3.8 and 3.9
+ Java 8 (`java8.al2`) and 11
+ .NET Core 3.1
+ Ruby 2.7
+ Custom runtime (`provided.al2`)

Graviton2 processor is well suited for Lambda functions which are processing heavy high-performance computations, video encoding, simulation workloads.

#### Setting up Graviton2 (arm64) powered Lambda function <a name="setting-up-arm64"></a>
When creating a new Lambda function from the web-console, you can select one of the architecture options - **x86_84** (default) or **arm64** (Graviton2 Processor).
![New Lambda Function](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432918743/zoVnvmDSM.png)
If you want to change the architecture for the existing Lambda function, navigate to the *Runtime settings*.
![Runtime settings](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432920634/Kn6Nbq7Ah.png)
Click on *Edit* to change the settings.
![Edit runtime settings](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432922129/Ni-U0_cyx.png)

If you are trying to change an existing Lambda function from x86_64 to arm64 architecture, ensure to check the [function code compatibility with arm64 architecture](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html#foundation-arch-considerations) and also [suggested migration steps](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html#foundation-arch-steps).

Creating and updating Lambda functions with [AWS SAM](https://aws.amazon.com/serverless/sam/) and [AWS CDK](https://aws.amazon.com/cdk/) will soon support both the architectures.

Running Lambda functions with arm64 architecture on x86_64 machine
```bash
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

#### Performance with arm64 vs x86_64 architecture <a name="arm64-vs-x86_64"></a>
For determining the performance I would be reusing the DynamoDB operation performance SAM template from my previous blog post.  
{% link https://dev.to/awscommunity-asean/dynamodb-operations-scan-vs-query-with-cloudwatch-custom-metrics-2mik %} 
With some modifications - 
+ Change the number of `StepFunctionIndex` to 10.
+ Added `LambdaMemory` and `LambdaProcessor` parameters to the CloudWatch Custom Metrics. `LambdaMemory` is taken from Lambda's environment variable and `LambdaProcessor` is a value set explicitly as environment variable. 
```JavaScript
await cloudwatch.putMetricData({
        'MetricData': [{
            'MetricName': 'LambdaProcessorMeteric',
            'Dimensions': [{
                    'Name': 'OPERATION',
                    'Value': metric
                },
                {
                    'Name': 'LambdaMemory',
                    'Value': process.env.AWS_LAMBDA_FUNCTION_MEMORY_SIZE
                },
                {
                    'Name': 'LambdaProcessor',
                    'Value': process.env.lambda_fn_arch
                }
            ],
            'Unit': 'Milliseconds',
            'Value': value
        }, ],
        'Namespace': 'LambdaProcessMetrics'
    }).promise()
```
<!--
StepFunction invocation 1 : x86_64 architecture with 128MB Memory.
Average ![StepFunction invocation 4](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432923400/rAieh4jGL.png)
Minimum ![Minimum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432924859/sGHToyq5z.png) 
Maximum ![Maximum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432926159/cYlNVDQrX.png)

StepFunction invocation 2 : x86_64 architecture with 256MB Memory.
Average ![StepFunction invocation 4](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432927446/acgFZjn7N.png) 
Minimum ![Minimum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432928726/62Sf7vMZH.png) 
Maximum ![Maximum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432930004/4hcPkXlBz.png)

StepFunction invocation 3 : arm64 architecture with 128MB Memory.
Average ![StepFunction invocation 4](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432931252/k_4BuzAhP.png)
Minimum ![Minimum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432932623/ba-nFIVpX.png)
Maximum ![Maximum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432933878/G9hQzNxAg.png)

StepFunction invocation 4 : arm64 architecture with 256MB Memory.
Average ![StepFunction invocation 4](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432935230/RQ5xgO1dx.png)
Minimum ![Minimum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432936539/ssR1uU7-8.png)
Maximum ![Maximum](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432937876/FQZSM78XM.png)
-->

arm64 architecture duration of several Lambda function invocations
![Duration](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432939149/YRJBCThcJ.png)
arm64 architecture average duration of several Lambda function invocations
![Average duration](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432940632/RpErRdHE1.png)
Duration graph
![Duration graph](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432941927/J2saPqfqb.png)
 
x86_64 architecture duration of several Lambda function invocations
![Duration](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432943225/GWej8uP0M.png)
x86_64 architecture average duration of several Lambda function invocations
![Average duration](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432945171/Ez24oqMOy.png)
Duration graph
<!--![Duration graph](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432946562/cdmNsWn18.png)-->
![Duration graph](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432947924/F7jcI_iuH.png)
 
All `query` operations with x86_64 and arm64 with 128MB and 256MB Memory
Average ![All query operations with x86_64 and arm64 with 128MB and 256MB Memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432949224/KlAaLXvSO.png)  
Minimum ![All query operations with x86_64 and arm64 with 128MB and 256MB Memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432950598/hdbXzm7a2.png)
Maximum ![All query operations with x86_64 and arm64 with 128MB and 256MB Memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432951954/cSogS4zvH.png) 

All `scan` operations with x86_64 and arm64 with 128MB and 256MB Memory
Average ![All scan operations with x86_64 and arm64 with 128MB and 256MB Memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432953231/HckF9Fz2g.png)
Minimum ![All scan operations with x86_64 and arm64 with 128MB and 256MB Memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432954584/SlCEuSLNJ.png)
Maximum ![All scan operations with x86_64 and arm64 with 128MB and 256MB Memory](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432955995/-V9VsrYbV.png) 

All operations 
Average ![All operations](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432957496/wL4qGd1g4.png)
Minimum ![All operations](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432958912/tfdLVVgO5.png) 
Maximum ![All operations](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432960363/w-qKv7EStL.png)

#### Pricing
Lambda function with **arm64 architecture** is cheaper than the **x86_64 architecture**. 
>And also both the architectures are included under the 1M free requests per month and 400,000 GB-seconds of compute time per month, usable for functions powered by both x86, and Graviton2 processors, in aggregate.

![image](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432962029/hT2aXDSbl.png)
You can know more about [Lambda pricing with different architectures](https://aws.amazon.com/lambda/pricing/).
For this demo, the complete execution cycle is part of the **Free Tier limit** inclusive of all StepFunction executions and Lambda function executions.

#### Conclusion
The operational and execution time with **x86_64** and **arm** has significant difference, the DynamoDB operations `Scan` and `Query` have different variations of execution time but the Lambda function as a whole, the time of duration is dropped which in-turn affects the billed execution time thus making the cost of Lambda function of **arm64** significantly lower than **x86_64**.
