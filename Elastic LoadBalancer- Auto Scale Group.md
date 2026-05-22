 

Stateless Application là ứng dụng không lưu trạng thái làm việc của user bên trong server/app instance.

Ví dụ: User A gửi request đến EC2 instance số 1. Request tiếp theo của user A có thể đi đến EC2 instance số 2 mà vẫn hoạt động bình thường.

Ưu điểm:

- Dễ dàng scale out
- Dễ dàng thay thế instance bị lõi 
- Phù hợp với Auto scaling
- LB có thể phân phối request tự do

Stateful Application là ứng dụng có lưu trữ trạng thái bên trong server hoặc phụ thuộc vào server cụ thể.

Ví dụ: User A login vào EC2 instance số 1, session được lưu trong memory của instance đó. Request tiếp theo nếu bị chuyển sang EC2 instance số 2 thì user có thể bị mất session.

Vấn đề khi scale:

- khó scale out hơn
- cần đồng bộ dữ liệu giữa các instance
- LB có thể cần sticky session
- Instance chết có thể làm mất state nếu không lưu bền vứng.

Scale Up nghĩa là nâng cấp instance_type của ec2 (scale down)

Scale Out nghĩa là tăng số lượng server/instance (ngược là scale in)

Auto Scaling Group

ASG là cơ chế dùng để tự động quản lý số lượng EC2 instances theo nhu cầu tải, sức khỏe hệ thống hoặc lịch định sẵn.

Hiểu đơn giản:

- Traffic tăng --> ASG tự tạo thêm EC2
- Trafic giảm --> ASG tự xóa bới EC2
- Ec2 bị lỗi --> ASG tự thay thế EC2

ASG làm việc với ECS, EKS và EC2, được tích hợp với các service

- Cloudwatch
- ELB - Elastic Load balancer
- EC2 Spot instance để tối ưu chi phí
- VPC network 

Cơ chế hoạt động của ASG:

Với kiến trúc Web App:

```
User
  ↓
Application Load Balancer
  ↓
Auto Scaling Group
  ↓
EC2 instances
```

Khi traffic tăng:

```
CPU trung bình > 70%
        ↓
CloudWatch Alarm kích hoạt
        ↓
ASG tạo thêm EC2
        ↓
ALB phân phối traffic sang EC2 mới
```

Khi traffic giảm:

```
CPU trung bình < 30%
        ↓
ASG terminate bớt EC2
        ↓
Giảm chi phí
```

Khi một EC2 bị lỗi:

```
EC2 unhealthy
        ↓
ASG terminate EC2 lỗi
        ↓
ASG tạo EC2 mới thay thế
```

Cold start là lúc hệ thống cần thời gian để khởi động tài nguyên mới trướ khi  xử lý request.

Với ASG, khi traffic tăng ASG tạo thêm EC2 mới nhưng EC2 không phục vụ request ngay lập tức. Trong thời gian đó, instance mới chưa nhận traffic.

set time cho cold start để ASG không bị scale spam khi tạo ra liên tiếp các EC2

Các loại scale trong ASG:

- Target Tracking Scaling: Bạn đặt target và aws tự điều chỉnh
- Step Scaling: Tự điều chỉnh mức tăng mức giảm
- Scheduled Scaling: Lập lịch scale
- Predictive Scaling: Scale theo lịch



Elastic Load Balancing

ELB trong AWS là dịch vụ dùng để phân phối traffic đến nhiều backend như EC2, container, IP address hoặc lambda.

Hiểu đơn giản:

```
User
  ↓
Elastic Load Balancer
  ↓
EC2 instance 1
EC2 instance 2
EC2 instance 3
```

Thay vì user gọi trực tiếp vào từng EC2, User gọi vào loadblancer, rồi LB tự chia request đến các server healthy.

Các loại ELB trong aws:

| Loại                                | Layer        | Dùng khi nào                                    |
| ----------------------------------- | ------------ | ----------------------------------------------- |
| **Application Load Balancer — ALB** | Layer 7      | HTTP/HTTPS, web app, API, path-based routing    |
| **Network Load Balancer — NLB**     | Layer 4      | TCP/UDP/TLS, cần performance rất cao, static IP |
| **Gateway Load Balancer — GWLB**    | Layer 3/4    | Firewall, IDS/IPS, network appliance            |
| **Classic Load Balancer — CLB**     | Layer 4/7 cũ | Legacy, không nên dùng cho hệ thống mới         |

- Listener là cổng mà Load Balancer lắng nghe

Ví dụ:

```
HTTP :80
HTTPS :443
```

- Rule là luật điều hướng traffic

Ví dụ:

```
Nếu path = /api/* --> chuyển đến API target group
Nếu host = admin.example.com --> chuyển đến Admin target group
```

- Target Group là nhóm backend nhận traffic. Target có thể là:

  - EC2 instance

  - IP address

  - ECS task

  - Lambda funtion với ALB

### Internet-facing và Internal Load Balancer

#### 	internet-facing LB

Dùng cho traffic từ internet vào hệ thống

```
Internet → Public ALB → Web app
```

Thường đặt ở public subnets

#### 	internal LB

Chỉ dung trong VPC/private network

```
Backend service → Internal ALB → Private service
```

Thường dùng cho microservice nội bộ

Cross-Zone LB

Trong Aws, cross-zone load balancer là tính năng cho phép Load balancer phân phối traffic đến target ở nhiều AZ, không chỉ trong AZ nới request đi vào.

