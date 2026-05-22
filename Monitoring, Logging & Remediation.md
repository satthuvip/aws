## 1. CLOUD WATCH

### 1.1 Khái niệm

Cloudwatch là dịch vụ dùng để monitoring và observability trong AWS. Nó giúp người dùng theo dõi hệ thông đang chạy như thế nào: 

- CPU có cao không
- RAM có đầy không
- Application có lỗi không
- Log có bất thường không

Và khi có vấn đề thì gửi cảnh báo hoặc tự động xử lý.

CloudWatch là nơi thu thập metrics, logs, alarms và dashboards để quan sát tài nguyên và ứng dụng.

### 1.2 Thành phần

CloudWatch có nhưng thành phần core như:

- Metrics: là các chỉ số đo lường hiệu năng hoặc trạng thái của tài nguyên.

- Alarms: Dùng để cảnh báo khi metric vượt ngưỡng, một alarm theo dõi metric hoặc biểu thức metric, rồi thực hiện hành động khi điều kiện bị vi phạm trong một số khoảng thời gian.

- Logs: Dùng để thu thập, lưu trữ và xem log. CloudWatch Logs có các khái niệm:
  - Log group: Nhóm log
  - Log stream: Dòng log cụ thể, thường theo instance
  - Log Event: Một dòng hoặc một sự kiện log cụ thể
  - Retention: Thời gian lưu log

- Events: Để xử lý event.

- Cloudwatch Agent: Agent cài trên EC2 hoặc server on-premises để gửi thêm metrics/logs về cloudwatch.

**1.3 CloudWatch Logs**

Namspace là nhóm metrics theo từng dịch vụ hoặc ứng dụng

| Namespace            | Ý nghĩa                               |
| -------------------- | ------------------------------------- |
| `AWS/EC2`            | Metrics của EC2                       |
| `AWS/RDS`            | Metrics của RDS                       |
| `AWS/Lambda`         | Metrics của Lambda                    |
| `AWS/ApplicationELB` | Metrics của Application Load Balancer |
| `AWS/DynamoDB`       | Metrics của DynamoDB                  |
| `CWAgent`            | Metrics do CloudWatch Agent gửi lên   |
| Custom namespace     | Namespace do bạn tự tạo cho ứng dụng  |

Dimension là thông tin dùng để lọc hoặc xác định metric thuộc về tài nguyên nào.

Ví dụ: Giả sử có 3 EC2

```
EC2-A: i-111
EC2-B: i-222
EC2-C: i-333
```

cả 3 đều gửi metric CPUUtilization vào CloudWatch. CloudWatch sẽ phân biệt bằng dimension

| Metric         | Dimension          |
| -------------- | ------------------ |
| CPUUtilization | InstanceId = i-111 |
| CPUUtilization | InstanceId = i-222 |
| CPUUtilization | InstanceId = i-333 |

Nếu không có dimension, bạn sẽ không biết CPU đó là của instance nào.

