---
title: "Chuẩn bị & Khởi tạo"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---
# Chuẩn bị & Khởi tạo

Trước khi tiến hành, hãy chuẩn bị môi trường:
*   Cài đặt Node.js, AWS CLI và AWS CDK.
*   Cấu hình thông tin xác thực với AWS (dùng `aws configure`).

![Thông tin tài khoản AWS](/images/5-Workshop/aws_configure_sts.png)
*Hình 2: Cấu hình thông tin tài khoản AWS và kết quả kiểm tra danh tính (get-caller-identity)*

### Khởi tạo dự án CDK
Tạo một thư mục mới và khởi tạo ứng dụng CDK bằng ngôn ngữ TypeScript:
```bash
mkdir ecommerce-platform-infra
cd ecommerce-platform-infra
cdk init app --language typescript
```
