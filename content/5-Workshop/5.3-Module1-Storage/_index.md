---
title: "Module 1: Storage & Database"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
# Module 1: Storage & Database

**Step 1: Initialize Amazon S3 Bucket**
Create an S3 Bucket to store product images and configure Block Public Access. Only allow access via Pre-signed URLs or CDN.

> 📸 **[PRACTICAL SCREENSHOT GUIDE 2 - S3 BUCKET BLOCK PUBLIC ACCESS CONFIGURATION]**
> *   **Step 1:** Log in to the AWS Management Console and open the **Amazon S3** service.
> *   **Step 2:** Select the bucket you initialized using CDK for the project (e.g., `musicstoreproductsbucket...` or `assetstoragebucket...`).
> *   **Step 3:** Switch to the **Permissions** tab.
> *   **Step 4:** Scroll down to the **Block public access (bucket settings)** section.
> *   **Step 5:** Take a screenshot showing this entire configuration section. Ensure that the **Block *all* public access** status is clearly shown as **On**.
> *   **Recommended image filename to save:** `/static/images/5-Workshop/s3_block_public_access.png`

**Step 2: Design and Initialize DynamoDB with Single-Table Design**
Use a single DynamoDB table (`ECommerceTable`) to store all entities such as Product, Order, User to optimize costs and queries.

![DynamoDB Single Table Design](/images/5-Workshop/amazon_dynamodb_single_table_design_guide.jpeg)
*Figure 3: Illustration of Single-Table Design on DynamoDB using combined Partition Key and Sort Key*

![DynamoDB Single-Table Data](/images/5-Workshop/dynamodb_single_table.png)
*Figure 4: Real data stored in the DynamoDB Single-Table*

---

### Define CDK Stack for Storage
Create the file `lib/storage-stack.ts` to implement the S3 bucket and DynamoDB table configuration:

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

    // Initialize DynamoDB Single Table for application data
    this.table = new dynamodb.Table(this, 'AppSingleTable', {
      partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST, // On-Demand billing
      pointInTimeRecovery: true, // Automate continuous backup
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Auto-delete on stack destruction
    });

    // Create private S3 Bucket for application assets
    this.assetBucket = new s3.Bucket(this, 'AssetStorageBucket', {
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL, // Block all public access
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });
  }
}
```
