API Gateway là dịch vụ giúp bạn tạo, publish, bảo mật, giám sát và quản lý API ở quy mô lớn.

API Gateway như là cổng vào cho client gọi tới backend như lambda, EC2, ECS, ALB, HTTP Service hoặc các AWS service khác.

API Gateway dùng để:

- Tạo endpoint API
- Kết nối client với backend => API Gateway nhận request rồi chuyển tiếp đến lambda hoặc backend khác
- Bảo mật API
- Giới hạn request, monitor và log.
- Transform request/response

Các loại API Gateway:

| Loại              | Dùng khi nào                                                 |
| ----------------- | ------------------------------------------------------------ |
| **REST API**      | Cần nhiều tính năng quản lý API, API key, usage plan, request validation, caching, AWS WAF, private API |
| **HTTP API**      | Muốn API đơn giản, nhanh, chi phí thấp hơn, thường dùng với Lambda hoặc HTTP backend |
| **WebSocket API** | Cần giao tiếp 2 chiều real-time như chat app, notification, live dashboard |

Các loại triển khai chính cho API Gateway:

- Edge-optimized endpoint: Đây là kiểu endpoint được tối ưu cho client phân tán toàn cầu

```
User global
  |
CloudFront edge location
  |
API Gateway
  |
Backend
```

Dùng khi:

```
Người dùng ở nhiều quốc gia
Muốn giảm latency cho API public
REST API public
```

API Gateway sẽ dùng CloudFront managed distribute phía trước.

- Regional endpoint: Là kiểu phổ biến hiện nay cho API public trong một region

```
Client
  |
Regional API Gateway endpoint
  |
Backend trong cùng region
```

Dùng khi:

```
Client chủ yếu ở một khu vực
Bạn muốn tự gắn CloudFront riêng nếu cần
Muốn kiểm soát CDN/WAF/custom domain tốt hơn
```

- Private endpoint: Là loại API không public ra internet, client chỉ gọi được API này từ trong VPC hoặc từ mạng on-prmises đã kết nối với VPC

Các thành phần của API Gateway:

- Resource: Là đường dẫn của API
- Method: Là HTTP Method (GET, PATCH, PUT, POST, DELETE)
- Integration: Là backend mà API Gateway gọi đến (Lambda, HTTP Endpoint ...)

- Stage: Là môi trường deploy API
- Deployment: Sau khi cấu hình REST API, cần deploy ra một stage thì client mới gọi được.

Nên dùng API Gateway khi:

```
Tạo REST/HTTP API cho Lambda
Xây dựng serverless backend
Bảo vệ API bằng Cognito/IAM/JWT
Giới hạn request
Expose backend private ra ngoài một cách có kiểm soát
Làm WebSocket real-time
Quản lý nhiều version/stage API
```

 

API Usage Plans & API Keys

API Key và Usage Plan thương đi chung với nhau để kiểm soát việc sử dụng API theo từng client/app

- API Key: Định danh client/app nào đang gọi API
- Usage Plan: Quy định client/app đó được gọi bao nhiêu request

API Key là một chuỗi key do API Gateway tạo ra hoặc tự import vào.

Client gọi API sẽ gửi API key trong header:

```
GET /products HTTP/1.1
Host: abc123.execute-api.ap-southeast-1.amazonaws.com
x-api-key: abc123xyz456
```

API Gateway sẽ kiểm tra key có hợp lệ hay không.

API Key dùng để :

```
Nhận diện app/client đang gọi API
Gắn client với Usage Plan
Áp dụng quota và throttling
Theo dõi mức sử dụng API theo từng key
```

Usage Plans là cấu hình kiểm soát các một API Key được phép sử dụng API. Một Usage Plan có thể quy định

```
Throttle rate
Burst limit
Quota limit
API stage được phép truy cập
```

ví dụ:

```
Basic Plan:
- Rate: 10 requests/second
- Burst: 20 requests
- Quota: 10,000 requests/month

Premium Plan:
- Rate: 100 requests/second
- Burst: 200 requests
- Quota: 1,000,000 requests/month
```

- throttling giới hạn tốc độ gọi API, 2 thông số chính:
  - Rate limit: Là tốc độ request ổn định cho phép
  - Burst limit: Số request tăng đột biến cho phép trong thời gian ngắn. Nghĩa là trong 1 thời điểm ngắn, client có thể burst lên cao hơn rate limit, nhung không được vượt quá bucket cho phép.
- Quota: Là tổng số request được cho phép trong một khoảng thời gian

```
10,000 requests/day
100,000 requests/week
1,000,000 requests/month
```

==> Khi vượt qua quota, APT sẽ reject request từ client