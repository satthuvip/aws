## 1. ĐỊNH NGHĨA

Là dịch vụ dùng để quản lý truy cập vào tài nguyên trên AWS, và được làm gì

Thành phần chính:

- User: Tài khoản IAM cho người hoặc ứng dụng cụ thể
- Group: Nhóm nhiều user, dùng để gán quyền chung
- Role: Vai trò tạm thơi, thường dùng cho EC2, Lambda, App hoặc cross-account access
- Policy: Tập luật JSON định nghĩa quyền được phép từ chối 
- Permission: Quyền thực hiện hành động, ví dụ: s3.GetObject; ec2:StartInstances
- Principal: Thực thể đang yêu cầu quyền: user, role, federated user, application

IAM hoạt động theo mô hình: Xác thực trước, phân quyền sau. Nghĩa là aws kiểm tra bạn là ai, rồi kiểm tra policy để quyết định bạn được phép làm hành động đó trên tài nguyên đó không.

Identity-based Policy: được apply policy cho user, group và role

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

Resource-based Policy:  được apply trực tiếp cho resource. Vì policy gắn vào resource thường có trường "Principal" để chỉ rõ ai được phép

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:role/ExternalReadRole"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

Quyền trên AWS sẽ ưu tiên hành động "deny". Nghĩa là:

- Mặc định là deny: Nếu không có policy nào cho phép thì hành động bị từ chối
- Có allow thì được phép: Nếu có policy cho phép hành động trên resource đó, request được cho phép.
- Nhưng nếu có deny trên resource rõ ràng thì luôn bị chặn.

## Bảo mật S3

Mặc định nên block public access

AWS có tính năng S3 block public access ở cấp account, bucket, access point và organization.

### IAM Policy

IAM policy là identity-based policy, chỉ định hành động cho phép trên AWS resources.

Ví dụ chỉ cho phép đock object trong một prefix

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-bucket/public/*"
    }
  ]
}
```

### Bucket Policy

Bucket policy là resource-based policy, có thể được attach vào S3 Bucket. Sử dụng ngôn ngữ AWS access policy.

Ví dụ: Cho CloudFront truy cập, hoặc deny nếu request không dùng HTTPS

Deny non-HTTPS:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

