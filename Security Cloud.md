Directory Service

Amazon Cognito 

Key Management Service - KMS

AWS KMS là dịch vụ dùng  để tạo, quản lý và kiểm soát encryption key trong kms. Là nơi quản lý key dùng để mã hóa và giải mã dữ liệu.

Ví dụ, có dữ liệu trong

```
S3
EBS
RDS
DynamoDB
EFS
CloudTrail
Secrets Manager
```

Có thể dùng KMS Key để mã hóa các dữ liệu nằm trong đó.

KMS Key là key chính dùng để bảo vệ các key khác hoặc bảo vệ dữ liệu

```
KMS key
  |
  |-- bảo vệ data key
        |
        |-- data key mã hóa file thật
```

KMS thường không dừng KMS key để mã hóa trực tiếp toàn bị dữ liệu lớn. Thay vào đó, aws dùng mô hình Envelope Encryption, luồng cơ bản:

```
1. Tạo data key
2. Data key mã hóa dữ liệu thật
3. KMS key mã hóa data key
4. Lưu encrypted data + encrypted data key
```

Mô hình:

```
Plaintext data
     |
     v
Data key mã hóa dữ liệu
     |
     v
Encrypted data

Data key
     |
     v
KMS key mã hóa data key
     |
     v
Encrypted data key
```

Khi cần đọc dữ liệu:

```
1. Lấy encrypted data key
2. Gửi encrypted data key cho KMS
3. KMS decrypt data key
4. Data key decrypt dữ liệu thật
```

Điểm quan trọng:

```
KMS key không rời khỏi AWS KMS
```

Các loại KMS Key:

- AWS owned key: là key do AWS sở hữu và quản lý hoàn toàn, thường không nhìn thấy hoặc quản lý trực tiếp loại key này
- AWS managed key: là key do AWS tạo và quản lý cho một số dịch vụ AWS cụ thể. Tên thường có dạng:

```
aws/s3
aws/ebs
aws/rds
aws/secretsmanager
```

- Customer managed key: là key do người dùng tạo và quản lý, ta có thể kiểm soát

  - Key policy
  - IAM Policy
  - Rotation
  - enable/disable key
  - Alias, tag, cross-account

  => Dùng khi cần kiểm soát bảo mật chặt chẽ hơn

So sánh:

| Loại key                 | Ai quản lý? | Bạn có thấy key không? | Khi nào dùng                              |
| ------------------------ | ----------- | ---------------------- | ----------------------------------------- |
| **AWS owned key**        | AWS         | Không                  | AWS tự mã hóa nội bộ                      |
| **AWS managed key**      | AWS         | Có thể thấy            | Dùng nhanh, đơn giản                      |
| **Customer managed key** | Bạn         | Có                     | Cần kiểm soát key policy, audit, rotation |

