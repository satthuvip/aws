Serverless application là ứng dụng được xây dựng mà bạn không cần quản lý server trực tiếp. Không tự tạo EC2, cài OS, Patch server, scale server hay quản lý capacity mà AWS lo phần hạ tầng, còn người dùng tập trung vào code, data và business logic.

Serverless giúp người dùng build và chạy ứng dụng mà không cần provision, scale và quản lý server thủ công. Các dịch vụ phổ biến gồm:

- Aws Lambda
- API Gateway
- DynamoDB

So sánh serverless với kiến trúc truyền thống

- Kiến trúc truyền thống: Ví dụ app web chạy trên EC2

```
User
  ↓
Load Balancer
  ↓
EC2 / Auto Scaling Group
  ↓
RDS
```

==> Phải quản lý:

```
EC2
OS patching
Runtime
Scaling
Load balancing
Security group
Monitoring
High availability
Backup
```

- Kiến trúc serverless: Ví dụ app web serverless

```
User
  ↓
API Gateway
  ↓
Lambda
  ↓
DynamoDB
```

==> Chủ yếu quả lý:

```
Code
IAM permission
Business logic
Data model
Monitoring
Cost
```

==> Phần server, scaling, avaibility hạ tầng do AWS quản lý.



2. AWS Lambda - chạy code serverless

Lambda là dịch vụ compute serverless. Người dùng upload code, Lambda chạy code khi có sự kiện xảy ra. Lambda tự scale up/down và tính phí theo mức sử dụng.

Ví dụ Lambda có thể chạy khi:

```
User gọi API
File được upload vào S3
Message đi vào SQS
Event xuất hiện trong EventBridge
DynamoDB stream có thay đổi
```

Một lambda function gồm:

```
Code
Runtime
Handler
IAM Role
Trigger
Configuration
Logs
```

Luồng cơ bản

```
1. Có event xảy ra
2. AWS service invoke Lambda
3. Lambda tạo execution environment nếu cần
4. Lambda chạy handler function
5. Code xử lý event
6. Lambda trả kết quả hoặc ghi kết quả ra service khác
7. Log được gửi vào CloudWatch Logs
```

Event Sources và trigger lambda function truyền event data dạng json để function xử lý. Lambda chạy code trong execution environment theo runtime như Node.js hoặc Python.

Trigger trong Lambda là nguồn kích hoạt lambda. Ví dụ:

```
API Gateway gọi Lambda khi user gọi API
S3 gọi Lambda khi có file upload
SQS gọi Lambda khi có message trong queue
EventBridge gọi Lambda theo lịch hoặc event
DynamoDB Streams gọi Lambda khi item thay đổi
SNS gọi Lambda khi có notification
CloudWatch Logs gọi Lambda để xử lý log
```

Mô hình:

```
Event Source / Trigger
        ↓
      Lambda
        ↓
  Xử lý logic
```

- Function là đơn vị code bạn deploy lên lambda, ví dụ:

```
createOrderFunction
resizeImageFunction
sendEmailFunction
processPaymentFunction
```

- Runtime: Là môi trường chạy code

```
Node.js
Python
Java
.NET
Go
Ruby
Custom runtime
Container image
```

- Handler: Là entrypoint của lambda function

```
def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": "Hello from Lambda"
    }
```

Trong đó:

```
event   = dữ liệu đầu vào từ trigger
context = thông tin runtime, request ID, timeout còn lại...
```

Ví dụ:

```
export const handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello from Lambda" })
  };
};
```

Invoke Lambda -  có 3 kiểu chính

- Synchronous invocation - Client gọi lambda và chờ kết quả

```
API Gateway → Lambda → trả response cho user
```

==> Nếu lambda lỗi, client nhận lỗi

- Asynchronous invocation - Service gọi Lambda nhưng không chờ kết quả ngay

```
S3 → Lambda
SNS → Lambda
EventBridge → Lambda
```

==> Nếu lỗi, Lambda có thể retry. Nên cấu hình DLQ hoặc on-failure destination để không mất event lỗi.

- Poll-base invocation / Event Source Mapping: 
  - Với SQS, DynamoDB Streams, Kinesis, Lambda không bị service push trực tiếp theo kiểu đơn giản. Thay vào đó, lambda dùng event source mapping để poll dữ liệu rồi invoke function theo batch.
  - event source mapping là Lambda resource đọc item từ stream/queue-based services và invoke function với batch records.

Event Source Mapping là cơ chế để lambda tự đọc dữ liệu từ một nguồn event dạng queue/stream, rồi gọi Lambda function để xử lý dữ liệu đó.

Event source mapping khác trigger bình thường

| Loại                          | Cách hoạt động               | Ví dụ                             |
| ----------------------------- | ---------------------------- | --------------------------------- |
| Service push event vào Lambda | Service chủ động gọi Lambda  | API Gateway, S3, SNS, EventBridge |
| Event source mapping          | Lambda chủ động đọc từ nguồn | SQS, DynamoDB Streams, Kinesis    |

Sử dụng event source mapping khi nguồn event không chủ động push trực tiếp vào Lambda mà Lambda cần tự poll/đọc dữ liệu từ nguồn đó.

Công thức: 

```
Queue / Stream → Event Source Mapping → Lambda
```

Lambda Version và Lambda Alias

Trong Lambda, Version và Alias dùng để quản lý việc release/deploy function an toàn hơn

```
version = bản snapshot cố định của lambda function
alias = tên đại diện trỏ tới một version cụ thể
```



Event Bridge là dịch vụ serverless event bus của AWS, dùng để nhận event từ nhiều nguồn rồi route event đó đến một hoặc nhiều dest để xử lý.

