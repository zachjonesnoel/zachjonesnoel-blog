---
title: "AWS SAM app with MongoDB Atlas Data APIs"
datePublished: Sun Jan 09 2022 15:39:56 GMT+0000 (Coordinated Universal Time)
cuid: cl347ssm900j5uqnvcqt8645l
slug: aws-sam-app-with-mongodb-atlas-data-apis
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432743418/PbpM1Ld_A.png

---

This is the implementation of a typical web app backend where the MongoDB Atlas is leveraged to store data.

### Overview of My Submission
The SAM app uses MongoDB Atlas for database choice which contains two collections - `shield_agents` and `shield_locations`. This stores the S.H.I.E.L.D info such as the database of all agents and non classified locations. The operation of `INSERT` and `GET` is integrated with [MongoDB Atlas Data APIs](https://docs.atlas.mongodb.com/api/data-api/) where the API credentials are created and passed into AWS Lambda fns as *environment variables* with CloudFormation Parameters. The APIs from SAM app are exposed with *AWS API Gateway*. Also have the Search enabled with indexes.
![MongoDB dashboard](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432743418/PbpM1Ld_A.png)
 
### Submission Category: 
**Choose Your Own Adventure**, the first time I'm using MongoDB and Data APIs so went a step to adapt it into my usual tech-stack (AWS Serverless) with a [AWS SAM application](https://aws.amazon.com/serverless/sam/). 

### Link to Code
{% github https://github.com/zachjonesnoel/shield-database %}

### Screenshots
Creating an agent with `insertOne` API
![Create agent](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432745020/leXwCNy5Y.png)
Getting one of the created agent with `findOne` API
![Get created agent](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432746639/3SfbrYadW.png)
Creating a location with `insertOne` API
![Create location](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432748304/JWCC4vJbQ.png)
Getting one of the created location with `findOne` API
![Get created location](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432749993/jNWxTiPNn.png)
[MongoDB Atlas Search](https://docs.atlas.mongodb.com/atlas-search/) helps in creating an `index` for a collection and using the index to query and check if the filter pattern matches any record. The matching records also return a match score.
![Atlas Search](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432751951/NSQhjiHzo.png)
[MongoDB Atlas Charts](https://docs.mongodb.com/charts/) can help you visualize data. 
No of locations - 
![MongoDB Atlas Charts - locations](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432753528/uT0TWIjFG.png)
Agents with security clearance level- 
![MongoDB Atlas Charts - security level](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432755080/2wktPzlDP.png)
 

### Additional Resources / Info
+ [MongoDB Atlas Data API](https://docs.atlas.mongodb.com/api/data-api/)
+ [MongoDB Atlas Search](https://docs.atlas.mongodb.com/atlas-search/)
+ [MongoDB Atlas Charts](https://docs.mongodb.com/charts/)
+ [AWS SAM application](https://aws.amazon.com/serverless/sam/)
