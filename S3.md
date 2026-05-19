1. Khái niệm

Amazone S3 - Simple Storage Service là dịch vụ object storage của AWS, dùng để lưu file/dữ liệu gần như mọi loại như: ảnh, video, backup, log, file tĩnh website, dữ liệu data lake

S3 không giống ổ đĩa EBS hay file system EFS. S3 lưu theo dạng object, mỗi object gồm:

```
Object = data + metadata + key + version ID nếu bật versioning
```

AWS mô tả S3 là dịch vụ lưu trữ cho internet, cho phép lưu và truy xuất lượng dữ liệu bất kỳ từ bất cứ đâu trên web

Khái niệm cốt lõi:

Bucket là thùng chứa object

Tên bucket phảu unique toàn cầu trong partition AWS tương ứng. Thực tế nên đặt tên rõ mỗi trường

Object là file bạn upload lên S3

S3 không có folder thật như file system. "Folder" chỉ là prefix trong key

Key là đường dẫn /ten object trong bucket. Ví dụ: s3://my-bucket/images/cat.jpg

Region, bucket nằm trong một aws region cụ thể. Chọn region gần user/app để giảm latency, hoặc theo yêu cầu compliance

2. S3 Storage class

S3 có nhiều storage class để tối ưu chi phí theo tần suất truy cập. AWS cho phép cấu hình storage class ở cấp object, và một bucket general purpose có thể chứa object ở nhiều storage class khác nhau.

| Storage class                     | Dùng khi nào                                  |
| --------------------------------- | --------------------------------------------- |
| **S3 Standard**                   | Truy cập thường xuyên, mặc định, latency thấp |
| **S3 Intelligent-Tiering**        | Không biết trước pattern truy cập             |
| **S3 Standard-IA**                | Ít truy cập nhưng cần lấy nhanh               |
| **S3 One Zone-IA**                | Ít truy cập, chấp nhận lưu ở 1 AZ             |
| **S3 Glacier Instant Retrieval**  | Archive nhưng cần lấy milliseconds            |
| **S3 Glacier Flexible Retrieval** | Archive, lấy trong phút đến giờ               |
| **S3 Glacier Deep Archive**       | Lưu cực rẻ, lấy lâu hơn                       |
| **S3 Express One Zone**           | Workload latency rất thấp trong một AZ        |

Versioning, Replication và Lifecycle

S3 Versioning  giúp S3 lưu nhiều phiên bản của cùng một object

Ví dụ bucket có file:

```
report.pdf
```

Bạn upload lại file mới cùng tên report.pdf, nếu không enable versioning thì file cũ sẽ bị ghi đè.

Nếu enable versioning, s3 sẽ giữ cả 2 file

```
report.pdf - version 1
report.pdf - version 2
```

Khi thực hiện đọc report.pdf, mặc định s3 sẽ trả về version mới nhất.

Versioning dùng để bảo vệ dữ liệu khởi

- bị ghi đè nhầm
- Xóa nhầm
- Rollback về bản cũ
- Ransomware hoặc lỗi ứng dụng
- Replication sang bucket khác 

Một điểm quan trọng là xóa object

Nếu không enable version, xóa object là mất object

Nếu bucket enable versioning, khi bạn xóa object S3 thường không xóa version thật ngay. Nó tạo ra một thứ gọi là delete marker

Ví dụ trước khi xóa:

```
config.json - version 1
config.json - version 2 latest
```

Sau khi xóa config.json

```
config.json - version 1
config.json - version 2
config.json - delete marker latest
```

Khi đó nếu GET config.json, bạn sẽ thấy như file không tồn tại. Vì delete marker đang là bản latest.

==> Muốn khôi phục, chỉ cần xóa delete marker, object cũ sẽ available.

S3 Versioning có 3 trạng thái:

