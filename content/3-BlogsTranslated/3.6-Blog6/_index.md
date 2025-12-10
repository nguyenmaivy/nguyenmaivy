---
title: "Blog 5"

weight: 1
chapter: false
pre: " <b> 3.6. </b> "
---


# Implementing message prioritization with quorum queues on Amazon MQ for RabbitMQ

by Akhil Melakunta and Vinodh Kannan Sadayamuthu on 23 JUL 2025 in[Advanced (300)](https://aws.amazon.com/blogs/compute/category/learning-levels/advanced-300/), [Amazon MQ](https://aws.amazon.com/blogs/compute/category/messaging/amazon-mq/), [Serverless](https://aws.amazon.com/blogs/compute/category/serverless/), [Technical How-to](https://aws.amazon.com/blogs/compute/category/post-types/technical-how-to/) 

[Quorum queues are now available on Amazon MQ for RabbitMQ from version 3.13.](https://aws.amazon.com/blogs/compute/introducing-quorum-queues-on-amazon-mq-for-rabbitmq/) Quorum queues are a replicated First-In, First-Out (FIFO) queue type that uses the [Raft consensus algorithm](https://raft.github.io/)  to maintain data consistency. Quorum queues on RabbitMQ version 3.13 lack one key feature compared to classic queues: message prioritization. However, RabbitMQ version 4.0 introduced support for [message priority](https://www.rabbitmq.com/blog/2024/08/28/quorum-queues-in-4.0#message-priorities), which behaves differently than classic queue message priorities. Migrating applications from classic queues with message priority to quorum queues on [Amazon MQ for RabbitMQ](https://aws.amazon.com/amazon-mq/) presents challenges for customers. This post describes the different approaches to implementing message prioritization in quorum queues in Amazon MQ for RabbitMQ.

Amazon MQ is a managed message broker service for [Apache ActiveMQ](https://activemq.apache.org/) and [RabbitMQ](https://www.rabbitmq.com/) that simplifies setting up and operating message brokers on AWS.

## Why message prioritization matters

Modern messaging systems require handling messages differently, depending on the business priority. Some messages are more time-sensitive or critical than others and prioritizing them can enhance the efficiency and responsiveness of applications. Message prioritization allows certain messages to be processed before others, aligning with business priorities and helping to ensure that high-value or time-critical messages receive the attention they need.

Message prioritization addresses critical business challenges across multiple industries. In insurance companies, it can expedite urgent claim processing by prioritizing high-priority messages over routine policy updates, reducing settlement times. Automotive manufacturers can make sure that critical production line alerts and safety notifications take precedence over standard telemetry data, preventing costly downtime. Energy utilities can prioritize real-time grid stability alerts and outage notifications, enabling faster responses to potential blackouts. By implementing message priority, industries can direct immediate attention to time-sensitive operations while efficiently managing routine processes within existing infrastructure. By using this approach to transform their communication strategies, organizations can respond more quickly and effectively to critical events.

## Classic queues compared to quorum queues message prioritization

In this section, explore the fundamental differences between classic queues and quorum queues when it comes to message prioritization capabilities. Examine how each queue type handles message priority, the built-in features available, and key considerations.

### Message prioritization with classic queues
In [classic queues](https://www.rabbitmq.com/docs/priority), RabbitMQ supports message priorities ranging from 1 to 255, with 1 being the lowest priority and 255 being the highest. However, it’s generally recommended to use a smaller range (for example, 1–5) for better performance, because RabbitMQ needs to maintain an internal sub-queue for each priority from 1 up to the maximum value configured for a given queue. A wider priority range adds more CPU and memory cost, which can impact broker performance.
Priority queue behavior in classic queues:
- Classic queues require x-max-priority argument to define the maximum number of priorities for a given queue
- A procedure sends a message with a priority property value
Consumers don’t need special configuration to handle priorities
Messages with higher priority are delivered before messages with lower priority
- Within the same priority level, messages are delivered in FIFO order
- Messages without a priority property are treated as if their priority is lowest
- Messages with a priority that is higher than the queue’s maximum are treated as if they were published with the maximum priority

> *Image 1. *

Example Python code for classic queue implementation with message priority:
``` python
#!/usr/bin/env python
import pika
import ssl
# Set up SSL context for secure connection
context = ssl.SSLContext(ssl.PROTOCOL_TLSv1_2)
# Define credentials
credentials = pika.PlainCredentials('username', 'password') # Replace with actual credentials
# Set up connection parameters for Amazon MQ RabbitMQ broker
connection_parameters = pika.ConnectionParameters(
    host='b-example.mq.us-west-2.on.aws', # Replace with actual broker endpoint
    port=5671,
    credentials=credentials,
    ssl_options=pika.SSLOptions(context)
)
# Establish connection and create a channel
connection = pika.BlockingConnection(connection_parameters)
channel = connection.channel()
# Declare a direct exchange
# - direct exchanges route messages based on routing key
channel.exchange_declare(
    exchange='priority_exchange',
    exchange_type='direct',
)
# Declare a priority queue
# - x-max-priority=5 sets maximum priority level (0-5)
# - x-queue-type=classic specifies classic queue implementation
channel.queue_declare(
    queue='classic_priority_queue',
    arguments={
        'x-max-priority': 5,
        'x-queue-type': "classic"
    }
)
# Bind queue to exchange with routing key
# - This connects the queue to the exchange
# - Messages sent to the exchange with matching routing key will be routed to this queue
channel.queue_bind(
    queue='classic_priority_queue',
    exchange='priority_exchange',
    routing_key='priority_queue'
)
# Publish messages with different priorities
# Low priority message (priority=1)
channel.basic_publish(
    exchange='priority_exchange',
    routing_key='priority_queue',
    body='Low priority message',
    properties=pika.BasicProperties(priority=1)
)
print(" [x] Sent 'Low priority message'")
# Medium priority message (priority=2)
channel.basic_publish(
    exchange='priority_exchange',
    routing_key='priority_queue',
    body='Medium priority message',
    properties=pika.BasicProperties(priority=2)
)
print(" [x] Sent 'Medium priority message'")
# High priority message (priority=5)
channel.basic_publish(
    exchange='priority_exchange',
    routing_key='priority_queue',
    body='High priority message',
    properties=pika.BasicProperties(priority=5)
)
print(" [x] Sent 'High priority message'")
# Close the connection
connection.close()
```

The preceding code demonstrates message prioritization in RabbitMQ using a classic queue with built-in priority handling. The implementation connects to a RabbitMQ broker using the [Python Pika library](https://pika.readthedocs.io/en/stable/) and declares a direct exchange, a classic queue with a maximum priority level of 5. Messages are then published to this single queue with explicitly assigned priority values (1 for low, 2 for medium, and 5 for high priority). When consumers fetch messages from this queue, RabbitMQ will deliver higher priority messages first.

### Message prioritization with quorum queues
Unlike classic queues, quorum queues in Rabbit MQ 3.13 don’t support message prioritization natively. However, there are effective patterns that you can implement to achieve message priority with Quorum queues.
#### *Using separate queues for different priorities*
A straightforward method is to create multiple quorum queues, each dedicated to different priority levels. For example, you might have a high-priority queue and a low-priority queue. Using RabbitMQ exchange and binding key route messages to the appropriate queues based on their priority, allowing the system to process high-priority messages more promptly, as shown in the following figure.

> *Image 2. *

Example to implement priority handling using separate quorum queues:
``` python
#!/usr/bin/env python
import pika
import ssl
# Set up SSL context for secure connection
context = ssl.SSLContext(ssl.PROTOCOL_TLSv1_2)
# Define credentials
credentials = pika.PlainCredentials('username', 'password') #Replace with actual credentials
# Set up connection parameters for Amazon MQ RabbitMQ broker
connection_parameters = pika.ConnectionParameters(
    host='b-example.mq.us-west-2.on.aws',
    port=5671,
    credentials=credentials,
    ssl_options=pika.SSLOptions(context)
)
# Establish connection and create a channel
connection = pika.BlockingConnection(connection_parameters)
channel = connection.channel()
# Declare a direct exchange
# - Direct exchanges route messages based on routing key
channel.exchange_declare(
    exchange='priority_exchange_qq',
    exchange_type='direct'
)
# Create separate quorum queues for different priority levels
# Low priority queue
channel.queue_declare(
    queue='low_priority_queue',
    durable=True,
    arguments={
        'x-queue-type': "quorum" 
    }
)
# Bind the low priority queue to the exchange with a specific routing key
# - This creates a rule that messages sent to 'priority_exchange' with routing_key='low_priority_1'
# - will be routed to the 'low_priority_queue'
channel.queue_bind(
    queue='low_priority_queue',
    exchange='priority_exchange_qq',
    routing_key='low_priority_1'
)
# Medium priority queue
channel.queue_declare(
    queue='medium_priority_queue',
    durable=True,
    arguments={
        'x-queue-type': "quorum" 
    }
)
# Bind the medium priority queue to the exchange with a specific routing key
# - Messages with routing_key='medium_priority_2' will be directed to the 'medium_priority_queue'
channel.queue_bind(
    queue='medium_priority_queue',
    exchange='priority_exchange_qq',
    routing_key='medium_priority_2'
)
# High priority queue
channel.queue_declare(
    queue='high_priority_queue',
    durable=True,
    arguments={
        'x-queue-type': "quorum" 
    }
)
# Bind the high priority queue to the exchange with a specific routing key
# - Messages with routing_key='high_priority_2' will be directed to the 'high_priority_queue'
channel.queue_bind(
    queue='high_priority_queue',
    exchange='priority_exchange_qq',
    routing_key='high_priority_5'
)
# Publish messages to different priority queues
print(" [x] Publishing messages to different priority queues")
# Low priority message
channel.basic_publish(
    exchange='priority_exchange_qq',  
    routing_key='low_priority_1',
    body='Low priority message'
)
print(" [x] Sent 'Low priority message'")
# Medium priority message
channel.basic_publish(
    exchange='priority_exchange_qq', 
    routing_key='medium_priority_2',
    body='Medium priority message'
)
print(" [x] Sent 'Medium priority message'")
# High priority message
channel.basic_publish(
    exchange='priority_exchange_qq', 
    routing_key='high_priority_5',
    body='High priority message'
)
print(" [x] Sent 'High priority message'")
# Close the connection
connection.close()
print(" [x] Connection closed")
```

The preceding code demonstrates a message prioritization approach in RabbitMQ using separate quorum queues for different priority levels (low, medium, and high). The implementation uses the Python Pika library to connect to a RabbitMQ server, a direct exchange and three separate quorum queues for different priority levels, and publish messages to different routing keys with different priority.

#### *Custom priority logic on consumers*

Implement custom logic within your application to handle messages based on their priority. For example, you can use headers or metadata to determine the priority of a message and then use this information to route messages to different queues or handle them in a specific order.
Higher priority queues should use more consumers or consumers with higher resources allocated to process messages more quickly than lower priority queues. Use the basic.qos ([prefetch](https://www.rabbitmq.com/docs/confirms#channel-qos-prefetch)) method in manual acknowledgement mode on your consumers to limit the number of messages that can be out for delivery at any time and allow messages to be prioritized. basic.qos is a value a consumer sets when connecting to a queue. It indicates how many messages the consumer can handle at one time. This method is shown in the following figure.

> *Image 3. *

*Note:* This solution implements message priority on a best-effort basis. There is a possibility that low and medium priority messages may be processed before high priority messages.

## Conclusion
Message prioritization in RabbitMQ brokers on Amazon MQ has different considerations for classic and quorum queues. Using quorum queues requires a thoughtful approach because of the lack of native support for message proritization in RabbitMQ. By employing separate queues and custom logic, you can achieve effective prioritization while maintaining the high availability and consistency that quorum queues offer. Embrace these strategies to optimize your messaging infrastructure, enhance application responsiveness, and make sure that critical messages are processed in a timely manner.
We recommend that you adopt quorum queues as the preferred replicated queue type on RabbitMQ 3.13 brokers. For more details, see [Amazon MQ documentation](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/quorum-queues.html). For more information, see [quorum queues](https://www.rabbitmq.com/docs/quorum-queues).
To learn more, see [Amazon MQ for Rabbit MQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/working-with-rabbitmq.html).
