---
title: "Custom Domain on Amplify with Route53"
datePublished: Mon Sep 06 2021 17:57:01 GMT+0000 (Coordinated Universal Time)
cuid: cl347ypok00jsuqnv9fsfh2gc
slug: custom-domain-on-amplify-with-route53
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652433031370/__wMM2GvD.jpeg

---

In my previous post, about [AWS Amplify's web hosting feature](https://docs.amplify.aws/start/getting-started/hosting/q/integration/vue/#add-hosting-to-your-app) the Amplify app was hosted with a production mode (HTTPS over CloudFront) is using Amplify's domain `amplifyapp.com`. 
{% post zachjonesnoel/amplify-your-web-hosting-3g4a %}
And in this post, we will use Route53 to define custom domains for Amplify hosted web apps. 

#### Key-takeways from the blog
+ Setting up Route53 hosted zone.
+ Mapping Route53 hosted zone to Amplify app.
+ Restricted access to web app.
+ Monitoring the web app.

#### Setting up Route53 hosted zone
[Route53](https://aws.amazon.com/route53/) is a managed DNS service from AWS. It resolves domain names such as `www.zachjonesnoel.com` to it's equivalent IP address from the domain provider. Route53 also offers traffic management with various routing policies.  

To create a hosted zone, navigate to AWS Console -> `Route53` -> `Hosted Zones` and click on the create button. You would be prompted with the below screen.
![Creating a hosted zone](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433015118/qOxaGCGEk.png)
You can choose public or private domains (have to be part of AWS VPC network) which are registered. Once the hosted zone is created, you can configure the rest of the name servers on your domain provider and also add the DNS records for redirection, email text records and many more. (For all security reasons will not be sharing the screenshots of how the records dashboard looks)
![Creating records](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433016750/vkDe3dUSq.png)

#### Mapping Route53 hosted zone to Amplify app
Here, I would be using a web app which is already hosted on Amplify's web hosting, you can refer to the linked blog post [Amplify your web hosting](https://dev.to/zachjonesnoel/amplify-your-web-hosting-3g4a) to create one.

In Amplify console, select the app which needs a custom domain and navigate to the `Domain management` menu.
![Domain management](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433018266/ZNimJs88k.png)
In `Domain management` dashboard, click on the `Create domain` button and you would be prompted with the below screen.
![Create domain](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433020157/H2xg_iou-.png)
The domain names are the registered domains which have their hosted zone defined on Route53. 
When the domain is created, you can manage them and select the available branch from your Git repository. 
![Update domain](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433021988/AjREZjYb0L.png)
You can not only define the domain but also manage other sub-domain. For eg. the web app has 3 stages -  `dev`, `prod` and `uat` so your web-app should be staged and also the respective branches should be hosted on those specific sub-domain. 

| Stage | Custom domain         | Amplify domain                  |
|:----: |:---------------------:| :------------------------------:|
| prod  | `www.zachjonesnoel.com` | `main.<app_id>.amplifyapp.com`    |
| dev   | `dev.zachjonesnoel.com` | `dev.<app_id>.amplifyapp.com`     |
| uat   | `uat.zachjonesnoel.com` | `uat.<app_id>.amplifyapp.com`     |

For a Route53 managed domain, the DNS records are added automatically by AWS but if your domain is managed by other domain provider, then you can view the text records details so that you can add them on your domain provider.  
![image](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433023983/OWNk8kzgo.png)
As the domain is served over HTTPS, you can view the certificates for the same.
 
The created hosted zone can be shared over different Amplify apps where you have a use-case in which, each web-app that is hosted is redirected to a specific sub-domain. 

| App | Custom domain         | Amplify domain                  |
|:----: |:---------------------:| :------------------------------:|
| App1 | `app1.example.com` | `app1.<app1_app_id>.amplifyapp.com`    |
| App2 | `app2.example.com` | `app2.<app2_app_id>.amplifyapp.com`    |
| App3 | `app3.example.com` | `app3.<app3_app_id>.amplifyapp.com`  |

#### Restricted access to web app
With multiple branches and sub-domains, in a real-world scenario it is ideal that `prod` is only publicly available whereas `dev` and `uat` are more access controlled for only developers or testers. Amplify facilitates with the same. 
![Restricted access](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433025459/p81bx8sdQ.png)
You can specify, which of the branch is a public facing web-app and which are access controlled so that users which the specific credentials are able to access it over the public internet. 

#### Redirection and Rewrites
Since the custom domains are treated as re-directional URLs to Amazon CloudFront distribution, Amplify web hosted apps create a CloudFront distribution which is directing to source of the S3 bucket where the actual built web application files reside. These CloudFront distributional URLs are the URL endpoints referred in the DNS TXT record for redirecting the custom domain to Amplify hosted app.
![Redirection and Rewrites](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433026891/l3XTo95eu.png)
CloudFront distribution endpoints are one of the secure way to expose any endpoints of Amazon which servers a web application as [Web Application Firewall (WAF)](https://aws.amazon.com/waf/) is available which ensures the web app is secure and is protected from common web attacks - traffic spike bots, SQL injection, cross-site scripting and many more.

#### Monitoring the web app
Whenever an Amplify hosted web-app is hosted irrespective of custom domain or Amplify's default domain, Amplify provides monitoring metrics which helps you understand if the web app is being served the right way. 
![Monitoring](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433028489/SrdfqTzP_.png)
Amplify offers with CloudWatch metrics which gives you details about the number of requests, errors (4xx and 5xx), time for first byte and data transfer.
All the access logs are also available which provides summary about all the logs along with the respective status code and the user agent details. 
You can also create your custom alarms similar to the CloudWatch alarms. 
 ![Alarms](https://cdn.hashnode.com/res/hashnode/image/upload/v1652433029918/5zo5PYeSV.png)
Alarms are for the metrics - requests, bytes downloaded, updated, errors, time for first bye where you can specify the rule of average number for the metric along with the consecutive time period. These alarms use SNS to notify you.

#### Pricing
The princing for the solution would be divided among the services - Amplify builds, Route53 and the cost for data transfer. 
Amplify pricing : [view pricing](https://aws.amazon.com/amplify/pricing/)
Route53 pricing : [view pricing](https://aws.amazon.com/route53/pricing/)

#### Conclusion
Amplify apps which leverage web-hosting are made feasible for DevOps that not only helps release cycles but also manage various stages and the feature of using your own domains instead of Amplify' default domain `amplifyapp.com` makes it easy for a production ready application with various stages configured. 
The example screenshots and custom domain over Route53 is added for my personal landing page [zachjonesnoel.com](https://zachjonesnoel.com/) which is using Amplify web hosting for NextJS and the [portfolio](https://portfolio.zachjonesnoel.com/#/) is another web app using Amplify web hosting for VueJS. In this one hosted zone is created for the domain and in domain mapping for the respective Amplify apps is redirecting to the domain or sub-domain. 