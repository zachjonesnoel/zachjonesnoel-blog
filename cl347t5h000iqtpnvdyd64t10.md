---
title: "Save more with DynamoDB table class"
datePublished: Sun Jan 02 2022 11:29:03 GMT+0000 (Coordinated Universal Time)
cuid: cl347t5h000iqtpnvdyd64t10
slug: save-more-with-dynamodb-table-class
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432771532/EFb0kdIZc.jpeg

---

During re:Invent 2021, an announcement was made about [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) with table classes which helps in saving cost for infrequently accessed items. Read about the announcement from [AWS blog](https://aws.amazon.com/blogs/aws/new-dynamodb-table-class-save-up-to-60-in-your-dynamodb-costs/).
{%twitter 1466102686412001283%}

#### Key takeaways from the blog
+ [The idea behind the storage class (S3) and table class (DynamoDB)](#idea)
+ [Setting up DynamoDB table class](#setting-up)
+ [How DynamoDB table class affects cost](#costs)
+ [Best fit for Standard IA](#best-fit)

#### The idea behind the storage class (S3) and table class (DynamoDB) <a name="idea"></a>
In most of the Serverless architectures, the *go-to solution* for storing infrequently access items with high availability is leveraging [Amazon S3](https://aws.amazon.com/s3/) with the storage class **Standard IA** or **One zone IA**. 
This is achieved in two ways -
+ Where periodically or based on events, DynamoDB table items are retrieved and stored in S3 with the storage class of Infrequently Accessed (IA). And the retrieved items from DynamoDB have to be deleted from DynamoDB.
+ With [DynamoDB TTL](https://dev.to/awscommunity-asean/dynamodb-and-ttl-177g) and [event filters for *REMOVE* operation](https://dev.to/aws-builders/deep-dive-into-lambda-event-filters-for-dyanmodb-320) only, this can be stored into S3 with he storage class of *Infrequently Accessed (IA)*.

This helps in storing data which are in other words *archived* to the application and would be accessed *one a while* but whenever it is accessed, it needs to be highly available.

Now with the latest announcement, we wouldn't have to architect using S3 for *infrequently accessed* items/objects and it is all part of DynamoDB table with the *Standard-IA* class with which you can perform the same DynamoDB operations for read/write. Now with the introduction of table classes, there are two classes now available - **DynamoDB Standard** and [**DynamoDB Standard-IA**](https://aws.amazon.com/dynamodb/standard-ia/). The table with *Standard-IA* class will eliminate the need of S3 based archival.

> **DynamoDB Standard**
The default general-purpose table class. Recommended for the vast majority of tables that store frequently accessed data, with throughput (reads and writes) as the dominant table cost.

> **DynamoDB Standard-IA**
Recommended for tables that store data that is infrequently accessed, with storage as the dominant table cost.

#### Setting up DynamoDB table class <a name="setting-up"></a>
You could use your existing deployment methods for creating or updating DynamoDB tables with *TableClass* attribute via CloudFormation and also from the web console.

##### Creating a table with DynamoDB Standard-IA
Whenever creating the table from console, under **Customized settings** selection, the choice of table class shows up where you can choose between **DynamoDB Standard** and **DynamoDB Standard-IA**.
![Create table with Standard IA](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432762275/S6Yhoxyqz.png)
 
##### Updating a table to DynamoDB Standard-IA
For updating an existing table, select the table and view details on the console. Under the **Actions** menu, select **Update table class**.
![Updating a table to DynamoDB Standard-IA](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432763835/fu80kOUdf.png) This will present you an UI where you can choose between **DynamoDB Standard** and **DynamoDB Standard-IA**.
![Update table to Standard IA](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432765430/zug3RDTH2.png)

##### Creating or updating with CloudFormation / SAM CLI
SAM CLI which uses CloudFormation based resource definition with YAML/JSON, the table class can be updated with the new property introduced [**TableClass**](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html#cfn-dynamodb-table-tableclass) which accepts **STANDARD** for DynamoDB Standard class or **STANDARD_INFREQUENT_ACCESS** for DynamoDB Standard IA class.

```YAML
DynamoDBTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5
      TableClass: STANDARD_INFREQUENT_ACCESS
        
```

#### How DynamoDB table class affects cost <a name="costs"></a>
The introduction of table classes, has changed how DynamoDB pricing works in terms of *storage*, *read-write requests* and *global tables*. 
The cost for storage with *DynamoDB Standard IA* class is much much **cheaper** when compared with *DynamoDB Standard* class with almost a difference of $0.15 per GB-month.
![Data storage](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432766849/DcSoD6Cuy.png) However, first 25 GB/month is available for free under *DynamoDB Standard* class with the *monthly free tier*.
But in terms of *read-write requests* and also *global tables*, the *DynamoDB Standard IA* class would make DynamoDB pricing expensive. 
![Global tables](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432768413/ygtuoc-ge.png) But comparing to the number of read requests which would happen with *DynamoDB Standard* and *DynamoDB Standard IA* class would be much much lesser as they are infrequently accessed.
![Read-write requests](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432770111/eFFmfIMA4.png)
There is a detailed information on DynamoDB pricing page for [On Demand](https://aws.amazon.com/dynamodb/pricing/on-demand/) and [Provisioned](https://aws.amazon.com/dynamodb/pricing/provisioned/). From the example of table classes - DynamoDB Standard IA classes with On Demand is **44.2% cheaper** than Standard and Provisioned is **37.6% cheaper** than Standard even though the read-write requests with both the pricing models, Stardard IA is **expensive** over Standard class.

#### Best fit for Standard IA <a name="best-fit"></a>
The new table class *DynamoDB Standard IA* may not fit into all the applications as if the DynamoDB table is often requested with read/write it would generate a high DynamoDB bill. Standard IA would fit in best for the scenarios such as - 
+ Social media app where posts prior to 1 year (365 days) are rarely accessed. 
+ Enterprise applications where the archives of years of data are maintained.
+ Healthcare systems where the patient bill records are rarely accessed once cleared by insurance.
+ Ecommerce applications where the low rated / less searched products are in IA state.