Ví dụ có ALB/NLB chạy trên 2 AZ:

Az-a: 2 EC2

Az-b: 4 EC2

Nếu không cross-zone, traffic đi vào node LB ở AZ-a thường chỉ được gửi đến EC2 trong AZ-a.

Giả sử có 6 EC2

```
AZ-a: EC2-1, EC2-2
AZ-b: EC2-3, EC2-4, EC2-5, EC2-6
```

Có 1000 request, nếu 50% traffic vào node LV ở AZ-a và 50 vào AZ-b

```
AZ-a nhận 500 request → chia cho 2 EC2
Mỗi EC2 ở AZ-a nhận khoảng 250 request

AZ-b nhận 500 request → chia cho 4 EC2
Mỗi EC2 ở AZ-b nhận khoảng 125 request
```

Nếu enable cross-zone, traffic có thể được chia đến EC2 ở AZ-a và AZ-b

LB chia traffic trên toàn bộ 6 EC2

```
1000 request / 6 EC2 = khoảng 166 request mỗi EC2
```

Mục đích của Cross-Zone:

- Chia tải đều hơn giữa các EC2
- Tránh tình trạng một AZ có it instance bị quá tải
- Hữu ích khi ASG phân bố instance không đều giữa các AZ
- Tăng khả năng chịu tải khi số target mỗi AZ khác nhau.

Với ALB, cross-zone load balancing thường được bật mặc định

Kiến trúc,

```
User
  ↓
ALB nodes ở nhiều AZ
  ↓
Target Group EC2 ở nhiều AZ
```

ALB có thể gửi request đến target health ở các AZ khác nhau

Với NLB, cross-zone load balancing có thể cần bật tùy cấu hình. Nếu không bật, mỗi node NLB thường chỉ gửi traffic đến targets trong cùng AZ

Với NLB, cần chú ý thêm cross-AZ data tranfer cost nếu traffic đi qua AZ khác.

Nên bật cross-xone khi:

```
Số EC2 giữa các AZ không đều
Traffic vào các AZ không đều
Một AZ thường xuyên bị tải cao hơn AZ khác
Muốn request phân bổ đều trên toàn bộ target
```

Cần cân nhắc khi:

```
Dùng NLB và quan tâm chi phí cross-az data tranfer
Ứng dụng rất nhạy với latency
Muốn traffic trong AZ nào xử lý nội bộ AZ đó
Kiến trúc yêu cầu strict AZ-local routing
```

#### Session State & Session Stickness

Session state là dữ liệu trạng thái của user trong một phiên làm việc.

Ví dụ: Người dùng đăng nhập vào website, hệ thống cần ghi nhớ:

```
User đã đăng nhập chưa?
UserID là gì?
Giỏ hàng đang có món nào?
Role của user là admin hay customer?
CSRF token là gì?
```

Những thông tin trên đó gọi là **session state**.

Lưu trữ session state:

- Lưu trong memory của EC2

  - ```
    User → ALB → EC2-1
    Session của user nằm trong RAM EC2-1
    ```

    Vấn đề:

    ```
    Request tiếp theo → ALB → EC2-2
    EC2-2 không có session
    User bị logout hoặc mất giỏ hàng
    ```

  ==> là kiểu statefule app

- Lưu bên ngoài EC2

  - ```
    User
      ↓
    ALB
      ↓
    EC2-1 / EC2-2 / EC2-3
      ↓
    Redis / DynamoDB / RDS
    ```

    Session được lưu ở nơi chung như:

    | Nơi lưu session   | AWS service         |
    | ----------------- | ------------------- |
    | Cache nhanh       | ElastiCache Redis   |
    | NoSQL bền hơn     | DynamoDB            |
    | Relational DB     | RDS                 |
    | Token phía client | JWT / cookie signed |

Khi đó EC2 nào nhận request cũng đọc được session.

==> Là kiểu stateless app, dễ scale out hơn.

Session stickness là cơ chế để LB cố gắng gửi request của một user tới cùng một backend target

Ví dụ:

```
Request 1 của User A → EC2-1
Request 2 của User A → EC2-1
Request 3 của User A → EC2-1
```

Thay vì:

```
Request 1 của User A → EC2-1
Request 2 của User A → EC2-2
Request 3 của User A → EC2-3
```

==> Stickey session thường được thực hiện bằng cookie

Cân sticky session: Hữu ích khi app lưu session state trong chính EC2.

Ví dụ:

```
EC2-1 RAM:
  session_user_A = logged_in

EC2-2 RAM:
  không có session_user_A
```

Nếu User A bị chuyển từ EC2-1 sáng EC2-2, app không biết user đó là ai ==> bật sticky session để user đó luôn vể EC2-1

#### Secure Listener LB

Trong AWS LB thường secure listener dùng HTTPS/TLS để nhận traffic được mã hóa từ client

```
User --HTTPS--> Load Balancer --HTTP/HTTPS--> EC2
```

Listener là cổng nghe của LB, Secure listener là listener có mã hóa, thường dùng:

```
HTTPS :443
TLS    :443
```

Secure listener là listener dùng certificate để mã hóa kết nối.

Ví dụ:

```
Client --> HTTPS:443 --> ALB
```

ALB cần có SSL/TLS certificate, thường lấy từ:

```
AWS Certificate Manager -- ACM
```

Sau đó ALB có thể forward traffic đến EC2 bằng HTTP hoặc HTTPS