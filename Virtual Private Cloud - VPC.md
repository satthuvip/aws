Global Infrastructure

AWS Global Infrastructure là hệ thống hạ tầng toàn cầu của AWS, gồm các datacenter, mạng cáp quang, vùng địa lý và các điểm biên giúp chạy dịch vụ AWS trên toàn thế giới.

Hiểu đơn giản:

```
AWS Global Infrastructure
│
├── Region
│   ├── Availability Zone 1
│   ├── Availability Zone 2
│   └── Availability Zone 3
│
├── Edge Location
├── Local Zone
└── Wavelength Zone
```

Region: là một khu vực địa lý riêng biết. Mỗi region độc lập với region khác. Khi tạo tài nguyên như EC2, RDS, VPC, S3 bucket, thường phải chọn region.

Availability Zone: là một hoặc nhiều data center nằm trong cùng một region. Mỗi AZ có nguồn điện, mạng, kết nối riêng. AWS mô tả AZ là các vị trí tách biệt trong region, gồm một hoặc nhiều DC với nguồn điện, networking và kết nối dự phòng. 

Mục đích của AZ là HA-high Availability, nếu một AZ gặp sự cố, hệ thống vẫn có thể chạy ở AZ khác.

Edge Location: Là điểm biên của AWS, thường dùng cho các dịch vụ cần đưa nội dung đến gần người dùng hơn, ví dụ:

- Amzone CloudFront
- Route 53
- AWS Shield
- AWS Global Accelerator

Ví dụ: Website đặt tại singapore, nhưng user ở Việt Nam, Thái Lan .... CloudFront có thể cache hình ảnh, video, file tĩnh ở Edge Location gần người dùng hơn để truy cập nhanh hơn.

Local Zone là phần mở rộng của region, đặt gần các thành phố lớn để giảm độ trễ. Dùng khi app cần latency rất thấp, ví dụ:

- Gaming
- Video streaming
- AR/VR
- Real-time analytics

Local Zone không đầy đủ dịch vụ như region chính, nhưng có thể chạy một số dịch vụ như compute, storage, database gần người dùng hơn.

Wavelength Zone: Đưa dịch vụ AWS vào gần mạng 5G của nhà mạng viễn thông. Dùng cho các ứng dụng cần latency cực thấp qua 5G như:

- Xe tự lái
- IoT
- Camera AI

VPC - Virtual Private Cloud là một mạng riêng ảo cho người dùng trên AWS, nơi đặt các tài nguyên như EC2, RDS, LoadBalancer, NAT Gateway, subnet, route table, Security Group...

```
AWS Cloud
└── Region: Singapore
    └── VPC: 10.0.0.0/16
        ├── Public Subnet: 10.0.1.0/24
        │   └── EC2 Web Server
        │
        ├── Private Subnet: 10.0.2.0/24
        │   └── EC2 App Server
        │
        └── Private Subnet: 10.0.3.0/24
            └── RDS Database
```

VPC giúp kiểm soát mạng trong AWS

| Mục đích           | Ý nghĩa                                       |
| ------------------ | --------------------------------------------- |
| Chia mạng          | Tạo public subnet, private subnet             |
| Bảo mật            | Kiểm soát traffic bằng Security Group, NACL   |
| Kết nối Internet   | Dùng Internet Gateway, NAT Gateway            |
| Kết nối nội bộ     | EC2, RDS, Lambda trong VPC giao tiếp với nhau |
| Kết nối on-premise | Dùng VPN hoặc Direct Connect                  |

Khi tạo VPC, cần chọn dải IP riêng, ví dụ:

```
10.0.0.0/16
172.16.0.0/16
192.168.0.0/16
```

Nghĩa là VPC có thể chứa nhiều địa chỉ IP bắt đầu bằng 10.0.x.x 

Subnet là mạng con bên trong VPC, mỗi subnet nằm trong một AZ

```
VPC
├── AZ-1
│   ├── Public Subnet
│   └── Private Subnet
│
└── AZ-2
    ├── Public Subnet
    └── Private Subnet
```

Có 2 loại subnet:

- Private Subnet: Không route trực tiếp ra internet gateway
- Public Subnet: là subnet có route đi ra internet gateway

Internet Gateway - IGW: Là cổng giúp VPC kết nối internet. Muốn EC2 trong public sibnet truy cập internet hoặc được internet truy cập, thường cần:

- EC2 có Public IP
- Subnet route table có route
- 0.0.0.0/0 --> Internet Gateway
- Security Group cho phép traffic phù hợp

Route Table quyết định traffic đi đâu.

VPC Peering giúp 2 VPC kết nối riêng với nhau bằng private IP. Ví dụ:

```
VPC A: 10.0.0.0/16
VPC B: 10.1.0.0/16

VPC A ↔ VPC B qua VPC Peering
```

Lưu ý: 

- CIDR của 2 vpc không được overlap
- VPC Peering không hỗ trợ transitive routing 

Ví dụ không transitive:

```
VPC A ↔ VPC B ↔ VPC C

A không tự đi được đến C thông qua B
```

