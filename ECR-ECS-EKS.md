ECR là dịch vụ của AWS dùng để lưu trữ, quản lý và chia sẻ docker container images. Giống như kho chứa image docker trên AWS.

Sau khi build lên một image, thực hiện push image đó lên ECR rồi các dịch vụ khác như ECS, EKS , Lambda, EC2 có thể pull image này về để chạy.

ECR có 2 loại chính:

- Private registry: Image chỉ được truy cập bởi người dùng hoặc service có quyền IAM
- Public registry: DÙng để chia sẻ image công khai

Một số tính năng quan trọng:

- Image Scanning: QUét lỗi bảo mật trong image
- Lifecycle policy: Tự động xóa image cứ
- Encryption: Mã hóa image khi lưu
- IAM Permission: Kiểm soát quyền push/pull image
- Cross-region replication: Sao chép image sang region khác
- Immutable tags: Không cho ghi đè tag image.

 ECS là dịch vụ AWS dùng để chạy và quản lý container, thường là Docker container mà không cần tự quản lý quá nhiều hạ tầng.

Các thành phần chính trong ECS:

- Cluster: là nơi chứa các container/task của người dùng.  
- Task Definition: Giống như bản thiết kế để chạy container. Nó khai báo

```
Dùng image nào?
Container cần bao nhiêu CPU/RAM?
Port nào được mở?
Biến môi trường là gì?
Log gửi đi đâu?
IAM role nào được dùng?
```

=> Task Definition không phải container đang chạy, mà chỉ là cấu hình mẫu.

- Task: Là một phiên bản đang chạy từ Task Definition

- Service: Giúp duy trì số lượng task mong muốn
- Container: Ứng dụng thực tế được chạy từ images.

Fargate là dịch vụ giúp chạy container mà không cần quản lý server EC2

Fargate là serverless cho container

ECS với Fargate là kiểu chạy container mà người dùng không cần quản lý EC2. Chỉ cần khai báo:

```
Image
CPU
RAM
Port
Số lượng task
```

==> AWS tự lo server phía dưới, đây là cách dễ dùng và phổ biến cho người mới bắt đầu.

ECS với EC2 là người dùng quản lý các EC2 instance trong cluster, contianer sẽ chạy trên các EC2 đó

==> Cách này phù hợp khi cần kiểm soát sâu hơn về server, chi phí, GPU, cấu hình đặc biệt.

| Tiêu chí          | ECS Fargate                    | ECS EC2                               |
| ----------------- | ------------------------------ | ------------------------------------- |
| Quản lý server    | AWS quản lý                    | Bạn quản lý EC2                       |
| Dễ dùng           | Dễ hơn                         | Phức tạp hơn                          |
| Scale             | Dễ scale task                  | Phải scale cả EC2                     |
| Kiểm soát hạ tầng | Ít hơn                         | Nhiều hơn                             |
| Phù hợp           | App thông thường, microservice | Workload đặc biệt, tối ưu chi phí sâu |

ECS Task Placement là cơ chế ECS dùng để quyết định, Task sẽ được chạy trên container instance nào trong ECS Cluster.

Task Placement strategy có 3 strategy chính:

- ***binpack***: cố gắng dồn task vào ít instance nhất có thể dựa trên CPU hoặc memory ==> Nhồi task cho đầy 1 instance rồi mới dùng instance khác

  => Giúp tiết kiệm chi phí. Nếu task được dồn vào ít EC2 hơn, có thể scale in bớt EC2 instance không dùng

- ***spread***: Cố gắng rải task đều theo một thuộc tính nào đó.

  => Phân tán task đều ra để tăng độ sẵn sàng.

  => Dùng khi muốn HA

- ***random***: Đặt task ngẫu nhiên vào các instance hợp lệ

ECS thường kết hợp với dịch vụ:

| Dịch vụ         | Vai trò                              |
| --------------- | ------------------------------------ |
| ECR             | Lưu Docker image                     |
| ALB             | Load balancing traffic vào container |
| CloudWatch Logs | Xem log container                    |
| IAM Role        | Cấp quyền cho task                   |
| VPC/Subnet      | Mạng cho container                   |
| Auto Scaling    | Tự động tăng/giảm task               |
| Secrets Manager | Lưu password, API key                |
| RDS/DynamoDB    | Database cho app                     |

EKS là dịch vụ của AWS dùng để chạy Kubernetes được quản lý trên AWS

Kiến trúc cơ bản:

```
EKS Cluster
├── Control Plane - AWS quản lý
│   ├── API Server
│   ├── Scheduler
│   ├── Controller Manager
│   └── etcd
│
└── Worker Nodes - nơi chạy container
    ├── EC2 Node
    ├── EC2 Node
    └── Pod / Container
```

Các thành phần chính:

- Cluster: là môi trường K8s trên aws, nó bao gồm:

```
Control plane
Worker nodes
Networking
IAM integration
Kubernetes resources
```

- Control Plane: Là bộ não của k8s, nó quyết định

```
Pod chạy ở node nào
Khi nào cần tạo pod mới
Khi nào cần restart pod
Trạng thái mong muốn của hệ thống là gì
```

- Worker node: Là máy chủ thật sự chạy container, worker node có thể là:

```
EC2 instance
Fargate
Managed Node Group
Self-managed Node
```

- Pod: Là đơn vị chạy nhỏ nhất trong K8s, một pod có thể chưa một hoặc nhiều container.

- Deployment: Quản lý số lượng Pod mong muốn

- Service: Dùng để expose Pod và tạo địa chỉ truy cập ổn định, vì Pod có thể bị xóa và tạo lại, IP của Pod có thể thay đổi. Service giúp các ứng dụng khác gọi Pod thông tua một endpoint ổn định.

