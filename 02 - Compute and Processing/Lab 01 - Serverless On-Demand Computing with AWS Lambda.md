# Lab 05: Serverless On-Demand Computing with AWS Lambda

## Objective
In this lab, you will create a serverless function using AWS Lambda that performs on-demand computing. You will learn how to:
- Create an AWS Lambda function.
- Write a simple function in Python to perform a computation.
- Configure an API Gateway to trigger the Lambda function.
- Test the function via API calls.

## Prerequisites
- **AWS Account:** Ensure you have an active AWS account with permissions to create Lambda functions and API Gateway endpoints.
- **Basic Programming Knowledge:** Familiarity with Python (or your preferred Lambda runtime) will be helpful.
- **AWS Management Console Access:** You can use the AWS Console to complete this lab.

## Architecture Overview

```yaml
   +---------------------------+
   |       API Gateway         |
   | (HTTP Request Trigger)    |
   +-------------+-------------+
                 |
                 v
          +-------------+
          | AWS Lambda  |
          |  Function   |  <-- On-demand compute function (e.g., process input, compute result)
          +-------------+
                 |
                 v
        +------------------+
        |  CloudWatch Logs |
        +------------------+
```

*Note:* API Gateway serves as the HTTP front door to invoke the Lambda function. The function runs on demand, performing the computation and returning a response.

## Step-by-Step Instructions

### 1. Create an AWS Lambda Function

#### a. Navigate to AWS Lambda
- Log in to the AWS Management Console.
- Navigate to the **Lambda** service.

#### b. Create a New Function
- Click **Create function**.
- Choose **Author from scratch**.
- **Function Name:** `OnDemandComputeFunction`
- **Runtime:** Select **Python 3.x** (or your preferred runtime).
- Click **Create function**.

### 2. Write the Lambda Function Code

In the inline code editor, replace the default code with a simple computation. For example, create a function that calculates the square of a number passed in the event:

```python
def lambda_handler(event, context):
    # Retrieve the number from the event. Expecting a query parameter "number"
    try:
        number = float(event.get('queryStringParameters', {}).get('number', 0))
    except (ValueError, TypeError):
        return {
            'statusCode': 400,
            'body': 'Invalid input. Please provide a numeric value.'
        }

    # Perform on-demand computation: calculate the square of the number
    result = number ** 2

    # Return the result as a JSON response
    return {
        'statusCode': 200,
        'body': f'The square of {number} is {result}.'
    }
```

Click **Deploy** to save your changes.

## 3. Set Up an API Gateway Trigger

**a. Create an API**
- In the Lambda function’s **Configuration** page, click **Add trigger**.  
- Select **API Gateway**.  
- Choose **Create an API**.  
- **API Type:** HTTP API (for simplicity and lower latency)  
- Click **Add** to create the API Gateway and attach it as a trigger.

**b. Configure API Gateway (Optional)**
- Customize settings (e.g., security, CORS) in the API Gateway console if needed.  
- Copy the generated API endpoint URL — you’ll use it to test the Lambda function.

## 4. Test the Lambda Function via API Gateway

**a. Using a Web Browser or cURL**
- Open a web browser or terminal.  
- Construct a URL to invoke your function with a query parameter. For example:  


```bash
https://<api-id>.execute-api.<region>.amazonaws.com/?number=5
```

- You should receive a response similar to:

```json
{
  "statusCode": 200,
  "body": "The square of 5.0 is 25.0."
}
```
**b. Using the AWS Console**
- In the Lambda function console, click Test.
- Create a new test event with the following JSON payload:

```json
{
  "queryStringParameters": {
    "number": "5"
  }
}
```
- Click **Test** and review the output.




## 5. Validate CloudWatch Logs
- Navigate to CloudWatch Logs in the AWS Console.
- Locate the log group for your Lambda function (usually named `/aws/lambda/OnDemandComputeFunction`).
- Review the logs to see the function’s execution details and any printed output for troubleshooting.

## 6. Clean Up Resources
After you have completed testing:
- **Remove API Gateway Trigger:** Detach the API Gateway trigger from the Lambda function if not needed.
- **Delete the Lambda Function:** Navigate to the Lambda console, select your function, and choose Delete.
- **Delete API Gateway (if created separately):** Go to the API Gateway console and delete the API to avoid additional charges.

## Lab Discussion Points
- **Serverless Benefits:** Understand how AWS Lambda provides on-demand computing without the need to manage servers.
- **Event-Driven Architecture:** Learn how API Gateway and Lambda work together to create scalable, event-driven solutions.
- **Cost Efficiency:** Explore the cost advantages of using Lambda, where you only pay for the compute time used.
- **Monitoring and Debugging:** Recognize the importance of CloudWatch Logs for tracking execution details and troubleshooting.

This lab exercise demonstrates the fundamentals of deploying a serverless function on AWS to perform on-demand computing using Lambda and API Gateway.
