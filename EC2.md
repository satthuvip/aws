## 1. KHÁI NIỆM

- EC2 (elastic compute) là những máy ảo chạy trên hạ tầng AWS, hỗ trợ nhiều hệ điều hành như window, linux ...
- EC2 liên quan tới CPU và Memory, Card Network trên aws
- Triển khai ec2 chạy window sẽ mắc hơn ec2 chạy linux vì window có trả phí cho license hệ điều hành còn linux thì không.

INSTANCE_TYPE:

- general_purpose
- compute_optimize
- memory_optimize
- storage_optimize
- GPU

## 2. USER DATA

User data là dữ liệu hoặc script nhập khi khởi tạo EC2 để instance tự chạy lúc boot.

Thường dùng để:

- Cài đặt package
- Update OS
- Cài đặt web server
- Clone source code
- Tạo file config 
- Start service

Khi EC2 khởi động lần đầu, nó tự chạy script. AWS ghi rõ không nên lưu dữ liệu nhạy cảm như password hoặc logn-lived encryption key trong user data.

## 3. METADATA

Instance metadata là thông tin về chính EC2 instance, có thể truy cập từ bên trong instance.

Ví dụ:

```
Instance ID
AMI ID
Instance type
Private IP
Public IP
Hostname
Region
Availability Zone
IAM role credentials nếu instance có gắn IAM Role
Security groups
```

Metadata được truy cập thông qua instance metadata service - IMDS chạy cục bộ trên mỗi EC2. IMDS là service local trên EC2 dùng để kiểm soát khả năng truy cập metadata.

Hiện nay dùng IMDSv2 vì bảo mật hơn khi sử dụng thêm TOKEN

```
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`

curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/  
```

## 4. PLACEMENT GROUP

Trong aws EC2 là các gợi ý cho AWS đặt các EC2 instance gần nhau hoặc tách nhau trên hạ tầng vật lý, tùy mục tiêu hiệu năng mạng hay giảm rủi ro lỗi cung lúc.

AWS có 3 loại placement group chính:

- Cluster: Đặt các EC2 gần nhau trong cùng một AZ để có network latency thấp và throughput cao
- Spread: Đặt các EC2 instance trên phần cứng vât lý khác nhau để giảm khả năng nhiều instance chết cùng lúc.
- Partition: Chia instance thành nhiều partition logic. Các instance ở partition khác nhau không dùng chung phần cứng vật lý.



## 5. ELASTIC IP

Elastic IP trong AWS là địa chỉ IPv4 public tĩnh mà bạn có thể gán cho EC2 instance hoặc một số tài nguyên mạng khác.

- public IP thường: Có thể đổi khi stop/start EC2
- Elastic IP: IP cố định, bạn giữ và tự gán lại được

Cần elastic IP khi:

- Khi tạo EC2 có public ipv4 bình thường, IP đó có thể thay đổi nếu EC2 stop rồi start lại instance còn elastic IP thì không. Ta vẫn có thể trỏ DNS về IP Public cũ này sau khi ec2 bị lỗi hoặc restart.

Elastic IP là IP có phí. Không sử dụng chỉ allocate cũng bị tính phí.

## 6.LIFECYCLE EC2

Lifecycle của EC2 instance là các trạng thai từ lúc tạo đến khi bị xóa hẳn

```
pending  ->  running  ->  stopping  ->  stopped  ->  pending/running
                        \                         /
                         -> shutting-down -> terminated
```

| State             | Ý nghĩa                                              | Billing                                                      |
| ----------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| **pending**       | Instance đang được khởi tạo, AWS đang cấp tài nguyên | Thường chưa tính compute                                     |
| **running**       | Instance đang chạy, có thể SSH/RDP/app hoạt động     | Có tính phí compute                                          |
| **stopping**      | Đang tắt instance                                    | Có thể vẫn tính phí ngắn trong quá trình chuyển trạng thái   |
| **stopped**       | Instance đã tắt, có thể start lại                    | Không tính phí compute, nhưng vẫn tính EBS, Elastic IP nếu áp dụng |
| **shutting-down** | Đang chuẩn bị xoá vĩnh viễn                          | Không dùng lại được                                          |
| **terminated**    | Đã xoá vĩnh viễn                                     | Không start lại được                                         |

- Reboot 

  - instanceID giữ nguyên
  - Private IP giữ nguyên
  - Public IP thường giữ nguyên nếu reboot
  - Dữ liệu trên EBS giữ nguyên
  - Vẫn tính phí compute

- Stop

  - Instance ID giữ nguyên
  - EBS root volume giữ nguyên
  - Private IP giữ nguyên, Public IP tự động có thể bị đổi lại sau khi start lại trừ khi dùng elastic IP
  - Không tính phí compute, nhưng EBS vẫn tính phí

- Start

  - Chạy lại instance đã stop
  - Có thể chạy trên physical host khác

- Terminate

  - Xóa instance vĩnh viễn
  - không thể start lại
  - root ebs và data ebs có thể giữ lại hoặc không, tùy cấu hình.

  

## 7.PRICING EC2

Các tùy chọn về trả phí :

- **On-demand**: Trả theo thời gian sử dụng, không cam kết dài hạn. Dùng bao nhiêu trả bấy nhiêu. Phù hợp cho test, dev, workload không ổn định.
- **Savings Plans:** Cam kết mức sử dụng theo USD/giờ trong 1 hoặc 3 năm, đổi lại được giảm giá. AWS cho biết Savings Plans có thể tiết kiệm tới 72% so với On-Demand. (dùng cho nhiều resource như ec2, lambda ...)
- **Reserved Instances:** Cam kết một cấu hình cụ thể như instance_type và region trong 1 hoặc 3 năm. Đây thực chất là billing discount, không phải một máy vật lý riêng. (dành riêng resource ec2)
- **Spot Instance:** Dùng capacity dư thừa của AWS, giá rẻ hơn nhiều nhưng có thể bị thu hồi.
- **Dedicated Hosts:** Thuê riêng một server vật lý từ aws để triển khai.
- **Dedicated Instances**: Thuê từng instance riêng biệt

Giá EC2 thường bị ảnh hưởng bởi:

- region
- Instance family
- Size
- OS/license: Linux thường rẻ hơn window
- Storcage: EBS tính riêng
- Data tranfer: Dữ liệu ra internet hoặc cross-region thường tính riêng.
