---
title: "Getting into AWS SDK JS v3 mindset"
datePublished: Tue Nov 02 2021 19:46:50 GMT+0000 (Coordinated Universal Time)
cuid: cl347usgu00ittpnv71bt35rp
slug: getting-into-aws-sdk-js-v3-mindset
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432848306/_d-JUhqUv.jpeg

---

[AWS SDK for JavaScript v3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/) is an upgraded version of v2 with features which will have you wanting to get started with JS SDK v3. But coming from SDK v2 experience, it makes it hard for migrating to JS SDK v3 as you need to get into the "*v3 mindset*".

{%twitter 1339000173184176133 %}

This blog post gives you a sense of how the pros of v3 helps you to develop better applications and also some cons I personally faced when switching from v2 to v3 thus helping you build up the "*v3 mindset*".

<center>

**Pros** | **Cons** 
--- | --- 
[Middleware stack](#middleware-stack) | [Long procedures](#long-procedures)
[The size after installation](#size-of-installation) | [Lambda node-module / Lambda layer size is far too high](#app-size)
[TypeScript support](#ts-support) | [Complicated JS SDK Documentation](#complicated-doc)
[Modular architecture](#modular-arch) | 
[Ease of mocking](#mocking) |
 
</center>

#### Middleware Stack <a name="middleware-stack"></a>
[Middleware stack](https://aws.amazon.com/blogs/developer/middleware-stack-modular-aws-sdk-js/) lets you define your own middleware between your application and cloud. The middleware can be used for various use-cases such as serializing the response, sanitizing the input/response, adding certain AWS Resource tags. These use-case can be custom built by your application itself. 

This example from [AWS Blogs](https://aws.amazon.com/blogs/developer/middleware-stack-modular-aws-sdk-js/) shows how middleware for S3 `putObject` could be used to add custom headers for your HTTP requests via SDK.
```JavaScript 
const { S3 } = require("@aws-sdk/client-s3");
const client = new S3({ region: "us-west-2" });
// Middleware added to client, applies to all commands.
client.middlewareStack.add(
  (next, context) => async (args) => {
    args.request.headers["x-amz-meta-foo"] = "bar";
    const result = await next(args);
    // result.response contains data returned from next middleware.
    return result;
  },
  {
    step: "build",
    name: "addFooMetadataMiddleware",
    tags: ["METADATA", "FOO"],
  }
);

await client.putObject(params);
```
This could help you in terms of security where your bucket policy, can allow `putObject` only when it has a specific header. 

Similarly, you can have series of middleware business logic which can help you build a *Middleware stack*.

#### TypeScript Support <a name="ts-support"></a>
TypeScript has become popular in-terms of the adoption and development preference as it is an extension of JavaScript with static type definitions so makes it easier for developers to handle various types. AWS JS SDK v3 is built on TypeScript which makes it for developers to go though the well-documented code and also understand the specific datatype which is required by the SDK. 

The AWS Blog post [First-class TypeScript support in modular AWS SDK for JavaScript](https://aws.amazon.com/blogs/developer/first-class-typescript-support-in-modular-aws-sdk-for-javascript/) explains why TS was preferred for building JS SDK v3.

#### Modular Architecture <a name="modular-arch"></a>
The complete SDK adapts modular architecture i.e. unlike JS SDK v2 which is published as one single package on Node Package Manager (NPM), SDK v3 uses dedicated packages for each service which can be imported from the same parent `@aws-sdk` package.
In v2 SDK, if you have to initialize DynamoDB client (DocumentClient) you would have to import `aws-sdk` package and then use DynamoDB class and create an object.
```JavaScript
var AWS = require('aws-sdk');
AWS.config.update({ region: 'us-west-2' });
var docClient = new AWS.DynamoDB.DocumentClient();
```
Even with v2, you can import DynamoDB alone and initialize the object.
```JavaScript
var ddb = require('aws-sdk/DynamoDB');
var docClient = new ddb.DocumentClient();
```
But with v3 SDK, you can directly import DynamoDB client from `@aws-sdk` and import the needed operation command. 
```JavaScript
const { DynamoDBClient, UpdateTableCommand } = require('@aws-sdk/client-dynamodb'); 
const client = new DynamoDBClient({ region: 'us-west-2' });
```
The modular architecture has segregated high-level commands and low-level commands so if you would like to `marshall` or `unmarshall` the DynamoDB item which is sent as input or received as response, this would have to be imported from `util-dynamodb` package.
```JavaScript
const { DynamoDBClient, QueryCommand  } = require("@aws-sdk/client-dynamodb");
const { marshall, unmarshall } = require("@aws-sdk/util-dynamodb");

const client = new DynamoDBClient({ region: 'us-west-2' });
let input = {
     "TableName": "cars-demo",
     "KeyConditionExpression": "pk = :pk",
     "ExpressionAttributeValues": marshall({
            ":pk":"CARS"
     })
}
const command = new QueryCommand(input);
const response = await client.send(command);
response.Items.forEach({item}=>{
     console.log(unmarshall(item));
})
```

####  The size after installation <a name="size-of-installation"></a>
The size after installation of the SDK has reduced significantly. 
{%twitter 1446596564969930753%}
Also, the blog post [How we halved the publish size of modular AWS SDK for JavaScript clients](https://aws.amazon.com/blogs/developer/how-we-halved-the-publish-size-of-modular-aws-sdk-for-javascript-clients/) explains in detail.

#### Ease of mocking <a name="mocking"></a>
Mocking library `aws-sdk-client-mock` which is used for unit tests can be used with any JS unit testing frameworks. 
Example from AWS blog [Mocking modular AWS SDK for JavaScript (v3) in Unit Tests](https://aws.amazon.com/blogs/developer/mocking-modular-aws-sdk-for-javascript-v3-in-unit-tests/)
```JavaScript
import { mockClient } from "aws-sdk-client-mock";
import { DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";
const ddbMock = mockClient(DynamoDBDocumentClient);

import { GetCommand } from "@aws-sdk/lib-dynamodb";

it("should get user names from the DynamoDB", async () => {
  ddbMock
    .on(GetCommand)
    .resolves({
      Item: undefined,
    })
    .on(GetCommand, {
      TableName: "users",
      Key: { id: "user1" },
    })
    .resolves({
      Item: { id: "user1", name: "Alice" },
    })
    .on(GetCommand, {
      TableName: "users",
      Key: { id: "user2" },
    })
    .resolves({
      Item: { id: "user2", name: "Bob" },
    });
  const names = await getUserNames(["user1", "user2", "user3"]);
  expect(names).toStrictEqual(["Alice", "Bob", undefined]);
});
```
This mock unit testing checks and validates with the user names and user IDs. If the strict match/equal is not found, the unit test fails.

***

#### Long Procedures <a name"long-procedures"></a>
SDK v3 has provided fantastic developer features but the volume of coding and writing long procedures has made it "*a little overhead in terms of adaption*", as you need to import multiple packages and the process of invoking SDK APIs is - 
+ Importing multiple packages.
```JavaScript
const { DynamoDBClient, QueryCommand  } = require("@aws-sdk/client-dynamodb");
```
+ Declaring and initializing client.
```JavaScript
const client = new DynamoDBClient({ region: 'us-west-2' });
```
+ Creating objects for commands with the input payload.
```JavaScript
let input = {
     "TableName": "cars-demo",
     "KeyConditionExpression": "pk = :pk",
     "ExpressionAttributeValues": {
            ":pk":"CARS"
     }
}
const command = new QueryCommand(input);
```
+ Executing the SDK API.
```JavaScript
const response = await client.send(command);
```
If you are leveraging [Middleware stack](#middleware-stack), the middleware stack definition is additional procedures which developers would have to be careful.

#### Lambda node-module / Lambda layer size is far too high <a name="app-size"></a>
The [The size after installation](#size-of-installation) because of [modular architecture](#modular-arch) has in-fact reduced the size of final installed package. But currently [AWS Lambda functions](https://aws.amazon.com/lambda/) come with v2 SDK (pre-installed and available) if you would wish to use v3 SDK, you would have to import it and create a layer. Since your layer is the common set of packages used across multiple Lambda functions, you would have to install all your dependent client SDKs i.e. if you have a Lambda function which operates on DynamoDB, publishes to SNS, posts to SQS queue, uses SSM, operates on Cognito's functionalities then you would need to install all these packages (both high-level and low-level) which would enlarge your layer. The other way would be imported needed SDK clients for your Lambda function would would eventual increase your application's dependency size i.e. one Lambda function which is doing DynamoDB operation is installed with DynamoDB client, another Lambda which is publishing to SNS with it's client SDK but there would be SDK's dependent packages redundantly installed across multiple Lambda functions. 

![Lambda node-module / Lambda layer size is far too high](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432841951/fWnN6Zegaf.png)

But once we have the Lambda functions natively supporting SDK v3, it would be lighter in size.
 
#### Complicated JS SDK Documentation <a name="complicated-doc"></a>
AWS SDK v2 documentation was a simple documentation which provided all the supported APIs, it's input structure and response structure. With the [complex long procedures](#long-procedures) and TypeDoc generated documentation, it has turned into a hyperlinked 4 page document. Where you would have to navigate between 3-4 hyperlinked pages to understand one API with it's input structure and response structure.
<!--
{%twitter 1348612503102758912 %}
-->
![complicated-documentation ](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432843902/b1jY7G8Ez.png)

***
#### Resources to get started with SDK v3
API reference documentation : https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/index.html
Developer guide : https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/welcome.html
Self-paced workshop : https://github.com/aws-samples/aws-sdk-js-v3-workshop
Source code : https://github.com/aws/aws-sdk-js-v3/
Sample code : https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/javascriptv3/example_code
 