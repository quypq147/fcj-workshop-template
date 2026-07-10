---
title: "Cleanup & Advanced"
date: 2026-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---
# Clean up Resources

To avoid unexpected charges after evaluation and grading of the project, we need to completely remove the application from AWS Cloud.

### 1. Destroy Backend Infrastructure Using CDK
Run the following command in the infra directory to tear down all CloudFormation Stacks and related resources (DynamoDB, SQS, API Gateway, Lambdas, etc.):

```bash
cdk destroy --all
```

![Remove CDK Destroy Resources](/images/5-Workshop/cdk_destroy_cleanup.png)
*Figure 11: Confirmation screen showing all stacks successfully deleted via cdk destroy*

### 2. Remove Application from AWS Amplify Console
Navigate to the AWS Amplify service on the Console, select your application, click **Actions** in the top-right corner, and choose **Delete app** to completely delete the frontend hosting and configuration.

---

### 3. Advanced Topic: Automated Infrastructure Testing (Jest)
Write automated tests to ensure the infrastructure always complies with project policies.
Create the file `test/infra.test.ts`:

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
