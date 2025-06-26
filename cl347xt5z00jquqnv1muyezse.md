---
title: "Securing access to S3 bucket"
datePublished: Thu Sep 23 2021 16:16:11 GMT+0000 (Coordinated Universal Time)
cuid: cl347xt5z00jquqnv1muyezse
slug: securing-access-to-s3-bucket
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432989319/9Hd40pxcs.jpeg

---

[Amazon S3](https://aws.amazon.com/s3/) has become one of the most popular object storage on cloud. This is available for various use-cases, to know more about why S3 is popular you can check my old post - 
{% link https://dev.to/zachjonesnoel/aws-s3-and-it-s-popularity-4ici %}

#### Key takeaways from this blog post 
+ [Need of securing your S3 bucket](#need-for-securing)
+ [Ways to secure your S3 bucket](#ways-to-secure)

#### Need of securing your S3 bucket <a name="need-for-securing"></a>
AWS S3 offers storage of objects - images, documents, videos, audios, executable files, source code and many more types of files. The objects in S3 which could be of any of the above mentioned type would require a managed/controlled access. 
Eg. If an image of your passport is stored on S3, since that is an official document, you would like it to be accessible only to you or your trusted sources.
The best way solution in this scenario is making your bucket **private**.
![private buckets](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432971467/IYDvRx1kq.png)
In a public bucket, if the path or the pattern of path is exposed or easily traceable then not only the S3 object which was shared is available over the internet but also all the contents of your S3 bucket.
Data insecurity is once aspect also if the S3 bucket is not secure, it can add up to your billing. As per S3's pricing model, you don't get billed only for the storage but also for the number of API calls from AWS SDK, CLI or console. For any of the actions `GET` or `PUT`, it is always the recommended best practice that SDK or CLI is authenticated with credentials such as *SecretKey & AccessKey* or *IAM role* for a Cognito authenticated user. Another best practice is **least privilege access** so that the *user/assume role* would be allowed to `GET` or `PUT` for a limited session.

#### Ways to secure your S3 bucket <a name="ways-to-secure"></a>
The different ways to ensure your S3 bucket is secure are - 
+ [Making your bucket is *private*.](#private-bucket)
+ [Using *S3 bucket policies*.](#bucket-policies)
+ [Using *S3 ACLs*.](#s3-acl)
+ [Using *assumed IAM roles* which has access to the bucket for read/write.](#assume-iam-role)
+ [Using *PreSigned URLs*.](#presigned-urls)
+ [Using *CloudFront Distribution* enabled for S3 bucket origin both for S3 objects and S3 static hosted websites.](#cloudfront-distribution)
+ [Ensuring the IAM user has the *least privilege* for performing actions on the S3 bucket.](#least-privilege-iam-user)
+ [For programmatic access using *IAM user's credentials - access key and secret key*.](#iam-programmatic-access)
+ [Using *Multi-factor authentication (MFA)* for delete operations.](#mfa-delete)
+ [Enabling *CloudTrail* and *Server access logging* for your bucket.](#enable-cloudtrail-and-logging)
+ [Using *S3 Access points*.](#access-points)
+ [Using *Access Analyzer for S3* to review public buckets.](#access-analyzer)

#### Making your bucket is *private*<a name="private-bucket"></a>
As mentioned in the need of securing, the first step to make your bucket private. To ease the process, AWS has a global setting for making the S3 buckets in your account as *private buckets*.
![Block Public Access settings for this account](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432973030/hxZ7eN4TU.png)
This setting will block all the public access and you would require AWS credentials to access and invoke S3 APIs via CLI, SDK or S3 REST APIs.
Based on your account's global setting, every you create a new bucket, it defaults to *private*.
![Create bucket](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432974790/V2cpaiRaN.png)
But in-case if you would want your buckets to be public, you can unselect *Block all public access*. And AWS expects prompts up a confirmation so that you can double check and confirm before making it public.  
You can know more about blocking public access on S3 from the [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html).

#### Using *S3 bucket policies*<a name="bucket-policies"></a>
S3 bucket policies are similar to IAM policies but the scope of access control is limited to the specific AWS S3 bucket resource only. Were we can set access to specific users with specific S3 actions and also control based on more conditional parameters such as *Source IP*, *VPC*, *Access Point ARN* and more.
Full access to S3 bucket
```json
// Full access to S3 bucket
{
  "Id": "Policy1632299568329",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1632299332911",
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<<bucketname>>",
      "Principal": "*"
    },
  ]
}
```
Access to one IAM user with specific PUT and GET actions on S3
```json
// Access to one IAM user with specific PUT and GET actions on S3
{
  "Id": "Policy1632299568329",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1632299444454",
      "Action": [
        "s3:ListBucket",
        "s3:ListBucketMultipartUploads",
        "s3:ListBucketVersions",
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:PutObjectLegalHold",
        "s3:PutObjectRetention",
        "s3:PutObjectTagging",
        "s3:PutObjectVersionAcl",
        "s3:PutObjectVersionTagging"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<<bucket_name>>",
      "Principal": {
        "AWS": [
          "arn:aws:iam::<<account_id>>:user/<<iam_username>>"
        ]
      }
    }
  ]
}
```
Full access to a key in S3 bucket when accessing the IAM Role when the condition satisfies the source IP address to be 192.168.2.1 and S3 prefix as key1/key2
```json
/* Full access to a key in S3 bucket when accessing the IAM Role 
when the condition satisfies the source IP address to be 192.168.2.1 and
S3 prefix as key1/key2 */
{
  "Id": "Policy1632299568329",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1632305278967",
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<<bucket_name>>/<<key>>/<<key>>",
      "Condition": {
        "ArnEquals": {
          "aws:SourceIp": "192.168.2.1"
        },
        "StringEquals": {
          "s3:prefix": "key1/key2"
        }
      },
      "Principal": {
        "AWS": [
          "arn:aws:iam::<<acount_id>>:role/<<iam_rolename>>"
        ]
      }
    }
  ]
}
```
#### Using *S3 ACLs* <a name="s3-acl"></a>
Configuring S3 Access control list (ACL) is another good way to control the access to a specific S3 bucket. Where you can set the permissions as *Read/Write* for the objects and bucket permissions. By default *Read and Write* is set for the bucket owner (i.e. the AWS account owner). And no access to AWS authenticated users, public access. 
![S3 ACL](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432976541/8CLC3bQhY.png)
The default settings can be changed to ensure you are providing the right access to the needed user/identity. 
![Edit S3 ACL](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432978188/alSQNIs_h.png)
Additionally, you can provide access to another AWS account or an account which is part of your AWS Organization with the needed *Read and Write* permission.

#### Using *assumed IAM roles* which has access to the bucket for read/write  <a name="assume-iam-role"></a>
Whenever using any of the AWS services programmatically or from the console AWS creates an *assume IAM role* which grants access to that particular AWS service actions for a specific AWS resource. In this case, the IAM policy would allow that role to perform needed AWS S3 actions only to a designated S3 bucket and this resource can even be further access controlled to a specific path/key in that AWS S3 bucket. 

IAM role with policy to a specific S3 key in a bucket with only `PutObject`, `PutObjectAcl` and `GetObject` actions
```json
{
  "Id": "Policy1632299568329",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1632305278967",
      "Action": [
          "s3:PutObject",
          "s3:PutObjectAcl",
          "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<<bucket_name>>/<<key>>/<<key>>",
      "Principal": {
        "AWS": [
          "arn:aws:iam::<<acount_id>>:role/<<iam_rolename>>"
        ]
      }
    }
  ]
}
```
#### Using *PreSigned URLs* <a name="presigned-urls"></a>
PreSigned URLs are the URLs which are authenticated with certain pre-defined headers. With a *PreSigned URL*, we can add the headers or URL parameters which is used for authenticating and authorizing the URL to access an S3 object and also making sure this authorization is limited with an expiry time attached to it. The *PreSigned URLs* we can access an object in a private S3 bucket for a limited time over the internet with the privileges of viewing and downloading the object. 
```JavaScript
let s3 = new AWS.S3();
let getPreSignedUrlParams = {
    Bucket: <<bucket_name>>,
    Key: <<key>>
}
let presignedURL = await s3.getSignedUrl(getPreSignedUrlParams).promise();
let putPreSignedUrlObject = {
    Bucket: <<bucket_name>>,
    Fields: {
       key: <<key>>
    }
}
let presignedURL = await s3.createPresignedPost(params).promise();
```
You can know more about PreSigned URLs from the [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)

#### Using *CloudFront Distribution* enabled for S3 bucket origin both for S3 objects and S3 static hosted websites <a name="cloudfront-distribution"></a>
CloudFront being a CDN network works securely and also is available cached in the EdgeLocations making it faster for end users to access S3 objects. A CloudFront origin can have multiple different origins, and one of it is S3.  
![CloudFront origin](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432979681/MMrJnLx1f.png)
With a CloudFront origin, we can refer to a S3 hosted website and also S3 bucket as a whole. The access moderation is prioritized to make sure developers can leverage this to make their S3 objects fetched from a CDN and also making sure the bucket is private. 
![CloudFront Origin](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432981142/C1GW_KzGL.png)
We can ensure the bucket can be restricted to CloudFront only and if anyone uses S3 URL it would error with `AccessDenied`. And the access from CloudFront also is authorized with a Identity access configuration.
Additionally, we can enable Origin Shield which helps in reducing the operational cost and making the S3 object highly available with better network performance.
![CloudFront addition settings](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432982674/bi8tD8QAg.png)
We can ensure the timeout seconds for the requests made via CloudFront.
This is one of the reasons why web-hosting from Amplify is performed, the secure method is over CloudFront.
You can refer to the a detailed blog about it - 
{% post https://dev.to/zachjonesnoel/amplify-your-web-hosting-3g4a %}

#### Ensuring the IAM user has the *least privilege* for performing actions on the S3 bucket <a name="least-privilege-iam-user"></a>
The concept of *least privilege* is to ensure that IAM users are not authorized to make a lot of changes which definitely raises the security and accountability. Similar to the *least-privilege* to any of the AWS services/resources, for S3 also the same holds good. If the users need more authorization for creating ACLs, changing ACLs, permissions, deleting S3 objects then there is a lower chance of things being played around with. Even as administrator access, it is good to drill down the specific access authorization for the needed operations only.

#### For programmatic access using *IAM user's credentials - access key and secret key* <a name="iam-programmatic-access"></a> 
The programmatic access via CLI or SDK should be authorize with the IAM user's credentials i.e the *access key and secret key*. From the recommended best practices, it is best to rotate the credentials in a timely format like every 60 days so that even if the credentials are compromised, we can revoke that access and create new ones. Again the IAM user should be provided with *least privileges*.

#### Using *Multi-factor authentication (MFA)* for delete operations <a name="mfa-delete"></a>
As administrators or IAM Users who have access to delete an object should be enabled with *Multi-factor authentication (MFA) delete* so that any deletion of S3 object is verified with 2 levels or authorization. This could prevent deletion of some important items, images, documents which are present in S3. 
This is one feature which is good to have across AWS for AWS resource deletion.

#### Enabling *CloudTrail* and *Server access logging* for your bucket <a name="enable-cloudtrail-and-logging"></a>
*Server access logging* is an add-on feature from S3 which logs all the access and requests which are made to the specific S3 bucket. When enabled, you would have to choose a destination S3 bucket where all the logs are going to be written into.
![Server access logging and CloudTrail](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432984234/DDbPIjQrRl.png)
[CloudTrail](https://aws.amazon.com/cloudtrail/) is one of the important services for governance and monitoring your AWS resources. With *CloudTrail* we can keep a log of all the APIs which are getting involved and the meta data of the invocation such as which IAM user/role invoked, what was the IP address that invoked the API. 
These logs from both *Server access logging* and *CloudTrail* can be stored on S3 bucket which can be analyzed from Athena to get better querying experience in terms of who / what / when / where the API was invoked. 

#### Using *S3 Access points* <a name="access-points"></a>
*S3 Access points* are the public named network endpoints which are used to serve the requests to S3 bucket such as `GetObject` and `PutObject`. Whenever any *access point* is configured, it is from a VPC network so that internal redirection to S3 happens over AWS' internal VPC network which is not only one of the secure ways to access but also it is faster for the access point to resolve the S3 object. You can create multiple *access points* and use the ARN to be referred from an EC2 instance or Lambda function.
You can read more about *S3 Access points* from [AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-points-policies.html?icmpid=docs_s3_hp_access_points_page).

#### Using *Access Analyzer for S3* to review public buckets <a name="access-analyzer"></a>
For enabling *Access Analyzer for S3*, the *IAM Access Analyzer* has to be created.
![IAM Access Analyzer](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432985646/q0yTaFnRl.png)
Once this is done, you can navigate to S3 console and go to *Access Analyzer for S3* menu. And AWS prepares a report based on the IAM Access Analyzer that was created for a specific region. This will provide you a report of all the buckets which have public access enabled and also the buckets which has S3 Policies and ACLs where other AWS accounts can access.![Access Analyzer for S3](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432987886/ilrWaREyq.png)
Based on this report, you can take the needed actions as making the public buckets private and also modifying the ACL and S3 bucket policies.

#### Conclusion
S3 being most of the popular cloud storage service, the need to make is secure is every Cloud Architect's and Security Manager's concern. AWS to address these concerns provides different ways which support the Security pillar of a **Well-Architected System**. 
You can also refer to one of the recent AWS blogs - [Top 10 security best practices for securing data in Amazon S3](https://aws.amazon.com/blogs/security/top-10-security-best-practices-for-securing-data-in-amazon-s3/)
