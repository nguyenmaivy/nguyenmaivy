---
title: "Blog 9"

weight: 1
chapter: false
pre: " <b> 3.9. </b> "
---



# Giới thiệu phiên bản v2 của Powertool for AWS Lambda (Java)

Các ứng dụng hiện đại ngày nay ngày càng dựa vào các công nghệ Serverless như Amazon Web Services (AWS) [Lambda](https://aws.amazon.com/lambda/) để đạt được khả năng scalability, cost efficiency, và agility. [Serverless Applications Lens trong AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/) tập trung vào cách thiết kế, triển khai, và kiến trúc hóa các ứng dụng Serverless của bạn để vượt qua những thách thức này.

[Powertools for AWS](https://github.com/aws-powertools) Lambda là một developer toolkit giúp bạn triển khai Serverless best practices và chuyển đổi trực tiếp các khuyến nghị trong AWS Well-Architected thành các tiện ích có thể hành động, thân thiện với developer. Sau khi cộng đồng thành công trong việc áp dụng Powertools cho AWS bằng Python, Java, TypeScript, và .NET, bài viết này công bố phiên bản chính thức của Powertools for AWS Lambda (Java) v2, đi kèm các cải tiến lớn về hiệu năng, nâng cấp các core utilities, và một Kafka utility hoàn toàn mới.

Powertools for AWS (Java) v2 bao gồm 3 core utilities được cập nhật:

- **Logging**: Mô-đun logging được thiết kế lại theo phong cách Java idiomatic, cung cấp structured logging giúp dễ dàng tổng hợp và phân tích log.
- **Metrics**: Trải nghiệm metrics được cải thiện, cho phép thu thập custom metrics bằng [CloudWatch Embedded Metric Format (EMF)](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Embedded_Metric_Format_Specification.html).
- **Tracing**: Cách tiếp cận dựa trên annotation để thu thập distributed tracing data với [AWS X-Ray](https://aws.amazon.com/xray/), giúp trực quan hóa và phân tích luồng request.

Các tính năng mới trong v2:

- **GraalVM native image support**: Hỗ trợ native image cho GraalVM trên tất cả các core utilities, giúp giảm thời gian Lambda cold start lên đến 75.61% (p95).
- **Kafka utility**: Tích hợp với [Amazon Managed Streaming for Apache Kafka (Amazon MSK)](https://aws.amazon.com/msk/) và self-managed Kafka event sources trên Lambda, cho phép deserialize trực tiếp thành Kafka native types như ConsumerRecords.

Xem thêm chi tiết về hướng dẫn nâng cấp trong tài liệu [upgrade guide](https://docs.powertools.aws.dev/lambda/java/latest/upgrade/).


---

## Bắt đầu với Powertools for AWS Lambda (Java) v2

Powertools for AWS Lambda (Java) v2 có sẵn dưới dạng Java package trên Maven Central và tích hợp với các công cụ build phổ biến như Maven và Gradle. Bài viết này tập trung vào ví dụ triển khai bằng Maven để bạn có thể bắt đầu nhanh chóng. Các ví dụ về Gradle được cung cấp trong [tài liệu](https://docs.powertools.aws.dev/lambda/java/) và [repository ví dụ](https://github.com/aws-powertools/powertools-lambda-java/tree/main/examples).
Toolkit này tương thích với Java 11 trở lên, đảm bảo bạn có thể sử dụng các tính năng Java hiện đại khi xây dựng ứng dụng Serverless. Hướng dẫn cài đặt từng utility được trình bày trong các phần sau, và cấu hình đầy đủ có sẵn trong [tài liệu Powertools](https://docs.powertools.aws.dev/lambda/java/).


---

## Logging

Utility Logging giúp triển khai structured logging khi chạy trên Lambda mà vẫn sử dụng được các thư viện logging quen thuộc của Java như slf4j, log4j, và logback. Phiên bản v2 của Logging cho phép bạn:
- Xuất structured JSON logs có thêm thông tin từ Lambda context.
- Chọn backend logging yêu thích: log4j2 hoặc logback.
- Thêm structured arguments vào log, được serialize thành nested JSON objects.
- Thêm global log keys bằng [Mapped Diagnostic Context (MDC)](https://www.slf4j.org/apidocs/org/slf4j/MDC.html) của slf4j.

Ví dụ thêm dependency vào Maven project (log4j2 backend):
```yaml
<!-- In the dependencies section -->
<dependency>
    <groupId>software.amazon.lambda</groupId>
    <artifactId>powertools-logging-log4j</artifactId>
    <!-- Alternatively, if you wish to use the logback backend
    <artifactId>powertools-logging-logback</artifactId> 
    -->
    <version>2.1.1</version>
</dependency>
<!-- In the build plugins section -->
<plugin>
    <groupId>dev.aspectj</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <configuration>
        <aspectLibraries>
            <aspectLibrary>
                <groupId>software.amazon.lambda</groupId>
                <artifactId>powertools-logging</artifactId>
                <version>2.1.1</version>
            </aspectLibrary>
        </aspectLibraries>
    </configuration>
</plugin>
```
Tạo một ứng dụng JsonTemplateLayout tùy chỉnh trong tệp log4j2.xml của bạn:
```yaml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="JsonAppender" target="SYSTEM_OUT">
            <JsonTemplateLayout eventTemplateUri="classpath:LambdaJsonLayout.json" />
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="JsonLogger" level="INFO" additivity="false">
            <AppenderRef ref="JsonAppender"/>
        </Logger>
        <Root level="info">
            <AppenderRef ref="JsonAppender"/>
        </Root>
    </Loggers>
</Configuration>
```
Để thêm tính năng ghi nhật ký có cấu trúc vào các hàm của bạn, hãy áp dụng chú thích @Logging cho trình xử lý Lambda của bạn và sử dụng [slf4j Java API](https://www.slf4j.org/) quen thuộc khi viết câu lệnh nhật ký. Điều này cho phép bạn áp dụng tiện ích ghi nhật ký mà không cần tái cấu trúc mã chính. Powertools xử lý việc định tuyến tới chương trình phụ trợ ghi nhật ký chính xác cho bạn. Ví dụ sau đây cho thấy cách thêm khóa nhật ký chung bằng MDC và thêm đối số mục nhập có cấu trúc vào thông điệp nhật ký của bạn:
```yaml
public class App implements RequestHandler<SQSEvent, String> {
    private static final Logger log = LoggerFactory.getLogger(App.class);

    @Logging
    public String handleRequest(final SQSEvent input, final Context context) {
        // Add a global log key using Mapped Diagnostic Context MDC
        MDC.put("myCustomKey", "willBeLoggedForAllLogStatements");

        // Log a message with a structured argument (any JSON serializable Object)
        log.info("My message", entry("anotherCustomKey", Map.of("nested", "object")));

        // ... return response
    }
}
```
Lambda gửi đầu ra có định dạng JSON sau tới [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) (lưu ý cách Java Map được tự động tuần tự hóa thành đối tượng JSON):
```yaml
{
  "level": "INFO",
  "message": "My message",
  "cold_start": true,
  "function_arn": "arn:aws:lambda:us-east-1:012345678912:function:AppFunction",
  "function_memory_size": 512,
  "function_name": "AppFunction",
  "function_request_id": "0150a2a4-c5aa-4277-9345-17bad039f6c0",
  "function_version": "$LATEST",
  "sampling_rate": 0.1,
  "service": "powertools-java-sample",
  "timestamp": "2025-05-20T08:35:28.565Z",
  "myCustomKey": "willBeLoggedForAllLogStatements",
  "anotherCustomKey": {
    "nested": "object"
  }
}
```


---

## Metrics

CloudWatch cung cấp các service metrics tích hợp sẵn giúp theo dõi throughput của ứng dụng, tỷ lệ lỗi, và mức sử dụng tài nguyên. Tuy nhiên, người dùng cũng cần thu thập các custom metrics đặc thù cho workload của họ — những metric có liên quan trực tiếp đến bài toán kinh doanh, theo đúng best practices trong AWS Well-Architected Framework.

Powertools for AWS (Java) cho phép bạn tạo custom metrics một cách bất đồng bộ (asynchronously) bằng cách xuất dữ liệu metric theo định dạng CloudWatch EMF (Embedded Metric Format) trực tiếp ra standard output — một cách tiếp cận không yêu cầu thêm bất kỳ cấu hình nào khác. Dịch vụ Lambda sẽ tự động gửi các metric có định dạng EMF đó đến CloudWatch thay bạn.

Metrics utility cho phép bạn:

- Tạo custom metrics bất đồng bộ bằng CloudWatch EMF.
- Giảm độ trễ (latency) nhờ tránh việc publish metric đồng bộ.
- Tự động theo dõi cold starts trong một custom CloudWatch metric.
- Loại bỏ việc phải xác thực thủ công output của bạn so với EMF specification.
- Giữ code gọn gàng, không cần flush thủ công ra standard output.

Để thêm Metrics utility vào project của bạn, hãy thêm dependency Maven sau đây:
```yaml
<dependency>
    <groupId>software.amazon.lambda</groupId>
    <artifactId>powertools-metrics</artifactId>
    <version>2.1.1</version>
</dependency>
<!-- In the build plugins section -->
<plugin>
    <groupId>dev.aspectj</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <configuration>
        <aspectLibraries>
            <aspectLibrary>
                <groupId>software.amazon.lambda</groupId>
                <artifactId>powertools-metrics</artifactId>
                <version>2.1.1</version>
            </aspectLibrary>
        </aspectLibraries>
    </configuration>
</plugin>
```
Để thêm số liệu tùy chỉnh vào hàm Lambda, hãy đặt chú thích @FlushMetrics trên trình xử lý Lambda của bạn. Thư viện đảm nhiệm việc xác thực và chuyển số liệu của bạn sang đầu ra tiêu chuẩn trước khi hàm Lambda kết thúc. Ví dụ sau đây cho thấy cách bạn có thể tự động nắm bắt số liệu khởi động nguội và đưa ra số liệu tùy chỉnh của riêng mình:
```yaml
public class App implements RequestHandler<SQSEvent, String> {
    private static final Logger log = LoggerFactory.getLogger(App.class);
    private static final Metrics metrics = MetricsFactory.getMetricsInstance();
    // This configures a default namespace and service dimension for all metrics
    @FlushMetrics(namespace = "ServerlessAirline", service = "payment", captureColdStart = true)
    public String handleRequest(final SQSEvent input, final Context context) {
        // The Metrics instance is a singleton
        metrics.addMetric("CustomMetric1", 1, MetricUnit.COUNT);
        // Publish metrics with non-default configuration options
        DimensionSet dimensionSet = new DimensionSet();
        dimensionSet.addDimension("Service", "AnotherService");
        metrics.flushSingleMetric("CustomMetric2", 1, MetricUnit.COUNT, "AnotherNamespace", dimensionSet);

        // ... return response
    }
}
```

![](/images/3-Blogstranslated/Blog9/cloudwatch-graph-view.png)
> *Hình 1. Chế độ xem biểu đồ số liệu AWS CloudWatch.*


---

## Tracing

Tiện ích Tracing cung cấp khả năng tích hợp dựa trên chú thích với X-Ray để theo dõi phân tán với cấu hình tối thiểu. Truy tìm cho phép bạn:

- Theo dõi các lời gọi hàm và tương tác với AWS services trong X-Ray console.
- Tự động ghi nhận lỗi, phản hồi, và cold start.
- Thêm custom metadata để hỗ trợ debug.
- Bật/tắt tracing qua environment variables mà không cần chỉnh code.

Để thêm tiện ích Tracing vào dự án của bạn, hãy thêm phần phụ thuộc Maven sau:

```yaml
<!-- In the dependencies section -->
<dependency>
    <groupId>software.amazon.lambda</groupId>
    <artifactId>powertools-tracing</artifactId>
    <version>2.1.1</version>
</dependency>
<!-- In the build plugins section -->
<plugin>
    <groupId>dev.aspectj</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <configuration>
        <aspectLibraries>
            <aspectLibrary>
                <groupId>software.amazon.lambda</groupId>
                <artifactId>powertools-tracing</artifactId>
                <version>2.1.1</version>
            </aspectLibrary>
        </aspectLibraries>
    </configuration>
</plugin>
```
Để bật tính năng theo dõi trong hàm Lambda, hãy chú thích trình xử lý Lambda và các phương thức tùy chỉnh mà bạn muốn theo dõi bằng chú thích @Tracing. Mỗi chú thích ánh xạ tới một phân đoạn phụ của trình xử lý Lambda chính của bạn trong X-Ray và hiển thị trong bảng điều khiển.

```yaml
public class App implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {
    private static final Logger log = LoggerFactory.getLogger(App.class);
    @Tracing
    public APIGatewayProxyResponseEvent handleRequest(final APIGatewayProxyRequestEvent input, final Context context) {
        // ... business logic
        // Get calling IP with tracing
        String location = getCallingIp("https://checkip.amazonaws.com")

        // ... return response
    }
    @Tracing(segmentName = "Location service")
    private String getCallingIp(String address) {
        // Implementation to get IP address
        log.info("Retrieving caller IP address");
        // Add custom metadata to current sub-segment
        URL url = new URL(address);
        putMetadata("getCallingIp", address);
        // ...
        return "127.0.0.1";
    }
}
```
Bảng điều khiển X-Ray hiển thị bản đồ dịch vụ được tạo khi lưu lượng truy cập bắt đầu chảy qua ứng dụng của bạn. Việc áp dụng chú thích Truy tìm cho phương thức xử lý hàm Lambda của bạn hoặc bất kỳ phương thức nào khác trong chuỗi thực thi sẽ cung cấp cho bạn khả năng hiển thị toàn diện về các mẫu lưu lượng truy cập trong toàn bộ ứng dụng của bạn. Hình sau đây cho thấy siêu dữ liệu tùy chỉnh được thêm vào trong ví dụ được liên kết với phân đoạn phụ tùy chỉnh như thế nào.

![](/images/3-Blogstranslated/Blog9/xray-trace-waterfall.jpeg)
> *Hình 2. Chế độ xem dấu vết thác nước AWS X-Ray.*

---

## Giảm thời gian Lambda Cold Start

Một tính năng chính trong Powertools dành cho AWS Lambda (Java) v2 là hỗ trợ hình ảnh gốc [GraalVM](https://www.graalvm.org/) cho tất cả các tiện ích cốt lõi. Việc biên dịch các hàm Lambda của bạn thành các tệp thực thi gốc cho phép bạn giảm đáng kể thời gian khởi động nguội và mức sử dụng bộ nhớ. Sử dụng Powertools v2 với GraalVM cho phép bạn giảm khả năng khởi động nguội lên tới **75,61% (p95)** so với sử dụng thời gian chạy Java được quản lý. Điểm chuẩn sau đây so sánh thời gian khởi động nguội của một ứng dụng sử dụng tất cả các tiện ích cốt lõi (ghi nhật ký, số liệu, theo dõi) trên thời gian chạy java21 được quản lý so với thời gian chạy Lambda do Lambda provided.al2023 chạy hình ảnh gốc được biên dịch GraalVM (đi đến [Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) để được hỗ trợ):

| Environment                           | p95 (ms) | Min (ms) | Avg (ms) | Max (ms) | Max Memory (MB) | N   |
| ------------------------------------- | -------- | -------- | -------- | -------- | --------------- | --- |
| Powertools for AWS (Java) v2: JVM     | 1682.92  | 1224.55  | 1224.55  | 2229.81  | 205.04          | 234 |
| Powertools for AWS (Java) v2: GraalVM | 542.86   | 404.92   | 504.77   | 752.85   | 93.46           | 369 |

Cải tiến này đặc biệt có giá trị đối với các ứng dụng và chức năng nhạy cảm với độ trễ thường xuyên mở rộng quy mô. Kiểm tra một [ví dụ hoạt động đầy đủ trên GitHub](https://github.com/aws-powertools/powertools-lambda-java/tree/main/examples/powertools-examples-core-utilities/sam-graalvm).

---

## Lambda MSK Event Source Mapping Integration

Kafka utility mới được giới thiệu trong Powertools for AWS Lambda (Java) v2 giúp đơn giản hóa việc làm việc với  [Lambda MSK Event Source Mapping (ESM)](https://docs.aws.amazon.com/lambda/latest/dg/with-msk.html) và self-managed Kafka event sources. Nó mang lại một trải nghiệm quen thuộc cho các developer đã từng làm việc với Apache Kafka bằng cách cho phép chuyển đổi trực tiếp các Lambda events sang các kiểu dữ liệu native của Kafka. Các tính năng chính bao gồm:

- Deserialization trực tiếp thành các đối tượng Kafka ConsumerRecords<K, V> đồng thời vẫn sử dụng Lambda-native RequestHandler interface.
- Hỗ trợ deserialization các bản ghi được mã hóa bằng JSON, [Avro](https://avro.apache.org/), và [Protobuf](https://protobuf.dev/) cho cả key và value fields, có hoặc không sử dụng Schema Registry khi tạo message.

Để thêm Kafka utility vào project của bạn, hãy bao gồm thư viện powertools-kafka như một Maven dependency trong file pom.xml:
```yaml
<!-- In the dependencies section -->
<dependency>
    <groupId>software.amazon.lambda</groupId>
    <artifactId>powertools-kafka</artifactId>
    <version>2.1.1</version>
</dependency>
<!-- Kafka clients dependency - compatibility works for >= 3.0.0 -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>4.0.0</version>
</dependency>
```
Hãy sử dụng annotation @Deserialization trên Lambda handler của bạn để deserialize các message thành các native Kafka ConsumerRecords. Đảm bảo rằng bạn chỉ định loại deserializer phù hợp. Ví dụ sau đây minh họa cách deserialize các bản ghi được mã hóa bằng Avro với String keys. Giống như trong Lambda handler thông thường, bạn chỉ cần khai báo kiểu dữ liệu đầu vào cho function của mình trong RequestHandler generic parameters, và utility sẽ tự động phát hiện các kiểu deserialization. Lớp AvroProduct trong ví dụ bên dưới là một Java class được auto-generated bằng thư viện Java org.apache.avro.avro.
```yaml
public class App implements RequestHandler<ConsumerRecords<String, AvroProduct>, Void> {
    private static final Logger log = LoggerFactory.getLogger(App.class);
    @Deserialization(type = DeserializationType.KAFKA_AVRO)
    public Void handleRequest(ConsumerRecords<String, AvroProduct> consumerRecords, Context context) {
        log.info("Deserialized {} records.", consumerRecords.records().size()); 
        // ... Business logic 
        return null;
    }
}
```

---

## Kết luận

Powertools for AWS Lambda (Java) v2 đại diện cho bước tiến mới trong bộ công cụ hỗ trợ xây dựng các ứng dụng Serverless có tính ổn định, khả năng quan sát cao và hiệu suất vượt trội. Trong suốt bài viết này, chúng ta đã cùng tìm hiểu về các core observability utilities được cải tiến cùng những tính năng mới, hiệu năng vượt trội nhờ GraalVM native image support, và Kafka utility hoàn toàn mới giúp bạn làm việc với các Kafka pattern quen thuộc trong Lambda.

Powertools còn cung cấp nhiều utility khác để xử lý các Serverless design pattern phổ biến, mỗi utility đều được thiết kế dựa trên nguyên tắc rõ ràng và tối giản overhead. Để tìm hiểu thêm:

1. Truy cập [tài liệu](https://docs.powertools.aws.dev/lambda/java/) chính thức để xem các hướng dẫn chi tiết và ví dụ minh họa.
2. Thử các [sample applications](https://github.com/aws-powertools/powertools-lambda-java/tree/main/examples) được cung cấp.
3. Tham gia [cộng đồng](https://github.com/aws-powertools/powertools-lambda-java) trên GitHub để chia sẻ trải nghiệm và nhận hỗ trợ.

Ứng dụng Serverless tiếp theo của bạn đang chờ đón — cùng khám phá với Powertools for AWS Lambda (Java) v2. Chúng tôi rất mong nhận được phản hồi từ bạn!

