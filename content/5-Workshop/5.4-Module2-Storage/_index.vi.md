---
title: "Module 2: Lưu trữ & DB"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
# Module 2: Storage & Database (Lưu trữ và Cơ sở dữ liệu)

Thiết lập bảng DynamoDB dạng Single Table và S3 bucket lưu ảnh sản phẩm.

Tạo tệp `lib/storage-stack.ts`:
```typescript
import * as cdk from 'aws-cdk-lib';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import * as s3 from 'aws-cdk-lib/aws-s3';
import { Construct } from 'constructs';

export class StorageStack extends cdk.Stack {
  public readonly table: dynamodb.Table;
  public readonly productImagesBucket: s3.Bucket;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Khởi tạo DynamoDB Single Table cho thực thể Product, Order, Payment
    this.table = new dynamodb.Table(this, 'ECommerceSingleTable', {
      partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST, // Thanh toán On-Demand
      pointInTimeRecovery: true, // Tự động backup liên tục
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Tự động xóa khi destroy stack
    });

    // Tạo S3 Bucket riêng tư lưu ảnh sản phẩm
    this.productImagesBucket = new s3.Bucket(this, 'ProductImagesBucket', {
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL, // Chặn hoàn toàn public access
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });
  }
}
```
