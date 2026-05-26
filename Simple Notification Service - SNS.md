Simple Notification Service 

Là dịch vụ pub/sub messaging của AWS, dùng để gửi thông báo hoặc event từ một nơi đến nhiều nơi khác cùng lúc.

SNS giống như loa phát thông báo. Ai đăng ký nghe thì sẽ nhận được message



SNS thường dùng để:

```
Gửi thông báo
Broadcast event đến nhiều service
Kết nối các hệ thống theo kiểu pub/sub
Fan-out message đến nhiều SQS queue
Kích hoạt Lambda
Gửi email, SMS, mobile push notification
```

Mô hình cơ bản của SNS:

```
Publisher → SNS Topic → Subscribers
```

Trong đó:

| Thành phần     | Ý nghĩa                         |
| -------------- | ------------------------------- |
| **Publisher**  | Service gửi message vào SNS     |
| **SNS Topic**  | Kênh trung gian để phát message |
| **Subscriber** | Nơi nhận message                |

- Topic: Là kênh để publisher gửi message vào, Subscriber nào đăng ký topic thì sẽ nhận message từ topic đó.
- Publisher: là nơi gửi message vào SNS topic
- Subscriber: Là nơi nhận message từ SNS Topic. SNS hỗ trợ nhiều loại subscriber:

| Subscriber              | Ví dụ                           |
| ----------------------- | ------------------------------- |
| **Email**               | Gửi email thông báo             |
| **SMS**                 | Gửi tin nhắn SMS                |
| **Lambda**              | Kích hoạt function              |
| **SQS**                 | Gửi message vào queue           |
| **HTTP/HTTPS endpoint** | Gửi request đến API             |
| **Mobile push**         | Gửi notification tới app mobile |

SNS hoạt động theo kiểu push -  có nghĩa là khi có message, SNS sẽ chủ động gửi message đến subscriber.

```
SNS Topic → Subscriber
```

Khác với SQS là pull-based:

```
Consumer → SQS: Có message không?
```

Dễ nhớ:

```
SNS = push thông báo
SQS = queue để consumer kéo message
```

SNS kết hợn với SQS: Fan-out pattern. 

```
Publisher → SNS Topic → nhiều SQS Queues
```

ví dụ:

```
Order Service
    |
    v
SNS Topic: OrderCreated
    |
    |--- SQS Queue: Payment
    |--- SQS Queue: Shipping
    |--- SQS Queue: Email
    |--- SQS Queue: Analytics
```

Mỗi service có queue riêng, lợi ích:

```
SNS broadcast message
SQS lưu message cho từng service xử lý riêng
Service nào lỗi thì message vẫn nằm trong queue của service đó
Các service không ảnh hưởng lẫn nhau
```

