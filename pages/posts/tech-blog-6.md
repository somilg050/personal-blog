---
title: Robust API Retry Mechanism with AWS Step Functions and Lambda
date: 2024/7/10
description: Learn How to Construct a Reliable System for Retrying External API Calls Without Getting 429
tag: Tech Blog
author: You
---

![MainImage](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*i-65KLMxwQz1JBRLZMYUzQ.png)

I have been working with external API calls for a while and have noticed they can sometimes fail for various reasons, such as network issues, server downtime, or rate limits on the server. So, I have built this solution to have a robust system to tackle this problem.

In this solution, we will leverage the AWS Step Function and Lambda Functions to construct a reliable retry mechanism. The State Machine will consist of a collection of Lambda functions invoked and stitched together to produce results. This article will walk you through the step-by-step guide.

### **The main objective we are trying to solve:**

While Step Functions inherently support retries within tasks, our specific challenge involves handling API rate limits from the server we are communicating with. The server imposes a rate limit and responds with a 429 status code if too many requests are made from the same IP address within a short period.

---

### **Prerequisites**

* AWS Account

* Basic understanding of AWS Lambda and Step Functions


### **1\. Architecture:**

![](https://miro.medium.com/v2/resize:fit:1500/1*odB9U9xDRe_GXFcC4ItMfA.png)

#### Workflow Explanation

1. **User Invokes Step Function State Machine:** The process begins when a user initiates the step function state machine. This could be triggered through an API call, a scheduled event, or another AWS service.

2. **Step Function Invokes Lambda (1st Attempt):** The step function invokes the first Lambda function (`Lambda 1`). This Lambda function is responsible for making the API call.

3. **Response: Status:** `Lambda 1` Executes the API call and returns a status response. This response indicates whether the API call was successful (e.g., status code 200) or failed (e.g., any status code other than 200).

4. **If Failure Status ≠ 200 (2nd Attempt):** If the response from `Lambda 1` If it indicates a failure (status code not equal to 200), the step function will proceed to invoke a retry mechanism. This could involve retrying the same Lambda function or invoking a different Lambda function (`Lambda 2`) to handle the retry attempt.

5. **Response: Status:** `Lambda 2` It attempts to execute the API call and returns a status response. Similar to the first Attempt, this response will indicate whether the retry was successful.

6. **If Success Status = 200:** If either `Lambda 1` or `Lambda 2` Successfully executes the API call and returns a status code of 200, the step function completes successfully, and the user is notified of the success.

7. **If Failure Even After Retries:** Then we will fail the step function and forward the API error to the user with the appropriate status code.


To explain the architecture easily, I have created the above diagram with one retry only, but we will build the solution with two retries. Below is the state machine diagram.

![](https://cdn-images-1.medium.com/max/1200/1*vfHQ3qJnAGjtXCQ-50TM4A.png)

---

### 2\. Step-by-Step Guide

* **Create a base lambda function:**  
  This lambda function will help us in orchestrating the state machine. Executing the state machine and handling logic based on the execution status.


```python
import boto3
import json
import time

def start_state_machine(body):
    # Create a session with AWS credentials
    session = boto3.Session(
        aws_access_key_id='',
        aws_secret_access_key='',
        region_name=''
    )
    
    # Create a client to interact with AWS Step Functions
    step_functions_client = session.client('stepfunctions')
    
    # Define the ARN of the Step Function that you want to start
    state_machine_arn = 'arn:aws:states::stateMachine:apiProxyStateMachine'
    
    # Define the input to pass to the Step Function
    input_data = body
    
    # Start the Step Function with the specified input
    response = step_functions_client.start_execution(
        stateMachineArn=state_machine_arn,
        input=json.dumps(input_data)
    )
    
    # Wait for the execution to complete
    while True:
        execution_status = step_functions_client.describe_execution(
            executionArn=response['executionArn']
        )['status']
        if execution_status in ['SUCCEEDED', 'FAILED', 'ABORTED']:
            break
    
    execution_output = step_functions_client.describe_execution(
        executionArn=response['executionArn']
    )
    
    if(execution_output['status'] == 'SUCCEEDED'):
        return execution_output['output']
    else:
        return execution_output['status']

def lambda_handler(event, context):
    event = event["body"]
    data = start_state_machine(event)
    response = json.loads(data)
    return {
        "statusCode": response["statusCode"],
        "body": response["body"]
    }
```
---

* **Create a function URL for the lambda function:**  
  Now that the lambda function is ready, we can set up a function URL to trigger/send a request to the lambda function using it. Refer to this [article](https://medium.com/technology-hits/how-to-use-aws-lambda-to-trigger-any-script-as-an-api-call-64f13d8b36e5?source=post_page-----ef4702bc36c8--------------------------------) to turn any lambda function into an API with a function URL.


* **Create child lambda functions:**  
  These will be simple lambda functions acting as a proxy; they will not handle any logic.

    ```python
    import json
    import requests
    
    def lambda_handler(event, context):
        api_url = "https://api.example.com/data"
        try:
            response = requests.get(api_url)
            response.raise_for_status()
            return {
                'statusCode': 200,
                'body': json.dumps(response.json())
            }
        except requests.exceptions.RequestException as e:
            return {
                'statusCode': response.status_code if response else 500,
                'body': json.dumps({'error': str(e)})
            }
    ```

  We have to create the same three lambda functions using the above code.


* **Define Step Function State Machine:**  
  Next, we'll create a Step Functions state machine with a retry mechanism. Here is an example definition in JSON.

    ```json
    {
      "Comment": "A description of my state machine",
      "StartAt": "Proxy 1",
      "States": {
        "Proxy 1": {
          "Type": "Task",
          "Resource": "arn:aws:states:::lambda:invoke",
          "OutputPath": "$.Payload",
          "Parameters": {
            "Payload.$": "$",
            "FunctionName": "arn:aws:lambda:*******:function:apiProxy1:$LATEST"
          },
          "Retry": [
            {
              "ErrorEquals": [
                "Lambda.ServiceException",
                "Lambda.AWSLambdaException",
                "Lambda.SdkClientException",
                "Lambda.TooManyRequestsException"
              ],
              "IntervalSeconds": 2,
              "MaxAttempts": 6,
              "BackoffRate": 2
            }
          ],
          "Next": "Pass"
        },
        "Pass": {
          "Type": "Pass",
          "Next": "Choice"
        },
        "Choice": {
          "Type": "Choice",
          "Choices": [
            {
              "Variable": "$.statusCode",
              "NumericEquals": 200,
              "Next": "Pass (1)"
            }
          ],
          "Default": "Pass (7)"
        },
        "Pass (7)": {
          "Type": "Pass",
          "Next": "Wait"
        },
        "Wait": {
          "Type": "Wait",
          "Seconds": 5,
          "Next": "Proxy 2"
        },
        "Pass (1)": {
          "Type": "Pass",
          "Next": "Success - 1"
        },
        "Success - 1": {
          "Type": "Succeed"
        },
        "Proxy 2": {
          "Type": "Task",
          "Resource": "arn:aws:states:::lambda:invoke",
          "OutputPath": "$.Payload",
          "Parameters": {
            "Payload.$": "$",
            "FunctionName": "arn:aws:lambda:*****:function:apiProxy2:$LATEST"
          },
          "Retry": [
            {
              "ErrorEquals": [
                "Lambda.ServiceException",
                "Lambda.AWSLambdaException",
                "Lambda.SdkClientException",
                "Lambda.TooManyRequestsException"
              ],
              "IntervalSeconds": 2,
              "MaxAttempts": 6,
              "BackoffRate": 2
            }
          ],
          "Next": "Pass (8)"
        },
        "Pass (8)": {
          "Type": "Pass",
          "Next": "Choice (1)"
        },
        "Choice (1)": {
          "Type": "Choice",
          "Choices": [
            {
              "Variable": "$.statusCode",
              "NumericEquals": 200,
              "Next": "Pass (2)"
            }
          ],
          "Default": "Pass (3)"
        },
        "Pass (3)": {
          "Type": "Pass",
          "Next": "Proxy 3"
        },
        "Pass (2)": {
          "Type": "Pass",
          "Next": "Success -2"
        },
        "Success -2": {
          "Type": "Succeed"
        },
        "Proxy 3": {
          "Type": "Task",
          "Resource": "arn:aws:states:::lambda:invoke",
          "OutputPath": "$.Payload",
          "Parameters": {
            "Payload.$": "$",
            "FunctionName": "arn:aws:lambda:*******:function:apiProxy3:$LATEST"
          },
          "Retry": [
            {
              "ErrorEquals": [
                "Lambda.ServiceException",
                "Lambda.AWSLambdaException",
                "Lambda.SdkClientException",
                "Lambda.TooManyRequestsException"
              ],
              "IntervalSeconds": 2,
              "MaxAttempts": 6,
              "BackoffRate": 2
            }
          ],
          "Next": "Pass (4)"
        },
        "Pass (4)": {
          "Type": "Pass",
          "Next": "Choice (2)"
        },
        "Choice (2)": {
          "Type": "Choice",
          "Choices": [
            {
              "Variable": "$.statusCode",
              "NumericEquals": 200,
              "Next": "Pass (5)"
            }
          ],
          "Default": "Pass (6)"
        },
        "Pass (6)": {
          "Type": "Pass",
          "Next": "Failure"
        },
        "Pass (5)": {
          "Type": "Pass",
          "Next": "Success - 3"
        },
        "Success - 3": {
          "Type": "Succeed"
        },
        "Failure": {
          "Type": "Succeed"
        }
      }
    }
    ```
    
---

### 3\. Testing the State Machine

Trigger the state machine execution using the first lambda function URL and monitor it through the AWS State Machine Console. You should see the retries and the final result, whether it succeeds or fails.

![](https://cdn-images-1.medium.com/max/1200/1*Ppua6lCoZHX2KuYjjM_A3Q.png)

---

### Conclusion —

Implementing a robust API retry mechanism using AWS Step Functions and Lambda is a powerful way to enhance the reliability of your API integrations. I have worked too much with the vendor APIs, and their reliability is something you can not trust. They have rate limits, server IP-based wait times, and so on. This retry using different lambda functions will give us different server URLs, preventing IP-based wait time blocking plus the retry mechanism.

This solution provides a visual workflow to monitor and debug your API calls. With AWS Step Functions and Lambda, you can build a fault-tolerant API integration with minimal effort.

---

*Thanks for reading the tutorial. I hope you learn something new today. If you want to read more stories like this, I invite you to follow me.*

*Till then, Sayonara! I wish you the best in your learning journey.*

*I am a software engineer who enjoys travelling and writing.*