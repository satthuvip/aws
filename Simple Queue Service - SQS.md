SQS là viết tắt của Amazon Simple Queue Service, đây là dịch vụ message queue của AWS dùng để lưu message tạm thời giữa các service để xử lý bất đồng bộ.

SQS giống như một hàng đợi. Service A gửi việc cần làm vào hàng đợi, service B lấy từng việc ra để xử lý.

Cần SQS vì: Giả sử có hệ thống đặt hàng

```
User đặt hàng → Order Service → Payment Service → Email Service → Shipping Service
```

Nếu Order Service gọi trực tiếp các service khác, vấn đề có thể xảy ra

```
Payment Service chậm → Order Service bị chậm
Email Service lỗi → Order Service bị ảnh hưởng
Shipping Service quá tải → request bị fail
```

Dùng SQS thì flow có thể như sau:

```
Order Service → SQS Queue → Worker xử lý đơn hàng
```

Order Service chỉ cần gửi message vào queue rồi trả kết quả nhanh cho user. Worker phía sau xử lý dần dần.

Mô hình SQS

```
Producer → SQS Queue → Consumer
```

Trong đó:

| Thành phần   | Ý nghĩa                       |
| ------------ | ----------------------------- |
| **Producer** | Service gửi message vào queue |
| **Queue**    | Hàng đợi lưu message          |
| **Consumer** | Service đọc message và xử lý  |

Ví dụ:

```
API Gateway / Lambda / EC2
        |
        v
      SQS
        |
        v
Lambda / ECS / EC2 Worker
```

Có 2 loại SQS :

- Standard Queue: là loại queue mặc định, đặc điểm:

| Đặc điểm               | Ý nghĩa                                 |
| ---------------------- | --------------------------------------- |
| Throughput rất cao     | Xử lý số lượng message lớn              |
| At-least-once delivery | Message có thể được gửi nhiều hơn 1 lần |
| Best-effort ordering   | Không đảm bảo thứ tự tuyệt đối          |

==> Dùng standard Queue khi:

```
Không cần thứ tự tuyệt đối
Chấp nhận xử lý trùng bằng idempotency
Cần throughput cao
```

- FIFO là viết tắt của FirstIn-FirstOut, đảm bảo message nào vào trước thì được xử lý trước. Đặc điểm:

| Đặc điểm                | Ý nghĩa                                     |
| ----------------------- | ------------------------------------------- |
| Strict ordering         | Đảm bảo thứ tự                              |
| Exactly-once processing | Giảm duplicate bằng deduplication           |
| Message group ID        | Nhóm message để giữ thứ tự trong từng group |

Tên FIFO queue phải kết thúc bằng:

```
.fifo
```

Dùng FIFO Queue khi:

```
Cần xử lý đúng thứ tự
Không muốn duplicate logic phức tạp
Ví dụ giao dịch tài chính, xử lý đơn hàng theo user
```



- Visibility Timeout: Là thời gian mà message bị "ẩn" sau khi consumer lấy message ra xử lý. Ví dụ:

```
Consumer lấy message A từ queue
SQS ẩn message A trong 30 giây
Nếu consumer xử lý xong và delete message → message biến mất
Nếu consumer lỗi và không delete → sau 30 giây message hiện lại trong queue
```

==> SQS không tự xóa message khi consumer nhận. Consumer phải xóa message sau khi xử lý thành công.

- Message Retention Period là thời gian SQS giữ message trong queue nếu chưa được xử lý. => dùng để tránh message tồn tại mãi mãi trong queue
- Dead-Letter Queue là queue dùng để chứa message bị xử lý lỗi nhiều lần. Ví dụ:

```
Message A → Consumer xử lý lỗi
Message A quay lại queue
Consumer xử lý lỗi lần 2
Message A quay lại queue
Consumer xử lý lỗi lần 3
Message A được đưa vào DLQ
```

DLQ giúp kiểm tra:

```
Message nào bị lỗi?
Vì sao xử lý không được?
Có cần replay lại không?
```

Có 2 cách polling trong SQS:

- Short Polling: Consumer hỏi, nếu chưa thấy message thì trả về ngay

```
Consumer → SQS: Có message không?
SQS → Consumer: Không có
```

==> Có thể gọi API nhiều lần, tốn chi phí hơn

- Long Polling: Consumer hỏi và SQS chờ một khoảng thời gian nếu chưa có message.

```
Consumer → SQS: Có message không?
SQS chờ tối đa vài giây
Nếu có message thì trả về ngay
Nếu hết thời gian vẫn không có thì trả về rỗng
```

Long polling thường tốt hơn vì:

```
Giảm empty response
Giảm số lần gọi API
Tiết kiệm chi phí
```



Delay queue: Làm cho message mới gửi vào queue không thể được consumer nhận ngay trong một khoảng thời gian ==> dùng khi muốn trì hoãn xử lý

Message Timer: cũng là trì hõan message, nhung áp dụng cho từng message riêng lẻ.