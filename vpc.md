NAT trong Public Addresses

NT cho public address thường được hiểu là: Các EC2 trong private subnet đi ra Internet bằng public IP / Elastic IP của NAT Gateway, thay vì mỗi EC2 phải có 1 public IP riêng.

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

Khi EC2 private subnet truy cập Internet, Nat gateway sẽ thay source IP từ private của EC2 thành public IP của NAT Gateway.

NAT device là dịch vụ cho phép instance trong private subnet kết nối ra ngoài VPC, nhưng bên ngoài không thể tự khởi tạo kết nối vào các instance đó.

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
  - Quản lý bới AWS
  - Elastic scale up lên tới 45Gbps
  - Có thể HA trong 1 AZ, có thể đặt trong multi AZ
  - Không thể truy cập
  - Sử dụng Elastic IP
  - Không có port forwarding
- 