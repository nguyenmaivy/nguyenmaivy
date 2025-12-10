---
title: "Blog 5"

weight: 1
chapter: false
pre: " <b> 3.6. </b> "
---


# Triển khai ưu tiên tin nhắn với hàng đợi số lượng trên Amazon MQ cho RabbitMQ 

Tác giả: Akhil Melakunta và Vinodh Kannan Sadayamuthu 

Xuất bản: Ngày 23 tháng 07 năm 2025 

[Advanced (300)](https://aws.amazon.com/blogs/compute/category/learning-levels/advanced-300/), [Amazon MQ](https://aws.amazon.com/blogs/compute/category/messaging/amazon-mq/), [Serverless](https://aws.amazon.com/blogs/compute/category/serverless/), [Technical How-to](https://aws.amazon.com/blogs/compute/category/post-types/technical-how-to/) 

[Hàng đợi Quorum hiện đã có sẵn trên Amazon MQ for RabbitMQ từ phiên bản 3.13.](https://aws.amazon.com/blogs/compute/introducing-quorum-queues-on-amazon-mq-for-rabbitmq/) Hàng đợi Quorum là loại hàng đợi FIFO (First-In, First-Out) được sao chép, sử dụng [Raft Consensus Algorithm](https://raft.github.io/) để duy trì tính nhất quán của dữ liệu. Hàng đợi Quorum trên RabbitMQ phiên bản 3.13 thiếu một tính năng chính so với hàng đợi cổ điển: ưu tiên tin nhắn. Tuy nhiên, RabbitMQ phiên bản 4.0 đã hỗ trợ tính năng [ưu tiên tin nhắn](https://www.rabbitmq.com/blog/2024/08/28/quorum-queues-in-4.0#message-priorities), hoạt động khác với các ưu tiên tin nhắn của hàng đợi cổ điển. Việc di chuyển các ứng dụng từ hàng đợi cổ điển với ưu tiên tin nhắn sang hàng đợi Quorum trên [Amazon MQ for RabbitMQ](https://aws.amazon.com/amazon-mq/) đặt ra nhiều thách thức cho khách hàng. Bài viết này mô tả các phương pháp khác nhau để triển khai tính năng ưu tiên tin nhắn trong hàng đợi Quorum trên Amazon MQ for RabbitMQ.
Amazon MQ là dịch vụ môi giới tin nhắn được quản lý dành cho [Apache ActiveMQ](https://activemq.apache.org/) và [RabbitMQ](https://www.rabbitmq.com/), giúp đơn giản hóa việc thiết lập và vận hành các môi giới tin nhắn trên AWS.

## Tại sao việc ưu tiên tin nhắn lại quan trọng
Các hệ thống nhắn tin hiện đại yêu cầu xử lý tin nhắn khác nhau, tùy thuộc vào mức độ ưu tiên của doanh nghiệp. Một số tin nhắn nhạy cảm về thời gian hoặc quan trọng hơn những tin nhắn khác, và việc ưu tiên chúng có thể nâng cao hiệu quả và khả năng phản hồi của các ứng dụng. Việc ưu tiên tin nhắn cho phép một số tin nhắn nhất định được xử lý trước những tin nhắn khác, phù hợp với các ưu tiên của doanh nghiệp và giúp đảm bảo rằng các tin nhắn có giá trị cao hoặc quan trọng về thời gian nhận được sự chú ý cần thiết.
Việc ưu tiên tin nhắn giải quyết những thách thức kinh doanh quan trọng trong nhiều ngành. Tại các công ty bảo hiểm, việc này có thể đẩy nhanh quá trình xử lý yêu cầu bồi thường khẩn cấp bằng cách ưu tiên các tin nhắn có mức độ ưu tiên cao hơn các bản cập nhật chính sách định kỳ, giảm thời gian giải quyết. Các nhà sản xuất ô tô có thể đảm bảo rằng các cảnh báo quan trọng trên dây chuyền sản xuất và thông báo an toàn được ưu tiên hơn dữ liệu đo từ xa tiêu chuẩn, ngăn ngừa thời gian ngừng hoạt động tốn kém. Các công ty điện lực có thể ưu tiên các cảnh báo về sự ổn định lưới điện theo thời gian thực và thông báo mất điện, cho phép phản hồi nhanh hơn đối với các sự cố mất điện tiềm ẩn. Bằng cách triển khai ưu tiên tin nhắn, các ngành có thể tập trung sự chú ý ngay lập tức vào các hoạt động nhạy cảm về thời gian, đồng thời quản lý hiệu quả các quy trình thường xuyên trong cơ sở hạ tầng hiện có. Bằng cách sử dụng phương pháp này để chuyển đổi chiến lược truyền thông, các tổ chức có thể phản ứng nhanh chóng và hiệu quả hơn với các sự kiện quan trọng.

## So sánh mức độ ưu tiên tin nhắn của hàng đợi cổ điển và hàng đợi quorum

Trong phần này, hãy khám phá những khác biệt cơ bản giữa hàng đợi cổ điển và hàng đợi quorum về khả năng ưu tiên tin nhắn. Xem xét cách mỗi loại hàng đợi xử lý mức độ ưu tiên tin nhắn, các tính năng tích hợp sẵn và các cân nhắc chính.

### Ưu tiên tin nhắn với hàng đợi cổ điển

Trong [hàng đợi cổ điển](https://www.rabbitmq.com/docs/priority), RabbitMQ hỗ trợ mức độ ưu tiên tin nhắn từ 1 đến 255, với 1 là mức độ ưu tiên thấp nhất và 255 là mức độ ưu tiên cao nhất. Tuy nhiên, thường nên sử dụng phạm vi nhỏ hơn (ví dụ: 1–5) để có hiệu suất tốt hơn, vì RabbitMQ cần duy trì một hàng đợi con nội bộ cho mỗi mức độ ưu tiên từ 1 đến giá trị tối đa được cấu hình cho một hàng đợi nhất định. Phạm vi ưu tiên rộng hơn sẽ làm tăng chi phí CPU và bộ nhớ, điều này có thể ảnh hưởng đến hiệu suất của broker.
Hành vi hàng đợi ưu tiên trong hàng đợi cổ điển:
- Hàng đợi cổ điển yêu cầu tham số x-max-priority để xác định số lượng ưu tiên tối đa cho một hàng đợi nhất định
- Một thủ tục gửi một thông điệp với giá trị thuộc tính ưu tiên
- Người dùng không cần cấu hình đặc biệt để xử lý các mức ưu tiên
- Các thông điệp có mức ưu tiên cao hơn sẽ được gửi trước các thông điệp có mức ưu tiên thấp hơn
- Trong cùng một mức ưu tiên, các thông điệp được gửi theo thứ tự FIFO
- Các thông điệp không có thuộc tính ưu tiên được coi là có mức ưu tiên thấp nhất
- Các thông điệp có mức ưu tiên cao hơn mức tối đa của hàng đợi được coi là có mức ưu tiên cao nhất

> *Hình 1. *

Ví dụ code Python cho thực thi hàng đợi cổ điển với ưu tiên tin nhắn:
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
Đoạn mã trên minh họa việc ưu tiên tin nhắn trong RabbitMQ sử dụng hàng đợi cổ điển với cơ chế xử lý ưu tiên tích hợp sẵn. Việc triển khai kết nối với một broker RabbitMQ bằng [thư viện Python Pika](https://pika.readthedocs.io/en/stable/) và khai báo một trao đổi trực tiếp, một hàng đợi cổ điển với mức ưu tiên tối đa là 5. Tin nhắn sau đó được xuất bản vào hàng đợi duy nhất này với các giá trị ưu tiên được chỉ định rõ ràng (1 cho mức thấp, 2 cho mức trung bình và 5 cho mức ưu tiên cao). Khi người dùng truy xuất tin nhắn từ hàng đợi này, RabbitMQ sẽ gửi các tin nhắn có mức ưu tiên cao hơn trước.
### Ưu tiên tin nhắn với hàng đợi Quorum
Không giống như hàng đợi cổ điển, hàng đợi Quorum trong Rabbit MQ 3.13 không hỗ trợ ưu tiên tin nhắn gốc. Tuy nhiên, có những mô hình hiệu quả mà bạn có thể triển khai để đạt được mức độ ưu tiên tin nhắn với hàng đợi Quorum.
#### *Sử dụng các hàng đợi riêng biệt cho các mức độ ưu tiên khác nhau*
Một phương pháp đơn giản là tạo nhiều hàng đợi Quorum, mỗi hàng đợi dành riêng cho các mức độ ưu tiên khác nhau. Ví dụ: bạn có thể có một hàng đợi có mức độ ưu tiên cao và một hàng đợi có mức độ ưu tiên thấp. Sử dụng RabbitMQ để trao đổi và liên kết các khóa định tuyến tin nhắn đến các hàng đợi thích hợp dựa trên mức độ ưu tiên của chúng, cho phép hệ thống xử lý các tin nhắn có mức độ ưu tiên cao nhanh hơn, như được hiển thị trong hình sau.

> *Hình 2. *

Ví dụ để triển khai xử lý mức độ ưu tiên bằng cách sử dụng hàng đợi quorum riêng biệt:
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
Đoạn mã trên minh họa phương pháp ưu tiên tin nhắn trong RabbitMQ bằng cách sử dụng các hàng đợi quorum riêng biệt cho các mức độ ưu tiên khác nhau (thấp, trung bình và cao). Việc triển khai sử dụng thư viện Python Pika để kết nối với máy chủ RabbitMQ, một trao đổi trực tiếp và ba hàng đợi quorum riêng biệt cho các mức độ ưu tiên khác nhau, đồng thời xuất bản tin nhắn đến các khóa định tuyến khác nhau với mức độ ưu tiên khác nhau.
#### *Logic ưu tiên tùy chỉnh trên consumer*
Triển khai logic tùy chỉnh trong ứng dụng của bạn để xử lý tin nhắn dựa trên mức độ ưu tiên của chúng. Ví dụ: bạn có thể sử dụng tiêu đề hoặc siêu dữ liệu để xác định mức độ ưu tiên của tin nhắn, sau đó sử dụng thông tin này để định tuyến tin nhắn đến các hàng đợi khác nhau hoặc xử lý chúng theo một thứ tự cụ thể.
Hàng đợi có mức độ ưu tiên cao hơn nên sử dụng nhiều consumer hơn hoặc consumer có tài nguyên được phân bổ cao hơn để xử lý tin nhắn nhanh hơn so với hàng đợi có mức độ ưu tiên thấp hơn. Sử dụng phương thức basic.qos ([prefetch](https://www.rabbitmq.com/docs/confirms#channel-qos-prefetch)) ở chế độ xác nhận thủ công trên consumer của bạn để giới hạn số lượng tin nhắn có thể được gửi đi bất cứ lúc nào và cho phép ưu tiên tin nhắn. basic.qos là một giá trị mà consumer thiết lập khi kết nối với hàng đợi. Giá trị này cho biết số lượng tin nhắn mà consumer có thể xử lý cùng một lúc. Phương pháp này được thể hiện ở hình sau.

> *Hình 3. *


*Lưu ý*: Giải pháp này triển khai ưu tiên tin nhắn theo cơ chế nỗ lực tối đa. Có khả năng các tin nhắn có mức độ ưu tiên thấp và trung bình sẽ được xử lý trước các tin nhắn có mức độ ưu tiên cao.

## Kết luận
Việc ưu tiên tin nhắn trong RabbitMQ broker trên Amazon MQ có những cân nhắc khác nhau đối với hàng đợi cổ điển và hàng đợi quorum. Việc sử dụng hàng đợi quorum đòi hỏi một cách tiếp cận thận trọng do RabbitMQ thiếu hỗ trợ gốc cho việc ưu tiên tin nhắn. Bằng cách sử dụng các hàng đợi riêng biệt và logic tùy chỉnh, bạn có thể đạt được mức độ ưu tiên hiệu quả trong khi vẫn duy trì tính khả dụng cao và tính nhất quán mà hàng đợi quorum mang lại. Hãy áp dụng các chiến lược này để tối ưu hóa cơ sở hạ tầng nhắn tin, nâng cao khả năng phản hồi của ứng dụng và đảm bảo các tin nhắn quan trọng được xử lý kịp thời.
Chúng tôi khuyên bạn nên sử dụng hàng đợi quorum làm loại hàng đợi sao chép ưu tiên trên các broker RabbitMQ 3.13. Để biết thêm chi tiết, hãy xem [tài liệu Amazon MQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/quorum-queues.html). Để biết thêm thông tin, hãy xem [hàng đợi quorum](https://www.rabbitmq.com/docs/quorum-queues).
Để tìm hiểu thêm, hãy xem [Amazon MQ dành cho Rabbit MQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/working-with-rabbitmq.html).
