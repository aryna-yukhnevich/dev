# Alerting event filters 

## Description

Describe alerting event filters for each AWS service in scope: AWS Glue, AWS Glue Workflow, AWS Step Functions, AWS Lambda, Amazon EMR Serverless.\
The full list of AWS services that generate events and send them to EventBridge can be found here: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-service-event-list.html \
The following event pattern in JSON format will be applied for each service in scope:
```json
{
  "source": "event source",
  "detail-type": "event name",  
  "detail": {
  }
}
```
This event pattern example only provides values for three fields: the top-level fields "source" and "detail-type", and the "state" field inside the "detail" object field. EventBridge ignores all the other fields in the event when applying the rule.

### Event Filters

AWS Glue Events:
- Job Failure
```json
{
  "source": ["aws.glue"],
  "detail-type": ["Glue Job State Change"],
  "detail": {   
    "state": ["FAILED", "ERROR", "STOPPED"]
  }
}
```
- Job Success
```json
{
  "source": ["aws.glue"],
  "detail-type": ["Glue Job State Change"],
  "detail": {   
    "state": ["SUCCEEDED"]
  }
}
```

AWS Glue Workflow Events:
- Workflow Failure
```json
{
  "source": ["aws.glue"],
  "detail-type": ["Glue Workflow State Change"],
  "detail": {
    "state": ["FAILED", "ERROR"]
  }
}
```

- Workflow Success
```json
{
  "source": ["aws.glue"],
  "detail-type": ["Glue Workflow State Change"],
  "detail": {
    "state": ["SUCCEEDED"]
  }
}
```

AWS Step Functions Events:
- Failure
```json
{  
  "source": ["aws.states"], 
  "detail-type": ["Step Functions Execution Status Change"],  
  "detail": {    
    "status": ["FAILED"]   
  }
}
```

- Success
```json
{  
  "source": ["aws.states"], 
  "detail-type": ["Step Functions Execution Status Change"],   
  "detail": {    
    "status": ["SUCCEEDED"]   
  }
}
```

Amazon EMR Serverless Job Events:
- Failure
```json
{
  "source": ["aws.emr-serverless"],
  "detail-type": ["EMR Serverless Job Run State Change"],
  "detail": {
    "state": ["FAILED"]
  }
}
```

- Success
```json
{
  "source": ["aws.emr-serverless"],
  "detail-type": ["EMR Serverless Job Run State Change"],
  "detail": {
    "state": ["SUCCESS"]
  }
}
```

## Research on Lambda events

### via CloudTrail
AWS CloudTrail records API activity in AWS account, including Lambda function invocations.

- Failure \
Filters events where the response status code indicates a failure. This includes HTTP status codes 400 (Bad Request) and 500 (Internal Server Error)
```json
{
    "source": ["aws.lambda"],
    "detail-type": ["AWS API Call via CloudTrail"],
    "detail": {
        "eventSource": ["lambda.amazonaws.com"],
        "eventName": ["InvokeFunction"],
        "responseElements": {
            "statusCode": [400, 500]
        },
        "requestParameters": {
            "functionName": ["YourLambdaFunctionName"]
        }
    }
}
```

- Success
```json
{
    "source": ["aws.lambda"],
    "detail-type": ["AWS API Call via CloudTrail"],
    "detail": {
      "eventSource": ["lambda.amazonaws.com"],
      "eventName": ["InvokeFunction"],
      "responseElements": {
           "statusCode": [200]
        },
        "requestParameters": {
            "functionName": ["YourLambdaFunctionName"]
        }
    }
}
```

### via AWS Lambda Destinations
This is a feature that provides visibility into Lambda function invocations and routes the execution results to AWS services (EventBridge). \
More details can be found here https://aws.amazon.com/blogs/compute/introducing-aws-lambda-destinations/ . 

With Destinations, we can route asynchronous function results as an execution record to a destination resource without writing additional code. An execution record contains details about the request and response in JSON format including version, timestamp, request context, request payload, response context, and response payload. For each execution status such as Success or Failure we can choose EventBridge.

Lambda passes the invocation record as the detail in the PutEvents call to EventBridge. The value for the source event field is lambda. The value for the detail-type event field is either "Lambda Function Invocation Result - Success" or "Lambda Function Invocation Result - Failure". The resource event field contains the function and destination Amazon Resource Names (ARNs).
The invocation record contains details about the request and response in JSON format:
- Success
```json
{
  "source": ["aws.lambda"],
  "detail-type": ["Lambda Function Invocation Result - Success"],
  "resources": ["Lambda function ARN"]
}
```
- Failure
```json
{
  "source": ["aws.lambda"],
  "detail-type": ["Lambda Function Invocation Result - Failure"],
  "resources": ["Lambda function ARN"]
}
```

### via CloudWatch Logs Group
Lambda function with logging information in a structured format can be filtered using CloudWatch Logs subscription filters.
CloudWatch Logs message event contains log data in Base64-encoded .gzip file archive:
```json
{
  "awslogs": {
    "data": "BASE64_ENCODED_LOG_DATA"
  }
}
```
We would need to decode the data field to get the actual log data. The log data will be in a JSON format and contain information about the log group, log stream, and the log events that matched the filter pattern.

