---
title: "Message Customization on Cognito"
datePublished: Sat May 22 2021 09:24:04 GMT+0000 (Coordinated Universal Time)
cuid: cl2vo3vmh033bt4nv8wxyfb83
slug: message-customization-on-cognito
canonical: https://dev.to/awscommunity-asean/message-customization-on-cognito-2bme
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1651915950558/Q8vveHgyL.jpeg
tags: aws, email, serverless, messaging

---

AWS Cognito has become one of the most adapted ways to authenticate your Modern applications [(Click here to know more)](https://dev.to/zachjonesnoel/modern-apps-going-cognito-1o77). The emails and SMS messages involved in the process of different actions, the modern applications need their own verbiage and also they have to be customization for each action. 
Eg. For sign-up the application would have to send an email like - 
```
Thank you for signing up. You can login with the <link>, you can download the app from the <link>.
```
For forgot password, the application would have to send an OTP or link for resetting the password.

And many more use cases where the message would have to be customized in various way. 

#### Ways to configure message customization
+ AWS Console
+ AWS CLI 
+ Custom message via Lambda function trigger

#### AWS Console
In AWS Console, when can navigate to AWS Cognito User Pool and select the user pool you want to edit and navigate to the "Message Customization" tab. 
![SES Configuration](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915944108/f83d_ccZn.png)
AWS gives the liberty of changing the sender email from the default "no-reply@verificationemail.com" to your own SES verified email identity, of-course this email identity has to be moved out of Sandbox environment. The selected email identity has to provide Cognito access to send emails, the respective IAM policy is required which is auto-created when you do from the web console.
```json
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "stmnt1234567891234",
            "Effect": "Allow",
            "Principal": {
                "Service": "cognito-idp.amazonaws.com"
            },
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "<your SES identity ARN>"
        }
    ]
}
```
Once the permission is setup, you can customize the message which has to be delivered to the user. You can customize both email and SMS body. Email body is supported with HTML tags to format and beautify the email body.
![Message customization](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915945653/y3HoxKvQq.png)
Cognito provides placeholders for username, password, code, link which are used to fill in the user specific values. 

#### AWS CLI
AWS CLI makes it easier for terminal fond developers where the complete configuration is done with a single command. With the terminal/CLI you would have to specify the details as a JSON parameters. CLI provides a *update-user-pool* command for *cognito-idp* which could be used to update the custom messages.
```
aws cognito-idp update-user-pool 
--user-pool-id us-east-1_xxxxxxxxx 
--email-verification-subject "Your temporary password" 
--email-verification-message "<p>Your username is {username} and temporary password is {####}. </p>"
--email-configuration SourceArn=<YourSESIdentityARN>,ReplyToEmailAddress=<VerifiedSESIdentity>,EmailSendingAccount="DEVELOPER",From=<VerifiedSESIdentity>
```

#### Custom message via Lambda function trigger
Lambda function could be added as a trigger to handle all custom message which gives you the total liberty to customize each and every message both email and SMS that is sent to the user.
![Custom message trigger](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915947290/tToCT5uwo.png)
Select your Lambda function and save the setting. This setting will trigger the selected Lambda function every time when Cognito is trying to send a message to the user. 

Different event triggers which gets invoked -
+ CustomMessage_SignUp - When a new user signs-up and Cognito will send out a verification email/SMS to verify the identity.
+ CustomMessage_AdminCreateUser - When the user is created with adminUserCreate() API from Cognito, the invitation email with temporary password is sent.
+ CustomMessage_ResendCode - When the user requests for the code to be resent.
+ CustomMessage_ForgotPassword - When user initiates a forgot password, a verification code would be sent over email/SMS to the verified identity.
+ CustomMessage_UpdateUserAttribute - Whenever the user's attribute is updated.
+ CustomMessage_VerifyUserAttribute - When the attribute (email address or mobile number) is changed, it has to be verified.
+ CustomMessage_Authentication - For MFA authentication code email and SMS message.

Sample event for Lambda function
```json
{
  "version": "1",
  "region": "us-east-1",
  "userPoolId": "us-east-1_xxxxxx",
  "userName": "xxxx@xxxx.com",
  "callerContext": {
    "awsSdkVersion": "aws-sdk-xxxx-2.22.1",
    "clientId": "xxxxxxxxxxxxxxxxx"
  },
  "triggerSource": "CustomMessage_ForgotPassword",
  "request": {
    "userAttributes": {
      "sub": "xxxxxxx",
      "cognito:user_status": "CONFIRMED",
      "email_verified": "true",
      "phone_number_verified": "true",
      "phone_number": "+1234567890",
      "preferred_username": "+1234567890",
      "email": "xxxxxxx@xxxxxxx.com"
    },
    "codeParameter": "{####}",
    "linkParameter": "{##Click Here##}",
    "usernameParameter": "xxxxx"
  },
  "response": {
    "smsMessage": null,
    "emailMessage": null,
    "emailSubject": null
  }
}
```

Sample Lambda function code 
```javascript
'use strict';

const generate_email_body = (emailBody) => `
    <html>
        <body>
            <table align="center"  cellpadding="0" cellspacing="0" width="600" >
                <tr>
                    <td bgcolor="#ffffff" style="padding: 40px 0 30px 0;"><img src="" alt="Logo"  height="230" style="display: block;"></td>
                </tr>
                <tr>
                    <td bgcolor="#ffffff"><p style="margin: 0;">${emailBody}</p></td>
                </tr>
                <tr>
                    <td bgcolor="#ffffff" style="font-weight: 500; font-size: 11px"><p>© Copyright ${new Date().getFullYear()}. All Rights Reserved.</p></td>
                </tr>
            </table>
        </body>
    </html>
`

const sign_up_message = async(event) => {
    let email = event.request.usernameParameter;
    let code = event.request.codeParameter;
    event.response = {
        emailSubject: "Confirm your sign up",
        emailMessage: generate_email_body("<p>Your username is " + email + " and password is " + code + "</p>")
    }
    return event
}
const admin_create_user_message = async(event) => {
    let email = event.request.usernameParameter;
    let code = event.request.codeParameter;
    event.response = {
        emailSubject: "Your temporary password",
        emailMessage: generate_email_body("<p>Your username is " + email + " and password is " + code + "</p>")
    }
    return event
}
const resend_code_message = async(event) => {
    let email = event.request.usernameParameter;
    let code = event.request.codeParameter;
    event.response = {
        emailSubject: "Resend code",
        emailMessage: generate_email_body("<p>Your username is " + email + " and code is " + code + "</p>")
    }
    return event
}
const forgot_password = async(event) => {
    let email = event.request.usernameParameter;
    let code = event.request.codeParameter;
    event.response = {
        emailSubject: "Forgot password",
        emailMessage: generate_email_body("<p>Your forgot password code is " + code + "</p>")
    }
    return event
}
const update_user_attribute_message = async(event) => {
    let email = event.request.usernameParameter;
    let code = event.request.codeParameter;
    event.response = {
        emailSubject: "User updated",
        emailMessage: generate_email_body("<p>Your username is " + email + "</p>")
    }
    return event
}
const verify_user_attribute = async(event) => {
    let email = event.request.usernameParameter;
    let code = event.request.codeParameter;
    event.response = {
        emailSubject: "Verify user attribute",
        emailMessage: generate_email_body("<p>Your username is " + email + "</p>")
    }
    return event
}
const authenitcation_message = async(event) => {
    let email = event.request.usernameParameter;
    let code = event.request.codeParameter;
    event.response = {
        emailSubject: "MFA Authenitcation",
        emailMessage: generate_email_body("<p>Your username is " + email + " and code is " + code + "</p>")
    }
    return event
}
exports.handler = async(event) => {
    switch (event.triggerSource) {
        case "CustomMessage_SignUp": //Sign-up trigger whenever a new user signs him/herself up.
            return sign_up_message(event)
        case "CustomMessage_AdminCreateUser": //When the user is created with adminCreateUser() API
            return admin_create_user_message(event)
        case "CustomMessage_ResendCode": //When user requests the code again.
            return resend_code_message(event)
        case "CustomMessage_ForgotPassword": //Forgot password request initiated by user
            return forgot_password(event)
        case "CustomMessage_UpdateUserAttribute": //Whenever the user attributes are updated
            return update_user_attribute_message(event)
        case "CustomMessage_VerifyUserAttribute": //Verify mobile number/email
            return verify_user_attribute(event)
        case "CustomMessage_Authentication": //MFA authenitcation code.
            return authenitcation_message(event)
        default:
            return event

    }
};

```

To test it out, you can head to console, and create a user with send invitation checked. On creating a user, the email address used would get the below email.
![Invite email](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915949133/Z52Z2ahwN.png)