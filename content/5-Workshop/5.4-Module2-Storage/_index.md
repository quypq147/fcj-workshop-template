---
title: "Module 2: Storage & Database"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
# Module 2: Storage & Database

Set up the DynamoDB single-table for metadata and S3 bucket for product images.

Create `lib/storage-stack.ts`:
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

    // Initialize DynamoDB Single Table for Product, Order, Payment metadata
    this.table = new dynamodb.Table(this, 'ECommerceSingleTable', {
      partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST, // On-Demand pricing
      pointInTimeRecovery: true, // Enables automated backups
      removalPolicy: cdk.RemovalPolicy.DESTROY, // Dev environment cleanup
    });

    // Create private S3 Bucket for product photos
    this.productImagesBucket = new s3.Bucket(this, 'ProductImagesBucket', {
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL, // Secure private bucket
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });
  }
}
```