| Trạng thái      | Ý nghĩa                      |
| --------------- | ---------------------------- |
| **Unversioned** | Chưa từng bật versioning     |
| **Enabled**     | Đang bật versioning          |
| **Suspended**   | Đã từng bật, sau đó tạm dừng |

Lưu ý: 

- Khi đã bật versioning, bạn không thể quay về "chưa từng bật" ==> chỉ có thể là suspend

S3 Replication

Replication là cơ chế tự động copy object từ bucket nguồn sang bucket đích.

Có 2 kiểu phổ biến:

| Loại                               | Ý nghĩa                                |
| ---------------------------------- | -------------------------------------- |
| **SRR — Same-Region Replication**  | Replicate sang bucket khác cùng Region |
| **CRR — Cross-Region Replication** | Replicate sang bucket khác Region      |

Mục đích sử dụng replication:

- Disaster Recovery
- Backup sang region khác
- Compliance
- Giảm latency cho user ở region khác
- Copy log sang account bảo mật
- Tách dữ liệu production và analytics

Điều kiện tiên quyết:

- Enable versioning tại Source Bucket và Destination Bucket
- Tạo IAM role cho S3 replication
- Replication Rule, chọn prefix/tag cần replication
- Chọn Destination bucket

Mặc định, replication rule thường áp dụng cho object mới sau khi rule được tạo. 

Object đã tồn tại trước đó thường không tự replication ngay. Nếu muốn replication object cũ, có thể cần dùng

```
S3 Batch Replication
S3 Batch Operations
```

Trong bucket enable versioning, delete thường tạo delete marker. Có thể bật/tắt việc replicate delete marker sang bucket đích.

Ví dụ:

```
Source: delete file A --> tạo delete marker
Destination: Cũng nhận delete marker nếu bặt replication delete marker
```

Nếu không bật, file ở destination có thể vẫn còn.



Lifecycle là rule tự động quản lý vòng đời của object. Lifecycle có thể:

- Chuyển object sáng storage class rẻ hơn
- Xóa object sau một thời gian hoặc old versions
- Xóa delete marker
- Dọn multipart upload dang dở

Các loại lifecycle:

- Transition action: hành động đối với object trong S3
- Expiration actions: Xóa object khi hết hạn

Các vòng đời của object trong S3:

- S3 Standard: là storage class mặc định, dùng cho dữ liệu truy cập thường xuyên.
- S3 Standard-IA: dùng cho dữ liệu ít truy cập nhưng khi cần thì phải lấy nhanh, ở mức milliseconds.
- S3 Intelligent-tiering: dùng khi không biết dữ liệu sẽ được truy cập nhiều hay ít. 
- S2 One Zone - IA: cũng là loại ít truy cập, nhưng dữ liệu chỉ lưu trong một AZ
- S3 Glancier Instant Retrieval: dùng cho dữ liệu archive nhưng vẫn cần lấy ngay lập tức, ở mức ms (milliseconds)
- S3 Glancier Flexible Retrieval: dùng cho dữ liệu archive nhưng không cần lấy ngay, khi muốn lấy object cần restore trước thường từ phút/giờ.
- S3 Glacier Deep Archive: là class rẻ nhất cho dữ liệu lưu cực lâu và gần như không truy cập.

MFA trong S3

MFA delete là tính năng bảo vệ thêm cho bucket đã enable Versioning. Khi enable MFA Delete, muốn làm các việc nguy hiểm như:

- Xóa vĩnh viễn một object version hoặc,
- Thay đổi trạng thái versioning của bucket

==> người thực hiện phải cung cấp thêm MFA.

AWS mô tả MFA Delete là lớp vảo mật yêu cầu 2 yếu tố xác thực cho các thao tác này.

MFA Delete khác MFA login aws:

| Loại MFA          | Dùng ở đâu                          | Ý nghĩa                        |
| ----------------- | ----------------------------------- | ------------------------------ |
| **MFA login AWS** | Khi đăng nhập AWS Console           | Bảo vệ tài khoản               |
| **MFA Delete S3** | Khi xóa version hoặc đổi versioning | Bảo vệ object version trong S3 |

