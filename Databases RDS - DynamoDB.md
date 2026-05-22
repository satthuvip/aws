Amazon RDS - Relational Database Service là dịch vụ quản lý cơ sở dữ liệu quan hệ trên AWS

RDS giống như MySQL, PostgreSQL, SQL Server ... nhung AWS quản lý giúp người dùng nhiều phần như cài đặt, backup, patch, monitoring, failover.

RDS hỗ trợ các database engine:

| Engine               | Mô tả                             |
| -------------------- | --------------------------------- |
| MySQL                | Phổ biến cho web app              |
| PostgreSQL           | Mạnh về tính năng, chuẩn SQL      |
| MariaDB              | Tương tự MySQL                    |
| Oracle               | Dùng nhiều trong doanh nghiệp     |
| Microsoft SQL Server | Dùng trong hệ sinh thái Microsoft |
| Amazon Aurora        | Database quan hệ do AWS tối ưu    |

 Đặc điểm: RDS dùng mô hình relational database, tức là dữ liệu được lưu trong bảng có quan hệ với nhau

RDS phù hợp khi ứng dụng cần:

- Dữ liệu có cấu trúc rõ ràng
- Quan hệ giữa các bảng
- Truy vấn SQL
- Transaction mạnh
- Join nhiều bảng
- Tính nhất quan dữ liệu cao

Thành phần chính trong RDS:

- DB instance: là máy chủ database được aws quản lý, kết nối app vào RDS thông qua endpoint
- Database Engine: là loại database được chọn như mysql, postgreSQL, Oracle ...
- Strorage: RDS lưu dữ liệu trên storage do AWS quản lý. Khi tạo database, chọn dung lượng lưu trữ. RDS có thể hỗ trợ tự động tăng dung lượng nếu bạn bật storage autoscaling.
- Endpoint: là địa chỉ để ứng dụng kết nối tới database

RDS thường dùng Security Group để kiểm soát ai được kết nối database

RDS hỗ trợ backup tự động. Có 2 khái niệm quan trọng là:

- Automated Backup: AWS tự động backup database theo lịch. Người dùng có thể cấu hình backup retention period. Nó giúp người dùng restore database về một thời điểm cụ thể, gọi là Point-in-Time Recovery
- Manual Snapshot: là bản backup được tạo thủ công

Multi AZ dùng để tăng HA cho database. Khi bật MultiAZ, RDS tạo một database chính ở một AZ và một bản standby ở AZ khác. 

Với kiểu multiAZ DB instance truyền thống, dữ liệu từ primary được đồng bộ sáng standby bằng synchronous replication, và AWS có thể tự động failover khi có sự cố.

ví dụ:

```
AZ-1: Primary RDS
AZ-2: Standby RDS
```

Nếu database chính ở AZ-1 lỗi:

```
AZ-1 Primary lỗi
        |
AWS tự động failover
        |
AZ-2 Standby trở thành Primary
```

Read Replica là bản sao chỉ đọc của database chính, mục đích chính là scale read

Nếu tất cả request đọc/ghi đều vào primary database, primary có thể bị quá tải. người dùng có thể tạo Read Replica:

```
Primary DB: ghi + đọc chính
Read Replica 1: đọc
Read Replica 2: đọc
```

Ứng dụng sẽ phân chia:

```
Write query → Primary DB
Read query  → Read Replica
```

Read replica là bản sao read-only của DB instance và giúp giảm tải primary bằng cách chuyển truy vấn đọc sang replica; dữ liệu được sao chép bất đồng bộ từ primary sang replica.

RDS Proxy là một dịch vụ trung gian nằm giữa application và Amazon RDS. Thay vì ứng dụng kết nối trực tiếp vào database, ứng dụng sẽ kết nối vào RDS Proxy. Proxy sẽ quản lý và tái sử dụng các kết nối database phía sau.

Database có giới hạn số lượng kết nối. Nếu quá nhiều client/app mở kết nối cùng lúc, database có thể bị quá tải.

RDS Proxy giúp connection pooling, nghĩa là nó giữ sẵn một nhóm kết nối tới database và tái sử dụng chúng cho nhiều request/app khác nhau. Thay vì mỗi request mở kết nối mới tới DB, app kết nối tới proxy, proxy dùng lại kết nối DB có sẵn.

Ví dụ:

- Không dùng proxy

```
1000 Lambda request → 1000 connection đến database
```

- Dùng RDS proxy

```
1000 Lambda request → RDS Proxy → dùng lại ít connection hơn đến database
```