Luồng hoạt động của EventBridge

```
Event Source
    ↓
Event Bus
    ↓
Rule
    ↓
Target
```

Giải thích:

```
Event Source = nơi phát sinh event
Event Bus    = nơi nhận và route event
Rule         = điều kiện lọc event
Target       = nơi nhận event để xử lý
```

- Event Source: Là nguồn tạo ra Event. Có 3 nhóm chính:

  - AWS Services: Là các service trong aws tạo ra event

  - Custom Application: Ứng dụng tự gửi event vào EventBridge

  - SaaS: EventBridge có thể nhận event từ một số SaaS provider

- Event Bus: Là nơi nhận event và route event đến target, các loại event bus thường gặp:
  - Default event bus: dùng cho event từ aws services
  - Custom event bus: Tự tạo để dùng cho application
  - Partner event bus: dùng cho event từ SaaS
- Rule: Là điều kiện để lọc event. Ví dụ chỉ muốn bắt event "OrderCreated"

```
{
  "source": ["myapp.orders"],
  "detail-type": ["OrderCreated"]
}
```

==> Khi event match rule này, EventBridge gửi event đến target

- Target: Là nơi EventBridge gửi event đến sau khi rule match. Một rule có thể gửi event đến nhiều target. Mỗi rule có thể định nghĩa tối đa 5 target. Các target phổ biến như:

```
AWS Lambda
SQS
SNS
Step Functions
Kinesis
ECS task
CodeBuild
CodePipeline
API Gateway
API Destination
Event bus khác
CloudWatch Log group
```

EventBridge Scheduler là dịch vụ serverless scheduler, hỗ trợ:

- cron/rate recurring schedules
- one-time invocations

 AWS khuyến nghị dùng EventBridge Scheduler để invoke target theo lịch.

 Ví dụ dùng:

```
Gửi báo cáo hằng ngày
Cleanup data cũ
Sync dữ liệu mỗi giờ
Stop EC2 ban đêm
Start EC2 buổi sáng
```

EventBridge có thể dùng scheduler để chaỵ job định kì. 

Ví dụ:

```
Mỗi ngày 8h sáng
  ↓
EventBridge Scheduler
  ↓
Lambda gửi report
```



------

Amazon Kinesis là nhóm dịch vụ của AWS dùng để thu thập, xử lý và phân tích dữ liệu streaming theo thời gian thực.

Khi dữ liệu được tạo ra liên tục từng giây, Kinesis giúp nhận dữ liệu đó, lưu tạm, xử lý và đẩy sang nơi khác như S3, Lambda ...

Cần Kinesis vì:

Giả sử website thương mại điện tử có hàng triệu user, mỗi giây phát sinh:

```
User click sản phẩm
User search
User add to cart
User thanh toán
Application log
Error log
```

Muốn phân tích gần như real-time:

```
Sản phẩm nào đang hot?
User nào có hành vi bất thường?
App có lỗi tăng đột biến không?
Doanh thu theo từng phút ra sao?
```

Nếu đợi cuối ngày mới xử lý batch thì quá chậm ==> Kinesis giúp xử lý vấn đề này

```
Data phát sinh liên tục → Kinesis → xử lý gần real-time
```

Mô hình cơ bản của Kinesis:

```
Producer → Kinesis → Consumer
```

trong đó:

| Thành phần   | Ý nghĩa                               |
| ------------ | ------------------------------------- |
| **Producer** | Nơi gửi dữ liệu vào Kinesis           |
| **Kinesis**  | Nơi nhận/lưu/điều phối streaming data |
| **Consumer** | Nơi đọc và xử lý dữ liệu              |

Kinesis bao gồm các nhóm chính:

- Kinesis Data Stream: Là dịch vụ nhận và lưu streaming data theo thời gian thực, sau đó consumer có thể đọc dữ liệu để xử lý.

  - Trong kinesis Data Streams, dữ liệu được chia vào các shard, có thể hiểu shard giống như lane / partition để Kinesis scale throughput.

  - Mỗi record gửi vào stream sẽ đi vào một shard dựa trên partition key

    - Record là một đơn vị dữ liệu trong data streams, một record thường bao gồm:

      ```
      Data
      Partition key
      Sequence number
      Timestamp
      ```

    - Consumer là ứng dụng đọc dữ liệu từ kinesis để xử lý, ví dụ:

    ```
    AWS Lambda
    EC2 application
    ECS service
    Kinesis Client Library application
    Amazon Managed Service for Apache Flink
    ```

- Amazon Data Firehose dùng để nhận streaming data và tự động đẩy đến destination như:

```
Amazon S3
Amazon Redshift
Amazon OpenSearch Service
Splunk
Snowflake
HTTP endpoint
```

AWS mô tả Firehose là dịch vụ fully managed để capture, transform và load streaming data vào S3 ...

| Tiêu chí          | Kinesis Data Streams   | Amazon Data Firehose                      |
| ----------------- | ---------------------- | ----------------------------------------- |
| Mục đích          | Xử lý stream linh hoạt | Gửi data đến destination                  |
| Consumer          | Tự viết consumer       | AWS quản lý delivery                      |
| Real-time         | Gần real-time hơn      | Near real-time, có batching               |
| Lưu dữ liệu tạm   | Có retention           | Chủ yếu delivery                          |
| Dùng Lambda xử lý | Có                     | Có thể transform bằng Lambda              |
| Độ kiểm soát      | Cao hơn                | Đơn giản hơn                              |
| Use case          | Custom processing      | Đẩy log/data vào S3, Redshift, OpenSearch |
