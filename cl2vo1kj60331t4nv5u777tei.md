---
title: "AWS S3 and it's popularity"
datePublished: Sun Apr 25 2021 16:40:31 GMT+0000 (Coordinated Universal Time)
cuid: cl2vo1kj60331t4nv5u777tei
slug: aws-s3-and-its-popularity
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1651915843005/BvYT51ZXR.jpeg

---

Amazon Web Services' **Simple Storage Service**, popularly known as [**AWS S3**](https://aws.amazon.com/s3/) is a cloud storage service providing high scalability, availability, affordable, most secure and also performant. AWS S3 is a object storage i.e. the files or folders are treated a data objects which are stored as a flat hierarchy and can have additional metadata along with the object for more of a object level management. 

Since it's launch in 2006, S3 popularity has grown immensely as it has evolved a lot by providing a new feature/enhancement to the existing feature every year. And in the month of March 2021, S3 celebrated it's 15th birthday!! 

####S3 Features
+ **High availability** - AWS S3 guarantees 99.999999999% popularly called the **11 9's** , well how does AWS S3 achieve it? S3 storage is spanned over multiple AZs (Availability Zones) provides high availability and also the storage classes enables the more frequently accessed files more available over the internet. 
+ **Storage classes** - As mentioned earlier, S3 provides storage classes which are very important when designing because it says a lot about how the object is going to be stored, how long it would take for retrieval of the object and also the cost of storage.
+ **CURD operations for object** - AWS S3 supports the basic CURD (Create, Update, Read, Delete) operation with HTTP APIs, AWS SDK, AWS CLI and also management console. And supports objects ranging from 0 bytes to 5 TBs. And for higher volumes of data, the Snow family is available.
+ **Version controlling** - For any storage service, versioning is an important feature, on S3 we can turn on versioning and whenever object with same key is uploaded, it creates a new version and we can even roll back to a desired version as needed.
+ **Ease of management** - Along with CURD operations, S3 provides life-cycle management rules which is more defined by the AWS Customer based on their use-cases, triggers on operations which can invoke SNS, Lambda and other services. configuration of S3 Access Points for management accessed of shared data.
+ **Web-hosting** - it provides web hosting which is where in the serverless world and you don't need to worry about creating an EC2 instance with Apache/Nginx for web-server hosting. Isn't that cool!? Web applications hosted on S3 can be delivered over CloudFront's CDN for low latency. And we could also redirect a domain to point to S3 with Route53.

#### Storage classes
As mentioned in the the features, AWS S3 provides a range of storage classes which are more used for storage type, cost effectiveness, availability of the objects. 
![Storage classes](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915839984/FT-p4GM4N.png)
Different [Storage classes](https://aws.amazon.com/s3/storage-classes/) - 
+ **S3 Standard** - the default storage class which offers high availability of 99.99%, this is spread over multiple AZs and popularly used.
+ **S3 Intelligent-tiering** - tailored for frequent and infrequent access and also provides good cost saving as S3 toggles the object between Frequent access tier and Infrequent access tier based on the number times the object is accessed over certain consecutive days.
+ **S3 Standard IA** - this is for the infrequently accessed object but which needs faster retrieval time. This is the most cost effective and high performance storage class. 
+ **S3 One Zone IA** - follows the same concepts of S3 Standard IA but the objects are available in a single AZ making it a low cost option and availability of 99.5%.
+ **S3 Glacier** - the most secure and lost cost storage designed for very less frequently access files and provides a retrieval time varying between minutes to 2-3 hours.
+ **S3 Glacier Deep Archive** - this storage compared to S3 Glacier where the objects have a retention period of 7-10 years with a retrieval time upto 12 hours. And often used for backups of data.

#### Life-cycle management
The life-cycle management provides AWS' Customer to configure a life-cycle based on their use-case. This allows the customer to toggle between different storage classes as the requirement demands. 
![Life-cycle management](https://cdn.hashnode.com/res/hashnode/image/upload/v1651915841436/TXZelP11p.jpeg)
As the example screenshot says, we can manage an object's life-cycle as when it is uploaded for the first time, it is in S3 Standard and then after 7 days from creation, the storage is transitioned to S3 Glacier and retained for upto 120 days and then the object is deleted. 
Similar to these, we can create custom life-cycle configurations and switch between storage classes, object management, version management, storage and cost management.

#### Common use-cases
Going through the features of S3 we understood that it has a lot of applicability in the real world use-cases. Some of the common use-cases include -
+ Backup and restore
+ Media storage for SaaS products
+ Web hosting
+ Data lakes
+ Access controlled retrievals

And also **DEV community** uses S3 for all the media storage. 
