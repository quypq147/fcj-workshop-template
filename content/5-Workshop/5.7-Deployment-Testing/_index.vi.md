---
title: "Triển khai & Kiểm thử"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---
# Triển khai & Kiểm thử (End-to-End Test)

**Bước 1: Triển khai Backend bằng CDK**
Để đẩy toàn bộ hạ tầng lên AWS, chạy các lệnh sau tại thư mục gốc của hạ tầng CDK:

```bash
# Khởi tạo các tài nguyên phụ trợ trên vùng (region) chỉ định
cdk bootstrap

# Kiểm tra sự khác biệt giữa local code và cloud
cdk diff

# Triển khai tất cả các stacks lên AWS Cloud
cdk deploy --all
```

![Kết quả triển khai backend CDK](/images/5-Workshop/cdk_deploy_outputs.png)
*Hình 8: Terminal log khi chạy thành công lệnh cdk deploy*

**Bước 2: Triển khai Frontend với AWS Amplify**
Kết nối GitHub repository chứa mã nguồn Next.js vào AWS Amplify, cấu hình các biến môi trường cần thiết từ Outputs ở Bước 1 và chạy quá trình Build & Deploy (hỗ trợ nền tảng `WEB_COMPUTE` cho Next.js SSR).

![Quy trình deploy trên AWS Amplify](/images/5-Workshop/amplify_build_deploy.png)
*Hình 9: Trạng thái triển khai frontend Next.js thành công trên AWS Amplify*

**Bước 3: Thực hiện E2E Test**
Truy cập vào tên miền (Domain) do Amplify cấp cho website. Thực hiện mua sản phẩm bất kỳ, thực hiện thanh toán và theo dõi log luồng xử lý hệ thống.

![Thanh toán thành công & Logs CloudWatch](/images/5-Workshop/e2e_test_success.png)
*Hình 10: Minh chứng thanh toán thành công và CloudWatch logs ghi nhận sự kiện*


