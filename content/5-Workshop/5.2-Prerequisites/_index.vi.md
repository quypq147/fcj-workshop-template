---
title: "Chuẩn bị & Khởi tạo"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---
# Chuẩn bị môi trường & Khởi tạo dự án

### 1. Các công cụ cần thiết
Trước khi bắt đầu, hãy đảm bảo bạn đã cài đặt các công cụ sau:
*   **Node.js** (LTS v18 hoặc v20)
*   **AWS CLI v2** cấu hình thông tin qua `aws configure`
*   **AWS CDK v2** cài đặt global: `npm install -g aws-cdk`
*   **TypeScript** compiler: `npm install -g typescript`

### 2. Khởi tạo dự án
Tạo một thư mục mới và khởi tạo ứng dụng CDK:
```bash
mkdir ecommerce-platform-infra
cd ecommerce-platform-infra
cdk init app --language typescript
```