Key Policy: là chính sách kiểm soát ai được sử dụng KMS Key. Ví dụ key policy cho phép một IAM role sử dụng key:

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:role/AppRole"
  },
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:GenerateDataKey"
  ],
  "Resource": "*"
}
```

Nghĩa là role AppRole được phép:

- Encrypt
- Decrypt
- GenerateDataKey

Với KMS, chỉ IAM policy thôi thường là chưa đủ.

- IAM policy = quyền của user/role
- Key policy = quyền trên chính KMS key

Muốn dùng được KMS key, thông thường cần: 

- IAM Policy cho phép hành động KMS, và
- Key Policy cho phép principal đó dùng key

Ví dụ: IAM policy cho role

```
{
  "Effect": "Allow",
  "Action": [
    "kms:Decrypt"
  ],
  "Resource": "arn:aws:kms:ap-southeast-1:111122223333:key/abcd-1234"
}
```

Nhưng nếu key policy không cho role đó dùng key, request vẫn có thể bị từ chối.

Các action KMS hay gặp

| Action                    | Ý nghĩa                        |
| ------------------------- | ------------------------------ |
| `kms:Encrypt`             | Mã hóa dữ liệu                 |
| `kms:Decrypt`             | Giải mã dữ liệu                |
| `kms:GenerateDataKey`     | Tạo data key để mã hóa dữ liệu |
| `kms:DescribeKey`         | Xem thông tin key              |
| `kms:CreateKey`           | Tạo KMS key                    |
| `kms:ScheduleKeyDeletion` | Lên lịch xóa key               |
| `kms:DisableKey`          | Vô hiệu hóa key                |
| `kms:EnableKeyRotation`   | Bật rotation                   |

KMS tích hợp với các dịch vụ AWS như:

```
Amazon S3
Amazon EBS
Amazon RDS
Amazon DynamoDB
Amazon EFS
Amazon Redshift
Amazon CloudTrail
AWS Secrets Manager
AWS Systems Manager Parameter Store
Amazon SNS
Amazon SQS
AWS Lambda
```

Key rotation là việc xoay vòng key theo thời gian để tăng bảo mật. Với customer managed symmetric KMS key, bạn có thể bật automatic rotation

Khi bật rotation:

```
AWS tạo key material mới định kỳ
Key ID không đổi
Alias không đổi
Dữ liệu cũ vẫn decrypt được
```

ACM - AWS Certificate Manager

Đây là dịch vụ dùng để tạo, quản lý và tự động gia hạn SSL/TLS certificate cho các dịch vụ AWS. 

Ví dụ website có địa chỉ:

```
http://example.com
```

Muốn chuyển qua HTTPS:

```
https://example.com
```

thì cần SSL/TLS certificate. 

==> ACM hỗ trợ cấp và quản lý certificate đó.

SSL/TLS certificate là chứng chỉ dùng để:

- Mã hóa dữ liệu giữa client và server
- Xác minh website là có thật
- Cho phép website chạy HTTPS

Ví dụ: Khi user truy cập:

```
https://myapp.com
```

trình duyệt sẽ kiểm tra certificate để đảm bảo website đáng tin cậy.

AWS Certificate Manager thường dùng để:

- Cấp public SSL/TLS certificate miễn phí
- Import certificate có sẵn
- Tự động gia hạn certificate
- Gán certificate cho Load Balancer, CloudFront, API Gateway
- Bảo mật HTTPS cho website/app



| Dịch vụ                    | Dùng ACM để làm gì          |
| -------------------------- | --------------------------- |
| **Elastic Load Balancer**  | HTTPS listener              |
| **CloudFront**             | HTTPS cho CDN/custom domain |
| **API Gateway**            | HTTPS cho custom domain     |
| **AWS Elastic Beanstalk**  | HTTPS app                   |
| **App Runner**             | HTTPS custom domain         |
| **Amazon Cognito**         | Custom domain               |
| **AWS Global Accelerator** | HTTPS/TLS endpoint          |

ACM Certificate bao gồm:

- Public certificate: Đây là certificate dùng cho website/app public trên Internet => dùng khi muốn website có HTTPS
- Private certificate: Dùng cho hệ thống nội bộ, không public ra internet, thường dùng cới AWS Private CA - AWS Private Certificate Authority, dịch vụ này có tính phí riêng
- Imported certificate: Có thể mua certificate từ bên ngoài như DigiCert, Sectigo, GlobalSign rồi import vào ACM

Quy trình cấp certificate public thường là:

- Request certificate trong ACM
- Nhập domain cần bảo vệ
- Chon cách validate domain
- ACM kiểm tra sở hữu domain
- Certificate được issued (đã phát hành)
- Gán Cert vào dịch vụ AWS

   

Khi request certificate cho domain:

```
app.example.com
```

ACM cần chắc chắn rằng người dùng sở hữu domain đó. Có 2 cách để validate:

- DNS validation: Đây là cách được khuyến nghị nhiều nhất, ACM sẽ cung cấp một bản ghi DNS dạng CNAME, người dùng thêm record đó vào DNS zone của domain
- Email validation: ACM gửi email xác thực đến các email quản trị domain, người dùng bấm approve trong email thì certificate được cấp.

WAF - AWS Web Application Firewall

Đây là dịch vụ firewall ở tầng ứng dụng, dùng để bảo vệ web application/API khỏi các request độc hại.

WAF có thể chặn:

- SQL Injection
- Cross-site scripting - XSS
- Bot request
- Black IP
- Request từ quốc gia cụ thể, quá nhiều trong thời gian ngắn 
- Request có header/cookie/query string bất thường

WAP hoạt động ở layer 7 - Application Layer, nó kiểm tra nội dùng HTTP/HTTPS request như:

- IP address
- HTTP header
- URI path
- Query string, body, cookie
- HTTP method
- Rate request

WAF không gắn trực tiếp vào EC2 như Security Group, nó thường được gắn vào các dịch vụ public entrypoint như:

| Dịch vụ                       | WAF dùng để bảo vệ            |
| ----------------------------- | ----------------------------- |
| **Amazon CloudFront**         | Website global/CDN            |
| **Application Load Balancer** | Web app chạy trên EC2/ECS/EKS |
| **Amazon API Gateway**        | REST API                      |
| **AWS AppSync**               | GraphQL API                   |
| **Amazon Cognito User Pool**  | Login/sign-up endpoint        |
| **AWS App Runner**            | Web app/container public      |
| **AWS Verified Access**       | Application access            |

Ví dụ luồng:

```
User
 |
 v
