Tier architecture là cách chia một ứng dụng thành nhiều lớp để dễ mở rộng, bảo trì và bảo mật hơn.

Trong Aws, kiến trúc phổ biến nhất là 3-tier architecture:

```
Users
  ↓
Presentation Tier
  ↓
Application Tier
  ↓
Data Tier
```

Presentation Tier: Là lớp người dùng truy cập vào. Nhận request từ người dùng và chuyển tiếp đến application tier.

```
User → Route 53 → CloudFront → S3 / ALB
```

Thường dùng:

| Thành phần     | AWS service               |
| -------------- | ------------------------- |
| DNS            | Route 53                  |
| CDN            | CloudFront                |
| Static website | S3                        |
| Load balancing | Application Load Balancer |

Application Tier: Là lớp xử lý logic nghiệp vụ, xử lý login, API, business logic, validation, payment, order ...

```
ALB → EC2 Auto Scaling Group
```

hoặc serverless

```
API Gateway → Lambda
```

| Thành phần    | AWS service           |
| ------------- | --------------------- |
| Compute       | EC2, ECS, EKS, Lambda |
| Load balancer | ALB                   |
| Scaling       | Auto Scaling Group    |
| Security      | Security Group, IAM   |

Data Tier: là lớp lưu trữ dư liệu

```
EC2 / Lambda → RDS / DynamoDB / ElastiCache
```

| Thành phần     | AWS service           |
| -------------- | --------------------- |
| Relational DB  | RDS, Aurora           |
| NoSQL          | DynamoDB              |
| Cache          | ElastiCache           |
| Object storage | S3                    |
| Backup         | AWS Backup, snapshots |

## 2. SERVERLESS Architecture

Kiến trúc serverless là cách xây dựng ứng dụng mà không cần quản lý server/EC2 trực tiếp. AWS sẽ lo phần hạ tầng như provisioning, scaling, patching, availability. Chúng ta tập trung vào code và logic nghiệp vụ.

```
User
  ↓
Route 53
  ↓
CloudFront
  ↓
S3 static website
  ↓
API Gateway
  ↓
Lambda
  ↓
DynamoDB / Aurora Serverless / S3
```

Các thành phần thường hay sử dụng

| Thành phần          | AWS service                      |
| ------------------- | -------------------------------- |
| Frontend static     | S3, CloudFront                   |
| API endpoint        | API Gateway                      |
| Compute             | Lambda                           |
| Database NoSQL      | DynamoDB                         |
| Database SQL        | Aurora Serverless                |
| File/object storage | S3                               |
| Authentication      | Cognito                          |
| Queue               | SQS                              |
| Event bus           | EventBridge                      |
| Workflow            | Step Functions                   |
| Monitoring          | CloudWatch, X-Ray                |
| Secrets             | Secrets Manager, Parameter Store |

## 3 Microservice Architecture

Kiến trúc này là cách xây dựng ứng dụng bằng nhiều service nhỏ, mỗi service phụ trách một chức năng riêng và có thể phát triển , deploy, scale độc lập.

Kiến trúc phổ biến như:

```
User
  ↓
Route 53
  ↓
CloudFront
  ↓
Application Load Balancer / API Gateway
  ↓
ECS / EKS / Lambda
  ↓
RDS / DynamoDB / S3 / ElastiCache
```

Các thành phần hay dùng

| Thành phần         | AWS service thường dùng             |
| ------------------ | ----------------------------------- |
| API Gateway        | API Gateway, ALB                    |
| Compute            | ECS, EKS, Lambda, EC2               |
| Container registry | ECR                                 |
| Database           | RDS, Aurora, DynamoDB               |
| Messaging          | SQS, SNS, EventBridge               |
| Service discovery  | Cloud Map                           |
| Monitoring         | CloudWatch, X-Ray                   |
| Secrets            | Secrets Manager, Parameter Store    |
| CI/CD              | CodePipeline, CodeBuild, CodeDeploy |

## 4. Hybrid Cloud

Là kiến trúc kết hợp (combines) giữa on-premise và aws. Có thể sử dụng:

- sử dụng IP public để kết nối qua internet
- sử VPN site-to-site, hoặc Direct Connect

Dùng trong trường hợp: Phù hợp với các tổ chức có như cầu dư liệu nằm tại chỗ.



## 5 HA & FT - High Availability & Fault Tolerance Architecture

Đây là kiến trúc nhắm hạn chế khả năng downtime của hệ thống.

- HA nghĩa là hệ thống được thiết kế để hạn chế downtime. Khi một thành phần lỗi, hệ thống có thể chuyển qua thành phần khác, nhưng có thể có một khoảng gián đoạn nhỏ
- FT nghĩa là hệ thống vẫn tiếp tục hoạt động ngay cả khi có lỗi, với rất ít hoặc gần như không có gián đoạn.

## 6 IoT Architecture

## 7 Multi-Region Active-Active Architecture

