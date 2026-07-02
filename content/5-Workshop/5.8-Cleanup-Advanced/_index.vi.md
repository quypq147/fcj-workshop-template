---
title: "Dọn dẹp & Nâng cao"
date: 2026-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---
# Dọn dẹp tài nguyên & Chủ đề nâng cao

### 1. Dọn dẹp tài nguyên
Để tránh các chi phí phát sinh ngoài ý muốn, hãy phá hủy toàn bộ tài nguyên khi kết thúc bài thực hành:
```bash
cdk destroy --all
```

### 2. Kiểm thử hạ tầng tự động (Jest)
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