VPC Endpoint giúp tài nguyên trong VPC truy cập dịch vụ AWS mà không cần đi qua internet.

Ví dụ EC2 private subnet cần truy cập S3:

Không tối ưu:

```
EC2 → NAT Gateway → Internet → S3
```

------

NAT trong Public Addresses

NAT cho public address thường được hiểu là: Các EC2 trong private subnet đi ra Internet bằng public IP / Elastic IP của NAT Gateway, thay vì mỗi EC2 phải có 1 public IP riêng.

Các hoạt động:

```
EC2 private subnet
Private IP: 10.0.2.10
        |
        | route 0.0.0.0/0
        v
NAT Gateway trong public subnet
Private IP: 10.0.1.x
Public IP / Elastic IP: 3.x.x.x
        |
        | Internet Gateway
        v
Internet
```

Khi EC2 private subnet truy cập Internet, NAT Gateway sẽ thay source IP từ private của EC2 thành public IP của NAT Gateway.

NAT device là dịch vụ cho phép instance trong private subnet kết nối ra ngoài VPC, nhưng bên ngoài không thể tự khởi tạo kết nối vào các instance đó.

Gồm 2 loại NAT:

- NAT Instance:
  - Bạn có thể tự quản lý được
  - Scale up (sử dụng instance_type)
  - Không HA
  - Cần assign Security Group
  - Sử dụng như một bastion host (vì là instance)
  - Sử dụng Elastic IP hoặc Public IP
  - Có thể triển khai port forwarding thông qua cấu hình manual 
- NAT Gateway
  - Quản lý bởi AWS
  - Elastic scale up lên tới 45Gbps
  - Có thể HA trong 1 AZ, có thể đặt trong multi AZ
  - Không thể truy cập
  - Sử dụng Elastic IP
  - Không có port forwarding

Trong VPC có 2 lớp bảo mật mạng quan trọng:

| Thành phần     | Cấp độ       | Stateful? |
| -------------- | ------------ | --------- |
| Security Group | Instance/ENI | Có        |
| NACL           | Subnet       | Không     |

Security Group giống firewall, gắn vào EC2, RDS, Loadbalancer. SG là statefull, nghĩa là nếu request đi vào được cho phép, response đi ra sẽ tự được cho phép.

NACL - Network ACL là firewall ở cấp subnet, NACL là stateless, nghĩa là inbound và outbound phải cấu hình riêng. Ví dụ: Nếu cho HTTP inbound, ta cũng cần đảm bảo outbound response được cho phép.

AWS Client VPN là dịch vụ VPN của aws giúp người dùng từ máy cá nhân, laptop hoặc văn phòng từ xa kết nối an toàn vào VPC trong AWS

Nó giúp người dùng truy cập tài nguyên private trong AWS mà không cần public chúng ra internet. Ví dụ:

- EC2 không có public IP
- RDS nằm trong private subnet
- Internal website chỉ chạy trong VPC

Bình thường từ laptop ở nhà bạn không truy cập được, nhưng nếu dùng aws client VPN thì khi đó máy laptop giống như đang nằm trong mạng private của AWS.

Các thành phần:

- Client VPN Endpoint: Là điểm kết nối VPN trên AWS, thực hiện tạo endpoint này trong AWS, cấu hình authentication, certificate, client CIDR, logging ... Client chỉ có thể kết nối sau khi endpoint được associate với target network đầu tiên.
- Target Network: Là subnet trong VPC mà Client VPN được associate vào
- Client CIDR: Là dải IP cấp cho User khi kết nối với VPN
- Authorization Rule: Là rule quyết định user VPN được phép truy cập network nào, nếu không có rule thì user có thể kết nối VPN nhưng không truy cập được tài nguyên nào trong VPC.
- Route Table: ClientVPN cũng cần có route table riêng

Site-to-Site VPN trong AWS là dịch vụ dùng để kết nối mạng nội bộ của công ty/on-premises với VPC trên AWS thông qua VPN bảo mật qua internet.

Sau khi kết nối, máy trong công ty có thể truy cập tài nguyên private trong AWS như EC2, RDS, internal application bằng private IP.

Các thành phần chính:

- Customer Gateway: Thiết bị hoặc thông tin đại diện cho phía công ty bạn, AWS cần biết public IP này để tạo VPN tunnel.
- Virtual Private Gateway: là gateway phía AWS gắn vào VPC, là đầu VPN phía AWS
- VPN Tunnel: thường tạo 2 VPN tunnels để dự phòng, nếu 1 tunnel lỗi, tunnel còn lại vẫn có thể hoạt động ==> High Avaiability.

Transit Gateway là dịch vụ dùng để kết nối nhiều VPC, VPN, Direct Connect và mạng on-premises lại với nhau thông qua một hub trung tâm. Thay vì kết nối từng VPC với nhau bằng nhiều (peering connect), ta thực hiện gom tất cả về transit Gateway.

Nếu không có Transit Gateway, giả sử có 4 VPC:

```
VPC A
VPC B
VPC C
VPC D
```

