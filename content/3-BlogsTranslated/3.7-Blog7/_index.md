---
title: "Blog 7"

weight: 1
chapter: false
pre: " <b> 3.7. </b> "
---


# Streamlining AWS Serverless workflows: From AWS Lambda orchestration to AWS Step Functions

This blog post discusses the [AWS Lambda](https://aws.amazon.com/lambda/) as orchestrator [anti-pattern](https://docs.aws.amazon.com/lambda/latest/operatorguide/anti-patterns.html) and how to redesign serverless solutions using [AWS Step Functions](https://aws.amazon.com/step-functions/) with native integrations.

Step Functions is a serverless workflow service that you can use to build distributed applications, automate processes, orchestrate microservices, and create data and machine learning (ML) pipelines. Step Functions provides native integrations with over [200 AWS services](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-services.html) in addition to external third-party APIs. You can use these integrations to deploy production-ready solutions with less effort, reducing code complexity, improving long-term maintainability, and minimizing technical debt when operating at scale.

---

## The Lambda as orchestrator anti-pattern

Let’s examine a common anti-pattern: using a Lambda function as an orchestrator for message distribution across multiple channels. Consider this real-world scenario where a system needs to send notifications through SMS or email channels based on user preferences, as shown in the following diagram.




![](/images/3-Blogstranslated/Blog7/anti-pattern.png)


The payload examples for this scenario are:  

1. Send SMS only:
  ```yaml
  {
    "body": {
        "channel": "sms",
        "message": "Hello from AWS Lambda!",
        "phoneNumber": "+1234567890",
        "metadata": {
            "priority": "high",
            "category": "notification"
        }
    }
}
```


2. Send email only:
  
  ``` yaml
  {
    "body": {
        "channel": "email",
        "message": "Hello from AWS Lambda!",
        "email": {
            "to": "recipient@example.com",
            "subject": "Test Notification",
            "from": "sender@example.com"
        },
        "metadata": {
            "priority": "normal",
            "category": "notification"
        }
    }
}

  ```
3. Send both SMS and email:

``` yaml
{
    "body": {
        "channel": "email",
        "message": "Hello from AWS Lambda!",
        "email": {
            "to": "recipient@example.com",
            "subject": "Test Notification",
            "from": "sender@example.com"
        },
        "metadata": {
            "priority": "normal",
            "category": "notification"
        }
    }
}
```

Here’s how it typically starts—with a Lambda function acting as an orchestrator:
```yaml
import boto3
import json
# Initialize Lambda client
# You can specify region if needed: boto3.client('lambda', region_name='us-east-1')
lambda_client = boto3.client('lambda')
def lambda_handler(event, context):
    try:
        # Parse the incoming event
        body = json.loads(event['body'])
        
        # Validate required fields
        if 'channel' not in body:
            return {
                'statusCode': 400,
                'body': json.dumps('Missing channel parameter')
            }
        
        if 'message' not in body:
            return {
                'statusCode': 400,
                'body': json.dumps('Missing message content')
            }
        
        if body['channel'] == 'both':
            # Invoke SMS Lambda function
            lambda_client.invoke(
                FunctionName='send-sns',
                InvocationType='Event',
                Payload=json.dumps(body)
            )
            
            # Invoke Email Lambda function
            lambda_client.invoke(
                FunctionName='send-email',
                InvocationType='Event',
                Payload=json.dumps(body)
            )
        else:
            # Validate channel value
            if body['channel'] not in ['sms', 'email']:
                return {
                    'statusCode': 400,
                    'body': json.dumps('Invalid channel specified')
                }
            
            # Invoke function based on specified channel
            function_name = 'send-sns' if body['channel'] == 'sms' else 'send-email'
            lambda_client.invoke(
                FunctionName=function_name,
                InvocationType='Event',
                Payload=json.dumps(body)
            )
        
        return {
            'statusCode': 200,
            'body': json.dumps('Messages sent successfully')
        }
        
    except json.JSONDecodeError:
        return {
            'statusCode': 400,
            'body': json.dumps('Invalid JSON in request body')
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }
```


This approach has the following problems:
- **IntrinsicComplex error handling**: The orchestrator needs to manage errors from multiple function invocations.
- **Tight coupling**:  Functions are directly dependent on each other. 
- **Limited execution time**: The orchestrator Lambda function continues running while sub Lambda functions execute. This could lead to the orchestrator Lambda function timing out.
- **Idle resources**:Because the orchestrator Lambda function is sitting idle waiting for returns from other Lambda functions, in this case, the user is [now paying](https://aws.amazon.com/lambda/pricing/) for idle resources.
  
---

## Rearchitecting with Step Functions

You can rebuild the logic using Step Functions and [Amazon States Language](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html) to replace the Lambda orchestrator function. You can use the [Choice state](https://docs.aws.amazon.com/step-functions/latest/dg/state-choice.html) in Amazon States Language to define logical conditions to follow a specific path. This approach reduces code maintenance complexity because you define the conditions using Amazon States Language. You can also use it to to extend the functionality with minimal changes to the codebase.

The following Step Functions workflow diagram shows the rearchitected version of the previous Orchestrator Lambda function:

![](/images/3-Blogstranslated/Blog7/step-functions.png)

The following Amazon State Language represents the workflow:
```yaml
{
  "Comment": "Multi-channel notification workflow",
  "StartAt": "ValidateInput",
  "States": {
    "ValidateInput": {
      "Type": "Choice",
      "Choices": [
        {
          "And": [
            {
              "Variable": "$.message",
              "IsPresent": true
            },
            {
              "Variable": "$.channel",
              "IsPresent": true
            }
          ],
          "Next": "DetermineChannel"
        }
      ],
      "Default": "ValidationError"
    },
    "ValidationError": {
      "Type": "Fail",
      "Error": "ValidationError",
      "Cause": "Required fields missing: message and/or channel"
    },
    "DetermineChannel": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.channel",
          "StringEquals": "both",
          "Next": "ParallelNotification"
        },
        {
          "Variable": "$.channel",
          "StringEquals": "sms",
          "Next": "SendSMSOnly"
        },
        {
          "Variable": "$.channel",
          "StringEquals": "email",
          "Next": "SendEmailOnly"
        }
      ],
      "Default": "FailState"
    },
    "ParallelNotification": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "SendSMS",
          "States": {
            "SendSMS": {
              "Type": "Task",
              "Resource": "arn:aws:states:::sns:publish",
              "Parameters": {
                "Message.$": "$.message",
                "PhoneNumber.$": "$.phoneNumber"
              },
              "End": true
            }
          }
        },
        {
          "StartAt": "SendEmail",
          "States": {
            "SendEmail": {
              "Type": "Task",
              "Parameters": {
                "FromEmailAddress.$": "$.email.from",
                "Destination": {
                  "ToAddresses.$": "States.Array($.email.to)",
                  "CcAddresses.$": "States.ArrayGetItem(States.JsonToString($.email.cc), $)",
                  "BccAddresses.$": "States.ArrayGetItem(States.JsonToString($.email.bcc), $)"
                },
                "Content": {
                  "Simple": {
                    "Subject": {
                      "Data.$": "$.email.subject",
                      "Charset": "UTF-8"
                    },
                    "Body": {
                      "Text": {
                        "Data.$": "$.message",
                        "Charset": "UTF-8"
                      },
                      "Html": {
                        "Data.$": "$.email.htmlBody",
                        "Charset": "UTF-8"
                      }
                    }
                  }
                },
                "ReplyToAddresses.$": "States.Array($.email.replyTo)",
                "EmailTags": [
                  {
                    "Name": "channel",
                    "Value": "email"
                  },
                  {
                    "Name": "messageType",
                    "Value.$": "$.email.messageType"
                  }
                ],
                "ConfigurationSetName.$": "$.email.configurationSet",
                "ListManagementOptions": {
                  "ContactListName.$": "$.email.contactList",
                  "TopicName.$": "$.email.topic"
                }
              },
              "Resource": "arn:aws:states:::aws-sdk:sesv2:sendEmail",
              "End": true
            }
          }
        }
      ],
      "End": true
    },
    "SendSMSOnly": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "Message.$": "$.message",
        "PhoneNumber.$": "$.phoneNumber"
      },
      "End": true
    },
    "SendEmailOnly": {
      "Type": "Task",
      "Parameters": {
        "FromEmailAddress.$": "$.email.from",
        "Destination": {
          "ToAddresses.$": "States.Array($.email.to)"
        },
        "Content": {
          "Simple": {
            "Subject": {
              "Data.$": "$.email.subject",
              "Charset": "UTF-8"
            },
            "Body": {
              "Text": {
                "Data.$": "$.message",
                "Charset": "UTF-8"
              },
              "Html": {
                "Data.$": "$.email.htmlBody",
                "Charset": "UTF-8"
              }
            }
          }
        }
      },
      "Resource": "arn:aws:states:::aws-sdk:sesv2:sendEmail",
      "End": true
    },
    "FailState": {
      "Type": "Fail",
      "Cause": "Invalid channel specified"
    }
  }
}
```
This Step Functions implementation offers several advantages:
- **Native service integration**: Direct integration with [Amazon Simple Notification Service (Amazon SNS)](https://aws.amazon.com/sns), [Amazon Simple Email Service (Amazon SES)](https://aws.amazon.com/ses), [Amazon DynamoDB](https://aws.amazon.com/dynamodb), and [Amazon CloudWatch](https://aws.amazon.com/cloudwatch) eliminates the need for wrapper Lambda functions
- **Visual workflow**: The execution flow is visible and maintainable through the AWS Management Console
- **Built-in error handling**: Retry policies and error states can be defined declaratively
- **Parallel execution**: The Parallel state handles multiple channel delivery efficiently
- **Simplified logic**: The Choice state replaces complex if-else statements
- **Centralized data flow**: Input and output are managed consistently across states
- **Enhanced workflow duration capabilities**: Step Functions Standard workflows support executions that run for up to one year, compared to the 15-minute maximum execution time for Lambda functions

---

## Comparing Lambda function as orchestrator to Step Functions

The summary of different features implemented on Lambda function as orchestrator and Step Functions is reflected in the following table:

| Feature                    | Lambda function as orchestrator                                             | Step Functions                                              |
| -------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Orchestration logic**    | Implemented in Python with nested if-else statements.                       | Defined declaratively using the Choice state                |
| **Multi-channel delivery** | Sequential function invocations. Parallel execution using function's logic. | Parallel execution using the Parallel state                 |
| **Service integration**    | Requires SDK calls or separate Lambda functions.                            | Direct integration with AWS services (Amazon SNS, DynamoDB) |
| **Error handling**         | Custom try-except blocks in Python.                                         | Built-in error states and retry policies                    |
| **Data persistence**       | Custom code to interact with DynamoDB.                                      | Native DynamoDB integration with putItem task               |
| **Metrics logging**        | Custom code to call CloudWatch.                                             | CloudWatch Metrics SDK integration                          |

---

## Implementation considerations

Review the following considerations when re-architecting a Lambda function orchestrator to Step Functions:

- [State machine type](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html): Choose between Standard (up to 1 year runtime) and Express (up to 5 minutes) workflows based on your needs.
- [Input/output management](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-input-output-filtering.html): Parameters manipulation reduces the development effort and give flexible alternatives to implement the workflow:
  + Parameters: Selects specific input fields to pass to the next state
  + ResultSelector: Filters the state response to include only relevant fields
  + ResultPath: Stores the processed result in a specific path of the state input
  + OutputPath: Determines what data passes to the next state

A code snippet for these features is:
```yaml
{
    "ProcessOrder": {
        "Type": "Task",
        "Resource": "arn:aws:states:::lambda:invoke",
        "Parameters": {
            "FunctionName": "ProcessOrderFunction",
            "Payload": {
                "orderId.$": "$.orderId",
                "customerId.$": "$.customerId"
            }
        },
        "ResultSelector": {
            "orderStatus.$": "$.Payload.status",
            "processedDate.$": "$.Payload.timestamp"
        },
        "ResultPath": "$.orderProcessing",
        "OutputPath": "$.orderProcessing",
        "Next": "NotifyCustomer"
    }
}
```
- [Error handling](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html): Implement retry policies and catch errors at both the task and state machine levels.
- [Monitoring](https://docs.aws.amazon.com/step-functions/latest/dg/procedure-cw-metrics.html): Set up CloudWatch logs and metrics for your state machine to track executions and performance.

---

## Benefits of using Step Functions
Using Step Functions for rearchitecting scenarios bring the following benefits:

- **Reduced code complexity**: The business logic is now defined in Amazon States Language rather than distributed across multiple Lambda functions.
- **Improved maintainability**: Developers can make workflow changes by modifying the Amazon States Language, often modifying several Lambda functions.
- **Native AWS service integrations**: Step Functions offers direct integrations with over 200 AWS services, which you can use to connect and coordinate AWS resources without writing custom integration code.
- **Cost optimization**: By using direct service integrations, there are fewer Lambda invocations and reduced costs.
- **Long-running processes**: Step Functions can manage workflows that run for up to a year, beyond the 15-minute limit for Lambda functions.

---

## Conclusion

Rearchitecting Lambda-based applications with Step Functions can significantly improve maintainability, scalability, and operational efficiency. By moving orchestration logic into Step Functions and using its native service integrations, you can create more robust and manageable serverless applications.

While this post focused on a message distribution workflow, the principles apply to many serverless architectures. As you develop your applications, consider how Step Functions can help you build more resilient and scalable solutions.

To learn more about serverless architectures visit [Serverless Land](https://serverlessland.com/).