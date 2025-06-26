---
title: "Configuring inbound rules on SES"
datePublished: Sun Oct 24 2021 11:56:53 GMT+0000 (Coordinated Universal Time)
cuid: cl347vefk00iwtpnv0r09av3q
slug: configuring-inbound-rules-on-ses
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432876819/QBqjnBgOr.jpeg

---

[Amazon Simple Email Service (SES)](https://aws.amazon.com/ses/) not only addresses the use-case of *outbound emails* to users but also have *inbound emails* where you can receive emails from a designated recipient.

To get started with SES, you can refer to my recent post.
{%post awscommunity-asean/amazon-ses-and-everything-to-set-it-up-49c6 %}

#### Key takeaways from the blog
+ [Configuring Inbound rule set](#inbound-rulesets)
+ [Configuring action rules](#destination-rules)

#### Configuring Inbound rule set <a name="inbound-rulesets"></a>
From the AWS console, you can navigate to SES (new console UI) and from the navigation **Rule Sets** under **Email Receiving**. 
![New console](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432856164/Efx8tAA0yR.png)
You can click on the **Create rule set** button to enter a rule set name and create a new rule set.
![Create rule set](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432857981/zU_8Db5cI.png)
Once the rule set is created, the console displays the details of the newly created *rule set*.
![Created rule set](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432859450/c6b7tf1IF.png)

#### Configuring action rules <a name="action-rules"></a>
For a rule set, we can have multiple recipients with multiple destination actions configured i.e. you can have a rule set that *receives all the emails from an email recipient `reach@zachjonesnoel.com` and on receiving the email SES can invoke an Lambda function so that the further processing can be done*.
To create the above rule, click the **Create rule** button and the console takes you through a 4 step process to create a new rule.

##### Step 1 : Define rule settings
You can define certain rule settings such as name and to enable/disable the rule and also security settings to enable - 
+ *Transport Layer Security (TLS)* which defines if to accept or drop the incoming email messages if the email aren't sent over a secure connection. 
+ *Spam and virus scanning* which enables scanning the incoming email message with spam check and virus scanning if the email is found to be malicious, the email is dropped.
![Define rule settings](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432861047/FLIPnyV83.png)

##### Step 2 : Add recipient conditions
As mentioned earlier, we can have multiple recipients and this configuration can be done in this step. 
Note : The recipient have to belong to the domain owned by the same AWS account i.e the domain has to be verified on SES. To verify your domain, you can refer to the details from my [previous post](https://dev.to/awscommunity-asean/amazon-ses-and-everything-to-set-it-up-49c6#domain-verification).
![Add recipient conditions](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432862722/LW2H7BxNh.png)

Different types of recipients supported on SES.

| Supported type        | Wildcard | Example           | 
| ------------- |:-------------:|:-------------:|
| All email identities of the verified domains in the AWS account (default) | - | - |
| Specify email identity | user@verified-domain | `reach@demo.com` |
| All mails of a domain except sub-domains | `verified-domain` | `demo.com` |
| All mails of a specific sub-domain only | `sub-domain.verified-domain` | `mails.demo.com` |
| All mails of a all sub-domains | `.verified-domain` | `.demo.com` |

##### Step 3 : Add actions
For a rule, we can have multiple actions which are configured. The supported types of actions are - 
+ *Add header* 
![Add header](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432864257/Rc5oYXfry.png)
Whenever the email is received, it adds a custom header to the email.
+ *Bounce response*
![Bounce response](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432865749/Q0aumgbVI.png)
If the email has to be rejected, this action performs it by bouncing the email.
+ *AWS Lambda function invocation*
![AWS Lambda function invocation](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432867288/o0U3lC8eW.png)
Whenever the email is received, it can invoke a Lambda function which can do the further processing.
+ *Publish to SNS topic*
![Publish to SNS topic](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432868871/GpKwZZrE_.png)
The received email either with `base64` or `utf-8` encoding can publish the email the message to a SNS topic and all the subscribers of that topic would receive the email.
+ *Upload the email to S3 bucket*
![Upload the email to S3 bucket](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432870477/b3ijuMtxC.png)
The email content can be uploaded into a designated S3 bucket.
+ *Integration with Amazon WorkMail*
![Integration with Amazon WorkMail](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432872161/F5k2vb5Ah.png)
[Amazon WorkMail](https://aws.amazon.com/workmail/) has integration with SES to receive all the emails and from WorkMail app, you could access the emails.
+ *Stop rule set*
![Stop rule set](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432873774/-mm8ZrXWC.png)
This terminates the SES rule.

<!-- ![Add actions](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432875252/qVCC3lUHW.png)-->

##### Step 4 : Review
You can review the configurations and create the rule.

#### Pricing
The blog post explains about incoming messages and the pricing from [SES pricing](https://aws.amazon.com/ses/pricing/) for incoming emails is as explained - 
> An incoming mail chunk is 256 kilobytes (KB) of incoming data, including headers, message content (text and images), and attachments. When you use Amazon SES to receive email, you pay $0.09 for every 1,000 incoming mail chunks.

Additionally, the action which uses other AWS services costs with it's respective pricing.

#### Conclusion
SES a managed service from AWS makes it easier for application developers to integrate as per their business logic. The inbound rules helps addressing use-cases which requires receiving emails and processing from the email content. 