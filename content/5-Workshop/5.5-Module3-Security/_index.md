---
title: "Module 3: Security & Roles"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
# Module 3: Security & Permissions

Set up Cognito User Pool for login and create fine-grained IAM policies following the principle of Least Privilege.

Create `lib/security-stack.ts`:
```typescript
import * as cdk from 'aws-cdk-lib';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import * as iam from 'aws-cdk-lib/aws-iam';
import { Construct } from 'constructs';

export class SecurityStack extends cdk.Stack {
  public readonly userPool: cognito.UserPool;
  public readonly userPoolClient: cognito.UserPoolClient;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    this.userPool = new cognito.UserPool(this, 'AppUserPool', {
      userPoolName: 'app-user-pool',
      selfSignUpEnabled: true,
      signInAliases: { email: true },
      autoVerify: { email: true },
      passwordPolicy: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireDigits: true,
        requireSymbols: true,
      },
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });

    this.userPoolClient = new cognito.UserPoolClient(this, 'AppUserPoolClient', {
      userPool: this.userPool,
      generateSecret: false,
    });

    // Custom role for notification system (Least Privilege)
    const notificationRole = new iam.Role(this, 'NotificationPublishRole', {
      assumedBy: new iam.ServicePrincipal('lambda.amazonaws.com'),
      description: 'Role for sending notifications with minimal permissions',
    });

    notificationRole.addToPolicy(new iam.PolicyStatement({
      actions: ['sns:Publish'],
      resources: ['*'],
      effect: iam.Effect.ALLOW,
    }));
  }
}
```