Điều kiện dùng MFA Delete:

- Bucket enable versioning
- Chỉ owner bucker (root account) mới bật/tắt được MFA Delete
- Chỉ dùng AWS CLI hoặc API để enable MFA Delete 

==> MFA Delete không thể enable qua AWS Management Console.



```
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled,MFADelete=Enabled \
  --mfa "arn:aws:iam::123456789012:mfa/root-account-mfa-device 123456"
```

Trong đó:

```
123456789012 = AWS account ID
root-account-mfa-device = MFA device của root account
123456 = mã MFA 6 số hiện tại
```

S3 Encryption

S3 Encryption là cơ chế mã hóa dữ liệu trong Aws S3 để bảo vệ object khỏi bị đọc trái phép.

Có 2 nhóm chính:

- Encryption at rest: mã hóa dữ liệu khi lưu trong S3
- Encryption in transit: mã hóa dữ liệu khi truyền qua mạng

Encryption at rest nghĩa là object được mã hóa khi nằm trong S3. S3 tự động áp dụng SSE-S3 làm mức mã hóa mặc định cho object mới upload, không tính thêm chi phí và không ảnh hưởng hiệu năng.

Các kiểu mã hóa server-side chính:

- SSE-S3
- SSE-KMS
- DSSE-KMS
- SSE-C

SSE-S3 là Server-side Encryption with Amazon S3 managed keys. Nghĩa là:

- S3 tự mã hóa object
- S3 quản lý encryption key
- Người dùng không cần quản lý key

Đây là kiểu mặc định hiện nay cho object mới upload lên S3.

SSE-KMS là server-side encryption with KMS Keys, nghĩa là object vẫn được s3 mã hóa nhưng key được quản lý qua aws kms.

AWS KMS hỗ trợ envelope encryption: dữ liệu được mã hóa bằng data key, rồi data key được mã hóa bằng KMS key.

**DSSE-KMS** là **Dual-layer Server-Side Encryption with AWS KMS keys**.

Nghĩa là object được mã hóa với **2 lớp mã hóa độc lập** dùng AWS KMS.

Dùng khi cần mức bảo vệ/compliance cao hơn SSE-KMS thông thường.

SSE-C là Server-side Encryption with Customer-Provided Keys. Nghĩa là:

- Bạn tự cung cấp encryption key trong request
- S3 dùng ket đó để mã hóa/giải mã object
- S3 không lưu key của bạn

AWS mô tả SSE-C là cách dùng key do khách hàng tự cung cấp để mã hóa object lưu trong S3.

Dùng khi:

- Muốn tự quản lý key hoàn toàn
- Không muốn aws kms quản lý key

Nhược điểm:

- Mỗi lần GET/PUT phải gửi key
- Phức tạp hơn, ít dùng hơn SSE-S3/SSE-KMS

Client-side Encryption

Ngoài server-side encryption, còn có client-side encryption

Nghĩa là:

- App tự mã hóa file trước khi upload lên s3
- S3 chỉ lưu dữ liệu đã bị mã hóa
- Khi download, app tự giải mã

S3 Event Notifications

S3 Event Notification là cơ chế để S3 tự động phát thông báo khi có sự kiện xảy ra với object trong bucket.

Ví dụ:

- Upload file vào bucket
- Xóa file trong bucket
- Restore file từ Glacier
- Replication hoàn tất
- Thay đổi tag của object
- Lifecycle chuyển/xóa object

S3 Event Notification dùng khi muốn hệ thống tự phản ứng với thay đổi trong S3.

