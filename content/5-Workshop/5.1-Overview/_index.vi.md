---
title: "Tổng quan Workshop"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---
# Tổng quan Workshop

### 1. Tại sao lựa chọn AWS CDK?
Trước đây, Hạ tầng dưới dạng mã (IaC) phụ thuộc vào các tệp mẫu YAML/JSON tĩnh. **AWS CDK** cho phép các nhà phát triển định nghĩa tài nguyên đám mây bằng các ngôn ngữ lập trình quen thuộc như **TypeScript**. Điều này mang lại tính năng kiểm tra kiểu dữ liệu mạnh mẽ, gợi ý mã tự động, tính kế thừa và tái sử dụng mã vào định nghĩa hạ tầng đám mây.

### 2. Kiến trúc Serverless hướng sự kiện
Các ứng dụng web hiện đại yêu cầu độ tin cậy và tối ưu chi phí tối đa. Workshop này trình bày một **Kiến trúc Serverless hướng sự kiện (Event-Driven)** tự động co giãn để xử lý các đợt lưu lượng tăng đột biến và đảm bảo tính sẵn sàng cao mà không cần quản trị máy chủ thực tế.

Chúng ta sẽ sử dụng Amazon API Gateway cho các REST endpoint, AWS Lambda cho việc tính toán serverless, Amazon DynamoDB để lưu trữ dữ liệu dạng Single-Table có khả năng mở rộng cao, Amazon SQS cho việc xếp hàng đợi tin nhắn và Amazon EventBridge để định tuyến sự kiện tùy chỉnh.