Database đỡ bị quá tải hơn.

Lợi ích RDS Proxy

| Lợi ích                 | Giải thích                                 |
| ----------------------- | ------------------------------------------ |
| Connection pooling      | Tái sử dụng connection đến DB              |
| Giảm quá tải database   | DB không phải mở quá nhiều connection      |
| Tăng khả năng scale app | App scale nhanh nhưng DB ổn định hơn       |
| Failover tốt hơn        | Proxy giúp chuyển kết nối khi DB failover  |
| Bảo mật hơn             | Tích hợp IAM và Secrets Manager            |
| Ít thay đổi code        | App đổi endpoint từ DB sang proxy endpoint |

RDS Proxy thường dùng AWS Secret Manager để lưu thông tin đăng nhập database

AMAZON ELASTICache 

ElastiCache là dịch vụ cache managed của AWS, là bộ nhớ đệm tốc độ cao. Giúp ứng dụng truy cập dữ liệu nhanh hơn bằng cách lưu dữ liệu thường dùng trong RAM thay vì lần nào cũng truy vấn database.

Giả sử có kiến trúc:

```
User → Application → RDS Database
```

Mỗi lần user mở trang sản phẩm , app lại query RDS:

```
SELECT * FROM products WHERE id = 100;
```

Nếu có rất nhiều user cùng xem sản phẩm đó, RDS phải xử lý cùng một truy vấn lặp đi lặp lại

==> Database load nặng, response chậm, chi phí database tăng ...

Khi thêm ElastiCache:

```
User → Application → ElastiCache → RDS
```

App sẽ kiểm tra dữ liệu trong cache trước:

```
Có dữ liệu trong cache → trả về ngay
Không có dữ liệu → query database → lưu vào cache → trả về user
```

ElastiCache hỗ trợ 2 engine chính:

| Engine         | Đặc điểm                                                   |
| -------------- | ---------------------------------------------------------- |
| Redis / Valkey | Nhiều tính năng, phổ biến, hỗ trợ data structure phong phú |
| Memcached      | Đơn giản, nhẹ, dùng cache cơ bản                           |

DynamoDB là dịch vụ NoSQL database do AWS quản lý hoàn toàn. DynamoDB không dùng bảng quan hệ như RDS. Nó lưu dữ liệu theo dạng key-value hoặc document.

DynamoDB phù hợp cho ứng dụng cần tốc độ cao, scale lớn, dữ liệu linh hoạt, không cần join phức tạp.

Ví dụ dữ liệu trong DynamoDB:

```
{
  "user_id": "U001",
  "name": "An",
  "email": "an@gmail.com",
  "cart": [
    {
      "product_id": "P01",
      "quantity": 2
    }
  ]
}
```

Đặc điểm 

DynamoDB là database serverless, gần như không cần quản lý server. AWS tự lo:

- Scale
- Replication
- Backup
- High availability
- Performance
- Strorage growth

DynamoDB phù hợp khi ứng dụng cần:

- Truy vấn cực nhanh
- Lượng request rất lớn
- Dữ liệu không quá phụ thuộc vào quan hệ
- Scale tự động
- Serverless architecture 
- Millisecond latency

So sáng RDS và DynamoDB:



| Tiêu chí          | Amazon RDS                         | Amazon DynamoDB                      |
| ----------------- | ---------------------------------- | ------------------------------------ |
| Loại database     | Relational Database                | NoSQL Database                       |
| Kiểu dữ liệu      | Bảng có quan hệ                    | Key-value, document                  |
| Ngôn ngữ truy vấn | SQL                                | API, PartiQL                         |
| Join table        | Có                                 | Không hỗ trợ trực tiếp               |
| Transaction       | Mạnh                               | Có hỗ trợ, nhưng không giống RDS     |
| Scale             | Cần cấu hình instance/read replica | Tự động scale tốt hơn                |
| Server            | Có DB instance                     | Serverless                           |
| Schema            | Cố định hơn                        | Linh hoạt hơn                        |
| Use case          | ERP, CRM, banking, order system    | Gaming, IoT, session, serverless app |
| Performance       | Tốt, phụ thuộc instance            | Rất nhanh ở quy mô lớn               |
| Quản lý hạ tầng   | AWS quản lý một phần               | AWS quản lý gần như toàn bộ          |

Thành phần cơ bản trong dynamoDB:

- Table: giống như bảng chứa dữ liệu
- Item: giống như một dòng dữ liệu trong table
- Attribute: giống như cột/field trong một item