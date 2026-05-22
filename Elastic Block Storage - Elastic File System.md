Elastic Block Store

1. Khái niệm

Đây là dịch vụ lưu trữ dạng block storage dùng để gắn vào EC2 instance, giống như ổ cứng gắn cho máy chủ.

Nói đơn giản:

==> EC2 là máy ảo, EBS là ổ đĩa của máy ảo đó.

EBS thường dùng để lưu:

- OS EC2
- Dữ liệu ứng dụng
- Database
- File cần lưu trữ lâu dài
- Log, cấu hình, dữ liệu persistent

EBS là block storage, hoạt động như một ổ cứng vật lý. Khi gắn vào EC2, hệ điều hành sẽ nhìn thấy nó như một disk

EBS tồn tại động lập với EC2, nếu stop EC2 dữ liệu trong EBS vẫn còn. Nếu termination EC2, EBS có thể:

- Bị xóa theo EC2 nếu enable Delete on termination
- Được giữ lại nếu disable Delete on termination

2. Type EBS

gp3 - là loại phổ biến nhất hiện nay.

gp2 - là thế hệ cũ hơn cảu General Purpose SSD. Hiện nay thường nên dùng gp3 thay vì gp2 vì gp3 linh hoạt và thường tối ưu chi phí hơn.

io1 / io2 - dùng cho workload cần hiệu năng IOPS cao và ổn định.

st1 - là HDD tối ưu cho throughput

sc1 - là HDD giá rẻ, dùng cho dữ liệu ít truy cập

 

Root Volume và Data volume

Khi tạo EC2, thường có một ổ chính gọi là root volume

```
Root Volume = Chứa hệ điều hành
Data volume = Chứa dữ liệu ứng dụng
```

Nên tách riêng root và data volume để dễ backup, resize và migrate.

EBS Snapshot là bản sao lưu của EBS volume, được lưu trong S3 phía backend của AWS. Không truy câp snapshot như một bject bình thường, nhưng AWS dùng S3 để lưu trữ nó.

Dùng snapshot để:

- Backup volume
- Khôi phục dữ liệu
- Tạo volume mới
- Copy volume sang region khác
- Tạo AMI

Snapshot là incremental backup, nghĩa là sau snapshot đầu tiên, các lần sau chỉ lưu phần dữ liệu thay đổi.



EBS Volume nằm trong một AZ cụ thể. Không thể gắn trực tiếp EBS ở AZ 1a vào EC2 nằm ở AZ 1b.

Muốn chuyển sang AZ khác cần:

```
EBS Volume → Snapshot → Create Volume ở AZ khác
```

  

Data Lifecycle Manager thường gọi là Amazon DLM hoặc EBS data lifecycle manager

Là dịch vụ dùng để tự động tạo, giữ lại và xóa snapshot / AMI theo lịch cho các tài nguyên như EBS volume hoặc EC2 instance.

DLM dùng làm gì?

Ví dụ: có một ec2 chạy database, dữ liệu nằm trên EBS volume. Thay vì mỗi ngày tự vào Aws console tạo snapshot thủ công, ta có thể tạo một policy DLM

```
Mỗi ngày lúc 1:00
--> Tạo snapshot cho EBS volume
--> Giữ lại 7 bản gần nhất
--> Bản cũ hơn thì tự xóa
```

EC2 Instance store volume là loại ổ đĩa gắn trực tiếp vào phần cứng vật lý nơi EC2 instance đang chạy.

Hiểu đơn giản: Instance Store là ổ đĩa tạm thời của EC2, rất nhanh nhưng dữ liệu không bền.

Khi EC2 chạy trên một physical host của AWS, một số loại instance có sẵn ổ đĩa local trên máy chủ vật lý đó.

So sánh với EBS

| Tiêu chí               | EBS                           | Instance Store            |
| ---------------------- | ----------------------------- | ------------------------- |
| Loại storage           | Network block storage         | Local block storage       |
| Dữ liệu bền            | Có                            | Không                     |
| Stop EC2               | Dữ liệu vẫn còn               | Mất                       |
| Terminate EC2          | Có thể giữ nếu cấu hình       | Mất                       |
| Snapshot               | Có                            | Không trực tiếp           |
| Gắn/tháo sang EC2 khác | Có                            | Không                     |
| Tốc độ                 | Tốt                           | Rất nhanh                 |
| Phụ thuộc AZ           | Cùng AZ với EC2               | Phụ thuộc host vật lý     |
| Use case               | OS, database, persistent data | Cache, temp, scratch data |

Elastic File System (EFS)

EFS là ổ đĩa dùng chung dạng file system cho nhiều EC2 cùng truy cập.

Nếu EBS giống ổ cứng gắn cho một EC2, thì EFS giống một thư mục mạng dùng chung cho nhiều server.

EFS dùng để lưu dữ liệu dạng file và cho nhiều máy cùng truy cập, ví dụ:

```
EC2-1  ─┐
EC2-2  ─┼──>  Amazon EFS
EC2-3  ─┘
```

