---
title: "Module 1: Lưu trữ & DB"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
# Module 1: Storage & Database

**Bước 1: Khởi tạo Amazon S3 Bucket**
Tạo một S3 Bucket lưu trữ ảnh sản phẩm và cấu hình Block Public Access. Chỉ cho phép truy cập qua Pre-signed URL hoặc CDN.

> 📸 **[HƯỚNG DẪN CHỤP ẢNH THỰC TẾ 2 - CẤU HÌNH S3 BUCKET BLOCK PUBLIC ACCESS]**
> *   **Bước 1:** Đăng nhập vào AWS Management Console và mở dịch vụ **Amazon S3**.
> *   **Bước 2:** Chọn bucket bạn đã khởi tạo bằng CDK cho dự án (ví dụ: `musicstoreproductsbucket...` hoặc `assetstoragebucket...`).
> *   **Bước 3:** Chuyển sang tab **Permissions** (Quyền truy cập).
> *   **Bước 4:** Cuộn xuống phần **Block public access (bucket settings)**.
> *   **Bước 5:** Thực hiện chụp màn hình hiển thị toàn bộ phần cấu hình này. Đảm bảo trạng thái **Block *all* public access** được hiển thị rõ là **On** (Đã bật).
> *   **Tên file ảnh khuyến nghị lưu:** `/static/images/5-Workshop/s3_block_public_access.png`

**Bước 2: Thiết kế và khởi tạo DynamoDB theo Single-Table Design**
Sử dụng một bảng DynamoDB duy nhất (`ECommerceTable`) để lưu toàn bộ thực thể như Product, Order, User nhằm tối ưu hóa chi phí và truy vấn.

![DynamoDB Single Table Design](/images/5-Workshop/amazon_dynamodb_single_table_design_guide.jpeg)
*Hình 3: Minh họa mô hình thiết kế Single-Table Design trên DynamoDB với các Partition Key và Sort Key kết hợp*

![Dữ liệu bảng DynamoDB Single-Table](/images/5-Workshop/dynamodb_single_table.png)
*Hình 4: Dữ liệu thực tế lưu trữ trong bảng DynamoDB Single-Table*

---


### Định nghĩa CDK Stack cho Storage
Tạo tệp `lib/storage-stack.ts` để triển khai mã nguồn cấu hình S3 bucket và bảng DynamoDB trên:

```typescript
import * as cdk from 'aws-cdk-lib';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import * as s3 from 'aws-cdk-lib/aws-s3';
import { Construct } from 'constructs';

export class StorageStack extends cdk.Stack {
  public readonly table: dynamodb.Table;
  public readonly assetBucket: s3.Bucket;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Khởi tạo DynamoDB Single Table cho dữ liệu ứng dụng nói chung
    this.table = new dynamodb.Table(this, 'AppSingleTable', {
      partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST, // Thanh toán On-Demand
      pointInTimeRecovery: true, // Tự động backup liên tục
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Tự động xóa khi destroy stack
    });

    // Tạo S3 Bucket riêng tư để lưu trữ tài nguyên ứng dụng
    this.assetBucket = new s3.Bucket(this, 'AssetStorageBucket', {
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL, // Chặn hoàn toàn public access
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });
  }
}
```

