---
title: "Amazon SES and everything to set it up"
datePublished: Sun Sep 12 2021 19:20:19 GMT+0000 (Coordinated Universal Time)
cuid: cl347y7bi00j9tpnvg8e14qoy
slug: amazon-ses-and-everything-to-set-it-up
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652433007557/JoDWeMMtP.jpeg

---

[Amazon Simple Email Service (SES)](https://aws.amazon.com/ses/) is a AWS provisioned Email service which is available for cloud native applications to integrate and implementing use-cases which requires email messages to be sent.

#### Key takeaways from the blog -
+ [Features of SES](#features)
+ [Common use-cases with SES](#use-case)
+ [Setting up SES on your AWS Account](#setting-up)
+ [Working with SES](#working)

#### Features of SES <a name="features"></a>
+ Sending emails to targeted audience.
+ Analytics and monitoring of emails which are delivered, failed, bounced.
+ Ease of integration with CLI and SDK.
+ Verified the identities - email and domain which has permission to send emails.
+ Highly scalable.

#### Common use-cases with SES <a name="use-case"></a>
+ Targeted email messages based on events or programmatically invoked from your application in Lambda Functions, EC2 instances or containers.
+ Valid email templates which could be reused.
+ With [Amazon Pinpoint](https://aws.amazon.com/pinpoint/) and [Amazon Personalize](https://aws.amazon.com/personalize/) sending marketing emails with content customized to user's preference.
+ Bulk emails for easier communications with user specific data.

#### Setting up SES on your AWS Account <a name="setting-up"></a>
To get started with SES, you would have to move your SES account out of sandbox environment to production so that you can send emails to any email address. Note, if your SES account is in sandbox environment then you would be able to send emails only to verified identities only.
![Sandbox environment](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432996514/vSk39NS56.png)
To move your SES account out of the sandbox, you would have to raise a support request with AWS and the use-case would evaluated based on which the account is moved to production access.
You can rase the support directly by clicking on *Edit your account details* button. And you would be prompted with the below screen, where you would have provide the email type and use-case description.
![Production access support](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432998119/dDAS7jPQB.png)
Another way to do it is, from the support page raising the limit increase support. 
You can find the detailed steps to move SES out of Sandbox in the [link](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/request-production-access.html).
And once, the support request is approved, your SES account would be moved out of Sandbox environment. The sending quota would also be reflected for your account. 
![Production access](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432999815/umWn_l5ul.png)

SES uses verified identities to send emails. The two type of identities supported by SES are - 
+ **Email addresses** : a valid email address should be verified by Amazon's verification process where the email identity would be receiving an email with a verification link.
![Email addresses](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433001621/yIIU4I0g_.png)
Once your email is verified, you can send a test email from the console. 
![Send test email](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433003136/PPfclf12R.png)
 
+ **Domains** : <a name="domain-verification"></a> a particular domain would be verified if the domain is on Route53, it automatically adds up DNS records for sending emails. Otherwise, it will prompt you with the DNS records which has to be added to your domain.
![Domains](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433004709/o2ww3S7PG.png)
Similar to email identities, once your domain is added and verified, you can send a test message. Now since your domain is enabled with email you can choose any email address of that domain.
![Send test email](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433006191/etjwFJHT1.png)
Note, even though the domain is verified and the email address is also part of the same verified domain, the emails will have the from header as `amazonses.com`.
 
#### Working with SES <a name="working"></a>
For sending emails with SES you can use both AWS CLI and AWS SDK. SDK also supports various languages, in this blog we will be using NodeJS SDK. 

**Creation of SES Email templates**
To create a template on SES for that we would have to define a HTML and text content for the email along with parameters which would be sent when invoking a templated Email message. Create, update and delete of templates is only supported on CLI and SDK. Since this is a one-time execution, we would be doing it from the CLI.
`template.json`
```json
{
  "Template": {
    "TemplateName": "SampleTemplate",
    "SubjectPart": "Greetings, {{name}}!",
    "HtmlPart": "<h1>Hello {{name}},</h1><p>this is a sample email with parameters -<br/> Name : {{name}}<br/>Email: {{email}}<br/></p>",
    "TextPart": "Dear {{name}},\r\nYour email is {{email}}."
  }
}
```
```bash
aws ses create-template --cli-input-json file://template.json
```
API reference : [JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SES.html#createTemplate-property), [Python](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.create_template)


**Different ways of sending SES emails -** 
+ **Formatted email** : The email body is constructed and sent with `SendEmail` API. This supports raw text format and also formatted HTML based content.  Here we can define multiple recipients (`to`, `cc` and `bcc`) with email address separated with comma but this is limited to 50 recipients per email. 
```JavaScript
 var params = {
  Destination: { 
   CcAddresses: [
      "test@zachjonesnoel.com"
   ], 
   ToAddresses: [
      "example@zachjonesnoel.com", 
      "demo@zachjonesnoel.com"
   ]
  }, 
  Message: {
   Body: {
    Html: {
     Charset: "UTF-8", 
     Data: "<p>This message body contains <b>HTML</b> formatting. It can, for example, contain links like this one: <a class=\"ulink\" href=\"https://zachjonesnoel.com/\" target=\"_blank\">zachjonesnoel</a>.</p>"
    }, 
    Text: {
     Charset: "UTF-8", 
     Data: "This is the demo message."
    }
   }, 
   Subject: {
    Charset: "UTF-8", 
    Data: "Test email"
   }
  }, 
  ReplyToAddresses: [
    "ro-repy@zachjonesnoel.com", 
  ], 
  Source: "ro-repy@zachjonesnoel.com", 
  SourceArn: ""
 };
 await ses.sendEmail(params).promise()
```
API reference : [JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SES.html#sendEmail-property), [Python](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.send_email)
+ **Raw email** : This is more flexible than *Simple Email* and additionally we can also send attachments with this. The rest of the functionalities are same. The message is a base64 encoded string.
```JavaScript
  var params = {
  Destinations: [
     "test@zachjonesnoel.com"
  ], 
  FromArn: "arn:aws:ses:us-east-1:xxxxxxxxx:identity/ro-repy@zachjonesnoel.com", 
  RawMessage: {
   Data: <Binary String>
  }, 
  Source: "ro-repy@zachjonesnoel.com", 
  SourceArn: "arn:aws:ses:us-east-1:xxxxxxxxx:identity/ro-repy@zachjonesnoel.com"
 }
 await ses.sendRawEmail(params).promise()
```
API reference : [JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SES.html#sendRawEmail-property), [Python](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.send_raw_email)

+ **Templated email** : The template of email is defined and the respective template is also created with API or CLI. And whenever the emails are programmatically sent, you would have pass the parameters which are defined in the email template.

```JavaScript
var params= {
        "Source": "no-reply@zachjonesnoel.com",
        "Template": "SampleTemplate",
        "Destination": {
            "ToAddresses": [
                "test@zachjonesnoel.com"
            ]
        },
        "TemplateData": JSON.stringify({ "name": "zachjonesnoel", "email": "test@zachjonesnoel.com" })
    }
    await ses.sendTemplatedEmail(params).promise();
```
API reference : [JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SES.html#sendTemplatedEmail-property), [Python](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.send_templated_email)

+ **Bulk templated emails** : Bulk emails also use templates where the same template would be used for multiple emails where the parameters are specific to the users. This would send dedicated emails to each user.

```JavaScript
var params= {
        "Source": "no-reply@zachjonesnoel.com",
        "Template": "SampleTemplate",
        "Destinations": [
           {
                "Destination": {
                    "ToAddresses": [
                         "test@zachjonesnoel.com"
                    ]
                },
                "ReplacementTags": [
                    {
                         "Name": "name",
                         "Value": "zach"
                    },
                    {
                         "Name": "email",
                         "Value": "test@zachjonesnoel.com"
                    }
                ]
           },
           {
                "Destination": {
                    "ToAddresses": [
                         "demo@zachjonesnoel.com"
                    ]
                },
                "ReplacementTags": [
                    {
                         "Name": "name",
                         "Value": "jones"
                    },
                    {
                         "Name": "email",
                         "Value": "demo@zachjonesnoel.com"
                    }
                ]
           }
        ]
    }
    await ses.sendBulkTemplatedEmail(params).promise();
```
API reference : [JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SES.html#sendBulkTemplatedEmail-property), [Python](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ses.html#SES.Client.send_bulk_templated_email)

#### Conclusion
AWS SES provides simplistic integration methods where application developers can leverage the SDK based APIs for `sendRawEmail`, `sendEmail`, `sendTemplatedEmail`, `sendBulkTemplatedEmail` available in JavaScript, Python, PHP, Go, Java and .NET. SES provides APIs where it eases sending emails from a verified email or domain so the authenticity of the sender is verified. This also provides good feasibility with HTML based templates and formatting of email content.
Before getting started, don't forget to look into [SES pricing](https://aws.amazon.com/ses/pricing/).

This sample requires identities to be verified on AWS before sending emails, so haven't provided GitHub repository references. If you encounter any issues, don't hesitate to ping me on [Twitter](https://twitter.com/ZachjNOEL) or [LinkedIn](https://www.linkedin.com/in/jones-zachariah-noel-n).