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
```json
{
  "source": ["aws.lambda"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["lambda.amazonaws.com"]
  }
}
```

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

Just ot mention that this feature works only when a lambda function is invoked asynchronously.
To configure Lambda Destinations on Lambda functions, the following invoke config looks can be applied:

```python
invoke_config = lambda_.CfnEventInvokeConfig(
            self,
            "MyLambdaInvokeConfig",
            function_name=lambda_function.function_name,
            qualifier='$LATEST',
            destination_config=lambda_.CfnEventInvokeConfig.DestinationConfigProperty(
                on_failure=lambda_.CfnEventInvokeConfig.OnFailureProperty(
                    destination=EventBusARN
                ),
                on_success=lambda_.CfnEventInvokeConfig.OnSuccessProperty(
                    destination=EventBusARN
                )
            ),
        ) 
```

And the permission to put events to EventBridge should be assigned to lambda:
```python
    lambda_role.add_to_policy(
            iam.PolicyStatement(
                actions=["events:PutEvents"],
                effect=iam.Effect.ALLOW,
                resources=[EventBusARN],
            )
        )
```

- Success event pattern
```json
{
  "source": ["lambda"],
  "detail-type": ["Lambda Function Invocation Result - Success"],
}
```
- Failure event pattern
```json
{
  "source": ["lambda"],
  "detail-type": ["Lambda Function Invocation Result - Failure"],
}
```

Additionally, event pattern applied to custom Event Bus should be updated as follows (to consider lambda events):
```json
{
  "source": [{
    "prefix": "aws"
  }, "lambda"]
}
```

### via CloudWatch Log Groups
Another option can be considered - to query Lambda runs using CloudWatch Logs. By default, Lambda sends logs to a log group named /aws/lambda/<function name>.
In this case we should assign "AWSLambdaBasicExecutionRole" service role to Lambda function in order to be able to send the logs to Amazon CloudWatch Logs:

```python
      lambda_role.add_managed_policy(
            iam.ManagedPolicy.from_aws_managed_policy_name(
                "service-role/AWSLambdaBasicExecutionRole"
            )
        )
```

The queries can be run using Monitoring Data Extraction component and the results can be stored in the Timestream table.
The query to extract failed lambda runs can look as follows:

```python
from helpers.cloudwatch import query_logs

 def retrieve_errors_list(self):
      log_group_name = f"/aws/lambda/{self.lambda_name}"
      query_string = """
      fields @timestamp, @message
      | filter @message like '[ERROR]'
      | sort @timestamp desc
      """
      self.errors_list = query_logs(self.client, log_group_name, query_string, self.start_time, self.end_time)  
      return self.errors_list
```