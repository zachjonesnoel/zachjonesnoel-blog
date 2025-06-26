---
title: "Streaming Responses via AWS Lambda"
seoTitle: "Streaming Responses via AWS Lambda"
seoDescription: "Learn how to use AWS Lambda's response streaming for improved performance with large payloads and real-time applications"
datePublished: Sun Jul 28 2024 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clz7x4ay3000g09l9cnj29qnz
slug: streaming-responses-via-aws-lambda
canonical: https://blog.theserverlessterminal.com/streaming-responses-via-aws-lambda
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722313480200/7e77ea1d-0eb2-4f6f-ab39-dc04f16baa3d.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1722313558394/206e9143-f726-468b-a6ca-5c95027883ec.png
tags: aws, apis, aws-lambda, lambda-function-urls, response-streaming

---

Originally posted on [The Serverless Terminal blog - Streaming Responses via AWS Lambda](https://blog.theserverlessterminal.com/streaming-responses-via-aws-lambda).

Building APIs on a Serverless stack includes using [AWS Lambda Function URLs](https://blog.theserverlessterminal.com/lambda-functions-over-urls) where the client traditionally requests the Lambda function and it responds after the Lambda function has completed execution.

![Traditional API request-response via Lambda Function URL](https://cdn.hashnode.com/res/hashnode/image/upload/v1722263777130/65ce6110-849f-4e6f-8a39-93160c7b3fc2.png align="center")

This pattern for large payloads induces latency that can negatively affect the application's performance.

In this blog, we will look at how Lambda function's Response Streaming would be helpful and identify when it would be ideal to use Response Streaming for your workloads.

## Lambda Response Streaming

AWS Lambda launched support for [response streaming](https://aws.amazon.com/blogs/compute/introducing-aws-lambda-response-streaming/) where this new pattern of API invocation allows the Lambda function to progressively send the response in chunks of data back to the client.

![Lambda functions' Response Streaming](https://cdn.hashnode.com/res/hashnode/image/upload/v1722264838781/c7d4cf5e-3f42-4f04-8d78-b1c589933623.png align="center")

### What does this pattern bring in?

* Improved time to first byte (TTFB) that improves the performance where the latency of the API requests is reduced and response is received as chunks of data as and when they are available.
    
* API response from Response Streaming supports larger payloads which have a soft limit of 20MB without the need to send it as a whole response.
    
* Enabling asynchronous data streaming which sends the API response when the data is available.
    

### Gotchas of this pattern

* **Runtime support**: AWS Lambda supports the capability of Response Streaming with only NodeJS runtime as part of the managed runtimes but with custom runtimes, you can leverage the Runtimes API to implement this.
    
* **Payload size**: The maximum response payload size supported is 20MB which is a soft limit you can request more via AWS Support. The first 6MB of the response is streaming without bandwidth constraints but as the data exceeds 6MB, you will have a maximum throughput of 2MB/s.
    
* **Network cost**: Similar to the payload limits, the first 6MB of the response is free and there would be data transfer charges for the total data streamed from the Lambda function.
    
* **Function timeout**: This goes without saying that you need to configure the right timeout for the Lambda function, the default 3s may be short but something beyond the 60s would also result in the API client terminating with the timeout error.
    
* **API endpoints**: The Response Streaming feature is available with only Lambda Function URL and expects the Function URL to use `ResponseStream` as the invocation mode. APIs are best managed with AWS API Gateway or Application Load Balancer (ALB) which doesn't support the response feature yet.
    

## Building AWS Lambda with Response Streaming

![We Learn Something New Everyday Here Learning GIF](https://media1.tenor.com/m/45kUy0Dpbi0AAAAd/we-learn-something-new-everyday-here-learning.gif align="center")

AWS Lambda uses the Node's Writable Stream API, so you can use `write()` from NodeJS to write into the stream, or Lambda functions introduced `pipeline()` which extends the capability of the stream from `util`.

### Node's `write()`

Response Streaming with `write()` would directly write into the Streams and whenever there is data in the stream, that would be sent to the client as a response. This method would expect you to manually handle the stream end.

```javascript
exports.handler = awslambda.streamifyResponse(
    async (event, responseStream, context) => {
        const httpResponseMetadata = {
            statusCode: 200,
            headers: {
                "Content-Type": "text/html",
            }
        };

        responseStream = awslambda.HttpResponseStream.from(responseStream, httpResponseMetadata);

        responseStream.write("<html>");
        responseStream.write("<p>Hello!</p>");

        responseStream.write("<h1>Let's start streaming response</h1>");
        await new Promise(r => setTimeout(r, 1000));
        responseStream.write("<h2>Serverless</h2>");
        await new Promise(r => setTimeout(r, 1000));
        responseStream.write("<h3>Is</h3>");
        await new Promise(r => setTimeout(r, 1000));
        responseStream.write("<h3>Way</h3>");
        await new Promise(r => setTimeout(r, 1000));
        responseStream.write("<h3>More</h3>");
        await new Promise(r => setTimeout(r, 1000));
        responseStream.write("<h3>Mature</h3>");
        await new Promise(r => setTimeout(r, 1000));
        responseStream.write("<p>DONE!</p>");
        responseStream.end();
    }
);
```

When you publish the above Lambda function and invoke the Function URL via a web browser, you can notice how the streaming responses are received.

![Streaming Response with NodeJS write()](https://cdn.hashnode.com/res/hashnode/image/upload/v1722273473238/4a609a62-3d62-4e06-9627-11774e0422ab.gif align="center")

### Lambda's `pipeline()`

Lambda function's `pipeline()` is a util extension that handles the end of the stream automatically. This is available in `util` package and use `pipeline(requestStream, responseStream)` to write data into the stream.

```javascript
import util from 'util';
import stream from 'stream';
const { Readable } = stream;
const pipeline = util.promisify(stream.pipeline);

export const handler = awslambda.streamifyResponse(async (event, responseStream, _context) => {
  const httpResponseMetadata = {
    statusCode: 200,
    headers: {
      "Content-Type": "text/html",
    }
  };

  responseStream = awslambda.HttpResponseStream.from(responseStream, httpResponseMetadata);
  let requestStream = Readable.from(Buffer.from(new Array(1024 * 1024).join('🚀')));
  await new Promise(r => setTimeout(r, 1000));
  requestStream = Readable.from(Buffer.from(new Array(1024 * 1024).join('⚡️')));
  await new Promise(r => setTimeout(r, 1000));
  requestStream = Readable.from(Buffer.from(new Array(1024 * 1024).join('🚀 Serverless is not dead!')));
  await pipeline(requestStream, responseStream);
});
```

Let's publish the Lambda function and invoke it via the Lambda Function URL.

![Response Streaming via pipeline()](https://cdn.hashnode.com/res/hashnode/image/upload/v1722275041437/ad4f0c78-bdf0-41ea-a767-741c5971367f.gif align="center")

### IaC to publish Lambda Function URL

While deploying and publishing the Lambda function, keep in mind to use the right `Timeout` and also `FunctionUrlConfig` with `InvokeMode` set as `RESPONSE_STREAM`.

```yaml
Resources:
  responseStreamingLambda:
    Type: AWS::Serverless::Function
    Properties:
      Timeout: 20
      Handler: index.handler
      Runtime: nodejs20.x
      Architectures:
        - x86_64
      FunctionUrlConfig:
        AuthType: NONE
        InvokeMode: RESPONSE_STREAM
```

## Response Streaming is the best fit in

It's important to understand when it would be ideal to use Response Streaming and some of the use cases include -

| Use Case | Why it's the best fit? |
| --- | --- |
| Real-time chat applications | Improved User Experience with low latency. |
| Streaming large files from S3 | Enabling downloading of large (&gt;6MB) S3 objects and receiving the data as and when available. |
| Server-side rending | When using SSR with incremental updates it improves the time to first byte (TTFB) while parts of the page render based on the data. |
| Streaming data from IoT devices | Enabling near real-time monitoring of data without delays and latency. |

## Wrap up!

Lambda functions' Response Streaming is ideal for web applications and monitoring systems where near real-time data is crucial. However, considering the limitations, you need to use Lambda Function URL with NodeJS runtime and be aware of constraints on cost and network bandwidth.