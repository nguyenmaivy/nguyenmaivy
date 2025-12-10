---
title: "Blog 7"

weight: 1
chapter: false
pre: " <b> 3.7. </b> "
---



# Tối ưu hóa quy trình làm việc Serverless trên AWS: Từ AWS Lambda orchestration đến AWS Step Functions

Bài đăng trên blog này thảo luận về [AWS Lambda](https://aws.amazon.com/lambda/) với tư cách là [mẫu chống điều phối](https://docs.aws.amazon.com/lambda/latest/operatorguide/anti-patterns.html) và cách thiết kế lại các giải pháp serverless bằng cách sử dụng AWS Step Functions với tích hợp gốc.

Step Functions là dịch vụ quy trình làm việc không có máy chủ mà bạn có thể sử dụng để xây dựng các ứng dụng phân tán, tự động hóa quy trình, điều phối các dịch vụ vi mô cũng như tạo đường dẫn dữ liệu và máy học (ML). Step Functions cung cấp khả năng tích hợp gốc với hơn [200 dịch vụ AWS](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-services.html) bên cạnh các API bên ngoài của bên thứ ba. Bạn có thể sử dụng các tiện ích tích hợp này để triển khai các giải pháp sẵn sàng sản xuất với ít nỗ lực hơn, giảm độ phức tạp của mã, cải thiện khả năng bảo trì lâu dài và giảm thiểu nợ kỹ thuật khi vận hành ở quy mô lớn.


---

## The Lambda as orchestrator anti-pattern

Hãy cùng xem xét một anti-pattern phổ biến: sử dụng một Lambda function như một orchestrator để phân phối thông điệp qua nhiều kênh khác nhau. Hãy tưởng tượng tình huống thực tế sau đây — một hệ thống cần gửi thông báo (notifications) thông qua SMS hoặc email tùy theo user preferences, như được minh họa trong sơ đồ sau.

![](/images/3-Blogstranslated/Blog7/anti-pattern.png)

Các ví dụ về tải trọng cho kịch bản này là:
1. Chỉ gửi SMS:
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
2. Chỉ gửi email:
```yaml
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

3. Gửi cả SMS và email:
```yaml
{
    "body": {
        "channel": "both",
        "message": "Hello from AWS Lambda!",
        "phoneNumber": "+1234567890",
        "email": {
            "to": "recipient@example.com",
            "subject": "Test Notification",
            "from": "sender@example.com"
        },
        "metadata": {
            "priority": "high",
            "category": "notification"
        }
    }
}
```
Đây là cách nó thường bắt đầu - với hàm Lambda đóng vai trò là người điều phối:

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


Cách tiếp cận này có các vấn đề sau:
- **Complex error handling**: Orchestrator cần quản lý lỗi từ nhiều lần gọi function khác nhau.
- **Tight coupling**: Các function phụ thuộc trực tiếp vào nhau.
- **Limited execution time**: Orchestrator Lambda function tiếp tục chạy trong khi các sub Lambda function đang thực thi. Điều này có thể dẫn đến việc orchestrator Lambda function bị timeout.
- **Idle resources**: Bởi vì orchestrator Lambda function đang ở trạng thái nhàn rỗi trong khi chờ kết quả trả về từ các Lambda function khác, trong trường hợp này, người dùng phải trả tiền cho tài nguyên không hoạt động.

---

## Rearchitecting with Step Functions

Bạn có thể xây dựng lại logic bằng cách sử dụng Step Functions và Amazon States Language để thay thế Lambda orchestrator function. Bạn có thể sử dụng Choice state trong Amazon States Language để xác định các điều kiện logic nhằm theo một nhánh cụ thể. Cách tiếp cận này giúp giảm độ phức tạp trong việc bảo trì code vì bạn định nghĩa các điều kiện bằng Amazon States Language. Bạn cũng có thể sử dụng nó để mở rộng chức năng với các thay đổi tối thiểu trong codebase.

Sơ đồ workflow Step Functions sau đây minh họa phiên bản được thiết kế lại của Orchestrator Lambda function trước đó:

![](/images/3-Blogstranslated/Blog7/step-functions.png)

Ngôn ngữ trạng thái Amazon sau đây thể hiện quy trình làm việc:
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
Cách triển khai Step Functions này mang lại một số lợi ích sau:
- **Native service integration**: Tích hợp trực tiếp với [Amazon Simple Notification Service (Amazon SNS)](https://aws.amazon.com/sns), [Amazon Simple Email Service (Amazon SES)](https://aws.amazon.com/ses), [Amazon DynamoDB](https://aws.amazon.com/dynamodb), và [Amazon CloudWatch](https://aws.amazon.com/cloudwatch) giúp loại bỏ nhu cầu sử dụng các wrapper Lambda functions.
- **Visual workflow**: Luồng thực thi có thể được quan sát và duy trì dễ dàng thông qua AWS Management Console.
- **Built-in error handling**: Chính sách Retry và error states có thể được định nghĩa một cách declaratively.
- **Parallel execution**: Parallel state xử lý việc gửi dữ liệu đến nhiều kênh một cách hiệu quả.
Simplified logic: Choice state thay thế cho các câu lệnh if-else phức tạp.
- **Centralized data flow**: Input và output được quản lý nhất quán giữa các states.
- **Enhanced workflow duration capabilities**: Step Functions Standard workflows hỗ trợ các phiên thực thi có thể chạy lên đến một năm, so với giới hạn 15 phút của Lambda functions.

---

## Comparing Lambda function as orchestrator to Step Functions

Tóm tắt các đặc điểm khác nhau được triển khai giữa Lambda function as orchestrator và Step Functions được thể hiện trong bảng sau:

| Tính năng               | Lambda function as orchestrator                                  | Step Functions                                                |
| ----------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------- |
| **Logic điều phối**     | Được triển khai trong Python với các câu lệnh if-else lồng nhau. | Được định nghĩa khai báo bằng trạng thái Choice               |
| **Phân phối đa kênh**   | Gọi hàm tuần tự. Thực thi song song sử dụng logic của hàm.       | Thực thi song song sử dụng trạng thái Parallel                |
| **Tích hợp dịch vụ**    | Yêu cầu SDK calls hoặc các Lambda functions riêng biệt.          | Tích hợp trực tiếp với các dịch vụ AWS (Amazon SNS, DynamoDB) |
| **Xử lý lỗi**           | Khối try-except tùy chỉnh trong Python.                          | Trạng thái lỗi tích hợp sẵn và chính sách thử lại             |
| **Duy trì dữ liệu**     | Code tùy chỉnh để tương tác với DynamoDB.                        | Tích hợp DynamoDB gốc với tác vụ putItem                      |
| **Ghi nhật ký số liệu** | Code tùy chỉnh để gọi CloudWatch.                                | Tích hợp SDK CloudWatch Metrics                               |

---

## Cân nhắc thực hiện

Xem xét các yếu tố sau khi re-architecting một Lambda function orchestrator sang Step Functions:
- [State machine type](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html): Chọn giữa Standard (thời gian chạy lên đến 1 năm) và Express (thời gian chạy lên đến 5 phút) tùy theo nhu cầu của bạn.
- [Input/output management](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-input-output-filtering.html): Việc thao tác với parameters giúp giảm khối lượng phát triển và mang lại các lựa chọn linh hoạt hơn để triển khai workflow:
  + Parameters: Chọn các trường input cụ thể để truyền sang state tiếp theo.
  + ResultSelector: Lọc state response để chỉ bao gồm các trường có liên quan.
  + ResultPath: Lưu processed result vào một path cụ thể trong state input.
  + OutputPath: Xác định dữ liệu nào sẽ được truyền sang state tiếp theo

Một code snippet cho các tính năng này như sau:

```yaml
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
- [Xử lý lỗi](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html): Triển khai retry policies và bắt lỗi (catch errors) ở cả hai cấp độ: task và state machine.
- [Giám sát](https://docs.aws.amazon.com/step-functions/latest/dg/procedure-cw-metrics.html): Thiết lập CloudWatch logs và metrics cho state machine của bạn để theo dõi executions và performance.


---

## Lợi ích của việc sử dụng Step Functions

Việc sử dụng Step Functions trong các kịch bản rearchitecting mang lại những lợi ích sau:
- **Reduced code complexity**: Business logic giờ đây được định nghĩa trong Amazon States Language thay vì phân tán qua nhiều Lambda functions.
- **Improved maintainability**: Các developer có thể thay đổi workflow bằng cách chỉnh sửa Amazon States Language, thay vì phải sửa đổi nhiều Lambda functions.
- **Native AWS service integrations**: Step Functions cung cấp direct integrations với hơn 200 AWS services, cho phép bạn kết nối và phối hợp các AWS resources mà không cần viết custom integration code.
- **Cost optimization**: Bằng cách sử dụng direct service integrations, số lượng Lambda invocations giảm đi, giúp tiết kiệm chi phí.
- **Long-running processes**: Step Functions có thể quản lý workflows chạy đến 1 năm, vượt xa giới hạn 15 phút của Lambda functions.


---

## Kết luận

Rearchitecting các Lambda-based applications bằng Step Functions có thể cải thiện đáng kể maintainability, scalability, và operational efficiency. Bằng cách chuyển orchestration logic vào Step Functions và tận dụng native service integrations, bạn có thể tạo ra các serverless applications mạnh mẽ và dễ quản lý hơn.
Mặc dù bài viết này tập trung vào message distribution workflow, nhưng các nguyên tắc được trình bày có thể áp dụng cho nhiều serverless architectures khác nhau. Khi bạn phát triển ứng dụng của mình, hãy cân nhắc cách Step Functions có thể giúp bạn xây dựng các giải pháp resilient và scalable hơn.
Để tìm hiểu thêm về serverless architectures, hãy truy cập [Serverless Land](https://serverlessland.com/).