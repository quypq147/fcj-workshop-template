---
title: "Dọn dẹp & Nâng cao"
date: 2026-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---
# Dọn dẹp tài nguyên (Clean up)

Để tránh phát sinh chi phí ngoài ý muốn sau khi nghiệm thu và chấm điểm bài tập lớn, chúng ta cần gỡ bỏ hoàn toàn ứng dụng ra khỏi AWS Cloud.

### 1. Xóa hạ tầng Backend bằng CDK
Chạy lệnh sau tại thư mục infra để xóa bỏ toàn bộ CloudFormation Stacks và các tài nguyên liên quan (DynamoDB, SQS, API Gateway, Lambdas, ...):

```bash
cdk destroy --all
```

![Gỡ bỏ tài nguyên CDK Destroy](/images/5-Workshop/cdk_destroy_cleanup.png)
*Hình 11: Màn hình xác nhận đã xóa sạch các stacks qua lệnh cdk destroy*

### 2. Gỡ bỏ ứng dụng trên AWS Amplify Console
Truy cập vào dịch vụ AWS Amplify trên Console, chọn ứng dụng của bạn, click vào **Actions** ở góc trên bên phải và chọn **Delete app** để xóa hoàn toàn hosting và cấu hình frontend.


---

### 3. Chủ đề nâng cao: Kiểm thử hạ tầng tự động (Jest)
Viết kiểm thử tự động để bảo đảm hạ tầng luôn tuân thủ các chính sách của dự án.
Tạo tệp `test/infra.test.ts`:

```typescript
import * as cdk from 'aws-cdk-lib';
import { Template } from 'aws-cdk-lib/assertions';
import * as Storage from '../lib/storage-stack';

test('DynamoDB table configuration is correct', () => {
  const app = new cdk.App();
  const stack = new Storage.StorageStack(app, 'TestStack');
  const template = Template.fromStack(stack);

  template.hasResourceProperties('AWS::DynamoDB::Table', {
    BillingMode: 'PAY_PER_REQUEST',
  });
});
```