Các EC2 có thể cùng đọc/ghi vào cùng một file system.

Dùng phổ biến cho:

- Webserver nhiều EC2 cần dùng chung source/file upload
- Application cần share storage
- ECS/EKS

EFS sử dụng giao thức NFS v4. EFS được thiết kế để hoạt động trên nhiều AZ trong một region. Khi tạo EFS, AWS sẽ tạo mount target trong các subnet/AZ để EC2 truy cập gần nhất.

Nếu một AZ gặp sự cố, EC2 ở AZ khác vẫn có thể truy cập EFS qua mount target còn lại.

Mount target là endpoint mạng để EC2 kết nối vào EFS. Mỗi target có:

- một IP trong subnet
- một security group
- Nằm trong một AZ

 EC2 muốn mount EFS thì phải kết nối được tới mount target qua NFS Port "TCP/2049"

EFS có 2 performance mode chính:

- General Purpose: Dùng hầu hết cho workload
- Max I/O: Dùng cho workload rất lớn, nhiều client cùng truy cập

Throughput mode

EFS có các chế độ throughput như:

- Bursting Throughput: Phụ thuộc vào dung lượng lưu trữ. Dữ liệu càng nhiều thì baseline throughput càng cao.
- Provisioned Throughput: Không phụ thuộc vào dung lượng lưu trữ. Dùng khi file system nhỏ nhưng cần throughput cao

Storage Class 

EFS có nhiều storage class để tối ưu chi phí:

| Storage class       | Ý nghĩa                                           |
| ------------------- | ------------------------------------------------- |
| **EFS Standard**    | Lưu dữ liệu thường xuyên truy cập, multi-AZ       |
| **EFS Standard-IA** | Infrequent Access, ít truy cập, multi-AZ          |
| **EFS One Zone**    | Lưu trong một AZ, rẻ hơn                          |
| **EFS One Zone-IA** | Ít truy cập, một AZ, rẻ hơn nữa                   |
| **EFS Archive**     | Dữ liệu rất ít truy cập, chi phí lưu trữ thấp hơn |

EFS hỗ trợ nhiều lớp bảo mật:

- Security Group: Kiểm soát network access đến mount target - cần mở port TCP/2049
- Encryption at rest: Mã hóa dữ liệu trên EFS bằng AWS KMS
- Encryption in transit: Mã hóa dữ liệu khi EC kết nối tới EFS bằng TLS
- IAM Authorization: Có thể dùng IAM để kiểm soát quyền mount/read/write thông qua EFS Access Point
- Posix permission

AWS storage gateway

Đây là dịch vụ giúp kết nối hệ thống lưu trữ on-premises với lưu trữ trên cloud.

Storage Gateway = Cầu nối giữa data center và aws storage

Strorage Gateway thường được triển khai như một appliance/gateway ở môi trường on-premises, có thể chạy trên:

- VMware ESXi
- Microsoft Hyper-V
- Linux KVM

Nó có local cache để giữ dữ liệu hay truy cập gần đây, giúp ứng dụng trên on-premises truy cập nhanh hơn.

AWS cũng mô tả gateway dùng cache local cho dữ liệu đọc/ghi gần đây và đồng bộ dữ liệu thay đổi lên aws.

Các loại AWS Storage Gateway chính:

- Amazon S3 File Gateway
- Amazon FSx File Gateway
- Volume Gateway
- Tape Gateway

S3 file Gate way: cho phép server local truy cập dữ liệu theo dạng file share, nhưng dữ liệu thật được lưu trong amazon S3

Nó hỗ trợ giao thức file như:

- NFS
- SMB

FSx file Gateway: dùng cho môi trường Windows file share, kết nối đến FSx for window file server

FSx File Gateway cung cấp truy cập on-premises tới cloud file shares được quản lý bởi Amazon FSx for window file server.

Volume Gateway: Cung cấp storage dưới dạng block storage cho server on-premises thông qua giao thức iSCSI.

Server local nhìn thấy nó như một ổ đĩa block, còn dữ liệu được bảo vệ/lưu trên AWS.

Volume Gateway có 2 kiểu quan trọng:

- Cached Volume:
  - Dữ liệu chính nằm trên AWS
  - Dữ liệu hay dùng được cache ở local
- Stored Volume: 
  - Dữ liệu chính nằm ở local
  - AWS dùng để backup snapshot

Tape Gateway: Dùng để thay thế hệ thống băng từ backup vật lý.



So sánh cách loại storage Gateway

| Loại Gateway         | Giao thức | AWS storage phía sau        | Dùng cho                         |
| -------------------- | --------- | --------------------------- | -------------------------------- |
| **S3 File Gateway**  | NFS/SMB   | Amazon S3                   | File backup, data lake, file app |
| **FSx File Gateway** | SMB       | FSx for Windows File Server | Windows file share               |
| **Volume Gateway**   | iSCSI     | S3/EBS snapshots            | Block storage, backup volume     |
| **Tape Gateway**     | iSCSI VTL | S3/Glacier                  | Tape backup/archive              |

 
