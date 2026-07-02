---
title: "Triển khai & Kiểm thử"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---
# Triển khai & Kiểm thử hệ thống

### 1. Các bước triển khai
Chạy các lệnh sau tại thư mục gốc của hạ tầng CDK:
```bash
# Khởi tạo các tài nguyên phụ trợ
cdk bootstrap

# Kiểm tra sự khác biệt hạ tầng
cdk diff

# Triển khai tất cả lên cloud
cdk deploy --all
```

### 2. Kiểm thử API bằng curl
Sau khi triển khai thành công, API Gateway sẽ hiển thị URL public của bạn. Sử dụng lệnh sau để kiểm tra:
```bash
# Gửi request GET public xem danh sách sản phẩm
curl -X GET https://<API-ID>.execute-api.us-east-1.amazonaws.com/prod/products
```
