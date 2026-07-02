---
title: "Dọn dẹp & Nâng cao"
date: 2026-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---
# Cleanup & Advanced Topics

### 1. Cleanup Resources
To prevent charges, always destroy the resources after you finish testing:
```bash
cdk destroy --all
```

### 2. Infrastructure Testing (Jest)
Verify stack resources properties automatically.
Create `test/infra.test.ts`:
```typescript
import * as cdk from 'aws-cdk-lib';
import { Template } from 'aws-cdk-lib/assertions';
import * as Network from '../lib/network-stack';

test('VPC configuration is correct', () => {
  const app = new cdk.App();
  const stack = new Network.NetworkStack(app, 'TestStack');
  const template = Template.fromStack(stack);

  template.hasResourceProperties('AWS::EC2::VPC', {
    EnableDnsSupport: true,
    EnableDnsHostnames: true,
  });
});
```