| Tình huống                      | Xử lý                                |
| ------------------------------- | ------------------------------------ |
| User upload ảnh                 | Trigger Lambda resize ảnh            |
| Upload CSV                      | Trigger Lambda import vào database   |
| Upload video                    | Đẩy message vào SQS để worker encode |
| Upload log                      | Gửi event sang analytics pipeline    |
| Xóa object                      | Ghi audit hoặc cleanup metadata      |
| Restore object Glacier hoàn tất | Notify app/user                      |
| Replication failed              | Gửi cảnh báo                         |

Các resource thường nhận event của S3:

- AWS Lambda
- Amazon SQS
- Amazon SNS
- Amazon EventBridge

Có thể cấu hình event chỉ chạy với một số object nhất định. Sử dụng prefix filter

Chỉ trigger với object nằm trong prefix:

```
uploads/
```

Ví dụ:

```
uploads/avatar.png -> trigger
logs/app.log       -> không trigger
```

Suffix filter, chỉ trigger với object có đuôi cụ thể như:

```
.jpg
.csv
.pdf
```

Hoặc ta có thể kết hợp cả prefix và suffix:

```
prefix = uploads/images/
suffix = .jpg
```

S3 Presigned URLs

S3 Presigned URL là một URL tạm thời cho phép người khác upload/download object trong S3 mà không cần có AWS credentials.

Hiểu đơn giản:

```
Presigned URL = Link tạm thời đã được ký bằng quyền của IAM user/role
```

Ai có link đó thì có thể thực hiện hành động được phép, ví dụ:

```
GET object -> download file
PUT object -> upload file
```

Nhưng chỉ trong thời gian URL cho phép

Multipart Upload 

Multipart upload là cơ chế upload một object lớn lên S3 bằng cách chia file thành nhiều part nhỏ, upload từng part riêng. Sau đó, S3 ghép các part lại thành một object hoàn chỉnh.

AWS khuyến nghị dùng multipart upload cho object từ 100MB trở lên, vì nếu một part upload lỗi thì chỉ cần upload lại part đó, không cần upload lại cả file.

Ví dụ:

```
video.mp4 = 5G

Chia thành:
part 1 = 100MB
part 2 = 100MB
part 3 = 100MB
...
part N = phần còn lại
```

Flow:

```
Client
  |
  | 1. Create multipart upload
  v
S3
  |
  | 2. Upload từng part
  |
  | 3. Complete multipart upload
  v
S3 ghép các part thành object hoàn chỉnh
```

So sánh Multipart Upload và Upload thường

Upload thường:

```
Upload file 5 GB
Đang upload đến 4.8GB thì mạng lỗi => Upload lại từ đầu
```

Multipart Upload:

```
File 5GB chia thành 50 part
Part 37 lỗi
=> chỉ upload lại part 37
```

Lợi ích sử dụng Multipart upload:

- Upload file lớn ổn định hơn
- Có thể upload song song nhiều part
- Retry từng part khi lỗi
- Tăng tốc upload
- Phù hợp mạng không ổn định

Tranfer acceleration 

Là tính năng giúp upload/download file đến S3 nhanh hơn khi client ở xá region của bucket.

Nó dùng hệ thống CloudFront edge locations. Client gửi dữ liệu đến edge location gần nhất, sau đó AWS chuyển dữ liệu về S3 qua network path tối ưu của AWS.

Flow bình thường:

```
User ở Việt Nam
   |
   | Internet public path
   v
S3 bucket ở us-east-1
```

Flow với transfer Acceleration:

```
User ở Việt Nam
   |
   | Gửi đến CloudFront edge gần nhất
   v
AWS Edge Location
   |
   | AWS optimized network
   v
S3 bucket ở us-east-1
```

Transfer Acceleration dùng khi:

- User upload/download từ nhiều quốc gia
- Client ở xa region của bucket
- File lớn
- Đường truyền internet public không ổn định
- Ứng dụng mobile/web có user toàn cầu

Transfer Acceleration đươc thiết kế để tối ứu tốc độ transfer từ khắp thế giời vào S3 general purpose bucket, và có thể phát sinh thêm phí data transfer.
