---
title: "Modern apps going Cognito"
datePublished: Sun May 16 2021 09:42:25 GMT+0000 (Coordinated Universal Time)
cuid: cl2vo38fc033at4nv1ksuauwl
slug: modern-apps-going-cognito
canonical: https://dev.to/zachjonesnoel/modern-apps-going-cognito-1o77
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1651915919940/U7XKmVDJh.jpeg

---

Modern mobile and web applications have a variety of options for implementing user management such as authorization, authentication, password management etc. AWS has the perfect service for AWS enthusiastic to incorporate in their applications both on the front-end and back-end. 

#### What is AWS Cognito?
[AWS Cognito](https://aws.amazon.com/cognito/) is a fully managed service which does user management such as sign-in, sign-up and also integrated with other social media identity providers such as Google, Apple, Amazon, Facebook. It also eases the enterprise identity providers via SAML 2.0 and OpenID Connect. This provides access to AWS resources with IAM Roles & Policies.

Offerings by AWS Cognito -
+ AWS Cognito User Pool
+ AWS Cognito Identity Pool

#### Why are modern apps choosing Cognito?
From the traditional systems, we have had challenges to get a basic sign-up done from our database design to implementing APIs keeping in mind the encryption of user data etc. and also the complete API integration on the front-end. This complete process is eased up with AWS Cognito. AWS Cognito makes it easy to integrate for modern web and mobile applications via AWS Amplify. The complete implementation of user sign-up to sign-in takes a couple of minutes. Amplify's SDK makes it easier for a front-end developer to even add the authentication modules and UI to their application even lot faster with hosted UI's by Cognito. And all this in the free tier for upto 50,000 MAU. 
![AWS Cognito](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915903441/u-jJksfEc.png)

#### Cognito User Pool
Cognito User Pools are the managed directory of users, whose data is entirely on AWS's storage with attributes for the users being configurable and the users can set which are the attributes needed for sign-up and also ease the process of sign-in with both email and mobile numbers. 
![User attributes](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915905006/jSaVx7riW.png)
These can also be configured to look up to a federated identity with Social media and enterprise identities. 
![Federated identities](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915906537/BcL7PKoGA.png)
We have the liberty to set the password policy needed for the application and also restrict user sign-up where we have an application which invites the user with a temporary password.
![Password policy](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915908012/5fBxANypX.png)

Cognito User Pools allows customization of messages both email and SMS based on different events, configurable from the Cognito console and also with Lambda triggers for programmatic change of message content. The beauty of Cognito is, it closely integrates with SES for email messages and SNS for SMS messages. 
![Message customization](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915909591/4LSNFA5iU.png)

Cognito has app client configuration in place for any of setting the expiry time of tokens for authentication and authorization. Also we can enable Lambda triggers for custom authentication flow if you need a very specific flow for your application.
![App clients](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915911166/sdvbRGMtS.png)
With the custom authentication and Lambda triggers we can totally customize or build an additional business logic.
![Lambda triggers](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915913230/Eq376ZePs.png)
Eg. You have a multi-tenant application and you have to check if the user has been invited to any of the instance in your application, you can do it with *Pre sign-up* trigger which gets invoked when the user tries to sign up and you can error out saying you have not been invited to any of the instance. 
For auditing purpose, you want to keep track of the user session, you can use the context details which Cognito provides such as IP address, device details etc on *Post authentication* trigger.

When integrating with Cognito, you need not develop the complete UI for the same, Cognito provides hosted UIs which is basically a web-page created by AWS and hosted on a AWS managed/custom domain which takes care of the complete authentication flow and also has scope of defining the UI based on our needs and app branding.
![Hosted UIs](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915915119/sMa61j7rL.png)
#### Cognito Identity Pool
Identity pools are the directory of users who have been authorized to access AWS resources. Here we can also enable unauthenticated access which is a temporary access to the users who have been verified externally or the guest users for the application. 
![Unauthenticated access](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915916571/h4em6_KSJ.png)
Any identity either authenticated or unauthenticated has a mapped IAM Role which defines what the user can access and what the user does not have access to. 
Identity Pools can be integrated with different authentication providers for access such as - Cognito User Pool, Social media and Enterprise federated identity providers
![Authentication providers](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915918381/0S-zfEqBR.png)

#### Cognito User Pool vs Identity Pool
Cognito User Pools are designed for user authentication where we verify the user's identity either with the directory of users created on Cognito User Pool or with Social media and other enterprise identity providers. 
Cognito Identity Pools are for authorization for the user to access AWS resources such as AWS API Gateway, S3, AWS AppSync and other services via an IAM Role and IAM Policy. These are for temporary access. 

#### Auth with Amplify
AWS Amplify SDK provides a smooth integration to AWS Cognito both User Pool and Identity Pool for web and mobile applications. Amplify SDK provides libraries in [JavaScript](https://docs.amplify.aws/lib/auth/getting-started/q/platform/js), [Android](https://docs.amplify.aws/lib/auth/getting-started/q/platform/android), [iOS ](https://docs.amplify.aws/lib/auth/getting-started/q/platform/ios) and [Flutter](https://docs.amplify.aws/lib/auth/getting-started/q/platform/flutter). With the Amplify CLI you can setup authentication for your project in just couple of minutes as it runs you through complete setup with set of questionnaire with the command -
```
amplify add auth
```
Amplify provides ready-to-use components for importing and using in the project code so that the component takes care of authentication flow.
```
<template>
  <amplify-authenticator>
    <div>
      My App
      <amplify-sign-out></amplify-sign-out>
    </div>
  </amplify-authenticator>
</template>
```
And also the SDK provides various APIs for sign-in, sign-up, forgot password etc.
```javascript
import { Auth } from 'aws-amplify';

async function signIn() {
    try {
        const user = await Auth.signIn(username, password);
    } catch (error) {
        console.log('error signing in', error);
    }
}
```

#### Conlusion
AWS Cognito's User Pool and Identity Pool provides the best in store authentication and authorization features for modern web and mobile applications to use. This provides fascinating ways to integrate to your application architecture for a secure user management and also eases UI development with hosted UIs and also with Amplify's SDK components. So now, you can literally get the authentication module of your application up and running in just a couple of minutes.

