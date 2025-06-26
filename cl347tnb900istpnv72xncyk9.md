---
title: "Deep dive into Lambda event-filters for DyanmoDB"
datePublished: Sun Dec 12 2021 14:30:00 GMT+0000 (Coordinated Universal Time)
cuid: cl347tnb900istpnv72xncyk9
slug: deep-dive-into-lambda-event-filters-for-dyanmodb
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1652432795059/7xGIVEhko.jpeg

---

[Amazon Lambda Functions](https://aws.amazon.com/lambda/) with the event-filtering for [DynamoDB](https://aws.amazon.com/dynamodb/) Streams enables ease of integration and application development. 

There are a couple of good blog posts with talks about how to setup triggers with filter patterns to your Lambda fns.

To know about what is DynamoDB Streams and it's use-cases - 
{%post aws-builders/using-dynamodb-streams-and-lambda-in-your-serverless-applications-3epl%}
To know about what is the announcement of Lambda functions Event-filtering -
{%post aws-builders/trigger-lambda-functions-with-event-filtering-2pnb%}
To know about DynamoDB Streams which are helpful with event-filtering - 
{%post aws-builders/new-dynamodb-streams-filtering-in-serverless-framework-3lc5%}
{%post nosqlknowhow/aws-lambda-event-filter-of-amazon-dynamodb-streams-16pc%}
{%post aws-builders/filtering-dynamodb-streams-before-lambda-p5p%}


#### Key takeaways 
In this blog post, we will deep dive into the feature itself and we will learn about - 
+ [How the filtering works](#working)
+ [Filter patterns supported](#supported-filters)
+ [Limitations with event-filtering](#limiations)
+ [Use-cases and it's associated filter-patterns](#usecase)

#### How the filtering works <a name="working"></a>
The filter expression or the filter pattern JSON uses a strict JSON format which is used to match the filtering criteria. For a DynamoDB Stream based event, the JSON is validated against the key `dynamodb` which is part of the event JSON.
```json
{
    "Records": [
        {
            ...
            "dynamodb": { }
            ...
        }
    ]
}
```
![Flow diagram](https://cdn.hashnode.com/res/hashnode/image/upload/v1652432793784/0lSZZf6XV.png) 
Whenever an action on DynamoDB `putItem` or `updateItem` or `deleteItem` occurs, the `INSERT` or `MODIFY` or `REMOVE` events are triggered respectively if DynamoDB Streams are enabled. This helps handling the needed business logic for specific actions. Now with *event filtering with patterns* are available, this performs additional validation before the Lambda function is invoked. 
The flow diagram explains, whenever there is a DynamoDB Stream invoked, it checks if the target Lambda fn has any event filtering pattern defined or not. If the pattern is available, event JSON is validated against it and if the event JSON matches the criteria then the Lambda fn gets invoked.
If nothing is matched, the event is discarded. If the pattern itself is not defined, the Lambda function gets invoked as *event-filter patterns* are optional.

Event which is triggered whenever a DynamoDB `INSERT` operation happens is - 
```json
{
    "Records": [
        {
            "eventID": "42a3c355ea58b7c68a96368be0f8fa68",
            "eventName": "INSERT",
            "eventVersion": "1.1",
            "eventSource": "aws:dynamodb",
            "awsRegion": "us-east-1",
            "dynamodb": {
                "ApproximateCreationDateTime": 1638968744,
                "Keys": {
                    "sk": {
                        "S": "CAR"
                    },
                    "pk": {
                        "S": "USA3271395584644fordgalaxie5001972-01-01"
                    }
                },
                "NewImage": {
                    "miles_per_gallon": {
                        "N": "14"
                    },
                    "acceleration": {
                        "N": "13"
                    },
                    "car_name": {
                        "S": "Ford Galaxie 500"
                    },
                    "horsepower": {
                        "N": "153"
                    },
                    "year": {
                        "S": "1972-02-01"
                    },
                    "origin": {
                        "S": "USA"
                    },
                    "sk": {
                        "S": "CAR"
                    },
                    "displacement": {
                        "N": "351"
                    },
                    "pk": {
                        "S": "USA3271395584644fordgalaxie5001972-01-01"
                    },
                    "cylinders": {
                        "N": "8"
                    },
                    "weight_in_lbs": {
                        "N": "4129"
                    }
                },
                "SequenceNumber": "530563900000000048422878623",
                "SizeBytes": 228,
                "StreamViewType": "NEW_AND_OLD_IMAGES"
            },
            "eventSourceARN": "arn:aws:dynamodb:us-east-1:xxxxxxx:table/cars-demo/stream/2021-11-28T18:05:10.127"
        }
    ]
}
```


#### Filter patterns supported <a name="supported-filters"></a>
The event filtering uses JSON based patterns which are capable with the following criteria checks -

Criteria | Example | Syntax
--- | --- | ---
Null | origin is null | `"origin": [ null ]`
Empty | origin is empty | `"origin": [""]`
Equals | origin is "USA" | `"origin": ["USA"]`
And | origin is "USA" and cylinders is 8 | `"origin": ["USA"], "cylinders": ["8"]`
Or | origin is "USA" or "Germany" | `"origin": ["USA","Germany"]`
Not | origin is not Japan | `"origin": { "anything-but": [ "Japan" ] }`
Exists | displacement exists | `"displacement ": [ { "exists": true } ]`
Does not exists | displacement does not exists | `"displacement ": [ { "exists": false} ]`
Begins with | pk begins with "USA" | `"pk": [ {"prefix": "USA" } ]`


#### Limitations with event-filtering <a name="limiations"></a>
+ The max number of *event-filtering patterns* that can be defined for a Lambda fn is 5. Additionally, you cannot have multiple event sources from the same DynamoDB table.
+ Each of the 5 patterns is validated against an *OR* condition i.e. pattern1 OR pattern2 OR pattern3 OR pattern4 OR pattern5
+ Numeric comparisons such as numeric equals or number in range cannot be performed as the DynamoDB Streams uses the marshalled JSON structure which has numbers also as strings. 
```
"horsepower": {
     "N": "153"
},
```
+ A individual pattern criteria can validate with an *AND* condition only. Eg. `"origin": ["USA","Germany"], "cylinders": ["8"], "pk": [ {"prefix": "USA" } ]` would validate when "origin" is either "USA" or "Germany" and "cylinders" is 8 and "year" begins with "1972".


#### Use-cases and it's associated filter-patterns <a name="usecase"></a>
Whenever you would want the Lambda function to be invoked only if -
+ Have atleast one cylinder.
```json
{
  "filters": [
    {
      "pattern": "{\"dynamodb\" : \"{ \"NewImage\" : \"cylinder\" : \"N\" :  { \"anything-but\": [ \"0\" ] } }\" }"
    }
  ]
}
```
+ A new car is added to the DynamoDB table.
```json
{
  "filters": [
    {
      "pattern": "{\"eventName\" : [\"INSERT\"], \"dynamodb\" : \"{ \"Key\" : \"sk\" : \"S\" :  [\"CAR\"] }\"  }"
    }
  ]
}
```
+ A car from USA origin is modified.
```json
{
  "filters": [
    {
      "pattern": "{\"eventName\" : [\"MODIFY\"], \"dynamodb\" : \"{ \"NewImage\" : \"origin\" : \"S\" :  [\"USA\"] }\"  }"
    }
  ]
}
```
+ A car name beginning with "Ford" is deleted.
```json
{
  "filters": [
    {
      "pattern": "{\"eventName\" : [\"REMOVE\"], \"dynamodb\" : \"{ \"OldImage\" : \"car_name\" : \"S\" :  [{\"prefix\": \"Ford\" }] }\"  }"
    }
  ]
}
```
+ Either if a new car is added or removed.
```json
{
  "filters": [
    {
      "pattern": "{\"eventName\" : [\"INSERT\",\"REMOVE\"], \"dynamodb\" : \"{ \"Key\" : \"sk\" : \"S\" :  [\"CAR\"] }\"  }"
    }
  ]
}
```
+ Either if a new car is added or if the any car with year 1971 is added / deleted / updated.
```json
{
  "filters": [
    {
      "pattern": "{\"eventName\" : [\"INSERT\"]}"
    },
    {
      "pattern": "{\"dynamodb\" : \"{ \"OldImage\" : \"year\" : \"S\" :  [{\"prefix\": \"1971\" }] }\"  }"
    }
  ]
}
```

#### Conclusion
The *event-filtering with DynamoDB Streams* helps in filtering out and focusing only on the pattern of data which needs the Lambda function to be invoked. This helps in efficient and optimized Lambda function code as the additional code to validate various conditions. This also adheres to the new *Well Architected Model*'s new pillar *Sustainability pillar* which ensures there is less carbon foot-print because of limited Lambda function executions.