CloudFront + AWS WAF
 |
 v
Application Load Balancer
 |
 v
EC2 / ECS / EKS
```

 

Khởi tạo Web ACL rồi attach Web ACL đó vào CloudFront, ALB, API Gateway ...

Trong Web ACL có nhiều rule như

```
Web ACL
 |
 |-- Rule 1: Block IP xấu
 |-- Rule 2: Block SQL Injection
 |-- Rule 3: Block XSS
 |-- Rule 4: Rate limit /login
 |-- Default action: Allow
```

Web ACL là tập hợp các rule kiểm tra request

Rule là điều kiện kiểm tra request, các action trong rule gồm:

| Action        | Ý nghĩa                                  |
| ------------- | ---------------------------------------- |
| **Allow**     | Cho request đi qua                       |
| **Block**     | Chặn request                             |
| **Count**     | Chỉ đếm, không chặn                      |
| **CAPTCHA**   | Bắt user giải CAPTCHA                    |
| **Challenge** | Kiểm tra client/browser một cách tự động |

Rule priority: Các rule được đánh giá theo thứ tự priority, rule nào match trước và có action kết thúc như Allow/Block thì WAF xử lý theo rule đó.

AWS Managed Rule là các bộ rule được AWS tạo sẵn để chống các kiểu tấn công phổ biến, dùng nó để giúp người dùng không cần tự viết ra tất cả rule từ đầu.

Custom rule: là rule do người dùng tự tạo theo nhu cầu riêng

Rate-Base rule:  dùng để giới hạn số request từ một nguồn trong một khoảng thời gian.

IP set: là danh sách IP/CIDR dùng trong WAF rule

Regex pattern set: Là tập hợp biểu thức chính quy để match request. Ví dụ muốn chặn request có path đáng ngờ:

```
/wp-admin
/phpmyadmin
/.env
```

Hoặc match pattern:

```
.*\.\./.*
```

để phát hiện patch traversal

WAF Captcha và Challenge:

- CAPTCHA: User phải giải CAPTCHA, dùng khi nghi ngờ bot nhung không muốn block hoàn toàn
- Challenge: WAF kiểm tra client một cách tự động, thường ít ảnh hưởng người dùng thật hơn CAPTCHA

Count mode là chế độ ghi nhận request match rule nhưng không chặn

AWS Shield là dịch vụ của AWS dùng để bảo vệ ứng dụng khỏi DDoS attack. DDoS là kiểu tấn công làm hệ thống bị quá tải bằng cách gửi lượng request/traffice rất lớn.

AWS Shield có 2 cấp độ chính:

- Shield Standard: được bật tự động cho tài khoản AWS mà không tính thêm phí
- Shield Advanced: là gói trả phí với khả năng bảo vệ và hỗ trợ cao hơn

AWS Guarduty là dịch vụ threat detection của AWS, phát hiện hành vi đáng nghi ngờ hoặc độc hại trong AWS account.

Nó liên tục phân tích log, network traffic và tín hiệu bảo mật để tìm các dấu hiệu như

```
Tài khoản AWS bị compromise
Access key bị lộ
EC2 giao tiếp với IP độc hại
Hành vi API bất thường
Reconnaissance / port scanning
Malware hoặc crypto mining
DNS query đáng ngờ
S3 bị truy cập bất thường
EKS/ECS runtime threat
```