Nếu dùng VPC Peering, ta phải nối từng cặp

```
VPC A ↔ VPC B
VPC A ↔ VPC C
VPC A ↔ VPC D
VPC B ↔ VPC C
VPC B ↔ VPC D
VPC C ↔ VPC D
```

Càng nhiều VPC thì càng rối.

Nếu có transit Gateway, mỗi VPC chỉ cần kết nối tới Transit Gateway

```
           VPC A
             │
             │
VPC B ── Transit Gateway ── VPC C
             │
             │
           VPC D
```

Traffic giữa cái VPC sẽ đi qua Transit Gateway.

Transit Gateway thường dùng để:

| Mục đích                    | Ý nghĩa                                  |
| --------------------------- | ---------------------------------------- |
| Kết nối nhiều VPC           | Nối nhiều VPC với nhau qua một hub       |
| Kết nối AWS với on-premises | Qua Site-to-Site VPN hoặc Direct Connect |
| Đơn giản hóa network        | Không cần quá nhiều VPC Peering          |
| Quản lý route tập trung     | Có route table riêng của Transit Gateway |
| Mô hình hub-and-spoke       | TGW là hub, VPC/VPN là spoke             |

Các thành phần:

- Transit Gateway: là router trung tâm, không chứa EC2 hay database. Nó chỉ dùng để định tuyến traffic giữa các mạng
- Attachment: là kết nối từ một mạng (VPC) vào transit gateway
- Route table: route table riêng để quyết định traffic đi đâu

AWS Direct Connect là dịch vụ giúp bạn tạo kết nối mạng riêng chuyên dụng từ hệ thống on-premises, data center hoặc văn phòng của công ty đến AWS

Khác với VPN đi qua internet, Direct Connect dùng kết nối riêng qua nhà mạng/đối tác AWS.

Direct Connect dùng khi công ty cần kết nối ổn định từ mạng nội bộ đến AWS, dùng nhiều trong mô hình hybrid cloud, tức là một phần hệ thống chạy ở công ty, một phần chạy trên AWS.

Direct Connect khác với Site-to-Site VPN

| Tiêu chí     | Site-to-Site VPN      | Direct Connect                      |
| ------------ | --------------------- | ----------------------------------- |
| Đường truyền | Qua Internet          | Đường truyền riêng                  |
| Mã hóa       | Có IPsec encryption   | Không mã hóa mặc định               |
| Độ ổn định   | Phụ thuộc Internet    | Ổn định hơn                         |
| Latency      | Có thể thay đổi       | Thấp và ổn định hơn                 |
| Bandwidth    | Thường thấp hơn       | Cao hơn                             |
| Triển khai   | Nhanh hơn             | Lâu hơn                             |
| Chi phí      | Thường rẻ hơn         | Thường đắt hơn                      |
| Dùng cho     | Kết nối nhanh, backup | Production, enterprise, traffic lớn |

Mặc định , Direct Connect không mã hóa traffic. Nếu cần mã hóa traffic, có thể:

- Chạy VPN over Direct Connect
- Dùng MACsec nếu kết nối hỗ trợ
- Mã hóa ở tầng ứng dụng như HTTPS/TLS

Thành phần của Direct Connect:

- Direct Connect Location: Là địa điểm có hạ tầng Direct Connect của AWS
- Connection: Là đường kết nối vật lý hoặc logic từ mạng của công ty đến AWS
  - dedicated connection: là kết nối chuyên dụng trực tiếp với AWS
  - hosted connection: Là kết nối thông qua AWS Direct Connect Partner
- Virtual Interface: Là interface logic chạy trên Direct Connect connection
  - Private VIF: Dùng để kết nối Direct Connect vào một VPC thông qua Virtual Private Gateway/Direct Connect Gateway
  - Public VIF: Dùng truy cập các AWS public services bằng public IP qua Direct Connect 
  - Transit VIF: Dùng để kết nối Direct Connect với Transit Gateway thông qua Direct Connect Gateway

Direct Connect Gateway là thanh phần giúp Direct Connect kết nối tới nhiều VPC hoặc nhiều Region dễ hơn. Nó giúp không cần tạo riêng từng connection vật lý cho từng region.

```
Direct Connect
      │
      ▼
Direct Connect Gateway
      │
      ├── VPC Singapore
      ├── VPC Tokyo
      └── VPC Sydney
```

Direct Connect dùng BGP để trao đổi route giữa router phía công ty và AWS.

Flow Logs là tính năng dùng để ghi lại thông tin traffic mạng đi vào và đi ra khỏi network interface trong VPC.

| Mục đích                | Ví dụ                                  |
| ----------------------- | -------------------------------------- |
| Troubleshooting network | Kiểm tra vì sao EC2 không kết nối được |
| Security analysis       | Phát hiện traffic lạ                   |
| Audit/compliance        | Lưu lịch sử traffic                    |
| Monitor traffic         | Xem instance nói chuyện với IP nào     |
| Debug SG/NACL           | Biết traffic ACCEPT hay REJECT         |