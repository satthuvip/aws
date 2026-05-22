ECR là dịch vụ của AWS dùng để lưu trữ, quản lý và chia sẻ docker container images. Giống như kho chứa image docker trên AWS.

Sau khi build lên một image, thực hiện push image đó lên ECR rồi các dịch vụ khác như ECS, EKS , Lambda, EC2 có thể pull image này về để chạy.

 